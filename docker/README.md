# Docker

Reef 镜像打包了完整的运行时（Slime、SGLang、Ray、Megatron），
因此部署可以完全在一个镜像内以普通进程的方式运行。这里
没有 docker-compose 层：`reef serve -c <config>` 会读取 YAML 中的
`services` 列表，并直接启动每个声明的进程，PID/日志文件位于
`/tmp/reef-stack` 下。

## 构建

```bash
docker build -f docker/Dockerfile.reef -t reef .
```

该镜像继承了 GPU 训练栈，并从 `pyproject.toml` 安装 Reef 精确锁定的运行时
版本，同时安装 SGLang、Ray 以及训练依赖。运行时存在于镜像中
并不意味着其训练驱动正在运行——只有你配置中列出的 `services` 才会运行。

默认镜像固定了带有 Reef 适配器接收器的 SGLang 修订版本，
因此 LoRA 训练（`--megatron-lora-rank`）可以在任何权重训练配方上运行，
无需单独的镜像。可通过
`--build-arg SGLANG_COMMIT=<sha>` 覆盖修订版本。

TTT-Discover 的 Qwen3-8B LoRA 实验使用可选的 `tttd` 目标。它
仅为生成程序添加 Erdős 评估器的求解器依赖，
并搭配一个经过验证的 Slime 摘要：

```bash
docker build -f docker/Dockerfile.reef --target tttd \
  --build-arg SLIME_IMAGE_TAG='latest@sha256:a97ec147e37bef050337a9b229036eda00b4aa9c4d02b31a0109dc850f8ca342' \
  -t reef-tttd:qwen3-8b .
```

## 演示配置

| 配置 | 启动的服务 | GPU | 是否更改权重 |
|---|---|---:|---:|
| `recipes/basic/local-sglang.yaml` | 本地 SGLang + Reef | 是 | 否 |
| `recipes/basic/external-provider.yaml` | Reef 代理到 HTTP 提供方 | 否 | 否 |
| `recipes/<method>/examples/<example>/serve.yaml` | Ray + Slime 桥接 + Reef 训练 | 2+ | 是 |

每个配置都以声明式方式声明其服务（`name`、`command`、
`ready` 探针、`depends_on`）。命令使用 `${a.b.c}` 插值来引用
配置中的其他部分，因此添加或更改服务只需编辑 YAML。
编排器会在内部使用相同的配置启动 Reef 的 HTTP 子进程；
所有其他服务都是普通的 shell 命令。有关训练
流程和数据契约，请参阅
[Evolve your model](https://reefinfra.ai/docs/user-guide/evolve-your-model/)。

## 运行

在镜像内（或任何已安装依赖的主机上）：

```bash
export REEF_TOKEN=$(openssl rand -hex 16)
# edit the stack yaml: set model paths / provider creds
reef serve -c recipes/basic/local-sglang.yaml
```

日志和 PID 位于 `/tmp/reef-stack/`。若要仅针对已在运行的提供方
运行 reef HTTP 服务，请使用 `services` 列表
仅包含 Reef 的配置：

```bash
reef serve -c recipes/basic/external-provider.yaml
```

## 持久化与清理

Agent 记录和导出的检查点持久化在 `reef.state_dir`
下（默认为 `/var/lib/reef`）。停止进程（`kill` `/tmp/reef-stack/`
中的 PID）但保留状态；若要同时删除已记录的 agent 记录，
请移除该目录。