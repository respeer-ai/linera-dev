# 编写Linera应用

在本章中，我们将会使用`counter`应用示例，展示使用Linera SDK创建Web3应用的具体步骤。

本章内容主要关注后端应用开发，其中包含两个主要部分：其一为*智能合约*，其二为该合约对应的GraphQL服务。

Linera应用的智能合约机器对应的服务都是用Rust开发，以来[`linera-sdk`](https://crates.io/crates/linera-sdk)crate，然后编译成Wasm字节码。

开发者应该将本章看作Linera SDK的编程指导，而非参考手册。参考手册参见[Linera SDK crate文档](https://docs.rs/linera-sdk/latest/linera_sdk/)。
