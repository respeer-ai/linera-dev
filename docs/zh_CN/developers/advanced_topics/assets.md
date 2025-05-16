# 处理资产的应用程序

一般来说，若你将代币发送至他人拥有的链，你需要依赖对方确保资产可用性：若对方不处理你的消息，你将无法访问自己的代币。

幸运的是，Linera提供了一种基于临时链的解决方案：若参与方数量有限且可提前确定，我们可以：

- 通过使用`linera change-ownership`命令将他们全部设为链的所有者，
- 仅允许该链上的单一应用程序执行操作，
- 并通过`linera change-application-permissions`命令限制仅该操作能关闭链。

此类应用程序应具备一个指定的操作或消息，用于触发链的关闭。当执行该操作时，应用程序应返还所有剩余资产，并调用运行时（runtime）的`close_chain`方法。

一旦链被关闭，所有者仍可创建区块以拒绝消息。通过这种方式，即使资产仍在传输过程中，也能被成功退回。

[`matching-engine` example application](https://github.com/linera-io/linera-protocol/tree/main/examples/matching-engine)正是这样做的：

```rust,ignore
    async fn execute_operation(&mut self, operation: Operation) -> Self::Response {
        match operation {
            // ...
            Operation::CloseChain => {
                for order_id in self.state.orders.indices().await.unwrap() {
                    match self.modify_order(order_id, ModifyAmount::All).await {
                        Some(transfer) => self.send_to(transfer),
                        // Orders with amount zero may have been cleared in an earlier iteration.
                        None => continue,
                    }
                }
                self.runtime
                    .close_chain()
                    .expect("The application does not have permissions to close the chain.");
            }
        }
    }
```


这实现了通过匹配引擎进行原子交换：如果你出价，你可以保证在任何时候拿回自己提供的代币，或者成功购买到的代币。