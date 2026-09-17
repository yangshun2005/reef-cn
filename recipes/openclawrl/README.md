# openclawrl

将 [OpenClaw-RL](https://arxiv.org/abs/2603.10165) 的个人代理实验复现为 Reef 权重训练配方。OpenClaw-RL 从代理的日常使用中训练它：每一轮由用户接下来的行为来评分，继续推进的后续行为算作接受，抱怨则算作拒绝，策略在持续服务的同时进行更新。代理使用 Reef 作为其推理端点，不发送训练报告；其测试框架应打上一个稳定的、对话唯一的 `x-reef-tag-session` 值，而处理器可以回退到匹配扩展的对话记录。该包包含方法本身，围绕它的模拟学生实验位于 [examples/openclawrl](examples/openclawrl/README.md)。

- 论文：[arXiv:2603.10165](https://arxiv.org/abs/2603.10165)
- 固定版本：`slime` 固定为 `THUDM/slime@41014d1f29e201137fdffce737bb8bac65bc5219`（通过 `pyproject.toml` 的 `dependency-groups.runtime`）；完成的运行使用 `Qwen3-4B-Thinking-2507` 作为策略和 PRM，`Qwen3-32B` 作为学生，以及 `hermes-agent@b6bcb3e791c673e63974029bbab40cc9326803ff`（参见示例 Dockerfile），在 72 个会话的 GSM8K 作业流上进行
- 声明范围：在流的前 36 个会话上进行的一次记录运行。该运行在第 14 个会话达到了论文的适应标准，即连续三个通过的会话，并且粗体和列表比率在整个运行过程中下降。一个学生角色，一个任务族；这是一个适应结果，而非基准范围的复现。

## 目录结构

```text
openclawrl/
  recipe.py        OpenClawRLRecipe：训练规范，损失族 "openclawrl"
  processor.py     计算反馈：从记录的流量重建会话，评判轮次
  objective.py     原始奖励优势和后端损失选择
  sessions.py      从 Reef 已保留的记录中重建会话
  prm.py           处理器在私有 worker 上运行的 PRM 评判器
  slime/           训练平面目标和提示条件化的教师
  examples/openclawrl/  可运行的实验：学生模拟器、Harbor 会话、结果
```

## 其余文档的位置

[openclawrl 配方页面](../../docs/user-guide/recipes/openclawrl.rst) 涵盖配置和服务栈，[示例 README](examples/openclawrl/README.md) 记录了实现细节、流协议和记录运行。