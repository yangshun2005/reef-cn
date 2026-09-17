

> **[中文化] reef**
>
> 此项目是 [Human-Agent-Society/reef](https://github.com/Human-Agent-Society/reef) 的中文翻译版本。
> - 原项目 Stars: 2633
> - 主语言: Python
> - 许可证: Apache-2.0
> - 翻译日期: 2026-09-17
> - 原始 README: [README_en.md](README_en.md)
>
> 如有翻译不准确之处，欢迎提 Issue 或 PR。

---


---

<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/assets/reef-logo-dark.svg">
  <img src="docs/assets/reef-logo-light.svg" alt="Reef" width="220">
</picture>

<h3>面向自我改进智能体的持续学习基础设施</h3>

[![CI](https://github.com/Human-Agent-Society/reef/actions/workflows/ci.yml/badge.svg)](https://github.com/Human-Agent-Society/reef/actions/workflows/ci.yml)
[![PyPI package: reef-infra](https://img.shields.io/pypi/v/reef-infra?label=PyPI%3A%20reef-infra&logo=pypi&logoColor=white)](https://pypi.org/project/reef-infra/)
[![Python](https://img.shields.io/badge/python-3.12%2B-3776AB?logo=python&logoColor=white)](pyproject.toml)
[![License](https://img.shields.io/badge/license-Apache--2.0-green)](LICENSE)

[English](README.md) | 中文

<div align="left">

Reef 是首个面向持续自我改进智能体的开源基础设施。
它连接了智能体推理、反馈、学习和版本化交付。你可以用它
通过 Slime 和 SGLang 训练模型权重，或改进智能体的运行框架（harness），包括其提示词、规则和技能。


</div>

**🚀 [快速开始](https://reefinfra.ai/docs/getting-started/quickstart/) |
🗺️ [路线图](https://github.com/Human-Agent-Society/reef/issues/25) |
📣 [发布文章](https://x.com/ao_qu18465/status/2094867930081337730) |
💬 [加入 Discord](https://discord.gg/5y8e5f937k) |
📱 [加入微信群](docs/community/wechat.md)**

</div>


## 🎯 何时使用 Reef

当你希望智能体仅通过从你与智能体的交互中学习就能持续改进时，请使用 Reef。

| 你的目标 | 学习路径 | 你需要什么 |
|---|---|---|
| 持续获得为你量身打造的更强模型 | 模型权重训练 | 一个可训练的模型、受支持的 GPU 技术栈，以及你的配方（recipe）可用的反馈 |
| 让你的运行框架实现自我改进 | 运行框架优化 | 一个模型端点、有代表性的任务和一个评估器；无需本地训练 GPU |
| 科学发现 | 测试时训练 | 一个执行环境、一个正确性检查器和一个可度量的目标 |


## 🧩 Reef 如何融入你的技术栈

| 能力 | 推理引擎（vLLM、SGLang、……） | RL 训练框架（Slime、veRL、AReaL、……） | **Reef** |
|---|:---:|:---:|:---:|
| 服务实时流量 | ✅ | ❌ | ✅ |
| 训练权重 | ❌ | ✅ | ✅ |
| 版本管理 | ❌ | ❌ | ✅ |
| 在更新期间保持在线 | ❌ | ❌ | ✅ |
| 超越权重的演进（技能、运行框架） | ❌ | ❌ | ✅ |


## 🔄 工作原理

<div align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/assets/loop-animation-dark.svg">
  <img src="docs/assets/loop-animation-light.svg" alt="Reef 服务请求、记录反馈、生成更新，并将被接受的更新提交到版本历史中。" width="76%">
</picture>
</div>

Reef 将每个学习周期分为四个步骤处理。下表还展示了实现每个步骤的模块。

| 步骤 | 发生什么 | 所在位置 |
|---|---|---|
| **1&nbsp;·&nbsp;服务** | 服务智能体请求并记录交互。 | [`service/`](reef/service) — 智能体请求和交互记录<br>[`runtime/`](reef/runtime) — 推理和构建产物更新 |
| **2&nbsp;·&nbsp;观察** | 将反馈与记录的交互进行匹配。 | [`storage/records.py`](reef/storage/records.py) — 存储的交互和反馈<br>[`train/processors/`](reef/train/processors) — 反馈匹配和资格判定 |
| **3&nbsp;·&nbsp;成长** | 从符合条件的记录中生成更新。 | [`recipe/`](reef/recipe) — 配方集成<br>[`train/`](reef/train) — 批次和更新任务 |
| **4&nbsp;·&nbsp;提交** | 应用配置的选择策略并发布被接受的更新。 | [`train/evaluation/`](reef/train/evaluation) — 候选评估<br>[`artifact/`](reef/artifact) — 版本历史<br>[`surface/`](reef/surface) — 构建产物交付 |


## 📦 安装

> 💡 **注意**
>
> Reef 的构建产物和检查点功能需要 `git-lfs` 系统包。
> Reef 会为其构建产物仓库在本地初始化 Git LFS。

我们推荐使用 [uv](https://docs.astral.sh/uv/) 来管理包，以下命令均使用它。

### 从 PyPI 安装

```bash
uv venv && source .venv/bin/activate
uv pip install reef-infra
python3 -c "import reef; print(reef.__version__)"
```

### 从源码安装

```bash
git lfs install
git clone https://github.com/Human-Agent-Society/reef.git
cd reef
uv venv && source .venv/bin/activate
uv pip install -e .
python3 -c "import reef; print(reef.__version__)"
```

开发和运行下面的训练示例时，请使用源码检出。


## 🔧 使用 Reef

Reef 支持两种学习面：模型**权重**和智能体**运行框架**。
部署的配方决定了其场景更新哪个学习面。

作为一个最小示例，将 Reef 作为纯推理服务器启动：

```bash
uv run reef serve --inference.model-path Qwen/Qwen2.5-1.5B-Instruct
```

### 权重训练部署

#### 启动部署

以下示例启动 SAO（arXiv:2607.07508）示例部署。请在满足
[演进你的模型](https://reefinfra.ai/docs/user-guide/evolve-your-model/) 中 GPU 要求的环境中，
从 Reef 源码检出运行它。

```bash
uv pip install -e ".[slime]" && uv pip install --no-deps --group runtime

export MODEL_PATH="Qwen/Qwen2.5-1.5B-Instruct"
export REEF_TOKEN="reef-local"

reef serve -c recipes/sao/examples/imo_answerbench/serve.yaml \
  --inference.model-path "$MODEL_PATH" \
  --reef.port "8900"

curl -f http://127.0.0.1:8900/healthz          # 已准备好提供服务
```

#### 发送推理请求并上报反馈

通过 Reef 发送推理请求，并为每个响应上报一个分数。SAO 配方
使用每个符合条件的已评分 rollout 来运行一个训练步骤。

Reef 的推理端点兼容 OpenAI 和 Anthropic：`/v1/chat/completions`
和 `/v1/messages` 接受对应提供商自己的请求体。请求包含
`x-reef-scenario` 请求头；新名称会使用部署所配置的配方创建一个场景。
请求不选择配方。

响应体使用提供商的 OpenAI 兼容格式。Reef 会添加
`x-reef-agent-record-id` 响应头。其值是**回执（receipt）**，后续上报用它来标识
这次交互。一次上报可以包含数值型 `score`、文本或结构化的 `feedback`，以及它所评估的回执。
此示例同时上报分数和简短说明。

```python
import os
import httpx

reef = httpx.Client(
    base_url="http://127.0.0.1:8900",
    headers={"Authorization": f"Bearer {os.environ['REEF_TOKEN']}", "x-reef-scenario": "hello-reef"},
    timeout=300,
)

# 发送一个提供商兼容的推理请求
response = reef.post(
    "/v1/chat/completions",
    json={
        "model": os.environ["MODEL_PATH"],
        "messages": [{"role": "user", "content": "Return exactly: reef is ready"}],
    },
)

response.raise_for_status()
receipt = response.headers["x-reef-agent-record-id"]
answer = response.json()["choices"][0]["message"]["content"]

# 发送关于这次推理的上报
matched = answer.strip() == "reef is ready"

reef.post(
    "/reef/report",
    json={"score": float(matched), "feedback": "matched" if matched else "wrong answer", "references": [receipt]},
).raise_for_status()
```

`feedback` 承载更丰富的信号，可以是纯文本或结构化对象，
供那些读取的不只是标量的配方使用。该端点会校验
**上报 Schema**（[`reef/core/reports/`](reef/core/reports)）。


#### 观察它学习和成长

一旦配方获得足够的反馈，它就会运行一个训练步骤，并将
更新后的权重同步到服务运行时。后续的推理请求将使用
当前版本，而无需重启 Reef。

### 运行框架演进部署

使用模型 API 而非 GPU，从自然语言请求中改进编码运行框架。

Reefine 是内置的运行框架改进配方，包含一个部署
配置；指定提供商 URL 和模型即可。在你的 Reef 源码检出和
已激活的 Python 环境中：

```bash
reef serve --recipe reefine \
  --inference.upstream-url http://127.0.0.1:11434 \
  --inference.upstream-model gemma4:26b
```

该示例连接到本地 Ollama 服务器。对于其他提供商，请更改
`--inference.upstream-url` 和 `--inference.upstream-model`，并在需要认证时设置
`REEF_UPSTREAM_API_KEY`。使用此配置，Reef
监听 `127.0.0.1:8901`，令牌为 `reef-local`，并将其状态保存在
`.reef/reefine/` 下（`--recipe harness-evolve` 是旧名称，会启动相同的
配置）。如需更改其他内容，请复制
[部署配置](reef/service/profiles/reefine.yaml) 并通过 `-c` 传入你的副本。

在另一个已激活相同 Python 环境的终端中（安装过程会将
该终端的 `python3` 固化到 `reef-pi` 中），创建一个场景、安装
运行框架，并请求一项更改：

```bash
export REEF_TOKEN=reef-local
curl -fsS -H "Authorization: Bearer $REEF_TOKEN" -H "Content-Type: application/json" \
  -d '{"name": "my-harness"}' http://127.0.0.1:8901/reef/scenarios
curl -fsS -H "Authorization: Bearer $REEF_TOKEN" -H "x-reef-scenario: my-harness" \
  'http://127.0.0.1:8901/reef/harness/install?adapter=pi' | bash

reef-pi harness "when I ask you to fix a bug, reproduce it with a failing test first"
```

在 `reef-pi` 会话中，`/reef-harness <text>` 会提交相同的请求。所服务的
模型会将更改写为技能、规则条目、智能体命令或 pi
扩展，下一次会话的更新通知会提供安装；使用 `/reef-versions` 查看
各版本，并在扩展被提供之前用
`/reef-versions <step> promote` 将其提升。如需更改模型，请用另一个
`--inference.upstream-model` 重启 `reef serve` 并重新运行安装
命令：安装会将模型 ID 写入本地运行框架配置。
脚本化的缺陷修复和研究演示请参见 [Reefine 教程](tutorials/reefine/README.md)，
配置请参见 [Reefine 指南](docs/user-guide/recipes/reefine.rst)。

## 📚 配方和示例

根据工作负载的**任务类型**以及**它应该演进什么**（模型权重还是智能体运行框架）
来选择配方。权重配方需要 GPU
训练技术栈，而运行框架配方只需要一个模型端点。下面的每个配方
都链接到其指南，每个经过测量的基准都链接到其结果
页面，而[配方目录](https://reefinfra.ai/docs/user-guide/recipes/)
为每个配方补充了代码和示例。Reefine 随 `reef-infra` 一起发布，
其他实现位于本仓库的 `recipes/` 示例集中，
通过点分类型引用选择，不随 Reef wheel 发布。

| 任务类型 | 任务形态 | 演进模型 | 演进运行框架 | 标准基准 |
|---|---|---|---|---|
| 科学发现 | 对一个难题进行反复尝试，并有可度量的目标 | [TTT-Discover](https://reefinfra.ai/docs/user-guide/recipes/tttd/)、[Guidance-TTT](recipes/tttd/examples/guidance_ttt/README.md) | 暂无 | 已测量：[TriMul](recipes/tttd/examples/guidance_ttt/results/README.md)、[圆形填充](recipes/tttd/examples/tttd/README.md#formal-8x64-results)、[Erdős 最小重叠](recipes/tttd/examples/tttd/README.md#formal-8x64-results)。 |
| 任务流上的持续学习 | 由验证器逐一评分的独立任务流 | [SAO](https://reefinfra.ai/docs/user-guide/recipes/sao/) | [Meta-Harness](recipes/meta_harness/README.md)、[GEPA](https://reefinfra.ai/docs/user-guide/recipes/gepa/) | 已测量：[AIME 2025](recipes/gepa/examples/aime/README.md#the-validation-contract)、[IMOAnswerBench](recipes/sao/examples/imo_answerbench/README.md#results)、[CEO-Bench](recipes/sao/examples/ceobench/README.md#results)、[Terminal-Bench](recipes/meta_harness/examples/terminal_bench/README.md#results)。 |
| 从使用中学习 | 无人上报分数的真实交互，或反馈延迟到达 | [OpenClaw-RL](https://reefinfra.ai/docs/user-guide/recipes/openclawrl/) | [SkillClaw](https://reefinfra.ai/docs/user-guide/recipes/skillclaw/)、[Reefine](docs/user-guide/recipes/reefine.rst) | 已测量：[带 GSM8K 任务流的模拟学生](recipes/openclawrl/examples/openclawrl/README.md#results)、[WildClawBench](recipes/skillclaw/README.md#the-2026-08-29-results-glm-53-flash-preliminary)。 |

[`recipes/basic/`](recipes/basic/) 是仅记录功能的起步技术栈，不在
目录中。关于反馈、候选编辑和发布的简短演练，请从
[编码运行框架教程](tutorials/evolve-your-harness/README.md) 开始。
每个结果页面都记录了其任务、评估设置、测量结果和
局限性。


## 📐 架构

<div align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/assets/architecture-dark.svg">
  <img src="docs/assets/architecture-light.svg" alt="Reef 架构：运行框架请求经场景流向推理。与回执关联的反馈馈入记录和配方训练；构建产物评估选择更新进行版本化发布。被拒绝的候选不会影响当前发布版本的服务。" width="1200">
</picture>
</div>

## 📖 了解更多

[文档](https://reefinfra.ai/docs/) 按以下顺序组织：

- [快速开始](https://reefinfra.ai/docs/getting-started/quickstart/)：安装 Reef、连接客户端并查看版本历史
- [HTTP API](https://reefinfra.ai/docs/reference/http-api/)：使用 HTTP API 并上报反馈
- [编写配方](https://reefinfra.ai/docs/developer-guide/write-a-recipe/)：配置 Reef 如何处理数据并生成更新
- [演进你的运行框架](https://reefinfra.ai/docs/user-guide/evolve-your-harness/)：演进运行框架而非模型权重
- [演进你的模型](https://reefinfra.ai/docs/user-guide/evolve-your-model/)：配置和运维训练部署
- [配方](https://reefinfra.ai/docs/user-guide/recipes/)：按任务类型组织的示例集
  配方目录，每个配方都包含代码、文档、示例和结果
- [核心循环](https://reefinfra.ai/docs/getting-started/core-loop/)：Reef 的核心循环
- [术语表](https://reefinfra.ai/docs/reference/glossary/)：所用术语的解释

## 🤝 社区与贡献

正在研究持续自我改进的智能体？

- [加入 Discord](https://discord.gg/5y8e5f937k) 分享你的配方、提出实现问题并讨论新功能。
- [加入微信群](docs/community/wechat.md)：群已满，请添加助手，它会邀请你。
- 加入 [GitHub Discussions](https://github.com/orgs/Human-Agent-Society/discussions) 提问、分享想法并与社区建立联系。
- 从[贡献指南](CONTRIBUTING.md)开始贡献。
- 通过 [RFC Issue](https://github.com/Human-Agent-Society/reef/issues/new?template=rfc.yml) 提出设计。
- 按照[安全策略](SECURITY.md)私下报告疑似漏洞。

如果 Reef 对你有用，请给它一个 ⭐ —— 这有助于社区发现并为项目做出贡献。


## 👥 团队

Reef 汇聚了探索智能体如何从经验中学习并
随时间改进的人们。以下成员帮助将这个想法变成可用的