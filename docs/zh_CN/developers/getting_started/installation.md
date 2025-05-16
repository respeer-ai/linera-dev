# 安装

让我们从Linera开发工具的安装开始。

## 概述

Linera工具链包含多个核心组件：

- `linera-sdk`是基于Rust的Linera应用开发核心库

- `linera-service` 定义了多个核心二进制组件，其中主工具链"linera"承担开发者钱包操作与本地测试网络启动功能

- `linera-storage-service` 提供轻量级数据库，支持本地验证节点的测试及开发运行

## 系统需求

当前Linera工具链支持在以下操作系统运行：

| Linux x86 64-bit | Mac OS (M1 / M2) | Mac OS (x86) | Windows  |
| ---------------- | ---------------- | ------------ | -------- |
| ✓ 主要平台  | ✓ 可以工作        | ✓ 可以工作    | 未测试 |

安装Linera工具链前应先安装Rust，Wasm和Protoc，在Linera上安装过程如下：

- Rust and Wasm

  - `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
  - `rustup target add wasm32-unknown-unknown`

- Protoc

  - `curl -LO https://github.com/protocolbuffers/protobuf/releases/download/v21.11/protoc-21.11-linux-x86_64.zip`
  - `unzip protoc-21.11-linux-x86_64.zip -d $HOME/.local`
  - 如果PATH环境变量不包含`~/.local`, 通过`export PATH="$PATH:$HOME/.local/bin"`添加

- 在部分Linux发行版中，可能需要安装`g++`、`libclang-dev`及`libssl-dev`等开发包。

MacOS支持及Linera协议自测所需的附加依赖项，请参阅[GitHub](https://github.com/linera-io/linera-protocol/blob/main/INSTALL.md)安装章节。

本手册通过以下Rust工具链完成验证：
```text
[toolchain]
channel = "1.85.0"
components = [ "clippy", "rustfmt", "rust-src" ]
targets = [ "wasm32-unknown-unknown" ]
profile = "minimal"
```

## 从crates.io安装

你可以通过如下命令安装Linera工具链

```bash
cargo install --locked linera-storage-service@0.14.0
cargo install --locked linera-service@0.14.0
```

然后使用linera-sdk作为Linera Wasm应用的依赖库：

```bash
cargo add linera-sdk@0.14.0
```

版本号`0.14.0`对应当前Linera Devnet，该版本号可能会频繁变更。

## 从GitHub安装

从[GitHub](https://github.com/linera-io/linera-protocol)下载源码：

```bash
git clone https://github.com/linera-io/linera-protocol.git
cd linera-protocol
git checkout -t origin/0.14.0  # Current release branch
```

如果希望从源码安装Linera工具链，执行如下命令：

```bash
cargo install --locked --path linera-storage-service
cargo install --locked --path linera-service
```

开发者和调试人员可以使用带有调试信息的可执行文件调试Linera，例如，通过`export PATH="$PWD/target/debug:$PATH"`将调试版本可执行文件添加到PATH中。

本文档在[Linera代码仓库](https://github.com/linera-io/linera-protocol)的如下提交记录测试通过：

```text
3e867613343b937b4bcdb6994a0c7b459e9d497e
```

## 寻求帮助

如果安装过程中遇到障碍，可以联系我们的团队(例如通过[Discord](https://discord.gg/linera))协助调试，或者在Github[创建一个issue](https://github.com/linera-io/linera-protocol/issues/new)。