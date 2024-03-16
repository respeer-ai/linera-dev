# 3.3. 定义ABI

Linera应用程序的应用程序二进制接口(ABI)定义了系统其他部分与应用的交互方式，其中包含数据结构，数据类型，以及链上合约和服务提供的函数。

ABIs通常定义在`src/lib.rs`文件中，可以编译为所有体系结构目标文件(Wasm和原生)。

ABI的参考文档可以查看[crate文档](https://docs.rs/linera-base/latest/linera_base/abi/)。

## [结构声明](https://linera-dev.respeer.ai/#/zh_CN/sdk/abi?id=defining-a-marker-struct)

应用中提供给外部调用的接口(通常定义在`src/lib.rs`文件中)必须定义一个公开的空结构，其中实现了`Abi` trait。

```rust
struct CounterAbi;
```

`Abi` trait聚合了`ContractAbi`和`ServiceAbi` traits，其中包含应用导出的所有数据类型和接口。

```rust
/// 包含Linera应用导出的所有数据类型和接口的trait(包含合约和服务)
pub trait Abi: ContractAbi + ServiceAbi {}
```

接下来，我们将会实现上面提到的两个traits。

## [合约ABI](https://linera-dev.respeer.ai/#/zh_CN/sdk/abi?id=contract-abi)

`ContractAbi` trait定义了应用程序的合约中使用的数据类型，每种类型都表示合约的一部分特定行为：

```rust
/// A trait that includes all the types exported by a Linera application contract.
pub trait ContractAbi {
    /// Immutable parameters specific to this application (e.g. the name of a token).
    type Parameters: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// Initialization argument passed to a new application on the chain that created it
    /// (e.g. an initial amount of tokens minted).
    ///
    /// To share configuration data on every chain, use [`ContractAbi::Parameters`]
    /// instead.
    type InitializationArgument: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The type of operation executed by the application.
    ///
    /// Operations are transactions directly added to a block by the creator (and signer)
    /// of the block. Users typically use operations to start interacting with an
    /// application on their own chain.
    type Operation: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The type of message executed by the application.
    ///
    /// Messages are executed when a message created by the same application is received
    /// from another chain and accepted in a block.
    type Message: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The argument type when this application is called from another application on the same chain.
    type ApplicationCall: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The argument type when a session of this application is called from another
    /// application on the same chain.
    ///
    /// Sessions are temporary objects that may be spawned by an application call. Once
    /// created, they must be consumed before the current transaction ends.
    type SessionCall: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The type for the state of a session.
    type SessionState: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The response type of an application call.
    type Response: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;
}
```

所有类型都必须实现`Serialize`, `DeserializeOwned`, `Send`, `Sync`, `Debug` traits，并且具有`'static`生命周期。

在我们的例子中，我们将`InitializationArgument`, `Operation`成员修改为`u64`类型，如下：

```rust
impl ContractAbi for CounterAbi {
    type InitializationArgument = u64;
    type Parameters = ();
    type Operation = u64;
    type ApplicationCall = ();
    type Message = ();
    type SessionCall = ();
    type Response = ();
    type SessionState = ();
}
```

## [服务ABI](https://linera-dev.respeer.ai/#/zh_CN/sdk/abi?id=service-abi)

概念上，`ServiceAbi`与`ContractAbi`雷同，所不同者，`ServiceAbi`提供的是应用程序的服务部分数据类型和接口。

```rust
/// A trait that includes all the types exported by a Linera application service.
pub trait ServiceAbi {
    /// Immutable parameters specific to this application (e.g. the name of a token).
    type Parameters: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The type of a query receivable by the application's service.
    type Query: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;

    /// The response type of the application's service.
    type QueryResponse: Serialize + DeserializeOwned + Send + Sync + Debug + 'static;
}
```

在计数器示例中，我们将会使用GraphQL请求应用程序，因此`ServiceAbi`需要定义如下：

```rust
impl ServiceAbi for CounterAbi {
    type Query = async_graphql::Request;
    type QueryResponse = async_graphql::Response;
    type Parameters = ();
}
```
