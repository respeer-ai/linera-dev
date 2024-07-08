# 合约最终化

当交易成功执行完成时，有一个最终步骤，所有加载的应用合约都会调用它们的 `Contract::store` 实现。这可以看作类似于执行析构函数。在这个意义上，应用可能希望在执行完成后执行一些最终操作。在最终化过程中，合约可以发送消息，读取和写入状态，但不能调用其他应用，因为它们都在最终化的过程中。

在最终化过程中，合约可以通过 panic 强制交易失败。即使整个交易操作在应用的 `Contract::store` 被调用之前已经成功完成，区块也会被拒绝。这使得合约能够在其他应用在响应跨应用调用后不遵循其建立的任何必要约束时，拒绝交易。

例如，一个执行跨应用调用 `Operation::StartSession` 的合约可能要求相同的调用者在交易结束之前执行另一个跨应用调用 `Operation::EndSession`。

```rust,ignore
pub struct MyContract {
    state: MyState;
    runtime: ContractRuntime<Self>;
    active_sessions: HashSet<ApplicationId>;
}

impl Contract for MyContract {
    type Message = ();
    type InstantiationArgument = ();
    type Parameters = ();

    async fn load(runtime: ContractRuntime<Self>) -> Self {
        let state = MyState::load(ViewStorageContext::from(runtime.key_value_store()))
            .await
            .expect("Failed to load state");

        MyContract {
            state,
            runtime,
            active_sessions: HashSet::new(),
        }
    }

    async fn instantiate(&mut self, (): Self::InstantiationArgument) {}

    async fn execute_operation(&mut self, operation: Self::Operation) -> Self::Response {
        let caller = self.runtime
            .authenticated_caller_id()
            .expect("Missing caller ID");

        match operation {
            Operation::StartSession => {
                assert!(
                    self.active_sessions.insert(caller_id),
                    "Can't start more than one session for the same caller"
                );
            }
            Operation::EndSession => {
                assert!(
                    self.active_sessions.remove(&caller_id),
                    "Session was not started"
                );
            }
        }
    }

    async fn execute_message(&mut self, message: Self::Message) -> Result<(), Self::Error> {
        unreachable!("This example doesn't support messages");
    }

    async fn store(&mut self) {
        assert!(
            self.active_sessions.is_empty(),
            "Some sessions have not ended"
        );

        self.state.save().await.expect("Failed to save state");
    }
}
```
