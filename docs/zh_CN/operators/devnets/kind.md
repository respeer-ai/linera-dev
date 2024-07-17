# 使用`kind`运行开发网络

本章将介绍使用`kind`怎么运行一个包含多个验证者的本地开发网络。

Kind(Kubernetes in Docker)是一个在Docker容器节点中运行本地Kubernetes集群的工具。Kind使用在Docker上创建的容器集群来模拟Kubernetes控制平面和工作节点，这样开发者就可以在本地主机方便地上创建、管理和测试多节点集群。

## 安装

本节涵盖了使用`kind`运行Linera网络的所有安装步骤。

### Linera工具链要求

当前Linera工具链对于各操作系统支持状况如下：

| Linux x86 64-bit | Mac OS (M1 / M2) | Mac OS (x86) | Windows |
| ---------------- | ---------------- | ------------ | ------- |
| ✓ 主要平台       | ✓ 可以工作       | ✓ 可以工作   | 未测试  |

安装Linera工具链之前需要现安装Rust，Wasm和Protoc，在Linux可以按照如下步骤安装：

- Rust和Wasm

  - `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
  - `rustup target add wasm32-unknown-unknown`

- Protoc

  - `curl -LO https://github.com/protocolbuffers/protobuf/releases/download/v21.11/protoc-21.11-linux-x86_64.zip`
  - `unzip protoc-21.11-linux-x86_64.zip -d $HOME/.local`
  - 需要将路径`~/.local`添加到环境变量：`export PATH="$PATH:$HOME/.local/bin"`

- 在某些Linux发行版上，可能需要安装`g++`、`libclang-dev` 和 `libssl-dev`等开发包。

MacOS系统参见[GitHub上的安装部分](https://github.com/linera-io/linera-protocol/blob/main/INSTALL.md)。

本手册基于下列工具链测试：

```text
[toolchain]
channel = "1.77.2"
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

安装Linera工具链需要先下载源码：

```bash
git clone https://github.com/linera-io/linera-protocol.git
cd linera-protocol
git checkout -t origin/devnet_2024_05_07  # Current release branch
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

从源码构建镜像需要一些时间。部署完毕后，终端将会输出包含开发网络钱包设置的环境变量：

```bash
export LINERA_WALLET="/tmp/.tmpIOelqk/wallet_0.json"
export LINERA_STORAGE="rocksdb:/tmp/.tmpIOelqk/client_0.db"
```

这些变量用于在新的终端中与上面部署的开发网络交互：

```bash
$ linera sync-balance
2024-05-21T22:30:12.061199Z  INFO linera: Synchronizing chain information and querying the local balance
2024-05-21T22:30:12.061218Z  WARN linera: This command is deprecated. Use `linera sync && linera query-balance` instead.
2024-05-21T22:30:12.065787Z  INFO linera::client_context: Saved user chain states
2024-05-21T22:30:12.065792Z  INFO linera: Operation confirmed after 4 ms
1000000.
```
