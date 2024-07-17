# 应用程序

Linera的编程模型设计旨在让开发者借助微链扩展他们的应用。

Linera使用WebAssembly(Wasm)虚拟机执行应用程序。当前，[Linera SDK](https://linera-dev.respeer.ai/#/zh_CN/sdk)主要针对[Rust](https://www.rust-lang.org/)编程语言。

Linera应用基于Rust开发者熟悉的**Rust crate**组织：应用程序依赖的外部接口(包括初始化参数、操作、消息和跨链调用)通常都有函数库的crate提供，应用程序核心部分将被编译为Wasm体系结构的二进制文件。

## [应用开发周期](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/applications?id=the-application-deployment-lifecycle)

Linera应用是可重用的，因此，应用的字节码和网络上正在运行的应用实例是不同的。

为了应用程序开发更加灵活，应用程序将会经历以下生命周期：

1. 将基于`linera-sdk`开发的Rust项目编译成字节码，
2. 将字节码发布到网络上的某一微链，并为其创建标识符，
3. 用户提供字节码标识符和初始化参数，创建一个新的应用程序实例，并获得应用标识。用户可以通过应用标识与应用交互。
4. 不同用户可以使用相同的字节码标识符创建不同的应用程序实例。

应用程序的部署生命周期与用户是分离的，开发者可以通过执行下面的命令发布一个应用：

```bash
linera publish-and-create <contract-path> <service-path> <init-args>
```

上面的命令除了发布字节码，也会在微链上创建一个应用实例。

## [应用程序剖析](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/applications?id=anatomy-of-an-application)

一个**应用程序**可以分为两个主要组件，*合约*和*服务*。

**合约**是gas计量的，应用程序在合约中执行操作、处理跨链消息、发起跨链调用和修改应用状态。关于合约在[SDK文档](https://linera-dev.respeer.ai/#/zh_CN/sdk)中包含更加详细的说明。

**服务**是只读的，服务的执行不消耗gas。服务主要用于查询应用状态，通过用户接口将查询结果返回到表示层(通常为前端)。

最后，合约和服务通过一种称为[视图](https://linera-dev.respeer.ai/#/zh_CN/advanced_topics/views)的形式共享应用状态，稍后我们将会详细介绍这一部分。

## [操作和消息](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/applications?id=operations-and-messages)

> 本节我们将使用示例应用"fungible"来展示完整的应用程序部署流程，用户可以通过该应用互相转账。

从系统层面来说，用户与应用的交互可以通过操作和消息完成。

**操作**是应用开发者定义的，每个应用可以拥有完全不同的操作集合，链所有者创建操作，并将操作包含到新区块中，实现与应用交互。

以"fungible token"为例，给另一个用户发送资金的操作定义如下：

```rust,ignore
# extern crate serde;
# use serde::{Deserialize, Serialize};
#[derive(Debug, Deserialize, Serialize)]
pub enum Operation {
    /// A transfer from a (locally owned) account to a (possibly remote) account.
    Transfer {
        owner: AccountOwner,
        amount: Amount,
        target_account: Account,
    },
    // Meant to be extended here
}
```

**消息**是操作或者其他消息执行的结果，可以在不同微链之间传递。区块创建者可以将消息包含在区块内，但是与操作不同，只有那些被其他微链(或者相同微链此前)创建的消息，才能按照正确顺序添加到区块中(意味着有些消息可能被忽略)。

在"fungible token"应用中，给账户记账的消息定义如下：

```rust,ignore
# extern crate serde;
# use serde::{Deserialize, Serialize};
#[derive(Debug, Deserialize, Serialize)]
pub enum Message {
    Credit { owner: AccountOwner, amount: Amount },
    // Meant to be extended here
}
```

### [认证](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/applications?id=authentication)

操作总是认证过的，给区块签名认证的角色天然就是本区块内包含的操作的认证者。消息也可以认证。执行操作时，应用程序可以创建消息并发送给其他微链，这些消息可以设置为认证过的。在这种情况下，接收方将收到与创建该消息的操作拥有相同认证信息的消息。同样，如果处理消息时创建了新消息，新消息也可以配置与旧消息相同的认证信息。

换句话说，区块签名者可以通过一系列的消息跨链广播其认证信息，这样，应用程序就可以安全地在微链上存储那些不能创建区块的用户状态。应用程序甚至可以仅允许被授权的用户更改其状态，且就算链所有者也不能修改这样的授权。

下面的图表展示了4条微链(A, B, C, D)以及这些微链上产生的区块。在这个示例中，每条微链都只有一个所有者(即地址)，所有者们负责创建新区块和使用他们的签名密钥验证新区块。图例中的有些区块展示了其中包含的操作和消息，操作和消息的验证信息在括号的附加内容中显示。可以看到，所有的操作都是被区块创建者认证过的，如果微链上只有一个用户(即链所有者)，区块创建者就是链所有者。消息携带的认证信息总是与创建消息的操作或消息相同。

其中的一个例子，微链A创建了一个区块，其中包含操作1，操作1由微链A的所有者认证(写作`(a)`)。操作1将向微链B发送一条消息，这条消息携带了发送者的认证信息，微链B将会接受该消息及其认证信息`(a)`。另一个例子是微链D创建了一个包含操作2的区块，该区块由微链D的所有者认证(写作`(d)`)。就像前面的例子一样，操作2向微链C发送了一条携带认证信息的消息，微链C处理该消息时，创建了一条新消息，并发送给微链B。当微链B收到消息时，同时也会收到最早的消息认证信息`(d)`。

```ignore
                            ┌───┐     ┌─────────────────┐     ┌───┐
       Chain A owned by (a) │   ├────►│ Operation 1 (a) ├────►│   │
                            └───┘     └────────┬────────┘     └───┘
                                               │
                                               └────────────┐
                                                            ▼
                                                ┌──────────────────────────┐
                            ┌───┐     ┌───┐     │ Message from chain A (a) │
       Chain B owned by (b) │   ├────►│   ├────►│ Message from chain C (d) |
                            └───┘     └───┘     │ Operation 3 (b)          │
                                                └──────────────────────────┘
                                                            ▲
                                                   ┌────────┘
                                                   │
                            ┌───┐     ┌──────────────────────────┐     ┌───┐
       Chain C owned by (c) │   ├────►│ Message from chain D (d) ├────►│   │
                            └───┘     └──────────────────────────┘     └───┘
                                                 ▲
                                     ┌───────────┘
                                     │
                            ┌─────────────────┐     ┌───┐     ┌───┐
       Chain D owned by (d) │ Operation 2 (d) ├────►│   ├────►│   │
                            └─────────────────┘     └───┘     └───┘
```

"Fungible"应用中有另一个转账例子，其中的`Claim`操作允许用户从另一条用户没有控制权(但用户信任该微链会创建新区块接受用户发送的消息)的微链提取资金。如果没有`Claim`操作，用户将仅能在自己的微链存入资金，多所有者微链和公开微链的资金将在所有有能力创建区块的用户之间共享(译者注：最后一句意思不是很明确)。

`Claim`操作允许用户在其他微链上存入资金，只要用户能够在该微链创建区块，或者相信该微链的所有者会创建区块处理他们的消息。这样的操作也确保即使微链有多个所有者，或者用户无权创建新区块时，用户的资金依然只有用户自己能动用。

## [注册跨链应用](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/applications?id=registering-an-application-across-chains)

如果Alice使用她自己的微链上的应用与Bob的应用交互，例如，通过`fungible`应用向Bob发送一些资金，那么，当Bob的微链接收到Alice发出的消息，并开始处理该消息时，应用就会在Bob的微链上自动注册。应用注册完成后，Bob就可以在自己的微链上执行应用操作，例如，给某人发送一些资金。

某些情况下，Bob想使用那些他的微链上没有的应用，该怎么办？例如，Alice运营了一个`social`应用，她会定期更新帖子，而Bob想订阅Alice的内容。

此时，Bob将不能执行`social`应用的操作，因为该应用没有在他的微链注册。Bob首先需要向Alice请求该应用：

```bash
linera request-application <application-id> --target-chain-id <alices-chain-id>
```

一旦Alice处理Bob的请求消息(当Alice将Linera客户端运行在服务模式，消息将被自动处理)，Bob就可以在他的微链上访问`social`应用。
