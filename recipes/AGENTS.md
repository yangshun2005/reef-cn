# 如何编写新示例

每个示例都是一个自包含的 Harbor 任务 + Reef 测试框架，由 `run.sh` 通过 [reef-eval](https://github.com/Human-Agent-Society/reef-eval) 将它们连接在一起。
复制 `recipes/basic/` 作为起点；某个方法的示例位于
`recipes/<method>/examples/<example>/` 下；这是最小有效结构。

## 目录布局

```text
my_example/
  harbor/                任务定义（由 reef-eval/Harbor 使用）
    task.toml             元数据、超时、资源限制
    instruction.md        展示给模型的提示词
    environment/
      Dockerfile          验证器运行所在的容器镜像
    solution/             可选的种子/初始解决方案文件
    tests/
      test.sh             验证器：将奖励写入 /logs/verifier/reward.txt
  harness/                智能体测试框架（导入 reef_client，而非 reef）
    __init__.py           延迟导出 HarborAgent
    agent.py              HarborAgent(BaseAgent) — 智能体逻辑
    report.py             可选：向 Reef 提交可训练的验证器奖励
  my_example.yaml         Reef 配置（recipe、推理、训练、HTTP 设置）
  run.sh                  启动 reef serve，然后使用测试框架运行 reef-eval
  pyproject.toml          使 harness/ 可被 reef-eval 的 uvx 环境导入
  README.md               该示例演示的内容以及如何运行
```

## 没有顶层 `__init__.py`

示例目录**不是** Python 包。没有
`my_example/__init__.py`。`harness/` 子目录才是包，由
`pyproject.toml` 声明：

```toml
[project]
requires-python = ">=3.12"
dependencies = [
    "reef-client",
    "reef-eval[harbor]",
]

[tool.setuptools.packages.find]
where = ["."]
include = ["harness*"]
```

只有不使用 reef-eval 的示例（例如，直接使用
`reef_client` 的活动驱动器）才应省略 reef-eval 依赖。

`run.sh` 通过
`--with-editable "$PWD"` 将其安装到 reef-eval 的临时环境中。

## harness/`__init__.py`

延迟导出 `HarborAgent`，这样导入测试框架包时就不需要
`harbor` 运行时 — reef-eval 在解析
`--agent harness:HarborAgent` 时加载它：

```python
__all__ = ["HarborAgent"]

def __getattr__(name: str):
    if name == "HarborAgent":
        from .agent import HarborAgent
        return HarborAgent
    raise AttributeError(f"module {__name__!r} has no attribute {name!r}")
```

## harness/`agent.py`

继承 `harbor.agents.base.BaseAgent`。`run()` 方法就是智能体
逻辑：通过 Reef 的 `/v1/chat/completions` 调用模型，将答案
写入指令指定的路径，并将推理回执暂存到
`context.metadata` 中，以便报告器能将验证器奖励关联回它。

连接设置来自 `run.sh` 设置的环境变量：

- `REEF_SERVICE_URL`（必需）
- `REEF_SCENARIO`（必需）
- `REEF_TOKEN`
- `REEF_TIMEOUT_S`

当任务验证器的奖励是场景 recipe 所消费的反馈时，
一个后台线程会监视 Harbor 的 `result.json`，并通过
`harness.report.post_report` 将其提交到 Reef。完整模式见 `basic/harness/agent.py`。当测试框架已经提交了其训练
反馈且最终的 Harbor 奖励仅用于评估时，不要添加该报告器；Harbor 会将该
值保留在试验结果中。

## harness/`report.py`（可选）

从 Harbor 的试验结果中提取验证器奖励，并将其作为
引用该试验推理回执的报告提交到 Reef。报告 id 是
确定性的（试验 id 的 `uuid5`），因此重复提交是无操作。

仅当 recipe 消费验证器奖励时才使用此路径。明确不符合训练条件的
终端评估应放在 Harbor 的
试验结果中，而不是训练场景的报告契约中。

## `my_example.yaml`

Reef 服务配置。最小的仅记录配置（无训练栈）是：

```yaml
schema-version: 2
reef:
  host: 127.0.0.1
  port: ${REEF_PORT}
  token: ${REEF_TOKEN}
inference:
  upstream-url: ${REEF_UPSTREAM_URL:?}
  upstream-model: ${REEF_UPSTREAM_MODEL:?}
  upstream-api-key: ${REEF_UPSTREAM_API_KEY}
recipe:
  implementation: recipe
storage:
  agent-record-dir: ${REEF_WORK}/agent-record
  artifact-repository: ${REEF_WORK}/artifacts.git
  artifact-work-dir: ${REEF_WORK}/artifact-work
  artifact-cache-dir: ${REEF_WORK}/artifact-cache
```

将随附的 Reef 配置保持在版本化的公共布局中。训练
示例将原生训练标志放在 `training.options` 中，将引擎标志放在
`inference.options` 中，将推理 GPU 容量/并行度放在 `inference.num-gpus`
和 `inference.tensor-parallel-size` 中；Reef 会组装它们的工作器拓扑。不要向版本 2 YAML 添加 `service` 或 `services`。HTTP
设置属于 `reef`。独立部署方法拥有的服务，并通过 recipe 字段传递
它们的端点；Reef 仅协调原生推理
和训练。外部服务启动和 GPU 隔离见 `recipes/openclawrl/examples/openclawrl/docker-compose.yaml`。Recipe 字段和拥有的部分放在
`recipe.config` 下。Docker Compose 和第三方任务文件保留它们
自己的 Schema。旧版 Reef 布局属于兼容性测试。

## `run.sh`

1. 设置环境变量（端口、令牌、场景、工作目录）；在 YAML 中选择
   部署 recipe。
2. 在后台启动 `reef serve -c <yaml>`；等待 `/healthz`。
   `reef serve` 会找到 YAML 旁边的 `recipes/` 包，并将其放入
   每个服务的 `PYTHONPATH`；脚本不设置它。
3. 使用 `--with-editable "$PWD"`（安装 `harness/`）和
   `--with reef-client`（从 PyPI 安装 SDK）运行 reef-eval。
4. reef-eval 解析 `--agent harness:HarborAgent` 并运行 Harbor 任务。

最小版本见 `basic/run.sh`（60 行）。`tttd/run.sh` 添加了
模型下载和 GPU 栈启动。

## harbor/`task.toml`

声明任务元数据、超时和资源限制：

```toml
version = "1.0"

[metadata]
author_name = "Reef"
difficulty = "easy"
category = "integration"
tags = ["reef", "harbor"]

[verifier]
timeout_sec = 30.0

[agent]
timeout_sec = 300.0

[environment]
build_timeout_sec = 120.0
cpus = 1
memory_mb = 512
storage_mb = 1024
gpus = 0
```

## harbor/`instruction.md`

展示给模型的提示词。保持其自包含：测试框架从 Harbor 的
`instruction` 参数读取它并将其传递给模型。

## harbor/`solutions/`（可选）

测试框架加载以引导其搜索的种子或初始解决方案文件。
测试框架在启动时读取它们（例如，为 PUCT 存档播种或提供
模型必须改进的基线）。并非每个任务都需要这个 — `basic/`
没有；`tttd/` 在运行时用空字符串为其存档播种。该
目录是一种约定，用于当任务附带具体的起点
太大或太结构化而无法内联到代码中时。

## harbor/environment/`Dockerfile`

验证器运行所在的容器。最小示例：

```dockerfile
FROM debian:bookworm-slim
WORKDIR /workspace
```

## harbor/tests/`test.sh`

验证器脚本。Harbor 在
`run()` 返回后在环境容器内运行它。将浮点数写入 `/logs/verifier/reward.txt`：

```sh
#!/bin/sh
set -eu
if [ -f /workspace/answer.txt ] && [ "$(cat /workspace/answer.txt)" = "391" ]; then
    echo 1 > /logs/verifier/reward.txt
else
    echo 0 > /logs/verifier/reward.txt
fi
```

## 测试

测试框架的单元测试位于仓库根目录的 `tests/test_<example>_harness.py`。
它们直接从
`recipes.<method>.examples.<example>.harness.*` 导入 — 测试将
`REPO_ROOT` 插入 `sys.path`，使 `recipes/` 可导入，并且每个 `examples/`
目录都解析为命名空间包（每个示例的 `harness/` 有自己的 `__init__.py`，但
示例目录本身没有）。

## 自包含性

每个示例必须完全自包含：没有跨示例的 Python 导入。
如果两个示例共享一个模式（评分、沙箱、搜索），每个都复制
自己的副本。唯一的共享依赖是外部的：`reef_client`、
`harbor` 和 `reef` 包本身（用于 `reef serve`）。