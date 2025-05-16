# ​Linera应用开发指南​​

本节将探讨如何使用 Linera SDK 构建 Web3 应用程序。

我们以一个简单的「计数器」应用作为贯穿示例。

重点解析其后台架构。该应用后端包含两大核心组件：​_​智能合约_​​与​​GraphQL 服务​​。

应用的后台组件（合约与服务）均使用 Rust 语言结合 crate [`linera-sdk`](https://crates.io/crates/linera-sdk)  进行开发，并编译为 Wasm 字节码。

本节定位为 SDK 的​​使用指南​​而非​​参考手册​​。
如需查阅详细 API 规范，请参考  [crate 的官方文档](https://docs.rs/linera-sdk/latest/linera_sdk/)。