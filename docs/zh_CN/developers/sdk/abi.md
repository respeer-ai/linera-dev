# 定义 ABI

Linera 应用程序的应用程序二进制接口（ABI）定义了如何与系统中的其他部分交互。它包括了链上合约和服务公开的数据结构、数据类型和函数。

ABIs通常在 `src/lib.rs` 中定义，并在所有架构（Wasm 和本地）上进行编译。

要查看参考指南，请查阅 crate 的 [文档](https://docs.rs/linera-base/latest/linera_base/abi/)。

## 定义标记结构体

应用程序的库部分（通常在 `src/lib.rs` 中）必须定义一个实现了 `Abi` trait 的公共空结构体。

```rust,ignore
struct CounterAbi;
```

`Abi` trait 结合了 `ContractAbi` 和 `ServiceAbi` trait，包括了你的应用程序导出的类型。

```rust,ignore
{{#include ../../../linera-protocol/linera-base/src/abi.rs:abi}}
```

接下来，我们将实现这两个 trait。

## 合约 ABI

`ContractAbi` trait 定义了合约中应用程序使用的数据类型。每种类型代表合约行为的特定部分：

```rust,ignore
{{#include ../../../linera-protocol/linera-base/src/abi.rs:contract_abi}}
```

所有这些类型都必须实现 `Serialize`、`DeserializeOwned`、`Send`、`Sync`、`Debug` trait，并具有 `'static` 生命周期。

在我们的示例中，我们想要将我们的 `Operation` 改变为 `u64`，如下所示：

```rust
# extern crate linera_base;
# use linera_base::abi::ContractAbi;
# struct CounterAbi;
impl ContractAbi for CounterAbi {
    type Operation = u64;
    type Response = ();
}
```

## 服务 ABI

`ServiceAbi` 与 `ContractAbi` 在原理上非常相似，只是针对应用程序的服务部分。

`ServiceAbi` trait 定义了应用程序服务部分使用的类型：

```rust,ignore
{{#include ../../../linera-protocol/linera-base/src/abi.rs:service_abi}}
```

对于我们的计数器示例，我们将使用 GraphQL 查询我们的应用程序，因此我们的 `ServiceAbi` 应该反映这一点：

```rust
# extern crate linera_base;
# extern crate async_graphql;
# use linera_base::abi::ServiceAbi;
# struct CounterAbi;
impl ServiceAbi for CounterAbi {
    type Query = async_graphql::Request;
    type QueryResponse = async_graphql::Response;
}
```
