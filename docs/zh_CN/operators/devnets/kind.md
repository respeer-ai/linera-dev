# 使用 `kind` 运行开发网络

在本节中，我们将使用 `kind` 在本地运行一个完整的开发网络（包含多个验证者）。

Kind（Kubernetes in Docker）是一个工具，用于使用 Docker 容器节点运行本地 Kubernetes 集群。Kind 使用 Docker 创建一组容器，模拟 Kubernetes 控制平面和工作节点，使开发人员能够在本地轻松创建、管理和测试多节点集群。

## 安装

本节涵盖了使用 `kind` 运行 Linera 网络所需的所有安装步骤。

### Linera 工具链要求

目前 Linera 工具链支持的操作系统总结如下：

| Linux x86 64-bit | Mac OS (M1 / M2) | Mac OS (x86) | Windows |
| ---------------- | ---------------- | ------------ | ------- |
| ✓ 主要平台       | ✓ 工作中         | ✓ 工作中     | 未测试  |

安装 Linera 工具链的主要先决条件包括 Rust、Wasm 和 Protoc。在 Linux 上，可以如下安装：

- Rust 和 Wasm

  - `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
  - `rustup target add wasm32-unknown-unknown`

- Protoc

  - `curl -LO https://github.com/protocolbuffers/protobuf/releases/download/v21.11/protoc-21.11-linux-x86_64.zip`
  - `unzip protoc-21.11-linux-x86_64.zip -d $HOME/.local`
  - 如果 `~/.local` 不在您的路径中，请添加：
    `export PATH="$PATH:$HOME/.local/bin"`

- 在某些 Linux 发行版上，您可能需要安装开发包，如 `g++`、`libclang-dev` 和 `libssl-dev`。

对于 MacOS 的支持，请参阅 [GitHub 上的安装部分](https://github.com/linera-io/linera-protocol/blob/main/INSTALL.md)。

本手册使用以下 Rust 工具链进行了测试：

```text
{{#include ../../../linera-protocol/rust-toolchain.toml}}
```

### 本地 Kubernetes 要求

要在本地运行 `kind`，您还需要以下依赖项：

1. [`kind`](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
2. [`kubectl`](https://kubernetes.io/docs/tasks/tools/)
3. [`docker`](https://docs.docker.com/get-docker/)
4. [`helm`](https://helm.sh/docs/intro/install/)
5. [`helm-diff`](https://github.com/databus23/helm-diff)
6. [`helmfile`](https://github.com/helmfile/helmfile?tab=readme-ov-file#installation)

### 安装 Linera 工具链

要安装 Linera 工具链，请从 [GitHub](https://github.com/linera-io/linera-protocol) 下载 Linera 源代码：

```bash
git clone https://github.com/linera-io/linera-protocol.git
cd linera-protocol
git checkout -t origin/{{#include ../../../RELEASE_BRANCH}}  # Current release branch
```

然后安装 Linera 工具链：

```bash
cargo install --locked --path linera-service --features kubernetes
```

## 使用 `kind` 运行

要使用 `kind` 在本地运行开发网络，请导航到 `linera-protocol` 存储库的根目录，并执行以下命令：

```bash
linera net up --kubernetes
```

这将花费一些时间，因为需要从 Linera 源代码构建 Docker 镜像。当集群准备就绪时，进程输出中会显示一些文本，其中包含配置开发网络钱包所需的导出内容，例如：

```bash
export LINERA_WALLET="/tmp/.tmpIOelqk/wallet_0.json"
export LINERA_STORAGE="rocksdb:/tmp/.tmpIOelqk/client_0.db"
```

在新的终端中导出这些变量将使您能够与开发网络进行交互：

```bash
$ linera sync-balance
2024-05-21T22:30:12.061199Z  INFO linera: Synchronizing chain information and querying the local balance
2024-05-21T22:30:12.061218Z  WARN linera: This command is deprecated. Use `linera sync && linera query-balance` instead.
2024-05-21T22:30:12.065787Z  INFO linera::client_context: Saved user chain states
2024-05-21T22:30:12.065792Z  INFO linera: Operation confirmed after 4 ms
1000000.
```
