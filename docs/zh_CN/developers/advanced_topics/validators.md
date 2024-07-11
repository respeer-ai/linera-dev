- # 验证者
  
  验证者运行允许用户下载和创建区块的服务器。它们验证、执行并通过密码学方式认证所有链的区块。
  
  > 在 Linera 中，每条链都由相同的一组验证者支持，并具有相同的安全级别。
  
  验证者的主要功能是保证基础设施的完整性，具体包括：
  
  - 每个区块都是有效的，即具有正确的格式，其操作是允许的，接收的消息顺序正确，并且例如余额计算正确。
  - 每条链接收到的每个消息确实是由另一条链发送的。
  - 如果特定高度的一个区块已被认证，那么同一高度的其他区块就不会被认证。
  
  只要三分之二的验证者（按其权益加权）遵循协议，上述属性就能得以保持。未来，偏离协议可能导致验证者被视为恶意，并且失去他们的质押。
  
  验证者还通过确保链的历史保持可用来促进系统的活跃性。然而，由于验证者在大多数链上不提出区块（见[下一节](block_creation.html)），它们并不保证任何特定操作或消息最终会在链上执行。相反，链的所有者决定是否以及何时提出新的区块，以及包括哪些操作和消息。Linera 客户端的当前实现自动将所有传入消息包含在新区块中。操作是链所有者明确添加的动作，例如转账。
  
  ## 验证者的架构
  
  由于每条链使用相同的验证者，增加更多链并不需要增加验证者。相反，它需要每个个体验证者通过添加更多的计算单元（也称为“工作者”或“物理分片”）来扩展。
  
  最终，一个 Linera 验证者类似于由以下组件构成的 Web2 服务：
  
  - 负载均衡器（即入口/出口），当前由二进制文件 `linera-proxy` 实现，
  - 一定数量的工作者，当前由二进制文件 `linera-server` 实现，
  - 共享数据库，当前由抽象接口 `linera-storage` 实现。

```ignore
Example of Linera network

                    │                                             │
                    │                                             │
┌───────────────────┼───────────────────┐     ┌───────────────────┼───────────────────┐
│ validator 1       │                   │     │ validator N       │                   │
│             ┌─────┴─────┐             │     │             ┌─────┴─────┐             │
│             │   load    │             │     │             │   load    │             │
│       ┌─────┤  balancer ├────┐        │     │       ┌─────┤  balancer ├──────┐      │
│       │     └───────────┘    │        │     │       │     └─────┬─────┘      │      │
│       │                      │        │     │       │           │            │      │
│       │                      │        │     │       │           │            │      │
│  ┌────┴─────┐           ┌────┴─────┐  │     │  ┌────┴───┐  ┌────┴────┐  ┌────┴───┐  │
│  │  worker  ├───────────┤  worker  │  │ ... │  │ worker ├──┤  worker ├──┤ worker │  │
│  │    1     │           │    2     │  │     │  │    1   │  │    2    │  │    3   │  │
│  └────┬─────┘           └────┬─────┘  │     │  └────┬───┘  └────┬────┘  └────┬───┘  │
│       │                      │        │     │       │           │            │      │
│       │                      │        │     │       │           │            │      │
│       │     ┌───────────┐    │        │     │       │     ┌─────┴─────┐      │      │
│       └─────┤  shared   ├────┘        │     │       └─────┤  shared   ├──────┘      │
│             │ database  │             │     │             │ database  │             │
│             └───────────┘             │     │             └───────────┘             │
└───────────────────────────────────────┘     └───────────────────────────────────────┘

```

在验证者内部，各组件通过验证者的内部网络进行通信。特别是，工作者之间使用直接的远程过程调用（RPC）来传递跨链消息。

请注意，每个验证者的工作者数量可能会有所不同。负载均衡器和共享数据库虽然表示为单个实体，但在生产环境中它们可以进行横向扩展。

> 在开发过程中的本地测试中，我们目前使用单个工作者和 RocksDB 作为数据库。

<!--
## 配置网络、工作者和代理

在[之前的章节](../getting_started/hello_linera.md)中，我们使用了 `linera net up` 命令来启动一个本地网络。当您运行本地网络时，这应该对大多数用例来说已经足够了。

```bash
linera net up
```

然而，您也可以自定义和配置网络的参数。

为此，您需要 `linera-protocol` 代码库和 `./scripts/run_local.sh` 脚本。

`run_local.sh` 使用位于 `configuration/` 目录下的 `validator_n.toml` 文件来配置第 `n` 个验证者。

```bash
linera-server generate --validators configuration/validator_{1,2,3,4}.toml --committee committee.json
```

这个命令生成密钥，并将其与来自 TOML 文件的选项一起写入到 `server_1.json` 到 `server_4.json` 中。它还将新验证者的公钥集合存储在 `committee.json` 文件中。

```bash
linera --wallet wallet.json --storage rocksdb:linera.db create-genesis-config 10 --genesis genesis.json --initial-funding 10 --committee committee.json
```

此命令为网络的初始状态创建配置文件 `genesis.json`，其中包含了 10 条链，每条链的余额为 10。它还为拥有所有这些链的客户创建了 `wallet.json`，并初始化相应的本地节点 `linera.db`。

要启动新配置的网络，每个验证者 `n` 需要启动他们的代理：

```bash
linera-proxy server_n.json &
```

以及所有的分片；对于分片 `i`：

```bash
linera-server run --storage rocksdb:server_n_i.db --server server_n.json --shard i --genesis genesis.json &
```

这将为每个分片创建一个单独的数据库文件 `server_n_i.db`。在生产网络中，这些会在不同的机器上运行。

## 更改验证者集合

如果一个新的验证者想要加入，或者一个旧的想要离开，所有的链都必须进行更新。

系统有一个指定的 *管理员链*，验证者可以在这里加入或离开，并定义新的 *时代* 。每个时代期间，验证者集合是固定的。如果您拥有管理员链，您可以使用 `set-validator` 和 `remove-validator` 命令来开始一个带有修改验证者集合的新时代：

```bash
linera --wallet wallet.json set-validator --name 5b611b86cc1f54f73a4abfb4a2167c7327cc85a74cb2a5502431f67b554850b4 --address 127.0.0.1:9100 --votes 3
linera --wallet wallet.json remove-validator --name f65a585f05852f0610e2460a99c23faa3969f3cfce8a519f843a793dbfb4cb84
```

链所有者然后需要创建一个区块，接收来自管理员链的 `SetCommittees` 消息，并由旧的验证者进行认证。只有他们链中的 *下一个* 区块才会由新的验证者集合进行认证！

目前 *管理员链* 是由单个用户管理的。未来，它将成为一个 *公共链*（即由验证者管理）。我们预计 Linera 的时代每天（或更少）会更改一次，并且连续的几个时代会重叠，以便链所有者有足够的时间迁移他们的链。（链迁移也可能委托给第三方。参见[next section](block_creation.html)。）

-->