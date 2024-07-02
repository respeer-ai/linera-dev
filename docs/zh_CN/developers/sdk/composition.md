# 调用其他应用程序

我们已经看到，在一个链上由应用程序发送的跨链消息始终由目标链上的 *同一个* 应用程序处理。

本节介绍如何使用 *跨应用程序调用* 调用其他应用程序。

这种调用发生在同一链上，并使用 [`ContractRuntime::call_application`](https://docs.rs/linera-sdk/latest/linera_sdk/struct.ContractRuntime.html#call_application) 方法进行：

```rust,ignore
pub fn call_application<A: ContractAbi + Send>(
    &mut self,
    authenticated: bool,
    application: ApplicationId<A>,
    call: &A::Operation,
) -> A::Response {
```

`authenticated` 参数指定了被调用方是否允许代表触发此调用的原始区块的签名者执行需要认证的操作。

`application` 参数是被调用方的应用程序 ID，`A` 是被调用方的 ABI。

`call` 参数是应用程序调用请求的操作。

## 示例：众筹

`crowd-funding` 示例应用程序允许应用程序创建者启动一个众筹活动，并设定一个筹款目标。该目标可以是任何类型的代币，基于 `fungible` 应用程序。其他人可以向该活动捐赠该类型的代币，如果在截止日期前未达到目标，则会退款。

例如，如果 Alice 使用 `fungible` 示例创建了一个 Pugecoin 应用程序（其吉祥物是一只可爱的哈巴狗），那么 Bob 可以创建一个 `crowd-funding` 应用程序，使用 Pugecoin 的应用程序 ID 作为 `CrowdFundingAbi::Parameters`，并在 `CrowdFundingAbi::InstantiationArgument` 中指定他的活动将持续一周，并且目标是 1000 个 Pugecoin。

现在假设 Carol 想向 Bob 的活动捐赠 10 个 Pugecoin 代币。

首先，她需要确保在她的链上有他的众筹应用程序，例如使用 `linera request-application` 命令。这将自动在她的链上注册 Alice 的 Pugecoin 应用程序，因为它是 Bob 的一个依赖项。

现在，她可以通过运行 `linera service` 并对 Bob 的应用程序进行查询来进行捐赠：

```json
mutation { pledge(owner: "User:841…6c0", amount: "10") }
```

这将在 Carol 的链上添加一个包含捐款操作的区块，该操作由 `CrowdFunding::execute_operation` 处理，结果是一个跨应用程序调用和两个跨链消息：

首先，`CrowdFunding::execute_operation` 调用 Carol 的链上的 `fungible` 应用程序，将 10 个代币转移到 Bob 链上的 Carol 账户：

```rust,ignore
// ...
let call = fungible::Operation::Transfer {
    owner,
    amount,
    target_account,
};
// ...
self.runtime
    .call_application(/* authenticated by owner */ true, fungible_id, &call);
```

这导致 `Fungible::execute_operation` 被执行，它将创建一个跨链消息，将数量 10 发送到 Bob 链上的 Pugecoin 应用程序实例。

在跨应用程序调用返回后，`CrowdFunding::execute_operation` 继续创建另一个跨链消息 `crowd_funding::Message::PledgeWithAccount`，通知 Bob 链上的众筹应用程序，这 10 个代币是为活动而捐赠的。

当 Bob 现在添加一个处理这两个传入消息的区块到他的链上时，首先执行 `Fungible::execute_message`，然后执行 `CrowdFunding::execute_message`。后者进行另一个跨应用程序调用，将 10 个代币从 Carol 的账户转移到众筹应用程序的账户（都在 Bob 的链上）。这是成功的，因为 Carol 现在在这条链上有了 10 个代币，并且她通过签名她的区块间接地认证了转账。众筹应用程序现在在 Bob 的链上的应用程序状态中记录了 Carol 捐赠了 10 个 Pugecoin 代币。

有关完整的代码，请查看 `linera-protocol` 仓库中 `examples` 文件夹下的 [`crowd-funding`](https://github.com/linera-io/linera-protocol/blob/{{#include ../../../.git/modules/linera-protocol/HEAD}}/examples/crowd-funding/src/contract.rs) 和 [`fungible`](https://github.com/linera-io/linera-protocol/blob/{{#include ../../../.git/modules/linera-protocol/HEAD}}/examples/fungible/src/contract.rs) 应用程序合约。