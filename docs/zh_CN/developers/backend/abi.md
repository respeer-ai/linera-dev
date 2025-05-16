# 定义ABI

应用的​​应用程序二进制接口（ABI）​​定义了如何从系统其他组件与该应用交互。其包含以下内容：
数据结构​​：应用状态与交互数据的存储模型，
数据类型​​：自定义类型与系统原生类型的映射规则，
函数接口​​：由链上合约与服务暴露的操作方法。

ABI 通常定义于 `src/lib.rs` 文件中，并针对所有目标架构（Wasm 虚拟机与原生平台）进行交叉编译。

如需查看完整的 ABI 规范，请参阅[官方文档](https://docs.rs/linera-base/latest/linera_base/abi/)。

## 标记结构体定义​​

应用程序的库组件（通常位于 `src/lib.rs` 文件）必须定义一个公共的空结构体，并实现 `Abi` 特性。

```rust
struct CounterAbi;
```

`Abi` 特性整合了 `ContractAbi`（合约ABI）与 `ServiceAbi`（服务ABI）特性，以包含应用程序导出的数据类型。

```rust,ignore
/// A trait that includes all the types exported by a Linera application (both contract
/// and service).
pub trait Abi: ContractAbi + ServiceAbi {}
```

接下来，我们将分别实现这两个特性。

## 合约ABI

`ContractAbi` 特性定义了应用程序在合约中使用的​​数据类型​​，每个类型对应合约行为的特定功能模块：

```rust,ignore
/// A trait that includes all the types exported by a Linera application contract.
pub trait ContractAbi {
    /// The type of operation executed by the application.
    ///
    /// Operations are transactions directly added to a block by the creator (and signer)
    /// of the block. Users typically use operations to start interacting with an
    /// application on their own chain.
    type Operation: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The response type of an application call.
    type Response: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// How the `Operation` is deserialized
    fn deserialize_operation(operation: Vec<u8>) -> Result<Self::Operation, String> {
        bcs::from_bytes(&operation)
            .map_err(|e| format!("BCS deserialization error {e:?} for operation {operation:?}"))
    }

    /// How the `Operation` is serialized
    fn serialize_operation(operation: &Self::Operation) -> Result<Vec<u8>, String> {
        bcs::to_bytes(operation)
            .map_err(|e| format!("BCS serialization error {e:?} for operation {operation:?}"))
    }

    /// How the `Response` is deserialized
    fn deserialize_response(response: Vec<u8>) -> Result<Self::Response, String> {
        bcs::from_bytes(&response)
            .map_err(|e| format!("BCS deserialization error {e:?} for response {response:?}"))
    }

    /// How the `Response` is serialized
    fn serialize_response(response: Self::Response) -> Result<Vec<u8>, String> {
        bcs::to_bytes(&response)
            .map_err(|e| format!("BCS serialization error {e:?} for response {response:?}"))
    }
}
```

所有这些类型必须实现 `Serialize`（序列化）、`DeserializeOwned`（反序列化拥有所有权）、`Send`（可跨线程发送）、`Sync`（可跨线程同步）、`Debug`（调试格式化）特性，且需具备 `static` 生命周期（即数据在整个程序运行期间有效）。

以我们的示例为例，需将 `Operation` 类型调整为 `u64`，具体实现如下：

```rust,ignore
pub struct CounterAbi;

impl ContractAbi for CounterAbi {
    type Operation = u64;
    type Response = u64;
}
```

## 服务ABI

`ServiceAbi` 在本质上与 `ContractAbi` 高度相似，​​仅适用于应用程序的服务组件​​。

`ServiceAbi` 特性定义了服务部分使用的​​数据类型​​：

```rust,ignore
/// A trait that includes all the types exported by a Linera application service.
pub trait ServiceAbi {
    /// The type of a query receivable by the application's service.
    type Query: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The response type of the application's service.
    type QueryResponse: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;
}
```

在 `Counter` 示例中，我们将通过 GraphQL 实现应用查询功能，因此 `ServiceAbi` 需体现该数据交互设计：

```rust,ignore
use async_graphql::{Request, Response};

impl ServiceAbi for CounterAbi {
    type Query = Request;
    type QueryResponse = Response;
}
```

## 参考资料​​

- Abi 特性的完整定义​​可在[此处](https://github.com/linera-io/linera-protocol/blob/3e867613343b937b4bcdb6994a0c7b459e9d497e/linera-base/src/abi.rs)找到。


- 完整的 Counter 示例应用​​可在[此处](https://github.com/linera-io/linera-protocol/tree/3e867613343b937b4bcdb6994a0c7b459e9d497e/examples/counter)找到。
