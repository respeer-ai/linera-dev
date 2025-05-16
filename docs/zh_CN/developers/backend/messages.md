# ​跨链消息

在 Linera 网络中，应用程序具备​​多链原生特性​​：
    ​​全局唯一性​​,同一应用在所有链上共享相同的 Application ID 与字节码;
    ​​状态隔离性​​,各链维护独立的应用状态（即每条链保存独立的状态副本）;
    ​​跨链协作机制​​,链间实例可通过​_cross-chain messages_​​进行状态同步，其核心特征包括：
        ​​确定性处理​​：目标链接收的消息必定由本链同一应用实例处理,
        ​​代码一致性​​：跨链处理逻辑与发送端完全一致（通过字节码强校验保证）,
        ​​状态差异性​​：各链状态因独立执行可能产生差异;



在您的应用开发中，您可以在实现`Contract`合约时指定任何可序列化类型作为`Message`类型。当需要发送消息时，可通过[`Contract::load`](https://docs.rs/linera-sdk/latest/linera_sdk/trait.Contract.html#tymethod.load)构造函数中提供的[`ContractRuntime`](https://docs.rs/linera-sdk/latest/linera_sdk/contract/type.ContractRuntime.html)运行时（该运行时通常存储在合约对象内部，正如我们在[`编写合约二进制文件`](../../../zh_CN/developers/backend/contract.md)时所做的那样）进行操作。具体步骤是：先调用[`ContractRuntime::prepare_message`](https://docs.rs/linera-sdk/latest/linera_sdk/contract/type.ContractRuntime.html#prepare_message)方法准备消息，再通过`send_to`将其发送至目标链。

```rust,ignore
    self.runtime
        .prepare_message(message_contents)
        .send_to(destination_chain_id);
```


也可以将消息发送至订阅频道（subscription channel），这样消息就会被转发给该频道的所有订阅者。只需将 [`ChannelName`](https://docs.rs/linera-base/latest/linera_base/identifiers/struct.ChannelName.html) 作为目标参数传递给 `send_to` 即可实现这一功能。

在发送链（ _sending_ chain）完成区块执行后，已发送的消息会被放入 _目标_ 链的收件箱（inbox）等待处理。但系统并不保证消息一定会被处理：只有当目标链的持有者（owner）在其某个区块的 `incoming_messages` 中包含该消息时，目标链才会触发对该消息的处理——此时，目标链上的合约 `execute_message` 方法将被调用。

在准备待发送的消息时，可选择启用身份验证转发（authentication forwarding）和/或消息追踪（tracking）功能。
    身份验证转发,接收方执行消息时，将使用与消息发送方相同的已认证签名者身份；
    消息追踪，若接收方拒绝处理该消息，系统会自动将其退回至发送方。
以下示例同时启用了这两个功能标志：

```rust,ignore
    self.runtime
        .prepare_message(message_contents)
        .with_tracking()
        .with_authentication()
        .send_to(destination_chain_id);
```

## 示例：同质化代币

在[`同质化`代币示例应用](https://github.com/linera-io/linera-protocol/tree/3e867613343b937b4bcdb6994a0c7b459e9d497e/examples/fungible)中，此类消息可以是将代币从一条链转移到另一条链的操作。当发送方在其链上执行 `Transfer`（转账）操作时，系统将减少其账户余额，并向接收方链发送 `Credit`（入账）消息：

```rust,ignore
async fn execute_operation(&mut self, operation: Self::Operation) -> Self::Response {
        match operation {
            Operation::Transfer {
                owner,
                amount,
                target_account,
            } => {
                self.runtime
                    .check_account_permission(owner)
                    .expect("Permission for Transfer operation");
                self.state.debit(owner, amount).await;
                self.finish_transfer_to_account(amount, target_account, owner)
                    .await;
                FungibleResponse::Ok
            }
            // ...
        }
}
```

```rust,ignore
    /// Executes the final step of a transfer where the tokens are sent to the destination.
    async fn finish_transfer_to_account(
        &mut self,
        amount: Amount,
        target_account: Account,
        source: AccountOwner,
    ) {
        if target_account.chain_id == self.runtime.chain_id() {
            self.state.credit(target_account.owner, amount).await;
        } else {
            let message = Message::Credit {
                target: target_account.owner,
                amount,
                source,
            };
            self.runtime
                .prepare_message(message)
                .with_authentication()
                .with_tracking()
                .send_to(target_account.chain_id);
        }
    }
```


在接收方链上，系统会调用 `execute_message` 方法，从而增加接收方的账户余额。

```rust,ignore
    async fn execute_message(&mut self, message: Message) {
        match message {
            Message::Credit {
                amount,
                target,
                source,
            } => {
                let is_bouncing = self
                    .runtime
                    .message_is_bouncing()
                    .expect("Delivery status is available when executing a message");
                let receiver = if is_bouncing { source } else { target };
                self.state.credit(receiver, amount).await;
            }
            // ...
        }
    }
```
