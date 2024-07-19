# 跨链消息

在Linera上，应用天然支持多链：它们在使用的每条链上都被实例化。一个应用在不同微链上拥有同样的应用ID与字节码，但是每条微链的应用状态是独立的。不同微链的应用实例可以通过发送*跨链消息*互相协调。发送消息与接收消息的应用总是*同一个*：这意味着消息总是被同样的代码片段处理，但是其状态可能是不同的。

我们可以在应用程序的合约实现中指定任何可以序列化的类型作为`Message`类型。正如[编写合约](./contract.md)章节中描述，我们在合约类型中声明了`runtime`字段。构造合约实例时，`Contract::load`函数携带的[`ContractRuntime`](https://docs.rs/linera-sdk/latest/linera_sdk/struct.ContractRuntime.html)参数被存储在合约实例的`runtime`字段。合约事务执行过程中，我们可以调用[`ContractRuntime::prepare_message`](https://docs.rs/linera-sdk/latest/linera_sdk/struct.ContractRuntime.html#prepare_message)准备消息，然后使用[`send_to`](https://docs.rs/linera-sdk/latest/linera_sdk/struct.MessageBuilder.html#send_to)将消息发送到目标链。

```rust,ignore
    self.runtime
        .prepare_message(message_contents)
        .send_to(destination_chain_id);
```

除了可以被发送到具体的目标微链，消息还可以被发送到订阅频道，从而转发到频道的所有订阅者。此时，只需要将`send_to`的目标设置为[`ChannelName`](https://docs.rs/linera-base/latest/linera_base/identifiers/struct.ChannelName.html)即可。

*发送*微链执行区块时，已发送消息将被放在*目标*微链的收件箱，等待处理。这些消息不是一定被处理：当目标微链的所有者(之一)将消息放到区块的`incoming_messages`中，合约的`execute_message`将会在目标微链被调用，消息就被处理了。

微链发送的消息中可以启用身份验证转发和/或消息追踪。身份验证转发意味着接收这执行该消息时，该消息的签名信息与最初发送该消息的用户签名一致。如果启用消息追踪，那些被接收者拒绝的消息将被发回发送者。下面的例子同时启用了身份认证转发和消息追踪：

```rust,ignore
    self.runtime
        .prepare_message(message_contents)
        .with_tracking()
        .with_authentication()
        .send_to(destination_chain_id);
```

## [示例: 同质化Token](https://linera-dev.respeer.ai/#/zh_CN/sdk/messages?id=example-fungible-token)

在[`fungible`示例应用](https://github.com/linera-io/linera-protocol/tree/main/examples/fungible)中，从一条微链向另一条微链转账就是一条消息。如果发送者在他的微链上包含一个`Transfer`操作，当操作被执行时，发送者的账户余额将减少，同时发送一个`Credit`消息到接收者的微链：

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

接收者微链这一侧，`execute_message`将会执行，增加接收者的余额。

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
