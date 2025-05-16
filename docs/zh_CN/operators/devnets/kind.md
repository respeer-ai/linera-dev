# 使用`kind`运行devnets

在本节中，我们使用 `Kind`在本地运行一个完整的开发者网络（即由验证者节点组成的网络）。

Kind（Kubernetes in Docker）是一种通过Docker容器节点运行本地Kubernetes集群的工具。它利用Docker创建一组容器，模拟Kubernetes的控制平面（control plane）和工作节点（worker nodes），使开发者能够轻松在本地机器上创建、管理和测试多节点集群。

## 安装

本节介绍使用 `Kind` 运行 Linera 网络所需的所有安装内容。

### Linera 工具链安装要求

Linera 工具链当前支持的操作系统可总结如下：

| Linux x86 64-bit | Mac OS (M1 / M2) | Mac OS (x86) | Windows  |
| ---------------- | ---------------- | ------------ | -------- |
| ✓ 主要平台  | ✓ 可以工作        | ✓ 可以工作    | 未测试 |


安装Linera工具链的主要先决条件包括Rust、WebAssembly（Wasm）和Protocol Buffers编译器（protoc）。在Linux系统上的安装方法如下：

- Rust and Wasm

  - `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
  - `rustup target add wasm32-unknown-unknown`

- Protoc

  - `curl -LO https://github.com/protocolbuffers/protobuf/releases/download/v21.11/protoc-21.11-linux-x86_64.zip`
  - `unzip protoc-21.11-linux-x86_64.zip -d $HOME/.local`
  - If `~/.local` is not in your path, add it:
    `export PATH="$PATH:$HOME/.local/bin"`

- 在某些Linux发行版上，可能需要安装`g++`、`libclang-dev` 和 `libssl-dev`等开发包。

MacOS系统参见[GitHub](https://github.com/linera-io/linera-protocol/blob/main/INSTALL.md)上的安装部分。

本手册基于下列工具链测试：

```text
[toolchain]
channel = "1.85.0"
components = [ "clippy", "rustfmt", "rust-src" ]
targets = [ "wasm32-unknown-unknown" ]
profile = "minimal"
```

### 本地Kubernetes要求

本地运行`kind`依赖下面的工具：

1. [`kind`](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
2. [`kubectl`](https://kubernetes.io/docs/tasks/tools/)
3. [`docker`](https://docs.docker.com/get-docker/)
4. [`helm`](https://helm.sh/docs/intro/install/)
5. [`helm-diff`](https://github.com/databus23/helm-diff)
6. [`helmfile`](https://github.com/helmfile/helmfile?tab=readme-ov-file#installation)

### 安装Linera工具链

安装Linera工具链需要先从[GitHub](https://github.com/linera-io/linera-protocol)下载源码：

```bash
git clone https://github.com/linera-io/linera-protocol.git
cd linera-protocol
git checkout -t origin/testnet_babbage  # Current release branch
```

然后编译安装：

```bash
cargo install --locked --path linera-service --features kubernetes
```

## 使用`kind`运行

使用`kind`运行本地开发网络需要进入`linera-protocol`仓库根目录，并执行：

```bash
linera net up --kubernetes
```

此过程可能需要一些时间，因为需要从Linera源代码构建Docker镜像。当集群就绪后，进程输出中会显示一些配置信息（包含用于配置开发者网络钱包所需的环境变量导出命令），例如：

```bash
export LINERA_WALLET="/tmp/.tmpIOelqk/wallet_0.json"
export LINERA_STORAGE="rocksdb:/tmp/.tmpIOelqk/client_0.db"
```

在新终端中导出这些变量即可与开发者网络进行交互：

```bash
$ linera sync-balance
2024-05-21T22:30:12.061199Z  INFO linera: Synchronizing chain information and querying the local balance
2024-05-21T22:30:12.061218Z  WARN linera: This command is deprecated. Use `linera sync && linera query-balance` instead.
2024-05-21T22:30:12.065787Z  INFO linera::client_context: Saved user chain states
2024-05-21T22:30:12.065792Z  INFO linera: Operation confirmed after 4 ms
1000000.
```
