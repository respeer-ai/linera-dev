# 编写Linera应用程序

在这个部分，我们将探讨如何使用Linera SDK创建Web3应用程序。

我们将以一个简单的“计数器”应用程序作为运行示例。

我们将专注于应用程序的后端部分，包括两个主要部分：一个智能合约和其GraphQL服务。

应用程序的合约和服务都是使用Rust编写的，使用[`linera-sdk`](https://crates.io/crates/linera-sdk) crate，并编译为Wasm字节码。

这一部分应该被视为SDK的指南，而不是参考手册。参考手册请查阅[crate的文档](https://docs.rs/linera-sdk/latest/linera_sdk/)。