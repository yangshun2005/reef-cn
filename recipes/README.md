# 配方与示例

每个示例目录只需设置一次（安装该示例的测试框架及其声明的依赖，包括 `reef-client`，以及在使用时用到的 [reef-eval](https://github.com/Human-Agent-Society/reef-eval)）：

```bash
pip install -e .
```

然后运行 `./run.sh` —— 它会启动 Reef（该示例的 stack YAML）并运行循环（`run.py`）。

下面的目录按配方所服务的**任务类型**以及它们**所演化的对象**（模型权重或智能体测试框架）进行分组。权重类配方需要 GPU 训练栈，而测试框架类配方只需要一个模型端点。Reefine 随 `reef-infra` 一起发布，这里其他所有配方都是 cookbook 包。[Basic](#basic) 是仅记录的起始栈，不在目录中，而 [beta 配方](#beta-配方)在发布学习结果后才会加入。根目录的 [README](../README.md#-recipes-and-examples) 和[配方指南](../docs/user-guide/recipes.rst)展示了相同的目录。

## 科学发现

一个具有可度量目标的难题，配方在其中进行反复尝试，并在测试时基于这些尝试进行训练。

| 配方 | 演化对象 | 代码 | 文档 | 示例 |
|---|---|---|---|---|
| TTT-Discover | 模型权重 | [`recipes/tttd/`](tttd/) | [TTT-Discover](../docs/user-guide/recipes/tttd.rst) | [TTT-Discover 在圆填充和 Erdős 最小重叠问题上的应用](tttd/examples/tttd/README.md) |
| Guidance-TTT | 引导模型权重；执行器保持冻结 | [`recipes/tttd/`](tttd/) | [Guidance-TTT](tttd/examples/guidance_ttt/README.md) | [Guidance-TTT 在 TriMul 上的应用](tttd/examples/guidance_ttt/README.md) |

[TTT-Discover](tttd/examples/tttd/README.md) 将普通的、与服务无关的 rollout 测试框架与其 Reef 适配器分离。它演示了分组发现 rollout、持续评估、精确的推理到报告引用，以及忠实于论文的 PUCT 状态复用。其 README 保留了正式的圆填充运行和一次 Erdős 运行，并附有存储的 W&B 历史记录。

[Guidance-TTT](tttd/examples/guidance_ttt/README.md) 训练一个仅做摘要的 Qwen 引导策略，同时一个冻结的外部执行模型编写经验证器评分的程序。它演示了如何接入一个执行模型，而不将其加入 Reef 的训练或推理 token 捕获路径。

## 任务流上的持续学习

一串独立任务，由验证器逐一评分，因此配方在下一个任务到来之前从每个分数中学习。

| 配方 | 演化对象 | 代码 | 文档 | 示例 |
|---|---|---|---|---|
| SAO | 模型权重 | [`recipes/sao/`](sao/) | [SAO](../docs/user-guide/recipes/sao.rst) | [SAO 在 IMOAnswerBench 上的应用](sao/examples/imo_answerbench/README.md)、[SAO 在 CEO-Bench 上的应用](sao/examples/ceobench/README.md) |
| GEPA | 测试框架树：规则、技能和智能体命令 | [`recipes/gepa/`](gepa/) | [GEPA](../docs/user-guide/recipes/gepa.rst) | [GEPA 在 AIME 2025 上的应用](gepa/examples/aime/README.md) |
| Meta-Harness | 测试框架：完整组合 | [`recipes/meta_harness/`](meta_harness/) | [Meta-Harness](meta_harness/README.md) | Meta-Harness 在 Terminal-Bench 上的应用：[示例](meta_harness/examples/terminal_bench/README.md)、[结果](meta_harness/RESULTS.md) |

[SAO](sao/examples/imo_answerbench/README.md) 是 cookbook 中 SAO 配方的功能性冒烟测试，也是最小的权重更新循环。三个 IMOAnswerBench 问题由 `run.py` 按顺序运行，每个问题通过 Reef 驱动六次带评分的 rollout，并带有可验证的二元奖励，每次带评分的 rollout 即为一个训练步骤。

[SAO 在 CEO-Bench 上的应用](sao/examples/ceobench/README.md) 将 [CEO-Bench](https://ceobench.com)（一个 500 天的模拟创业公司）作为一个 Harbor 任务运行。测试框架是该基准自带的 bash 智能体，从宿主机上运行，其提示词、工具和工具执行器取自任务镜像中固定版本的检出，其模型调用由 Reef 提供；两个模拟器角色留在 Reef 之外，验证器根据其 `world.nmdb` 对运行进行评分，每一完成周的公司价值变化会在 episode 运行期间对照该周的决策轮次进行报告。它演示了如何将基准的智能体采纳为 Reef 测试框架，以及如何为一个长 episode 塑造在线的、按周期的奖励。

[GEPA](gepa/examples/aime/README.md) 在同一机制上将反思式提示词演化重建为一个方法包：`propose` 是一次 GEPA 迭代——从方法自身的档案中进行 Pareto 采样选择一个父代，用更强的模型针对所服务组合的失败流量反思一个组件，并且只有当子代在小批量上胜过其父代时才接受它——而 `selection` 仅在完整验证集上取得严格平均改进时才发布。其中没有任何内容导入上游包。其 AIME 示例就是验证：驱动程序嵌入 Reef 服务，将快速入门中的 45 个训练问题每次三个地通过它运行，并将两次 150 题的测试通过结果与保留的官方记录进行封存比对（AIME 2025 上从 26.67% 提升到 38.67%，seed 0）；该方法自身的 seed-0 运行在相同问题上从相同父代进行反思，达到了 46.67%，其 seed-1 运行则获得了官方的那 12 分。

[Meta-Harness](meta_harness/README.md) 使用所有保留的候选和分数搜索完整的测试框架组合。它选择严格平均分改进，并用 Reef 的服务状态提交种群。参见 [Terminal-Bench 示例](meta_harness/examples/terminal_bench/README.md)、[结果](meta_harness/RESULTS.md)和选定的测试框架。

## 从使用中学习

真实交互中没有人报告分数，或者反馈到达得很晚，因此配方从它已经服务的流量中读出信号。

| 配方 | 演化对象 | 代码 | 文档 | 示例 |
|---|---|---|---|---|
| OpenClaw-RL | 模型权重 | [`recipes/openclawrl/`](openclawrl/) | [OpenClaw-RL](../docs/user-guide/recipes/openclawrl.rst) | [OpenClaw-RL 在 GSM8K 作业流上的应用](openclawrl/examples/openclawrl/README.md) |
| SkillClaw | 测试框架技能池 | [`recipes/skillclaw/`](skillclaw/) | [SkillClaw](../docs/user-guide/recipes/skillclaw.rst) | [SkillClaw 在 WildClawBench 上的应用](skillclaw/README.md) |
| Reefine | 测试框架：技能、规则、智能体命令和 pi 扩展 | [`reef/recipe/reefine/`](../reef/recipe/reefine/) | [Reefine](../docs/user-guide/recipes/reefine.rst) | [Reefine 在 reef-pi 上的应用](../tutorials/reefine/README.md) |

[OpenClaw-RL](openclawrl/examples/openclawrl/README.md) 将论文中的个人智能体实验作为 reef-eval 任务流运行：一个模拟学生将 72 个 GSM8K 作业问题带给一个 Hermes 智能体，其模型调用经过 reef，指标是智能体的答案匹配学生偏好之前的会话数。该方法（会话相关性、PRM 评判、提示条件化的教师）就是 `openclawrl` cookbook 包，因此该示例只包含测试框架一侧。

[SkillClaw](skillclaw/README.md) 在同一机制上将 SkillClaw 复现重建为一个方法包：`propose` 是封存的夜晚（每个技能组一个决策加上无技能桶）映射到一个复合变异序列，`selection: always` 像论文的无门控机制那样发布每个非跳过夜晚，并且该方法自带其交付方式——一个配方表面，将所服务池的目录注入每个被代理的请求。活动驱动程序嵌入 Reef 服务，在 docker 中运行冻结的 60 任务 WildClawBench 日，从 `GET /reef/harness` 拉取已发布的池，并按照从封存活动中逐字沿用的预注册增益标准封存轮次。其 `harbor/` 是一个以标准 Harbor 格式内置的 WildClawBench 任务（自包含镜像、基准自带的程序化评分器），而 `run.py solve` 是对它的单 episode reef-eval 冒烟测试。

[Reefine](../docs/user-guide/recipes/reefine.rst) 是内置配方，将自然语言请求转化为测试框架更新。所服务的模型提出更改，门控对其进行评分，代码扩展在运行前等待提升。`reef serve --recipe reefine` 无需检出即可启动其配置，而 [Reefine 教程](../tutorials/reefine/README.md)记录了一个 bug 修复流程、一个研究循环，以及哪些请求赢得了门控。

## Basic

[Basic](basic/) 是核心的、仅记录的 `recipe` 上的一切——一个不学习任何东西的部署，以及围绕它的最小完整循环。它的两个 stack 文件是部署在选择方法之前的起点，也是快速入门所服务的内容：

- `external-provider.yaml` —— 无 GPU、无本地模型：一个 Reef 进程代理到 HTTP 提供方（`reef serve -c recipes/basic/external-provider.yaml`）。
- `local-sglang.yaml` —— 本地推理：一个 SGLang 服务器加上 Reef，无训练。

每个都是完整且可运行的：一个扁平的 `reef:` 部分（由 [`reef/service/deploy/service_config.py`](../reef/service/deploy/service_config.py) 转换为冻结的 `ServiceConfig`）加上一个 `services:` 列表，编排器按依赖顺序启动它们，并支持 `${VAR}` 环境变量和 `${dotted.path}` 配置插值。`${REEF_PYTHON}` 默认为运行 `reef serve` 的解释器，因此使用它的 Python 服务共享 Reef 的环境，而不改变字面 `python` 命令的含义。复制一个并加以调整；`reef serve -c <stack> --<section.field> <value>` 会覆盖匹配的 YAML 设置，例如 `--inference.model-path /models/demo` 或 `--reef.port 9000`。它们绑定的 `recipe` 是 [`reef/recipe/base.py`](../reef/recipe/base.py) 中的基础契约；绑定某个方法的 stack 与该方法是放在一起的（`recipes/<method>/examples/<example>/serve.yaml`；最小的权重训练示例是 [`recipes/sao/examples/imo_answerbench/serve.yaml`](sao/examples/imo_answerbench/serve.yaml)）。两个契约保证这套内容可靠：[`test_training_server.py`](../tests/reef_service/test_training_server.py) 从每个 cookbook stack 启动内部服务，而 [`docs/site/scripts/check-doc-contracts.mjs`](../docs/site/scripts/check-doc-contracts.mjs) 从 `local-sglang.yaml` 推导出文档化的端口和健康检查路由。

围绕这些 stack，循环基于 Harbor 任务标准：一个 [Harbor](https://github.com/laude-institute/harbor) 任务（`harbor/`）、一个 Harbor 智能体测试框架，它通过 Reef 记录其模型调用并在试验结束时报告验证器奖励（`harness/`）、写出的循环（`run.py` —— [reef-eval](https://github.com/Human-Agent-Society/reef-eval) 的 `Lab.run`，一个 episode），以及一个启动器（`run.sh`），它从 `external-provider.yaml` 以本地覆盖启动 Reef 并运行它。

## Beta 配方

[CORAL TTT](beta/coral/README.md) 及其 [`coral_demo`](beta/coral/examples/coral_demo/) 示例处于 beta 阶段。在完整、可复现的学习结果发布之前，它们位于 `recipes/beta/coral/` 下。集成和冒烟测试验证了接线，但不确立学习性能。参见该配方的[验证说明](beta/coral/README.md#verifying-without-gpus)。

[CORAL TTT](beta/coral/README.md) 运行一个 [CORAL](https://github.com/Human-Agent-Society/CORAL) 发现任务——在 git worktree 中并行运行的编码智能体，针对一个问题进行评分的尝试——每个智能体调用都通过 Reef 服务并归属。CORAL 的网关流量将 Reef 回执带入一个仅追加的调用日志；一个监视器对每次最终化的尝试恰好报告一次，并附有其精确的推理引用，而同一父提交的兄弟尝试作为一个分组相对奖励步骤进行训练（复用 TTT-Discover 的目标和损失族）。其示例是一个由 CORAL 自身运行时驱动的真实 CORAL 任务，外加一条无 GPU 冒烟通道，用预设模型针对生产 Reef 服务运行整个循环。

[SPADE](beta/spade/README.md)（自适应合成可执行环境中的自我对弈，[arXiv:2608.19197](https://arxiv.org/abs/2608.19197)）处于 beta 阶段，位于 `recipes/beta/spade/` 下：一个策略同时扮演编写可执行环境的环境设计者和在其中学习的推理智能体。Reef 只认识一种任务格式——Harbor，而设计者直接编写它：一条指令、一个容器、一个验证器和一个参考解，任何 Harbor 智能体都能玩的任务。编写、检查和游玩这些任务是 Reef 的职责（`reef.record2dataset`，即 `reef serve` 在 HTTP 服务旁启动的生成器服务）；该包承载方法：设计者的对抗性经验部分、每个任务所用的两条臂、针对设计者回执报告的基于提示的遗憾，以及推理智能体在普通臂 episode 上的分组相对训练，全部由其处理器驱动。设计者自身的训练紧随其后。