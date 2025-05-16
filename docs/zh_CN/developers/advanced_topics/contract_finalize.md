# 合约执行完毕

当一笔交易成功执行完毕后，最终阶段会调用所有已加载应用合约的 `Contract::store` 实现。这类似于执行析构函数。从这个意义上说，应用可能希望在运行结束后执行某些收尾操作。在最终确定阶段，合约可以发送消息、读写状态，但​​不能调用其他应用​​——因为此时所有相关应用也正处于最终确定过程中。

在最终确认阶段，合约可以通过主动触发异常（panic）强制使交易失败。即使整个交易操作在应用程序调用`Contract::store`之前已成功执行，区块仍会被拒绝。这一机制使得合约能够在其响应跨应用调用后，若其他应用未遵循它设定的必要约束条件，便可拒绝相关交易。

例如，某个合约在执行带有`Operation::StartSession`的跨应用调用后，可以要求同一调用方在交易结束前，必须再执行一次带有`Operation::EndSession`的跨应用调用。

```rust,edition2021
# extern crate serde;
# extern crate linera_sdk;
# use serde::{Deserialize, Serialize};
# use linera_sdk::linera_base_types::*;
# use linera_sdk::*;
# use linera_sdk::abi::*;
# use std::collections::HashSet;
# use linera_sdk::views::{linera_views, RegisterView, RootView, ViewStorageContext};
# use crate::linera_sdk::views::View as _;
# use linera_sdk::linera_base_types::ApplicationId;

#[derive(RootView)]
#[view(context = "ViewStorageContext")]
pub struct MyState {
    pub value: RegisterView<u64>,
    // ...
}

#[derive(Serialize, Deserialize, Debug)]
pub enum Operation { StartSession, EndSession }

pub struct MyAbi;

impl ContractAbi for MyAbi {
    type Operation = Operation;
    type Response = ();
}

pub struct MyContract {
    state: MyState,
    runtime: ContractRuntime<Self>,
    active_sessions: HashSet<ApplicationId>,
}

impl WithContractAbi for MyContract {
    type Abi = MyAbi;
}

impl Contract for MyContract {
    type Message = ();
    type InstantiationArgument = ();
    type Parameters = ();
    type EventValue = ();

    async fn load(runtime: ContractRuntime<Self>) -> Self {
        let state = MyState::load(runtime.root_view_storage_context())
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
        let caller_id = self.runtime
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

    async fn execute_message(&mut self, message: Self::Message) {
        unreachable!("This example doesn't support messages");
    }

    async fn store(mut self) {
        assert!(
            self.active_sessions.is_empty(),
            "Some sessions have not ended"
        );

        self.state.save().await.expect("Failed to save state");
    }
}
```
