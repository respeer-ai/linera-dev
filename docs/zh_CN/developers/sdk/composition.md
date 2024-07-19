# 调用其他应用

前面的章节中我们已经了解跨链消息总是在一条微链的应用实例发送，然后在另一条微链上的*相同*应用的不同实例处理。

本节中我们将会讲解使用*跨应用调用*实现应用之间通信。

跨应用调用发生在同一条微链的不同应用实例之间，通过调用 [`ContractRuntime::call_application`](https://docs.rs/linera-sdk/latest/linera_sdk/struct.ContractRuntime.html#call_application) 方法实现：

```rust,ignore
pub fn call_application<A: ContractAbi + Send>(
    &mut self,
    authenticated: bool,
    application: ApplicationId<A>,
    call: &A::Operation,
) -> A::Response {
```

`authenticated`参数设置被调用者是否允许执行那些需要触发此次调用的原始区块签名者认证的行为。

`application`参数为被调用者的应用ID，`A`为被调用者的ABI声明。

`call`参数是被调用者定义的应用调用参数。

## [示例: 筹款](https://linera-dev.respeer.ai/#/zh_CN/sdk/composition?id=example-crowd-funding)

`crowd-funding`应用允许应用创建者启动一个筹款活动，筹款目标可以是在`fungible`应用中指定的任意Token金额。其他人可以向活动质押需要的Token，如果到期未能达到目标，质押将退回。

假定Alice使用`fungible`示例创建了一个Pugecoin应用(以一只令人印象深刻的哈巴狗作为吉祥物)，然后Bob可以创建一个`crowd-funding`应用，并使用Pugecoin的应用ID作为`CrowdFundingAbi::Parameters`，并在`CrowdFundingAbi::InstantiationArgument`中指定本次活动时长为一周，筹款目标为1000 Pugecoins。

现在，Carol想质押10 Pugecoins到Bob的活动。

首先，Carol需要确定她的微链上有`crowd-funding`应用，例如，通过`linera request-application`命令。当Carol执行上述命令后，作为Bob的应用依赖，Alice的应用也将在Carol的微链上自动注册。

然后，Carol可以通过运行`linera service`，并发送如下请求到Bob的应用质押她的Pugecoin了：

```
mutation { pledge(owner: "User:841…6c0", amount: "10") }
```

上面的mutation请求将会在Carol的微链创建一个新区块，该区块包含质押操作，由`CrowdFunding::execute_operation`处理。执行质押操作将会产生一次跨应用调用，和两条跨链消息：

首先，`CrowdFunding::execute_operation`调用Carol的微链上的`fungible`应用，从Carol的账户中转移10 Tokens到Bob的微链(上的Carol的账户)：

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

该调用将会在`Fungible::execute_operation`执行，创建一条跨链消息，以发送10 Pugecoin到Bob的微链上的Pugecoin应用实例。

跨应用调用返回后，`CrowdFunding::execute_operation`继续创建另一条`crowd_funding::Message::PledgeWithAccount`消息，通知Bob的微链上的筹款应用Carol已经向活动质押了10 Tokens。

Bob将在他的微链创建新区块处理两条收到的消息，首先处理`Fungible::execute_message`，然后处理`CrowdFunding::execute_message`。`CrowdFunding::execute_message`将会触发一次跨应用调用，从Carol的账户中转10 Tokens到筹款应用的活动账户(两个账户都在Bob的微链上)。由于Carol在她的微链上有10 Tokens，并且她通过签名认证区块的方式间接授权了向活动账户转入10 Tokens质押，因此上述过程能够成功。Bob的微链上的筹款应用现在可以在其应用状态中记录Carol质押了10 Pugecoins。

完整代码请查看 `linera-protocol` 仓库中 `examples` 文件夹下的 [`crowd-funding`](https://github.com/linera-io/linera-protocol/examples/crowd-funding/src/contract.rs) 和 [`fungible`](https://github.com/linera-io/linera-protocol/examples/fungible/src/contract.rs) 。
