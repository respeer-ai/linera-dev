# 安装

让我们从安装Linera开发工具开始。

## 概述

Linera工具链由两个crate组成：

- `linera-sdk` 是用于在Rust中编程Linera应用程序的主要库。
- `linera-service` 定义了一些二进制文件，包括：
  - `linera` -- 主要的客户端工具，用于操作开发钱包，
  - `linera-proxy` -- 代理服务，作为每个验证者的公共入口点，
  - `linera-server` -- 每个验证者工作节点后面隐藏的服务。

## 要求

Linera工具链当前支持的操作系统总结如下：

| Linux x86 64-bit | Mac OS (M1 / M2) | Mac OS (x86) | Windows  |
| ---------------- | ---------------- | ------------ | -------- |
| ✓ 主要平台       | ✓ 可用           | ✓ 可用       | 未经测试 |

安装Linera工具链的主要前提条件包括Rust、Wasm和Protoc。在Linux上可以如下安装：

- Rust和Wasm

  - `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
  - `rustup target add wasm32-unknown-unknown`

- Protoc

  - `curl -LO https://github.com/protocolbuffers/protobuf/releases/download/v21.11/protoc-21.11-linux-x86_64.zip`
  - `unzip protoc-21.11-linux-x86_64.zip -d $HOME/.local`
  - If `~/.local` is not in your path, add it:
    `export PATH="$PATH:$HOME/.local/bin"`

- 在某些Linux发行版上，你可能需要安装开发包，如 `g++`、`libclang-dev` 和 `libssl-dev`。

关于MacOS支持以及测试Linera协议本身所需的其他要求，请参阅GitHub上的安装部分 [GitHub](https://github.com/linera-io/linera-protocol/blob/main/INSTALL.md)。

本手册使用了以下Rust工具链版本：

```text
{{#include ../../../linera-protocol/rust-toolchain.toml}}
```

## 从 crates.io 安装

你可以使用以下命令从 crates.io 安装Linera二进制文件：

```bash
cargo install --locked linera-service@{{#include ../../../RELEASE_VERSION}}
```

并且使用 `linera-sdk` 作为Linera Wasm应用程序的库：

```bash
cargo add linera-sdk@{{#include ../../../RELEASE_VERSION}}
```

版本号 `{{#include ../../../RELEASE_VERSION}}` 对应于当前Linera的Devnet，可能会频繁更改。

## 从 GitHub 安装

从 [GitHub](https://github.com/linera-io/linera-protocol) 下载源代码：

```bash
git clone https://github.com/linera-io/linera-protocol.git
cd linera-protocol
git checkout -t origin/{{#include ../../../RELEASE_BRANCH}}  # Current release branch
```

要从源代码本地安装Linera工具链，可以运行：

```bash
cargo install --locked --path linera-service
```

或者，用于开发和调试，你可以使用调试模式编译的二进制文件，例如使用 `export PATH="$PWD/target/debug:$PATH"`。

本手册使用了以下存储库的提交版本： [repository](https://github.com/linera-io/linera-protocol):

```text
{{#include ../../../.git/modules/linera-protocol/HEAD}}
```

## Bash 辅助工具（可选）

考虑将 `linera net helper` 的输出添加到你的 `~/.bash_profile` 中，以帮助进行 [automation](../core_concepts/wallets.md#automation-in-bash)。

## 获取帮助

如果安装失败，请联系团队（例如在 [Discord](https://discord.gg/linera) 上寻求帮助，或者在 [GitHub](https://github.com/linera-io/linera-protocol/issues/new) 上 [创建问题](https://github.com/linera-io/linera-protocol/issues/new)。