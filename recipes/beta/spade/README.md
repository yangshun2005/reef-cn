# spade

**状态：beta。** 在完整、可复现的学习结果发布之前，SPADE 一直保留在 `recipes/beta/spade/` 下。测试针对替身验证了经验部分、处理器的分组及其生成结果；它们并不确立学习性能。有关各个部分，请参见 [issue #482](https://github.com/Human-Agent-Society/reef/issues/482) 和 [issue #498](https://github.com/Human-Agent-Society/reef/issues/498)；有关它们所属的流水线，请参见 [issue #447](https://github.com/Human-Agent-Society/reef/issues/447)；有关此分类，请参见 [issue #422](https://github.com/Human-Agent-Society/reef/issues/422)。

[SPADE](https://arxiv.org/abs/2608.19197)，即自适应合成可执行环境中的自我对弈，作为一个 Reef 配方：一个策略扮演环境设计者（Environment Designer），编写可执行环境；另一个策略扮演推理智能体（Reasoning Agent），在这些环境中学习。Reef 知道一种任务格式——Harbor，并在 `reef.record2dataset` 中编写、检查和游玩 Harbor 任务：任务契约作为提示词，被服务的模型通过 Reef 请求（每个提议都是一条带凭据的记录），回复受编写规则约束并用 `reef.core.tasks` 写入，Harbor 的 oracle 和 nop 智能体作用于结果，任务游玩器负责各个回合。这些作为生成器服务运行，由 `reef serve` 在 HTTP 服务旁边启动。SPADE 所添加的是方法，而它就住在这里。

- 论文：[arXiv:2608.19197](https://arxiv.org/abs/2608.19197)
- 参考代码：[spade-rl/spade](https://github.com/spade-rl/spade)

## 布局

```text
beta/spade/
  generation.py   the experience section (results sorted by regret into the frontier, the mastered and the out of reach), a generation's records
  processor.py    the reported half: episodes grouped by task; the task generation half: the Designer's generations, run on a worker
  objective.py    group relative advantages per task group, on Tinker's importance sampling loss
  recipe.py       the configuration that binds them
  examples/tinker/serve.yaml   a Tinker deployment with the generator service
```

## 处理器

`SpadeProcessor` 同时实现了 Reef 的两个处理器契约。

作为上报反馈处理器，它接收任务游玩器的报告：每个普通回合在 `metadata.task` 下指明其任务，一个任务的回合组成一个组，一个组在 `rollouts-per-task` 个回合时完成，一个批次容纳 `tasks-per-step` 个完整的组。`SpadeObjective` 在每个任务组内对每个回合的奖励进行中心化和缩放（一个组内所有奖励都相同则给出 0），Tinker 内置的 `importance_sampling` 损失将该优势施加到每个响应 token 上，因此该配方目前运行在 `tinker` 后端上，而在 Slime 上会在选择阶段失败，因为 Slime 没有该名称的损失族。设计者自己的报告（`metadata.role: designer`）和提示臂的回合（`arm: hint`）共享该场景；处理器将它们未组装地释放，仅对普通臂进行训练。

作为任务生成处理器（`reef.train.processors.TaskGenerationProcessor`），它运行设计者。一次生成是私有工作线程上的一个作业，脱离训练器线程：对 `skills` 进行 `count` 次提议，每次都是对 `generate` 的调用（设计者通过生成器服务被请求，提示词中包含上一次生成的结果作为 SPADE 的经验部分），写入生成器的任务根目录下（重复的会被拒绝；同一代较早尝试在重载取消它之前占用的名称会被替换），由 `validate` 检查（Harbor 的 oracle 和 nop 智能体：参考解决方案得分为 1，什么都不做得分低于 1），按原样游玩 `rollouts-per-task` 次（训练数据）以及追加 `solution/hint.txt` 后游玩 `hint-plays` 次（仅用于测量），并对照设计者的凭据进行报告，以其遗憾值作为分数，被拒绝的为 0。遗憾值是平均提示奖励减去平均普通奖励；普通平均值将任务归入其区间（掌握：高于 0.9；遥不可及：低于 0.1；否则为前沿），而前沿按遗憾值从高到低排序，就是下一个提示词所展示的内容。任务按设计者的记录 id 拆分到任务根目录下的 `manifest-<generation>.json` 中，而 `state-dir/generation-<generation>.json` 保留每个提议、拒绝和测量；重启时读取它以继续下一代和上一次经验。

第一代在处理器首次寻找批次时开始；下一代在自上一代开始以来已确认 `batches-per-generation` 个批次后开始（其回合在它运行期间进行训练），因此设计者为当下正在训练的策略编写；没有测量到任何任务的代会被立即跟进；`generations` 对它们进行上限限制。`GET /reef/status` 显示进行中的代、已完成的数量以及最后的错误。

## 运行它

```bash
export REEF_TOKEN=reef-local REEF_SPADE_STATE_DIR="$PWD/work/spade"   # and TINKER_API_KEY
reef serve -c recipes/beta/spade/examples/tinker/serve.yaml
curl -s -X POST -H "Authorization: Bearer $REEF_TOKEN" -H "Content-Type: application/json" \
  -d '{"name": "spade"}' http://127.0.0.1:8900/reef/scenarios
```

场景是处理器所归属的对象，而 `reef serve` 自身不会创建任何场景：上面的 `POST /reef/scenarios`（或第一个指明该场景的模型调用）将其创建出来，之后第 0 代在处理器首次寻找批次时开始。

部署的 `generator` 部分使 `reef serve` 在 HTTP 服务之前启动生成器服务，并将其地址作为 `${endpoints.generator}` 交给配方。生成器在 Reef 的解释器下运行；其主机需要 Docker 和 `harbor` 命令行，而 Reef 服务本身不需要。`execution: {generator: ray}` 将其放置到别处。`generator.designer-url` 和 `generator.designer-model` 将设计者指向另一个服务（例如 OpenRouter 上的强模型，而推理智能体是正在训练的部署）；默认情况下两个角色都是被服务的模型。`generator.designer-options` 为设计者的聊天请求添加字段：一个在编写环境之前思考数千 token 的模型会超出服务的推理截止时间，而 `{"reasoning_effort": "none"}` 使其仅限于回复。有关每个键，请参见[生成器部分](../../../docs/reference/configuration.rst)。

使用 `generations: 0` 时，处理器不生成任何内容，而是对任务游玩器所报告的任何内容进行训练，这就是在别处编写的任务上进行训练的方式：

```bash
python -m reef.harness.client.tasks --reef-url http://127.0.0.1:8900 --scenario spade --model Qwen/Qwen3-8B \
  --manifest tasks/manifest-00000.json --tasks-root tasks --side train --work-dir work/play --label arm=plain
```

思考保持关闭，因为思考模型的回合永远不会组装成一个样本：智能体的历史携带了先前轮次但没有它们的思考，因此第二轮提示词不再延续第一轮的 token。关闭思考后，Qwen3 的生成提示词仍以一个空的 think 块结尾，而历史会丢弃它；`scaffold-tolerance` 让组装重新对齐那些被掩码的 token。需要一个位于 Docker 与主机共享路径下的任务根目录（在 macOS 上，位于主目录下），否则验证器的奖励文件永远不会到达主机。

已知限制：一次生成会运行数小时，而权重每一步都会重新加载，因此一个任务组可能容纳两个权重版本的回合；`max-staleness` 对此进行限制。设计者自身的训练——以其遗憾值作为其提议的奖励——随后进行。