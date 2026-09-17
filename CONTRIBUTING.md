# 为 Reef 做贡献

感谢你帮助改进 Reef。贡献可以包括缺陷报告、文档、测试、示例、代码、设计提案和评审。

## 开始之前

- 在创建新的 Issue 或 Pull Request 之前，先搜索已有的 Issue 和 Pull Request。
- 让每个 Issue 和 Pull Request 专注于一个问题或提案。
- 对于缺陷，请使用缺陷报告表单，并附上可复现的示例。
- 对于方向或范围不确定的变更，请先创建一个 Issue，再投入实现。
- 切勿在 Issue、Pull Request、日志、测试夹具或提示词记录中包含凭据、私有数据、模型提供商令牌或其他机密信息。

## 选择合适的 Issue 路径

使用与工作内容匹配的结构化模板：

- **缺陷报告（Bug report）** 用于可复现的错误行为。请附上最小的完整复现步骤和环境详情。
- **性能报告（Performance report）** 用于可测量的延迟、吞吐量、内存、利用率或可扩展性回退。请附上等效的基线和当前测量结果。
- **使用问题（Usage question）** 当文档以及已开启或已关闭的 Issue 无法回答某个具体问题时使用。
- **功能提案（Feature proposal）** 用于范围尚不需要持久架构决策的具体用户问题。
- **实验（Experiment）** 用于论文复现、基准测试或经验性问题，需固定模型、工作负载、基线、指标并保存结果。
- **示例（Example）** 用于可运行的用户向配方或参考部署，需有文档化的设置和预期结果。
- **RFC 提案（RFC proposal）** 当变更可能影响公共接口、持久化、信任边界、拓扑、配方扩展契约、训练后端或项目级策略时使用。
- **维护者任务（Maintainer task）** 用于维护者已创建或批准的范围明确的实现、重构、清理或项目工作。
- **路线图（Roadmap）** 仅用于维护者拥有的、有时间限制的协调 Issue。

### Issue 标题与分类

Issue 标题以一个受控类型前缀开头，且仅有一个：

| 前缀 | 用途 |
| --- | --- |
| `[Bug]` | 可复现的错误行为 |
| `[Feature]` | 新能力、算法或集成 |
| `[Performance]` | 延迟、吞吐量、内存、利用率或规模 |
| `[Task]` | 范围明确的重构、清理、文档、CI 或项目工作 |
| `[Experiment]` | 基准测试、复现或经验性评估 |
| `[Example]` | 可运行的配方、演示或参考部署 |
| `[RFC]` | 持久的架构、接口或策略决策 |
| `[Roadmap]` | 季度或发布级别的协调索引 |
| `[Question]` | 具体的使用问题 |

标题的其余部分应写成一个简洁的成果，通常以动词开头：`[Feature] Add W&B experiment tracking for training`。不要为领域、项目、状态或方法添加第二个方括号标签。特别地，应使用 `[Roadmap] Reef 2026 Q3`，而不是 `[Roadmap][Draft] ...`；在路线图正文中记录 `Draft`、`Active` 或 `Complete`。

标签承载独立的分类维度。当工作跨越边界时，维护者可以应用多个 `area:*` 标签：

- `area: core` — 核心契约和场景生命周期；
- `area: service` — 服务 API、部署和 CLI；
- `area: training` — 训练执行、运行时和算法；
- `area: artifacts` — 构建产物存储、元数据、保留和发布；
- `area: versioning` — 身份、血统、激活、提升和回滚；
- `area: observability` — 指标、日志、追踪和实验跟踪；
- `area: harness` — harness 集成、配方和演化界面；
- `area: examples` — 示例、配方和参考部署；
- `area: packaging` — 包、容器和内置集成；
- `area: ci` — 自动化、CI 和开发者工具；以及
- `area: docs` — 用户和开发者文档。

生命周期（`status:*`）、优先级（`priority:*`）、贡献和加速器验证标签不应出现在标题中。

安全漏洞不应出现在公开 Issue 中。请遵循 [SECURITY.md](SECURITY.md)，并在发送漏洞利用细节之前建立私密报告渠道。

所有新 Issue 以 `status: needs-triage` 开始。维护者在评审后添加领域标签和一个生命周期状态。如果维护者请求信息，Issue 可能会获得 `status: waiting-author`；Issue 作者的新评论会自动将其返回分诊。`help wanted` 表示维护者欢迎外部贡献者提出计划并请求分配。这不是在没有协调的情况下提交竞争性 Pull Request 的邀请。

## 需要 RFC 的变更

在实现以下变更之前先编写 RFC：

- 在 `reef` 下添加新的顶层包；
- 添加共享训练后端或更改配方扩展契约；
- 做出重大或向后不兼容的公共接口变更；
- 更改持久化格式、传输契约、信任边界或服务拓扑；或
- 建立将约束后续工作的项目级策略。

小型缺陷修复、文档改进、测试、保持行为的内部重构，以及已接受 RFC 的实现工作通常不需要新的 RFC。

创建一个 [RFC Issue](https://github.com/Human-Agent-Society/reef/issues/new?template=rfc.yml) 并在 Issue 正文中完成完整提案。该 Issue 就是 RFC 及其决策记录；不要在 `docs/rfcs` 下添加新文档。将实质性设计变更、维护者决策和实现链接保留在 Issue 上。RFC 必须在那里被明确接受，其实现才会被视为已批准的项目方向。

## AI 辅助贡献

当 AI 辅助工作由人类贡献者指导、理解和验证时，是受欢迎的。不要提交自主或批量生成的 Issue、Pull Request、评审或评论。

当 AI 工具做出非平凡贡献时：

- 在 Pull Request 中披露该工具及其使用方式；
- 审查并理解每一行提交内容和事实性声明；
- 自己复现问题，而不是信任生成的诊断；
- 运行相关检查并报告其实际结果；
- 移除推测性修复、无关清理、生成的评论和不必要的抽象；以及
- 用自己的语言与维护者沟通，并对后续评审和维护负责。

AI 辅助不会降低测试、文档、兼容性、安全性或长期维护的标准。当评审低质量或无法验证的生成工作比直接复现或实现变更成本更高时，可能会在没有详细评审的情况下关闭。

## 设置仓库

遵循[开发指南](https://reefinfra.ai/docs/contributing/development/)来初始化子模块、创建环境、安装依赖并启用 `pre-commit`。

常见的本地检查是：

```bash
pre-commit run --all-files
pytest tests/
```

### 保持根 README 同步

`README.md` 和 `README.zh.md` 是一对经过评审的文档。对任一文件的更改必须在需要时更新另一个文件，并保持相同的标题、列表、表格、链接目标和围栏代码。在评审两种语言后，记录它们确切的 Git blob 哈希并验证该对文件：

```bash
python .github/scripts/check_readme_i18n.py --write
python .github/scripts/check_readme_i18n.py
```

`README.i18n.yaml` 中的记录会使任何后续的单方面编辑在本地 pre-commit 检查和 CI 中失败。结构检查不评判翻译质量，因此其成功不能替代对含义和措辞的人工评审。

完整的测试套件需要受支持的容器环境和训练依赖。有关具体命令和依赖详情，请参阅[测试指南](https://reefinfra.ai/docs/contributing/testing/)。

## 理解代码库

使用[代码库结构图](https://reefinfra.ai/docs/contributing/codebase-structure/)来决定哪个包拥有某项变更。每个包的 `__init__` 文档字符串陈述其领域内的不变量；该图是仓库范围的路由指南。

在添加配方、训练集成、运行时、界面、构建产物类型、服务路由或 harness 适配器时，请遵循[组件操作手册](https://reefinfra.ai/docs/contributing/adding-components/)。它们列出了同一 Pull Request 中预期的实现位置、注册点、测试、配置和文档。

## Python 编码风格

Python 变更应遵循 [PEP 8](https://peps.python.org/pep-0008/)、[Google Python 风格指南](https://google.github.io/styleguide/pyguide.html)以及以下原则。当通用指南与 Reef 的机械风格不同时，以 `pyproject.toml` 中的仓库配置为准：Black 格式化代码，isort 排序导入，Ruff 检查常见错误和可维护性问题，mypy 检查 `reef` 包。不要手动格式化代码以对抗这些工具。

### 编写 Pythonic 代码

- 优先使用直接的 Python 构造和标准库，而非自定义抽象。代码应使其控制流和数据流显而易见。
- 当迭代器、推导式、上下文管理器、解包和标准协议能提高可读性时使用它们。当推导式需要复杂条件或副作用时，使用普通循环。
- 保持函数专注。当提取辅助函数能为概念赋予有用的名称或消除有意义的重复时，才进行提取，而不仅仅是为了缩短函数。
- 避免聪明的元编程、隐藏的全局状态和意外的副作用。使依赖和状态转换显式化。
- 为新接口或变更的接口添加类型注解。除非边界确实是动态的，否则不要使用 `Any` 来避免描述类型。
- 当公共模块、类、函数和方法的目的、契约或失败模式无法从签名中清晰看出时，为其编写文档字符串。注释应解释*为什么*某个选择是必要的，而不是重述代码*做了什么*。

### 有意识地使用面向对象设计

- 当数据和行为构成具有状态、不变量、生命周期或多态契约的内聚对象时，使用类。对于无状态转换优先使用函数，对于纯数据值优先使用 dataclass。
- 给每个类一个明确的职责，并保持其公共接口小。构造有效的对象，而不是依赖调用者按特定顺序设置属性。
- 优先使用组合和小型抽象接口，而非深层继承层次。继承应表示真正的可替换关系，而不仅仅是重用实现。
- 封装可变状态并暴露能揭示意图的操作。当直接属性访问或属性更清晰时，不要添加 Java 风格的 getter 和 setter。
- 将 I/O 和框架集成保持在边缘，以便核心行为可以用普通 Python 对象测试。

### 避免动态设计捷径

- 不要使用 `TYPE_CHECKING`。注解所需的导入在运行时也必须有效。通过将共享契约移到较低层模块、纠正依赖方向或在集成边界使用本地运行时导入来解决循环。
- 不要使用 `typing.Protocol`、`typing_extensions.Protocol` 或 `runtime_checkable`。定义抽象基类并显式继承它们。Python 设计检查在所有第一方 Python 文件中强制执行此规则，包括 `reef/`、`recipes/`、`tests/`、`tutorials/`、`docker/`、`docs/`、`.github/scripts/` 和根文件。它使用 [.github/scripts/check_python_design.py](.github/scripts/check_python_design.py) 中的路径排除第三方代码、本地依赖/构建树、内置基准输入、已发布的结果程序和黄金夹具。Protocol 发现项不能通过设计基线豁免。现有的 `TYPE_CHECKING` 和 Callable 检查保持其 `reef/`、`recipes/` 和 `tests/` 范围。
- 不要将长期行为建模为 `Callable` 构造函数参数、可调用值字段或回调容器。定义一个具有有意义方法的抽象基类或一个内聚的类，使契约、状态和生命周期显式化。
- 单个短生命周期回调对于算法、装饰器或标准库适配器可能是合适的。需要多个回调的函数表明这些操作属于同一个接口。
- 当显式接口或普通多态能表达相同设计时，不要通过 `getattr`、猴子补丁或运行时检查使用动态分派。第三方框架的边界适配器必须将此类行为保持在本地，并记录其必要性。
- 不要用类型别名或 lint 抑制来隐藏设计策略违规。任何罕见的例外必须范围狭窄、在评审中说明理由，并记录在设计检查基线中。

### 为读者选择名称

- 模块、函数、方法和变量使用 `snake_case`；类使用 `PascalCase`；常量使用 `UPPER_SNAKE_CASE`。
- 用具体名词短语命名类，用动词短语命名函数。谓词名称应读作问题，如 `is_ready`、`has_capacity` 或 `can_retry`。
- 优先使用领域词汇和完整、熟悉的术语，而非内部简写。当存在更具体的名称时，避免使用 `data`、`info`、`obj`、`manager`、`helper` 或 `utils` 等模糊名称。
- 当歧义可能导致缺陷时，包含单位或表示形式，例如 `timeout_seconds`、`token_count` 或 `checkpoint_path`。
- 在代码、配置、日志和文档中保持术语一致。一个概念应有一个名称。
- 在小范围内使用短的约定名称是可以的（索引循环中的 `i`，本地文件句柄的 `f`）。不要将它们带到更大的范围。

例如，优先使用：

```python
def select_ready_workers(workers: Iterable[Worker]) -> list[Worker]:
    return [worker for worker in workers if worker.is_ready]
```

而非隐藏领域和意图的名称：

```python
def process(data):
    return [x for x in data if x.status]
```

### 自动化检查与评审

自动化检查强制执行本指南中可可靠评估的部分：

- Black 和 isort 强制执行格式和导入顺序。
- Ruff 的 Pythonic 和正确性规则捕获不必要的复杂构造、易错模式和常见性能问题。
- Ruff 的 `pep8-naming` 规则强制执行上述命名形式。已建立的公共 API 和数学符号的狭窄例外记录在 `pyproject.toml` 中。
- mypy 检查 `reef` 包中的类型一致性。
- Python 设计策略检查拒绝 `Protocol`、`runtime_checkable`、`TYPE_CHECKING` 以及新的基于 Callable 的对象状态或回调捆绑。其基线标识 Callable 模式的现有迁移债务；Protocol 和 `TYPE_CHECKING` 发现项不能加入基线。

自动化无法确定一个类是否是合适的抽象、标识符是否使用了最清晰的领域术语，或接口是否具有一个内聚的职责。作者和评审者必须在评审期间评估这些设计和可读性要求。不要仅仅为了消除警告而添加 lint 例外；保持其狭窄并解释为什么通用规则不适用。

在为 Python 变更请求评审之前，运行：

```bash
pre-commit run --all-files
python -m mypy
pytest tests/
```

## 创建 Pull Request

在请求评审之前：

- 保持 diff 尽可能小，并移除无关更改；
- 解释更改了什么、为什么需要以及如何验证；
- 当存在相关 Issue 或 RFC 时链接它们；
- 为行为变更添加或更新测试；
- 为公共接口变更添加契约测试；
- 更新受变更影响的用户、运维或开发者文档；
- 运行相关的本地检查并报告命令和结果。

当设计或实现尚未准备好接受时，使用草稿 Pull Request。不要将功能性变更与顺手格式化、生成的改写或无关清理混在一起。

草稿 Pull Request 仅运行 lint、类型检查和静态 Dockerfile 检查。一旦准备就绪，每次更新会自动在 Python 3.12 上运行源码、沙箱和已安装 wheel 套件，包括组合覆盖率检查。当文件变更时，文档构建和真实 harness 冒烟测试会自动运行。在本地运行相关检查，并在推送前批量处理每一轮评审修复。

在合并之前，在最终修订版上运行完整的 Python 3.10/3.11/3.12 测试和包矩阵。任何具有仓库写权限的贡献者都可以通过重新运行整个最新的 `ci` 工作流来请求：

```bash
gh run rerun RUN_ID --repo Human-Agent-Society/reef
```

Actions UI 中的等效操作是 **Re-run all jobs**。重新运行整个工作流会重新计算矩阵以进行完整验证；**Re-run failed jobs** 可能会重用之前的矩阵，旨在重试失败，而非扩展覆盖范围。不需要单独的维护者批准。没有仓库写权限的作者可以请协作者请求最终完整运行。

常规运行会让所需的 Python 3.10/3.11 检查保持待定状态。这些检查必须在合并前的最新修订版上通过；仅 Python 3.12 常规运行是不够的。新的推送会取消较旧的运行并返回常规矩阵。完整重跑会拒绝已关闭、草稿或已被取代的 PR 修订版。主分支推送和手动工作流调度会运行所有受支持的 Python 版本；使用 PR 的 `ci` 重跑来满足其合并检查。

## 评审与接受

维护者根据受影响的领域安排评审。[维护模型](.github/MAINTAINER.md)定义了维护者、合并值班和领域评审者的职责以及合并流程。评审者可以要求对正确性、接口、测试、文档、兼容性、可操作性或范围进行更改。作者应回应实质性评论，并在请求不清楚或不同意时说明。

只有当所需评审完成、所需检查通过且没有阻塞性讨论时，Pull Request 才准备好合并。通过 CI 并不保证被接受：维护者还会考虑项目方向、长期维护成本、兼容性和评审者容量。

如果评审已经沉寂，欢迎在 Pull Request 上给出简洁而礼貌的提醒。项目无法保证响应或合并时间线。非草稿 Pull Request 在 60 天无活动后被标记为过期，并在 21 天后关闭；Issue 在 90 天无活动后被标记，并在 30 天后关闭。当预期无活动时，维护者可以应用 `status: keep-open` 或 `status: blocked`。已关闭的工作在再次变得相关时可以重新打开或重新提交。

## 许可证

通过贡献，你同意你的贡献可以根据仓库的 [Apache License 2.0](LICENSE) 分发。