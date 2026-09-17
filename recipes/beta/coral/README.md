# 通过 Reef 和 Slime 进行 CORAL 测试时训练

**状态：beta。** CORAL TTT 及其 `coral_demo` 示例在完整、可复现的学习结果发布之前，将一直保留在 `recipes/beta/coral/` 下。下方的集成测试和冒烟测试验证的是接线是否正确；它们并不能证明学习性能。关于此分类，请参见 [issue #422](https://github.com/Human-Agent-Society/reef/issues/422)；关于该实验，请参见 [issue #3](https://github.com/Human-Agent-Society/reef/issues/3)。

运行一个真实的 CORAL 任务，并具备完全可归因的推理：CORAL 自身的运行时（agent 工作树、网关、评分守护进程）驱动编码 agent CLI，其每一次调用都流经 Reef，评估器分数作为训练数据返回，权重更新，后续尝试由新修订版本提供服务。实现了 [issue #3](https://github.com/Human-Agent-Society/reef/issues/3)。

固定的上游版本：[CORAL](https://github.com/Human-Agent-Society/CORAL) 提交 `0123dfb`。

## 快速开始（2 块 GPU）

```bash
cd recipes/beta/coral/examples/coral_demo
pip install -e .[coral]        # demo deps + CORAL at the pinned commit
npm install -g opencode-ai     # or any CORAL runtime CLI; select with --runtime
./run.sh
```

`run.sh` 启动 Reef 训练栈（`serve.yaml`：`CoralRecipe`、Qwen3-8B LoRA），然后 `run.py` 根据 `task/` 中的真实任务构建 CORAL 的 `AgentManager` —— CORAL 通过 `task/seed` 生成其 agent 运行时，从 `task/litellm_config.yaml` 启动其 LiteLLM 网关（其唯一上游是 Reef 服务），并使用 `task/grader` 中打包的评分器对 `coral eval` 提交进行评分。Reef 关联层在任何 agent 生成之前就被拼接在网关之下；一个尝试监视器将每个已完成的尝试报告给 Reef 以进行训练。运行会在达到其尝试预算（`--max-attempts`）时自动停止，并写入 `work/coral-demo/bundle.json`。

适配器测试既不需要 GPU 也不需要 CORAL：`python -m pytest tests/test_coral_*.py`。

## 在没有 GPU 的情况下验证

`examples/coral_demo/smoke/run_smoke.sh` 针对生产 Reef 服务运行未经修改的 `run.py`，使用一个预设的推理后端（`smoke/echo_reef_service.py`）和一个可脚本化的 CORAL 运行时（`smoke/scripted_runtime.py`）。每一次线路交互 —— 工作树、网关密钥交换、请求头标记、回执捕获到日志、在评分器 venv 中的评分守护进程评分、恰好一次报告、结果包 —— 都是真实的代码路径；只有模型的答案和 agent 的"智能"是预设的。`coral validate task/` 单独证明该任务包是一个格式良好的 CORAL 任务。

## 各部分如何对应

```
CORAL AgentManager (real runtime: worktrees, agent CLIs, per-agent proxy keys,
   |               grader daemon, heartbeats, restarts, attempt budget)
   v
CORAL gateway (identity: x-coral-agent-id, x-coral-session-id)
   v
recipes.beta.coral.middleware      stamps x-reef-scenario + x-reef-tag-coral-{run,agent,commit},
   v                          captures reef receipts -> journal
LiteLLM -> reef serve         stores INFERENCE records with tags,
   v                          answers with x-reef-agent-record-id / receipt SSE frame
CORAL grader daemon finalizes the attempt (.coral/**/attempts/*.json)
   -> recipes.beta.coral.watcher: resolves the attempt's captured references,
      POST /reef/report {score, references, metadata.coral}, exactly once
   -> CoralProcessor groups siblings of one parent commit -> Slime LoRA step
   -> new revision served to the next attempts
```

一个发现问题就是一个 reef 场景；agent 和工作树身份存在于标签中，因此并行的 agent 共享不断演化的策略，而不会使场景碎片化。

## 关联模型

Reef 随每条 INFERENCE 记录存储的 `coral-agent`/`coral-commit` 标签是主要的关联键，并且能在任何代理行为下存活。日志还额外捕获 reef 的响应回执，因此报告引用的是确切的记录 id；被剥离的回执会降级为仅标签关联，且永远不会致命。报告携带一个确定性的、由客户端提供的 id，因此报告器的重试或崩溃重放会在服务端去重。

CORAL 的网关在调用时用工作树的 HEAD 标记每次调用 —— 即 agent 当时正在编辑的*父*提交，而不是之后 `coral eval` 创建的提交。因此，监视器在（agent，父提交）坐标处解析尝试的引用，按顺序认领日志记录，这样来自同一父提交的连续尝试（一次回退、一次重试的评估）永远不会共享同一个引用。

## 报告的内容

真实的尝试，在其第一个终态时，恰好报告一次。CORAL 的 `grader_error` 尝试（评估机制损坏，无策略信号）和 `tune` 尝试（CORAL 自身从预算中排除的配置扫描）会被跳过；已归档的尝试会被忽略。

## 训练步骤如何工作

训练侧将 CORAL 的尝试树视为分组相对奖励训练的分组结构。每个尝试都指明它从哪个提交开始（`parent_hash`），因此同一父提交的已评分兄弟节点是一组从相同代码开始并产生分歧的 rollout，这正是相对奖励步骤所需的比较集。第一代尝试没有父提交，会被分组在一起：它们都从任务种子产生分歧。

从报告到梯度：

1. **分组。** `CoralProcessor` 将每个报告归档到其 `parent_hash` 下。一个组在累积到 `group_size` 个已评分兄弟节点后释放（`serve.yaml` 为 demo 设置为 2；默认值为 4）。从未累积到足够已评分子节点的父提交永远不会训练，该屏障是配方设置，而非 CORAL 不变量。
2. **样本组装。** 报告的有序推理引用被物化回 token、损失掩码，以及 reef 随每条 INFERENCE 记录存储的 rollout 时对数概率。一个尝试是多调用轨迹，因此多轮组装默认开启。
3. **版本守卫。** 调用跨越一次权重更新的尝试会被拒绝：其 rollout 对数概率属于两个策略，无法支持一个重要性比率。成员在不同修订版本上训练的组会被整体丢弃，因为相对奖励只能在一个策略版本内公平比较。格式错误的尝试会引发显式的训练数据错误；被丢弃的组在处理器的 `status()`（`discarded_groups`）中可见。
4. **步骤。** 一个已释放的组成为一个训练单元。配方复用 `tttd` 训练目标（分组留一法优势）和 `tttd` Slime 损失族，CORAL 兄弟组与 TTT-Discover 步骤具有相同的形状，因此配方只添加了组屏障。一个所有兄弟节点得分都相同的组仍会作为一个定义良好的零梯度步骤进行训练，而不是让屏障饥饿。
5. **循环闭合。** 该步骤更新所服务的 LoRA（demo：Qwen3-8B，在 `linear_qkv`/`linear_proj` 上的 rank-32 LoRA，使用 rollout 对数概率和针对基础模型的 KL 项进行训练，参见 `examples/coral_demo/serve.yaml`）。Reef 提供新修订版本，后续尝试既在其上运行，也被归因于它。

设计理由就在代码旁边：`processor.py`（分组和守卫）、`recipe.py`（为何复用 `tttd`）、`serve.yaml`（部署旋钮）。

## 任务

`task/` 是一个普通的 CORAL 任务 —— `coral validate task/` 接受它。demo 问题（实现一个稳定的双列表合并，按通过的检查比例评分）被故意设计得很小，以便循环在 2-GPU 服务栈上快速周转；通过编辑 `task/` 换入任何 CORAL 任务 —— 接线不会改变。`task/litellm_config.yaml` 是唯一 reef 特有的部分：网关的唯一上游是 Reef 服务，因此所服务的策略是 agent 唯一的模型。

## 已知限制

- `attach_reef_adapter_to_agent_manager` 拼接在 CORAL 管理器内部构建的中间件之下。CORAL 的网关现在暴露了一个 `header_provider` 钩子，但 reef 层还会将请求头镜像到请求体中并捕获响应回执 —— 这超出了请求头钩子的触及范围 —— 因此该拼接会一直保留，直到 CORAL 长出响应侧钩子。
- LiteLLM 构建一个全新的上游请求，因此中间件将场景和标签镜像到请求体的 `extra_headers` 中；回执头通过后缀匹配，因为转发代理会给它们加前缀。这两种行为都来自实时的 GPU 运行。
- demo 默认使用 `opencode` 运行时；CORAL 注册的任何运行时 CLI 都可以工作（`--runtime claude_code`、`codex`、……），但该 CLI 必须已安装，并且必须接受用于本地服务模型的 OpenAI 兼容网关端点。