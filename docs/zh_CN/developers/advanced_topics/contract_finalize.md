# 合约执行完毕

事务执行的最后一步调用`Contract::store`函数，相当于合约实例析构，某些应用可能希望在执行结束前执行一些特定操作。应用可以在`Contract::store`函数中发送消息和读写状态，然而由于其他依赖应用此时也处于正在结束状态，我们不应该在这一步骤调用其他应用。

结束过程中，合约可以通过panic强制事务失败，哪怕调用`Contract::store`前的所有事务操作都已经成功，panic发生后区块也会被拒绝。这样的设计使得合约可以在检测到跨应用调用不能满足需要的约束时拒绝事务。

例如，执行`Operation::StartSession`可能要求相同的调用者在事务结束前执行`Operation::EndSession`。

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
