# 3.7. 跨链消息

在Linera上，应用天然是跨链的：当一个应用在一条新微链被使用，该应用将在新微链初始化。一个应用在不同微链上拥有同样的应用ID与字节码，但是每条微链的应用状态是独立的。不同微链的应用实例可以通过发送*跨链消息*互相协调。发送消息与接收消息的应用总是*同一个*：这意味着消息总是被同样的代码片段处理，但是其状态可能是不同的。

我们可以在应用的`ContractAbi`实现中指定任何可以序列化的类型作为`Message`类型。当我们需要发送一条消息，将其包含在[`ExecutionOutcome`](https://docs.rs/linera-sdk/latest/linera_sdk/struct.ExecutionOutcome.html)函数返回值`messages`中即可：

```rust
    pub messages: Vec<OutgoingMessage<Message>>,
```

`OutgoingMessage`的`destination`成员可以是一条微链，也可以是一个频道。如果是一个频道，消息将被发送给所有订阅者。

如果`authenticated`为`true`，被调用者将可以执行需要引发此条用的原始区块签名者认证的行为。

`message`成员包含消息本身，其类型为此前我们在`ContractAbi`中指定的消息类型。

方便起见，我们也可以使用[`ExecutionOutcome::with_message`](https://docs.rs/linera-sdk/latest/linera_sdk/struct.ExecutionOutcome.html#method.with_message)和[`with_authenticated_message`](https://docs.rs/linera-sdk/latest/linera_sdk/struct.ExecutionOutcome.html#method.with_authenticated_message)构造消息.

*发送*微链执行区块时，可以通过`ExecutionOutcome`返回消息，这些消息将被放在*目标*微链的收件箱，等待处理。这些消息不是一定被处理：当目标微链的所有者(之一)将消息放到区块的`incoming_messages`中，合约的`execute_message`将会在目标微链被调用，消息就被处理了。

## [示例: 同质化Token](https://linera-dev.respeer.ai/#/zh_CN/sdk/messages?id=example-fungible-token)

在[`fungible`示例应用](https://github.com/linera-io/linera-protocol/tree/main/examples/fungible)中，从一条微链向另一条微链转账就是一条消息。如果发送者在他的微链上包含一个`Transfer`操作，当操作被执行时，发送者的账户余额将减少，同时发送一个`Credit`消息到接收者的微链：

```rust
async fn execute_operation(
    &mut self,
    context: &OperationContext,
    operation: Self::Operation,
) -> Result<ExecutionOutcome<Self::Message>, Self::Error> {
    match operation {
        Operation::Transfer {
            owner,
            amount,
            target_account,
        } => {
            // ...
            self.debit(owner, amount).await?;
            let message = Message::Credit {
                owner: target_account.owner,
                amount,
            };
            Ok(ExecutionOutcome::default().with_message(target_account.chain_id, message))
        }
        // ...
    }
}
```

接收者微链这一侧，`execute_message`将会执行，增加接收者的余额。

```rust
async fn execute_message(
    &mut self,
    context: &MessageContext,
    message: Message,
) -> Result<ExecutionOutcome<Self::Message>, Self::Error> {
    match message {
        Message::Credit { owner, amount } => {
            self.credit(owner, amount).await;
            Ok(ExecutionOutcome::default())
        }
        // ...
    }
}
```
