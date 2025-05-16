# 应用程序

Linera的编程模型设计旨在帮助开发者利用微链实现应用的横向扩展。

Linera通过[WebAssembly（Wasm）](https://webassembly.org)虚拟机执行用户应用程序。当前，[Linera SDK](../../../zh_CN/developers/backend.md)主要面向[Rust语言](https://www.typescriptlang.org/)（后端开发）与[TypeScript](https://www.typescriptlang.org/)（前端开发）提供支持。

Linera应用程序基于**Rust crate**概念构建：应用程序的外部接口（包括实例化参数、操作及消息）通常定义在其crate的库模块中，而应用核心逻辑则编译为适用于Wasm架构的二进制文件。

## 应用部署生命周期​​

Linera应用程序设计强调功能强大且可重用性。基于此设计理念，系统严格区分了字节码与网络中的应用实例。

Applications undergo a lifecycle transition aimed at making development easy and
flexible:

1. 字节码由包含 `linera-sdk` 依赖的 Rust 项目构建生成。
2. 该字节码会发布到网络中的微链，并分配唯一标识符。
3. 用户可通过提供字节码标识符与实例化参数创建新应用实例，此过程将返回应用标识符，用于后续引用及交互操作。
4. 同一字节码标识符可被不同用户按需多次调用，以创建独立应用实例。

关键是应用部署生命周期对用户透明​​,用户只需通过单条命令即可发布应用：

```bash
linera publish-and-create <contract-path> <service-path> <init-args>
```

此操作将同时完成字节码发布与应用程序实例化。

## 应用程序结构解析​​

**应用程序**由两大核心组件构成：​​_合约_ ​​与 _​​服务_ ​​。

​**合约**​​具备Gas计费机制，负责执行业务操作与消息处理、发起跨应用调用，并直接修改应用状态。其详细机制将在[应用后端开发指南](../../../zh_CN/developers/backend.md)中深入解析。

**​服务**​​为无Gas消耗且只读的组件，主要用于查询应用状态，并为前端展示层（如用户界面）提供所需的数据支撑。

## 操作与消息​​

> 在本节中，我们将使用名为 "Fungible" 的示例应用简化版本。该应用支持用户间代币转账功能，可用于理解基础代币交互逻辑。

在系统层面，与应用程序的交互通过​​操作指令​​与​​消息协议​​实现。

**​操作指令**​​由应用开发者定义，每个应用可拥有完全独立的操作集。链所有者通过主动创建操作指令并将其打包至区块提案中，实现对应用的交互操作。其他应用也可通过向目标应用提供操作指令发起调用（称为​​跨应用调用​​），此类调用仅限于同一链内执行。跨应用调用的操作指令可能向调用方返回响应值。

以"Fungible"代币应用为例，用户向另一用户转账的操作指令定义如下：

```rust
# extern crate serde;
# extern crate linera_sdk;
# use serde::{Deserialize, Serialize};
# use linera_sdk::linera_base_types::*;
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

**消息**由操作指令执行或其他消息传递触发产生。消息可在链间发送，但仅限于同一应用内交互。区块提议者可主动将消息打包至区块提案中，但与操作指令不同，其必须遵循顺序规则（允许跳过部分消息），且仅能包含由其他链（或同一链的前序区块）生成的有效消息。同一事务产生的消息在接收区块中作为单个事务处理。

以"Fungible Token"代币应用为例，账户信用操作对应的消息定义如下：

```rust
# extern crate serde;
# extern crate linera_sdk;
# use serde::{Deserialize, Serialize};
# use linera_sdk::linera_base_types::*;
#[derive(Debug, Deserialize, Serialize)]
pub enum Message {
    Credit { owner: AccountOwner, amount: Amount },
    // Meant to be extended here
}
```

### 认证机制

区块中的操作始终经过认证，而消息可能经过认证。区块签名者成为该区块内所有操作的认证主体。当操作被应用程序执行时，可能生成并发送消息至其他链。这些消息在创建时可配置是否启用认证——若启用，则继承创建操作的认证信息。若处理传入消息时生成新消息，这些衍生消息也可配置继承原始消息的认证。

简言之，区块签名者的权限可通过消息链式传播至跨链场景。这使得应用程序能将用户状态安全存储在用户无权生成区块的链上，且仅授权用户可修改状态——即便链所有者也无法覆盖。

下图展示了四条链（A/B/C/D）及其生成的区块。示例中每条链由单一所有者（即地址）管理，负责生成区块并使用私钥签名。部分区块标注了其接受的操作与传入消息，括号内为认证标识：所有操作均由区块提议者认证（若为单用户链，则提案者即链所有者）,认证消息继承创建者或来源消息的认证。

​示例说明​​：
    ​​链A​​生成包含操作1的区块，由链A所有者认证（标记为`(a)）`。该操作向链B发送消息，若启用认证转发，链B将以`(a)`认证执行该消息。
    ​​链D​​生成包含操作2的区块，由链D所有者认证（标记为`(d)`）。该操作向链C发送消息，链C以`(d)`认证执行。
    链C处理该消息后生成新消息发送至链B，链B以`(d)`认证执行此消息。


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

以"Fungible"代币应用为例，其"申领操作"允许用户从不受控链（但用户信任该链会生成接收消息的区块）中提取资金。若无此操作，用户只能将代币存储在自有链上，且多所有者/公共链的代币将被任何具备区块生成能力的实体共享。

通过"申领操作"，用户可将代币存入两类链： ​​可自主生成区块的链, ​​信任链所有者会生成接收消息区块的链。
仅代币持有者可转移资产（即使在多所有者链或无区块生成权限的链中），跨链资产转移通过认证链式传递实现权限控制。
