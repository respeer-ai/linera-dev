# 安装

让我们从安装Linera开发工具开始。

## [概述](zh_CN/developers/getting_started/installation.md#概述)

Linera工具链由以下两个crate构成（译者注：crate即为rust发布包）：

- `linera-sdk`：开发Linera应用的Rust基础库，其中包含Linera基础类型、函数定义等。

- `linera-service`：包含下述可执行文件：
  - `linera` -- Linera的基础客户端工具，用于操作钱包，
  - `linera-proxy` -- 代理服务，用作验证者的接入点，
  - `linera-server` -- 验证者的工作节点运行的服务，隐藏在代理服务后面。

## [运行环境](zh_CN/developers/getting_started/installation.md#运行环境)

当前Linera工具链支持在以下操作系统运行：

| Linux x86 64-bit | Mac OS (M1 / M2) | Mac OS (x86) | Windows  |
| ---------------- | ---------------- | ------------ | -------- |
| ✓ 主要平台  | ✓ 可以工作        | ✓ 可以工作    | 未测试 |

安装Linera工具链前应先安装Rust，Wasm和Protoc，在Linera上安装过程如下：

- Rust和Wasm
  - `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
  - `rustup target add wasm32-unknown-unknown`
- Protoc
  - `curl -LO https://github.com/protocolbuffers/protobuf/releases/download/v21.11/protoc-21.11-linux-x86_64.zip`
  - `unzip protoc-21.11-linux-x86_64.zip -d $HOME/.local`
  - 如果PATH环境变量不包含`~/.local`, 通过`export PATH="$PATH:$HOME/.local/bin"`添加
- 在特定的Linux发行版上，你可能需要安装诸如`g++`，`libclang-dev`和`libssl-dev`等开发工具和库

MacOS支持，以及测试Linera协议的一些附加需求可以参见[GitHub安装文档](https://github.com/linera-io/linera-protocol/blob/main/INSTALL.md)的相应章节。

本手册测试使用的Rust工具链配置如下：

```text
[toolchain]
channel = "1.77.2"
components = [ "clippy", "rustfmt", "rust-src" ]
targets = [ "wasm32-unknown-unknown" ]
profile = "minimal"
```

## [从crates.io安装](zh_CN/developers/getting_started/installation.md#从crates.io安装)

你可以通过如下命令安装Linera工具链

```bash
cargo install linera-sdk@0.11.3
cargo install linera-service@0.11.3
```

然后使用`linera-sdk`作为Linera Wasm应用的依赖库：

```bash
cargo add linera-sdk@0.11.3
```

版本号`0.11.3`对应当前Linera Devnet，该版本号可能会频繁变更。

## [从GitHub安装](zh_CN/developers/getting_started/installation.md#从GitHub安装)
从[GitHub](https://github.com/linera-io/linera-protocol)下载源码：

```bash
git clone https://github.com/linera-io/linera-protocol.git
cd linera-protocol
git checkout -t origin/devnet_2024_05_07  # 当前发布分支
```

如果希望从源码安装Linera工具链，执行如下命令：

```bash
cargo install --locked --path linera-service
```

开发者和调试人员可以使用带有调试信息的可执行文件调试Linera，例如，通过`export PATH="$PWD/target/debug:$PATH"`将调试版本可执行文件添加到PATH中。

本文档在[Linera代码仓库](https://github.com/linera-io/linera-protocol)的如下提交记录测试通过：

```text
2ada2e77e6a2f3dfa3bd32f4dc609bdadd0fbf3a
```

## [Bash助手(可选)](zh_CN/developers/getting_started/installation.md#Bash助手(可选))

可以通过在`~/.bash_profile`文件追加`linera net helper`快速[自动](zh_CN/developers/core_concepts/wallets.md#Bash环境自动设置)设置Linera运行时环境变量

## [寻求帮助](zh_CN/developers/getting_started/installation.md#寻求帮助)

如果安装失败，联系我们的团队(例如通过[Discord](https://discord.gg/linera))协助调试，或者在Github[创建一个issue](https://github.com/linera-io/linera-protocol/issues/new)。
