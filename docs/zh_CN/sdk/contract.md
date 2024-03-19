# 3.4. 编写合约代码

合约是Linera应用程序的第一个组件，合约的执行将实际上改变应用状态。

我们需要实现如下的`Contract` trait来实现一个合约：

```rust
#[async_trait]
pub trait Contract: WithContractAbi + ContractAbi + Send + Sized {
    /// The type used to report errors to the execution environment.
    type Error: Error + From<serde_json::Error> + From<bcs::Error> + 'static;

    /// The desired storage backend used to store the application's state.
    type Storage: ContractStateStorage<Self> + Send + 'static;

    /// Initializes the application on the chain that created it.
    async fn initialize(
        &mut self,
        context: &OperationContext,
        argument: Self::InitializationArgument,
    ) -> Result<ExecutionOutcome<Self::Message>, Self::Error>;

    /// Applies an operation from the current block.
    async fn execute_operation(
        &mut self,
        context: &OperationContext,
        operation: Self::Operation,
    ) -> Result<ExecutionOutcome<Self::Message>, Self::Error>;

    /// Applies a message originating from a cross-chain message.
    async fn execute_message(
        &mut self,
        context: &MessageContext,
        message: Self::Message,
    ) -> Result<ExecutionOutcome<Self::Message>, Self::Error>;

    /// Handles a call from another application.
    async fn handle_application_call(
        &mut self,
        context: &CalleeContext,
        argument: Self::ApplicationCall,
        forwarded_sessions: Vec<SessionId>,
    ) -> Result<ApplicationCallResult<Self::Message, Self::Response, Self::SessionState>, Self::Error>;

    /// Handles a call into a session created by this application.
    async fn handle_session_call(
        &mut self,
        context: &CalleeContext,
        session: Self::SessionState,
        argument: Self::SessionCall,
        forwarded_sessions: Vec<SessionId>,
    ) -> Result<SessionCallResult<Self::Message, Self::Response, Self::SessionState>, Self::Error>;

}
```

完整的trait定义可以参见[这里](https://github.com/linera-io/linera-protocol/blob/main/linera-sdk/src/lib.rs)。

这里涉及到很多内容，所以让我们逐个进行分解和讨论。

在示例应用中，我们将会使用`initialize`和`execute_operation`方法。

## [初始化应用](https://linera-dev.respeer.ai/#/zh_CN/sdk/contract?id=initializing-our-application)

首先，我们需要使用`Contract::initialize`初始化应用。

`Contract::initialize`仅在应用被创建时调用一次，并且仅发生在创建应用的微链上。

当应用在其他微链部署，根据应用状态的不同约束，应用将使用不同的初始化参数。如果应用状态被`SimpleStateStorage`修饰，应用状态将初始化为默认值；如果应用状态被`ViewStateStorage`修饰，应用状态将使用所有子视图的默认值进行初始化。

在我们的示例中，我们希望应用可以通过初始化参数指定任何数值作为计数器的初值：

```rust
    async fn initialize(
        &mut self,
        _context: &OperationContext,
        value: u64,
    ) -> Result<ExecutionOutcome<Self::Message>, Self::Error> {
        self.value.set(value);
        Ok(ExecutionOutcome::default())
    }
```

## [实现增量操作](https://linera-dev.respeer.ai/#/zh_CN/sdk/contract?id=implementing-the-increment-operation)

现在，我们实现了一个计数器状态，可以通过传递任意数值初始化，我们需要实现一个方法来增加计数器的值。区块创建者修改应用状态的行为统称为'操作'。

我们需要使用方法`Contract::execute_operation`来创建一个新的操作。在计数器示例中，该方法将会收到一个`u64`类型的数值，用来作为计数值的增量。

```rust
    async fn execute_operation(
        &mut self,
        _context: &OperationContext,
        operation: u64,
    ) -> Result<ExecutionOutcome<Self::Message>, Self::Error> {
        let current = self.value.get();
        self.value.set(current + operation);
        Ok(ExecutionOutcome::default())
    }
```

## [声明ABI](https://linera-dev.respeer.ai/#/zh_CN/sdk/contract?id=declaring-the-abi)

最后，我们通过如下的代码将上面实现的`Contract` trait与应用程序的ABI相关联：

```rust
impl WithContractAbi for Counter {
    type Abi = counter::CounterAbi;
}
```
