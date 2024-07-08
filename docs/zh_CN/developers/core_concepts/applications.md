# 应用程序

Linera的编程模型旨在使开发人员能够利用微链来扩展其应用程序。

Linera使用[WebAssembly (Wasm)](https://webassembly.org)虚拟机执行用户应用程序。目前，[Linera SDK](../sdk.md)专注于[Rust](https://www.rust-lang.org/)编程语言。

Linera应用程序结构采用熟悉的**Rust crate**概念：应用程序的外部接口（包括实例化参数、操作和消息）通常放在其crate的库部分，而每个应用程序的核心则编译为适用于Wasm架构的二进制文件。

## 应用程序部署生命周期

Linera应用程序被设计为强大且可重复使用。因此，字节码与网络上的应用程序实例之间存在区别。

应用程序经历一系列生命周期转换，旨在使开发变得简单灵活：

1. 从具有`linera-sdk`依赖项的Rust项目构建字节码。
2. 将字节码发布到微链网络，并分配一个标识符。
3. 用户可以通过提供字节码标识符和实例化参数来创建新的应用程序实例。此过程返回一个应用程序标识符，可用于引用和与应用程序交互。
4. 相同的字节码标识符可以被多次使用，为多个用户创建不同的应用程序。

重要的是，应用程序部署生命周期对用户来说是抽象的，可以使用单个命令发布应用程序：

```bash
linera publish-and-create <contract-path> <service-path> <init-args>
```

这将为您发布字节码并实例化应用程序。

## Anatomy of an Application

An **application** is broken into two major components, the _contract_ and the
_service_.

The **contract** is gas-metered, and is the part of the application which
executes operations and messages, make cross-application calls and modifies the
application's state. The details are covered in more depth in the
[SDK docs](../sdk.md).

The **service** is non-metered and read-only. It is used primarily to query the
state of an application and populate the presentation layer (think front-end)
with the data required for a user interface.

## 应用程序的解剖

一个**应用程序**分为两个主要组件，即**合约**和**服务**。

**合约**是按燃气计量的，是应用程序的一部分，负责执行操作和消息、进行跨应用程序调用以及修改应用程序的状态。详情可以参考[SDK文档](../sdk.md)。

**服务**是非计量且只读的。主要用于查询应用程序的状态，并为用户界面的呈现层提供所需的数据。

## 操作和消息

> 在本节中，我们将使用一个简化版本的示例应用程序“可互换”，用户可以相互发送代币。

在系统级别上，与应用程序交互可以通过操作和消息完成。

**操作**由应用程序开发人员定义，每个应用程序可以拥有完全不同的操作集。链所有者通过主动创建操作，并将它们放入其区块提案中与应用程序交互。其他应用程序也可以通过为其提供操作来调用应用程序执行操作，这称为跨应用程序调用，始终发生在同一链内。跨应用程序调用的操作可能会向调用者返回响应值。

以“可互换代币”应用程序为例，用户向另一个用户转账的操作如下所示：

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

****消息**由操作或其他消息的执行结果产生。消息可以在同一应用程序内从一条链发送到另一条链。区块提议者还可以在其区块提案中主动包含消息，但与操作不同的是，它们只能按正确的顺序（可能跳过某些消息），并且只有在它们确实是由另一条链（或同一链的前一个区块）创建时才能包含。

在我们的“可互换代币”应用程序中，信用账户的消息看起来像这样：

```rust,ignore
# extern crate serde;
# use serde::{Deserialize, Serialize};
#[derive(Debug, Deserialize, Serialize)]
pub enum Message {
    Credit { owner: AccountOwner, amount: Amount },
    // Meant to be extended here
}
```

### 认证

区块中的操作始终是经过身份验证的，而消息可能是经过身份验证的。区块签名者成为该区块中所有操作的验证者。随着应用程序执行操作，可以创建消息发送到其他链。当创建消息时，可以配置为经过身份验证。在这种情况下，消息将接收与创建它的操作相同的认证。如果处理传入消息会创建新的消息，那些消息也可以配置为具有与接收到的消息相同的认证。

换句话说，区块签名者可以通过一系列消息将其权限传播到不具备产生区块权限的链上，从而允许应用程序安全地在用户可能无法产生区块的链上存储用户状态。应用程序可以只允许授权用户更改该状态，即使链所有者也无法覆盖该状态。

下图显示了四条链（A、B、C、D）及其生成的一些区块。在本示例中，每条链都由单个所有者（即地址）拥有。所有者负责生成区块，并使用其签名密钥签署新的区块。一些区块显示了它们接受的操作和传入的消息，其中认证显示在括号内。所有生成的操作都由区块提议者验证，如果这些链都是单用户链，那么提议者始终是链所有者。具有认证的消息使用创建它们的操作或消息的认证。

在图中的一个示例是，链A生成了一个带有操作1的区块，由链A所有者验证（写成`(a)`）。该操作向链B发送了一个消息，假设消息是启用了认证转发的，它将在链B中以`(a)`的认证接收并执行。另一个示例是，链D生成了一个带有操作2的区块，由链D所有者验证（写成`(d)`）。该操作发送了一个消息到链C，在链C中使用`(d)`的认证执行，类似前面的示例。在链C处理该消息时生成了一个新的消息，该消息发送到链B。当链B接收到该消息时，将使用`(d)`的认证执行。

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

在可互换代币应用程序中使用的一个示例是，`Claim`操作允许用户从他们无法控制但仍然信任会接收他们消息并生成区块的链上提取资金。如果没有`Claim`操作，用户只能在自己的链上存储代币，而多所有者和公共链会使他们的代币在任何能够生成区块的人之间共享。

有了`Claim`操作，用户可以将他们的代币存储在另一条链上，他们可以生成区块，或者他们信任的所有者会生成区块并接收他们的消息。只有他们能够转移他们的代币，即使在共享所有权或者不能生成区块的链上也是如此。

## 在多条链注册应用程序

如果Alice在她的链上使用一个应用程序，并且通过该应用程序与Bob进行交互，例如使用可互换代币示例向他发送一些代币，该应用程序会自动在Bob的链上注册，只要他处理了传入的跨链消息。之后，他也可以在他的链上执行应用程序的操作，并向其他人发送代币。

但是，还有一些情况，Bob可能想要开始使用他还没有的应用程序。例如，Alice经常使用社交示例发布帖子，而Bob想要订阅她。

在这种情况下，尝试执行特定于应用程序的操作将会失败，因为该应用程序尚未在他的链上注册。他需要首先向Alice发出请求：

```bash
linera request-application <application-id> --target-chain-id <alices-chain-id>
```

一旦Alice处理了他的消息（如果她正在服务模式下运行客户端，则会自动处理），他就可以开始使用该应用程序。