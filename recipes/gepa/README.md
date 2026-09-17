# gepa

将 [GEPA](https://arxiv.org/abs/2507.19457) 复现为 Reef 的 harness 演化配方。GEPA 维护一个候选提示词档案库，通过 Pareto 采样选择一个父代，并基于其训练小批量的轨迹进行反思，以重写某一个组件。子代只有在训练小批量上击败父代后才能存活，随后接受完整的验证流程；Reef 会选取平均验证分数最高的候选者进行服务。该包包含方法本身；验证该方法的循环位于 [examples/aime](examples/aime/README.md)。

- 论文：[arXiv:2507.19457](https://arxiv.org/abs/2507.19457)
- 版本锁定：上游 GEPA v0.1.2（`92dadfffbe98c8ecf508179a1cab09c1bb85cd32`）、Pi `0.84.2`、任务模型 `gpt-4.1-mini-2025-04-14`，以及反思模型 `gpt-5-2025-08-07`；示例对 45/45/150 的 AIME 训练/验证/测试划分进行了版本锁定和哈希校验，并使用 150 次指标调用的搜索预算。该方法不依赖上游 GEPA 运行时。
- 结论范围：[AIME-2025 验证契约](examples/aime/README.md#the-validation-contract)在相同模型、划分、预算、评分器和 worker 数量下，将两个种子与上游 GEPA 进行配对。平均留出集提升为该方法的 +16.00 个百分点，上游为 +12.67；所选分数的差异在不同种子之间符号会发生变化，因此这些结果验证了该方法，但并未确立其优越性。确定性测试覆盖了评分器和驱动器，并在安装了 GEPA 时进行上游保真度对比。

## 目录结构

```text
gepa/
  archive.py      candidates, Pareto fronts, seeded sampling, and search budget
  backend.py      committed algorithm state and the post-commit archive mirror
  components.py   harness components as prompt texts and mutations
  method.py       reflection proposals, minibatch acceptance, and selection
  reflection.py   attributed upstream reflection prompt and record formatting
  recipe.py       GEPARecipe: configuration and evolution backend binding
  examples/aime/  the runnable validation loop: harness, gepa.yaml, and results
```

## 其余文档的位置

[gepa 配方页面](../../docs/user-guide/recipes/gepa.rst)介绍了算法和配置，[演化你的 harness](../../docs/user-guide/evolve-your-harness.rst)描述了演化引擎，而[示例 README](examples/aime/README.md)记录了环境搭建、实现细节、与上游协议的差异，以及已完成的验证。