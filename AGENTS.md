# Reef 的 Agent 指令

这些指令适用于 `Human-Agent-Society/reef` 中的 AI 辅助工作。
`AGENTS.md` 是共享的唯一事实来源；`CLAUDE.md` 是指向该文件的相对符号链接。
更新共享指令时，请编辑此文件。

阅读 [CONTRIBUTING.md](CONTRIBUTING.md) 了解项目政策。遵循你所修改区域中更具体的 `AGENTS.md`。用户的明确指令优先于仓库指南。

## 贡献工作流

- 编辑前先检查工作树，保留用户已有的更改。让每项更改聚焦于所请求的问题。
- 在创建 Issue 或 Pull Request 之前，搜索现有 Issue 和 PR，查找是否有重叠的工作。对于架构、公共契约、持久化或项目政策的更改，请遵循 `CONTRIBUTING.md` 中的 RFC 标准。
- 在更改之前，先复现 bug 并检查相关实现。避免推测性修复、无关的格式调整和不必要的抽象。
- 使用 [.github/PULL_REQUEST_TEMPLATE.md](.github/PULL_REQUEST_TEMPLATE.md)。说明问题、产生的行为、兼容性影响以及实际验证结果。披露非平凡的 AI 辅助。人类贡献者仍需负责审查和理解该更改。
- 将凭据和私人记录排除在日志、测试夹具和提交之外。漏洞报告请遵循 [SECURITY.md](SECURITY.md)。

## 代码库与边界

Reef 为模型权重和 agent harness 连接推理、反馈、学习和版本化交付。发行包为 `reef-infra`；导入时使用 `reef`。使用[代码库地图](docs/contributing/codebase-structure.rst)和受影响包的 `__init__.py` 文档字符串来查找更改的归属方。

| 位置 | 职责 |
| --- | --- |
| `reef/core/` | 共享值类型、通信契约和错误 |
| `reef/dispatcher.py`、`reef/scenario/` | 协调、场景状态、提交排序和恢复 |
| `reef/service/` | HTTP、认证、流式传输和部署 |
| `reef/recipe/`、`reef/train/` | Recipe 契约、处理器、训练、评估和后端集成 |
| `reef/runtime/` | 后端中立的运行时契约、调度和发布协调 |
| `reef/inference/` | 具体推理集成、引擎控制和权重接收 |
| `reef/surface/` | 已发布构建产物的交付 |
| `reef/artifact/` | 版本化构建产物和仓库 |
| `reef/storage/` | 记录存储契约、持久化和保留 |
| `reef/harness/` | Harness 适配器、渲染、运行器和轨迹 |
| `reef/record2dataset/` | 生成器服务：Harbor 任务的设计器提示词、创作关卡和 oracle 检查、任务播放器作业 |
| `recipes/`、`tutorials/` | 方法实现、可运行示例和教程 |
| `tests/`、`docs/`、`docker/` | 验证、文档和部署环境 |

- 将共享机制放在 `reef/` 中，将方法特定的策略放在 `recipes/` 中。核心不得导入 cookbook 方法；`recipes/` 不会随 Reef wheel 一起发布。
- 保持 `reef-client` 为独立的、无依赖的协议客户端。Harness 使用 `reef_client`；它们不应需要 Reef 的服务或训练栈。
- 保持基础安装可在 CPU 上使用。GPU 依赖属于受支持的训练环境，具体适配器属于其集成之下。声明或固定第三方依赖，而不是复制它们。
- 保留提供方请求体、场景隔离、回执到反馈的关联以及构建产物发布/恢复契约。

## 开发环境

Python 工作请使用 `uv` 和仓库虚拟环境。复用现有环境；对于新检出的代码，通常的设置是：

```bash
git submodule update --init --recursive
uv venv --python 3.12
source .venv/bin/activate
uv pip install -e ".[dev]" -e ./third_party/reef-client
pre-commit install
```

Reef 需要 Python 3.12 或更高版本，服务和它启动的每个子进程使用同一个解释器；CI 测试 3.12。构建产物/检查点工作需要使用 Git LFS。

训练相关工作可能还需要：

```bash
uv pip install -e ".[slime]"
uv pip install --no-deps --group runtime
```

`slime` extra 提供 Python 侧的适配器依赖；`runtime` group 固定 Slime 本身。保留 `--no-deps`，以免安装替换容器中 CUDA 兼容的软件栈。遵循[开发指南](docs/contributing/development.rst)和 [docker/README.md](docker/README.md) 了解所选后端所需的环境。

## Python 风格与设计

- 遵循 `pyproject.toml`：Black 和 isort 格式为 119 列；Ruff 检查代码和命名；mypy 检查 `reef`。与附近代码保持一致，并为新增或更改的接口添加类型。
- 优先使用聚焦的函数、用于值的 data class，以及用于状态和生命周期的内聚对象。对行为使用组合和显式抽象基类。
- 不要使用 `typing.Protocol`、`typing_extensions.Protocol` 或 `runtime_checkable`。定义 `ABC` 并显式继承它。CI 检查所有第一方 Python 文件，包括教程、Docker/docs/CI 脚本和根目录文件，没有基线例外。第三方代码、本地依赖/构建树、vendored 基准测试、已发布的结果程序和黄金夹具被排除；参见 [.github/scripts/check_python_design.py](.github/scripts/check_python_design.py) 中的路径。
- 不要使用 `TYPE_CHECKING`。修复依赖方向或移动共享契约，使注解导入在运行时可用。
- 不要将长期存在的行为建模为 `Callable` 构造函数参数、可调用值字段或回调容器。使用显式接口。将不可避免的第三方动态行为保留在其边界适配器本地。
- 不要在 `reef/` 中使用 `assert` 或 `del` 语句；使用显式验证、异常和变更 API。测试断言是允许的。
- 使用描述性的领域名称和简短注释来解释意图或约束。遵循 `CONTRIBUTING.md` 中的 Google 风格文档指南。
- 不要通过添加广泛的抑制或扩充 `.github/python-design-baseline.txt` 来容纳新的违规，从而绕过检查。

### 代码结构与可读性

在编写或审查代码时应用这些规则。[代码审查风格指南](https://github.com/zhaochenyang20/sglang-diffusion-routing/issues/32)是参考；当两者不同时，以这里的规则为准。

- **避免碎片化的函数。** 当仅使用一两次的小辅助函数只是拆分一个连续操作时，将其内联。为有意义的职责或大量复用而提取函数，而不是为了满足任意的行数限制。将相关逻辑保持在一处可读。
- **使用普通的标识符名称。** 只有嵌套在其他函数内部的函数才应使用前导下划线。其他变量、字段、常量、模块级函数和方法应以英文字母开头。保留 Python 要求的特殊名称，如 `__init__`。
- **处理现实的失败。** 在入口点验证类型和数据，然后在内部依赖这些契约。保持 `try/except` 块狭窄，仅在有意义的恢复或错误转换时捕获特定的、预期的失败。不要添加推测性回退、重复检查或针对不可能状态的测试。测试应简洁，覆盖可观察行为和操作的主要失败模式。
- **将常量与其消费者放在一起。** 如果某个常量在一个模块中定义，仅为了被另一个模块导入，则应在消费模块中定义它。当常量有实际共享用途时才共享；避免不必要的导入和仅为单一用途值而设的独立模块。
- **显式描述类型。** 不要使用 `Any` 字段或注解来绕过 pre-commit、mypy 或其他检查。对实际值类型和接口建模，而不是削弱注解来消除失败。
- **命名实际的数量或概念。** 变量和函数名称必须表达其领域、科学或物理含义。在需要时包含单位或表示，例如 `timeout_seconds`、`token_count` 或 `weight_dtype`；避免任意缩写和模糊的占位符名称。
- **使分支完整且浅层。** 在选择值、赋值变量或返回替代结果时，使用显式的 `else` 分支。提前返回、抛出或继续的守卫子句不需要 `else`。尽早检查类型和无效数据，以保持主路径清晰，避免过多的分支或深度嵌套的 `if/else` 块。
- **使用显式属性和接口。** 不要使用 `getattr`、`hasattr` 或动态属性变更来猜测对象能力或添加防御性默认值。在正在更改的代码中，用类型化属性、直接访问和显式接口替换这些模式；修复契约，而不是在运行时探测属性。

## 命名与术语

优先使用本仓库中已使用的简单、常规术语。

当更简单的名称足够时，不要引入抽象或不常见的术语。
特别要避免以下术语：

- ledger
- sidecar
- provenance
- evidence

除非该术语已在代码库中确立，或是该概念的标准技术术语。

优先使用具体的替代方案，例如：

- metadata
- manifest
- record
- source information
- checksum
- validation result
- training metadata

添加新概念时，在发明新术语之前，先复用现有的仓库词汇。

在代码、UI 文本、日志和文档中，优先使用具体的名称而非隐喻：

- 对于条件使用 `check`，对于检查有效性使用 `validation`，对于运行和评分任务使用 `evaluation`，而不是 `gate`。
- 对于结果使用 `result`，对于候选选择使用 `selection_result`，对于审查使用 `review_result`，而不是 `verdict`。
- 命名实际的操作或数量：`evaluation_score`、`evaluation_tasks`、`result_html` 和 "passed the checks" 比隐喻性名称更清晰。
- 将术语更改应用于标识符、序列化键、生产者、消费者、测试和文档。新记录应使用首选的键。在需要兼容性的地方，通过有针对性的回退读取或 API 别名保留较旧的记录和调用方；将这些遗留名称保留在边界处，而不是将其传播到新代码或显示的标签中。
- 保留标准技术名称，如神经网络 gate 和第三方标识符。

## 测试与检查

从最小的现有测试套件开始，该套件覆盖更改的行为。扩展附近的测试和夹具；检查可观察结果和失败案例。公共接口更改需要契约测试。对于训练或性能更改，包含相关的评估或基线比较。

```bash
# 聚焦测试示例；替换为与更改相关的测试套件。
.venv/bin/python -m pytest tests/reef_service/test_reef_artifacts.py -q

# Python 检查（先激活 .venv 以使用 pre-commit 的本地 Hook）。
pre-commit run --files path/to/changed_file.py
.venv/bin/python -m mypy

# 为 Python 更改请求审查前的完整检查。
pre-commit run --all-files
.venv/bin/python -m pytest tests/
```

完整测试套件即使仅收集也需要训练依赖。一些测试会跳过不可用的可选运行时；其他测试直接导入 Slime 和 torch。使用[测试指南](docs/contributing/testing.rst)和 [CI 配置](.github/workflows/ci.yml)来复现所需的环境。CI 使用 `GIT_CONFIG_GLOBAL=/dev/null` 和 `GIT_CONFIG_SYSTEM=/dev/null` 进行 Git LFS 测试隔离。如实记录跳过或不可用的检查。

对于 Python 3.12 上的完整测试套件覆盖率，使用
`.venv/bin/python -m pytest tests --cov=reef --cov-report=term`。
配置的覆盖率下限适用于整个包，而非聚焦运行。

Pre-commit 即使使用 `--files` 调用，也包含仓库范围的 Python 设计、语句和 README 配对检查。审查格式化工具的编辑，并保持无关的工作树更改完好。

## 区域特定指南与文档

- 在添加组件之前，阅读
  [adding-components.rst](docs/contributing/adding-components.rst)。
- 对于 recipe 和示例，阅读 [recipes/AGENTS.md](recipes/AGENTS.md) 和该方法的 README。保持示例自包含。
- 对于 harness 适配器，阅读
  [harness-adapters.rst](docs/developer-guide/harness-adapters.rst)。
  `tests/reef_service/data/harness_goldens/` 下的黄金 harness 树是测试夹具，包括其指令文件；保留精确输出。
- 对于文档站点更改，阅读 [docs/site/AGENTS.md](docs/site/AGENTS.md)。
  使用 Node.js 22，从 `docs/site/` 运行 `npm ci`、`npm run check:docs`、`npm run lint` 和 `npm run build`，与 docs CI 作业保持一致。
- 保持 `README.md` 和 `README.zh.md` 同步。审查两者后，运行
  `.venv/bin/python .github/scripts/check_readme_i18n.py --write`，然后
  `.venv/bin/python .github/scripts/check_readme_i18n.py`，并将更新后的 `README.i18n.yaml` 包含在更改中。
- 随代码一起更新受影响的 API、配置和用户文档。保持本指南简洁，并将详细规则链接到其所属文档。