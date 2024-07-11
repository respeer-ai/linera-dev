# 3.4. 编写合约代码

合约是Linera应用程序的第一个组件，合约的执行将实际上改变应用状态。

我们需要实现如下的`Contract` trait来实现一个合约：

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

完整的trait定义可以参见[这里](https://github.com/linera-io/linera-protocol/blob/main/linera-sdk/src/lib.rs)。

这里涉及到很多内容，所以让我们逐个进行分解和讨论。

在示例应用中，我们将会使用`load`, `execute_operation`和`store`方法。

## [合约生命周期](https://linera-dev.respeer.ai/#/zh_CN/sdk/contract?id=the-contract-lifecycle)

实现合约的第一步是创建合约类型：

```rust,ignore
pub struct CounterContract {
    state: Counter,
    runtime: ContractRuntime<Self>,
}
```

合约通常至少包含两个字段：前述章节已经定义的持久化状态`state`和`runtime`句柄。`runtime`句柄可以访问当前执行状态，合约也可以通过`runtime`句柄发送消息，或执行一些其他需要与微链执行状态交互的操作。合约类型中也可以添加其他字段，但这些数据是非持久化的，只存在于当前事务执行期间，事务执行完毕后非持久化字段的数据将被丢弃。

事务开始执行时，`Contract::load`将创建合约实例。`Contract::load`携带`runtime`句柄，作为参数，该句柄用于加载应用状态。我们使用如下的实现加载应用状态并创建`CounterContract`实例：

```rust,ignore
    async fn load(runtime: ContractRuntime<Self>) -> Self {
        let state = Counter::load(ViewStorageContext::from(runtime.key_value_store()))
            .await
            .expect("Failed to load state");
        CounterContract { state, runtime }
    }
```

事务执行成功后，合约需要对事务执行杰作最终检查确定，然后将执行状态持久化存储。这一步骤将通过`Contract::store`实现，大体上我们可以认为这是执行事务的析构函数。下面的实现将事务执行的最终状态持久化存储：

```rust,ignore
    async fn store(mut self) {
        self.state.save().await.expect("Failed to save state");
    }
```

事实上最后一步可以做的事情不仅仅是保存状态，更多细节参见[合约最终化](../advanced_topics/contract_finalize.md)章节。

## [初始化应用](https://linera-dev.respeer.ai/#/zh_CN/sdk/contract?id=initializing-our-application)

首先，我们需要使用`Contract::instantiate`初始化应用。

`Contract::instantiate` 只在应用创建时调用一次，而且仅在创建该应用的微链上调用。

当应用在其他微链部署，应用状态将使用所有子视图的默认值进行初始化。

在我们的示例中，我们希望应用可以通过初始化参数指定任何数值作为计数器的初值：

```rust,ignore
    async fn instantiate(&mut self, value: u64) {
        self.state.value.set(value);
    }
```

## [实现增量操作](https://linera-dev.respeer.ai/#/zh_CN/sdk/contract?id=implementing-the-increment-operation)

现在，我们实现了一个计数器状态，可以通过传递任意数值初始化，我们需要实现一个方法来增加计数器的值。区块创建者修改应用状态的行为统称为'操作'。

我们需要使用方法`Contract::execute_operation`来创建一个新的操作。在计数器示例中，该方法将会收到一个`u64`类型的数值，用来作为计数值的增量。

```rust,ignore
    async fn execute_operation(&mut self, operation: u64) {
        let current = self.value.get();
        self.value.set(current + operation);
    }
```

## [声明ABI](https://linera-dev.respeer.ai/#/zh_CN/sdk/contract?id=declaring-the-abi)

最后，我们通过如下的代码将上面实现的`Contract` trait与应用程序的ABI相关联：

```rust,ignore
impl WithContractAbi for Counter {
    type Abi = counter::CounterAbi;
}
```
