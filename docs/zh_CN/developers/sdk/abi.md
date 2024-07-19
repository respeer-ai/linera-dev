# 定义ABI

Linera应用程序的应用程序二进制接口(ABI)定义了系统其他部分与应用的交互方式，其中包含数据结构，数据类型，以及链上合约和服务提供的函数。

ABIs通常定义在`src/lib.rs`文件中，可以编译为所有体系结构目标文件(Wasm和原生)。

ABI的参考文档可以查看[crate文档](https://docs.rs/linera-base/latest/linera_base/abi/)。

## [结构声明](zh_CN/developers/sdk/abi.md#结构声明)

应用中提供给外部调用的接口(通常定义在`src/lib.rs`文件中)必须定义一个实现了`Abi` trait的公开空结构。

```rust
struct CounterAbi;
```

`Abi` trait聚合了`ContractAbi`和`ServiceAbi` traits，其中包含应用导出的所有数据类型和接口。

```rust
/// 包含Linera应用导出的所有数据类型和接口的trait(包含合约和服务)
pub trait Abi: ContractAbi + ServiceAbi {}
```

接下来，我们将会实现上面提到的两个traits。

## [合约ABI](zh_CN/developers/sdk/abi.md#合约ABI)

`ContractAbi` trait定义了应用程序的合约中使用的数据类型，每种类型都表示合约的一部分特定行为：

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
}
```

所有类型都必须实现`Serialize`, `DeserializeOwned`, `Send`, `Sync`, `Debug` traits，并且具有`'static`生命周期。

在我们的例子中，我们将`Operation`成员修改为`u64`类型，如下：

```rust
extern crate linera_base;
use linera_base::abi::ContractAbi;
struct CounterAbi;
impl ContractAbi for CounterAbi {
    type Operation = u64;
    type Response = ();
}
```

## [服务ABI](zh_CN/developers/sdk/abi.md#服务ABI)
概念上，`ServiceAbi`与`ContractAbi`雷同，所不同者，`ServiceAbi`提供的是应用程序的服务部分数据类型和接口。

```rust,ignore
/// A trait that includes all the types exported by a Linera application service.
pub trait ServiceAbi {
    /// The type of a query receivable by the application's service.
    type Query: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The response type of the application's service.
    type QueryResponse: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;
}
```

在计数器示例中，我们将会使用GraphQL请求应用程序，因此`ServiceAbi`需要定义如下：

```rust
extern crate linera_base;
extern crate async_graphql;
use linera_base::abi::ServiceAbi;
struct CounterAbi;
impl ServiceAbi for CounterAbi {
    type Query = async_graphql::Request;
    type QueryResponse = async_graphql::Response;
}
```
