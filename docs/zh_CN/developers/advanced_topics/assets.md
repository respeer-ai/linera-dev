# 处理资产的应用程序

通常情况下，如果您向他人拥有的链发送代币，您依赖于他们确保资产的可用性：如果他们不处理您的消息，您将无法访问您的代币。

幸运的是，Linera 提供了基于临时链的解决方案：如果事先知道和限定参与方的数量，我们可以：

- 使用 `linera change-ownership` 命令使它们成为所有者，
- 只允许链上一个应用程序的操作，
- 并允许仅该操作关闭链，使用 `linera change-application-permissions`。

这样的应用程序应该有一个指定的操作或消息，触发它关闭链：当执行该操作时，它应该将所有剩余的资产退回，并调用运行时的 `close_chain` 方法。

一旦链关闭，所有者仍然可以创建拒绝消息的区块。这样，即使资产正在传输中，也可以将其返回。

[`matching-engine` 示例应用程序](https://github.com/linera-io/linera-protocol/tree/main/examples/matching-engine) 就是这样做的：

```rust,ignore
    async fn execute_operation(&mut self, operation: Operation) -> Self::Response {
        match operation {
            // ...
            Operation::CloseChain => {
                for order_id in self.state.orders.indices().await? {
                    match self.modify_order(order_id, ModifyAmount::All).await {
                        Ok(transfer) => self.send_to(transfer),
                        // Orders with amount zero may have been cleared in an earlier iteration.
                        Err(MatchingEngineError::OrderNotPresent) => continue,
                        Err(error) => return Err(error),
                    }
                }
                self.runtime
                    .close_chain()
                    .expect("The application does not have permissions to close the chain.");
            }
        }
    }
```

这使得使用匹配引擎进行原子交换成为可能：如果您提交一个竞价，您可以确保在任何时间点都可以收回您提供的代币或您购买的代币。