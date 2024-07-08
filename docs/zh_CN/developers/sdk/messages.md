# 跨链消息

在 Linera 上，应用程序被设计为多链应用：它们在每个使用它们的链上都会被实例化。一个应用程序在任何地方都有相同的应用程序 ID 和字节码，但在每个链上有独立的状态。为了协调，实例可以向彼此发送 *跨链消息*。由应用程序发送的消息始终由目标链上的 *相同* 应用程序处理：处理代码保证与发送代码相同，但状态可能不同。

对于您的应用程序，可以在 `Contract` 实现中将任何可序列化类型指定为 `Message` 类型。要发送消息，可以使用作为合约的 [`Contract::load`] 构造函数参数提供的 [`ContractRuntime`](https://docs.rs/linera-sdk/latest/linera_sdk/struct.ContractRuntime.html)。运行时通常存储在合约对象内部，就像我们在[编写合约二进制](./contract.md)时所做的那样。然后，我们可以调用 [`ContractRuntime::prepare_message`](https://docs.rs/linera-sdk/latest/linera_sdk/struct.ContractRuntime.html#prepare_message) 来开始准备消息，然后使用 [`send_to`](https://docs.rs/linera-sdk/latest/linera_sdk/struct.MessageBuilder.html#send_to) 将其发送到目标链。

```rust,ignore
    self.runtime
        .prepare_message(message_contents)
        .send_to(destination_chain_id);
```

还可以将消息发送到订阅频道，以便将消息转发给该频道的订阅者。只需将 [`ChannelName`](https://docs.rs/linera-base/latest/linera_base/identifiers/struct.ChannelName.html) 指定为 `send_to` 的目标参数即可。

在 *发送* 链上的块执行后，发送的消息将放置在 *目标* 链的收件箱中以供处理。不能保证它会被处理：为了使其被处理，目标链的所有者需要在他们的块中将其包含在 `incoming_messages` 中。当这种情况发生时，目标链上的合约的 `execute_message` 方法将被调用。

在准备要发送的消息时，可以启用身份验证转发和/或跟踪。身份验证转发意味着接收方使用与消息发送方相同的经过身份验证的签名者执行消息，而跟踪意味着如果接收方拒绝消息，则将消息发送回发送方。下面的示例启用了这两个标志：

```rust,ignore
    self.runtime
        .prepare_message(message_contents)
        .with_tracking()
        .with_authentication()
        .send_to(destination_chain_id);
```

## 示例：可替代令牌

在 [`fungible` 示例应用程序](https://github.com/linera-io/linera-protocol/tree/{{#include ../../../.git/modules/linera-protocol/HEAD}}/examples/fungible)中，这样的消息可以是从一条链向另一条链转移令牌。如果发送方在其链上包含一个 `Transfer` 操作，它会减少其账户余额并向接收方的链发送一个 `Credit` 消息：

```rust,ignore
async fn execute_operation(&mut self, operation: Self::Operation) -> Self::Response {
    match operation {
        // ...
        Operation::Transfer {
            owner,
            amount,
            target_account,
        } => {
            self.check_account_authentication(owner)?;
            self.state.debit(owner, amount).await?;
            self.finish_transfer_to_account(amount, target_account, owner)
                .await;
            FungibleResponse::Ok
        }
        // ...
    }
}

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

在接收方的链上，将调用 `execute_message` 方法，以增加其账户余额。

```rust,ignore
async fn execute_message(&mut self, message: Message) {
    match message {
        Message::Credit {
            amount,
            target,
            source,
        } => {
            // ...
            self.state.credit(receiver, amount).await;
        }
        // ...
    }
}
```
