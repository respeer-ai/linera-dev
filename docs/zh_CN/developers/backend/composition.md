#  调用其他应用程序

我们已了解到：由某条链上的应用程序发送的跨链消息，始终由目标链上的`同一`应用程序处理。

本节将介绍如何使用跨应用调用（_cross-application calls_ ）来调用其他应用程序。

此类调用发生在同一条链上，并通过辅助方法 [`ContractRuntime::call_application`](https://docs.rs/linera-sdk/latest/linera_sdk/contract/type.ContractRuntime.html#call_application)实现：

```rust,ignore
    pub fn call_application<A: ContractAbi + Send>(
        &mut self,
        authenticated: bool,
        application: ApplicationId<A>,
        call: &A::Operation,
    ) -> A::Response
```

参数 `authenticated` 用于指定是否允许被调用方（callee）执行需要身份验证的操作，具体包括以下两种情况：

-  代表触发此次调用的原始区块签名者执行操作。
-  代表调用方应用程序执行操作。

参数 `application` 表示被调用方（callee）的应用ID，而 `A` 代表被调用方的应用二进制接口（ABI）。

参数 `call` 表示应用程序调用所请求的操作指令。

## 示例：众筹应用​​

该`众筹`示例应用支持应用创建者发起设定资金目标的募资活动。资金目标可采用基于`同质化代币`应用的任意类型代币作为计量单位。其他用户可通过质押该类代币参与募资，若活动截止时未达目标金额，则参与者的质押资产将自动退还。

若Alice基于`同质化代币`示例创建了 Pugecoin 应用（以印象深刻的哈士奇作为吉祥物），则Bob可构建`众筹应用`，将 Pugecoin 的应用 ID 用作 `CrowdFundingAbi::Parameters`，并在 `CrowdFundingAbi::InstantiationArgument` 中指定其活动周期为一周，目标金额为 1000 枚 Pugecoins。

假设卡罗尔需向Bob的众筹活动质押10枚 Pugecoin 代币，其可通过运行 `linera service`并向Bob的应用发送查询实现：

```json
mutation { pledge(owner: "User:841…6c0", amount: "10") }
```

此操作将在卡罗尔的链上新增包含质押记录的区块，该记录通过 `CrowdFunding::execute_operation` 合约方法处理，最终引发一次跨应用调用和两条跨链消息：

`CrowdFunding::execute_operation` 首先调用卡罗尔Carol链上的 `​​同质化代币`应用​​，将10枚代币转移至卡罗尔在Bob链上的账户：

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

此操作将触发运行 `Fungible::execute_operation` 合约方法，该方法会生成一条跨链消息，将金额10发送至Bob链上的 Pugecoin 应用实例。

在跨应用调用完成后，`CrowdFunding::execute_operation` 将继续生成另一条跨链消息 `crowd_funding::Message::PledgeWithAccount`，该消息会向Bob链上的众筹应用传递关键状态：10枚代币已锁定为活动专用资金。

当Bob在其链上添加处理两条传入消息的区块时，系统将依次执行`Fungible::execute_message`和`CrowdFunding::execute_message`。其中，CrowdFunding::execute_message 会触发第二次跨应用调用，将10枚代币从Carol的账户转移到Bob链上的众筹应用账户（两者均在同一链）。此操作成功原因在于Carol已在该链持有10枚代币,其通过区块签名间接授权了此次转账。最终，众筹应用在Bob链的应用状态中记录：​​Carol已为活动质押10枚Pugecoin代币​​。

# 参考文献​​

请参阅 `linera-protocol` 代码库中 `examples` 目录下的[`crowd-funding`](https://github.com/linera-io/linera-protocol/blob/3e867613343b937b4bcdb6994a0c7b459e9d497e/examples/crowd-funding/src/contract.rs)和[`fungible`](https://github.com/linera-io/linera-protocol/blob/3e867613343b937b4bcdb6994a0c7b459e9d497e/examples/fungible/src/contract.rs)以下合约实现。

[本文件](https://github.com/linera-io/linera-protocol/blob/3e867613343b937b4bcdb6994a0c7b459e9d497e/linera-sdk/src/contract/runtime.rs)定义了合约可用的 Runtime 实现。