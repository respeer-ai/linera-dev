# 应用程序中的资产

通常，当你向其他人拥有的微链发送一些token，这些token的可用性将取决于接收者的行为：如果接收者不处理消息，则这些token是不可用的。

Linera基于临时微链实现了一种解决方案：如果参与各方是有限的，并且事先可知，那么我们可以：

- 通过`linera change-ownership`将参与各方设置为链所有者，
- 设置该微链上只允许一个应用的操作，
- 通过`linera change-application-permissions`设置可以关闭微链的操作。

上述微链上被允许的应用应该有一个指定的操作或消息可以关闭微链：当该操作执行时，所有剩余资产将被退回，然后调用运行时的`close_chain`方法。

微链关闭时，所有者仍然可以创建区块拒绝消息，这样，即使资产已经发送，只要没有被接收方确认，就可以安全返回到发送者账户。

[`matching-engine` 示例程序](https://github.com/linera-io/linera-protocol/tree/main/examples/matching-engine)实现了上述流程：

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

通过上述设计，我们可以在匹配引擎中进行原子交换：当一个竞价被提交后，发送者将确保在任何时间点要么收到购买的token，要么收到自己的token。
