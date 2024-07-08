# 编写合约二进制代码

合约二进制代码是 Linera 应用程序的第一个组件，它实际上可以改变应用程序的状态。

要创建一个合约，我们需要创建一个新类型，并为其实现 `Contract` trait，其定义如下：

```rust,ignore
pub trait Contract: WithContractAbi + ContractAbi + Sized {
    /// The type of message executed by the application.
    type Message: Serialize + DeserializeOwned + Debug;

    /// Immutable parameters specific to this application (e.g. the name of a token).
    type Parameters: Serialize + DeserializeOwned + Clone + Debug;

    /// Instantiation argument passed to a new application on the chain that created it
    /// (e.g. an initial amount of tokens minted).
    type InstantiationArgument: Serialize + DeserializeOwned + Debug;

    /// Creates a in-memory instance of the contract handler.
    async fn load(runtime: ContractRuntime<Self>) -> Self;

    /// Instantiates the application on the chain that created it.
    async fn instantiate(&mut self, argument: Self::InstantiationArgument);

    /// Applies an operation from the current block.
    async fn execute_operation(&mut self, operation: Self::Operation) -> Self::Response;

    /// Applies a message originating from a cross-chain message.
    async fn execute_message(&mut self, message: Self::Message);

    /// Finishes the execution of the current transaction.
    async fn store(self);
}
```

完整的 trait 定义可以在这里找到： [链接到 GitHub](https://github.com/linera-io/linera-protocol/blob/{{#include ../../../.git/modules/linera-protocol/HEAD}}/linera-sdk/src/lib.rs)。

这里涉及的内容比较多，让我们逐步分解并逐个方法来看。

对于这个应用程序，我们将使用 `load`、`execute_operation` 和 `store` 方法。

## 合约生命周期

为了实现应用程序合约，首先创建合约类型：

```rust,ignore
pub struct CounterContract {
    state: Counter,
    runtime: ContractRuntime<Self>,
}
```

这个类型通常至少包含两个字段：之前定义的持久化 `state` 和运行时的句柄。运行时提供了当前执行的信息访问，并允许发送消息等。可以添加其他字段，用于存储仅在当前事务执行期间存在并在之后丢弃的易失性数据。

当执行事务时，通过调用 `Contract::load` 方法创建合约类型。此方法接收一个合约运行时的句柄，合约可以使用它来加载应用程序状态。对于我们的实现，我们将加载状态并创建 `CounterContract` 实例：

```rust,ignore
    async fn load(runtime: ContractRuntime<Self>) -> Self {
        let state = Counter::load(ViewStorageContext::from(runtime.key_value_store()))
            .await
            .expect("Failed to load state");
        CounterContract { state, runtime }
    }
```

当事务成功执行结束时，有一个最后步骤，所有加载的应用程序合约都会被调用来做任何最终检查，并将其状态持久化到存储中。这最后一步是调用 `Contract::store` 方法，可以类比于执行析构函数。在我们的实现中，我们将状态持久化回存储中：

```rust,ignore
    async fn store(mut self) {
        self.state.save().await.expect("Failed to save state");
    }
```

可以做的事情不仅仅是保存状态，[合约最终化章节](../advanced_topics/contract_finalize.md)提供了更多细节。

## 实例化我们的应用程序

当应用程序从字节码创建时，首先发生的事情是实例化它。这通过调用合约的 `Contract::instantiate` 方法完成。

`Contract::instantiate` 只在应用程序创建时调用一次，而且仅在创建该应用程序的微链上调用。

部署在其他微链上将使用状态使用视图范例中所有子视图的 `Default` 值。

对于我们的示例应用程序，我们希望将应用程序的状态初始化为可以在创建应用程序时使用其实例化参数指定的任意值：

```rust,ignore
    async fn instantiate(&mut self, value: u64) {
        self.state.value.set(value);
    }
```

## 实现增量操作

现在我们有了计数器的状态以及一种初始化它的方式，我们需要一种方式来增加计数器的值。来自区块提议者或其他应用程序的执行请求通常称为 '操作'。

要处理操作，我们需要实现 `Contract::execute_operation` 方法。对于计数器来说，它将接收到一个 `u64` 值，用于增加计数器的值：

```rust,ignore
    async fn execute_operation(&mut self, operation: u64) {
        let current = self.value.get();
        self.value.set(current + operation);
    }
```

## 声明 ABI

最后，为了将我们的 `Contract` trait 实现与应用程序的 ABI 关联起来，添加以下代码：

```rust,ignore
impl WithContractAbi for CounterContract {
    type Abi = counter::CounterAbi;
}
```
