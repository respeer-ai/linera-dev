#  验证者

验证者运行着允许用户下载和创建区块的服务器。它们对所有链上的区块进行验证、执行并加密认证。

> 在Linera中，每个链均由同一组验证者支持，并具有相同的安全级别。

验证者的主要功能是确保基础设施的完整性，具体而言是指：

- 每个区块均有效，即其格式正确、操作合法、接收到的消息顺序正确，例如余额计算无误。

- 每个链收到的每条消息实际上都是由另一个链发送的。

- 如果某一高度的一个区块被认证，则同一高度的其他区块均不会被认证。

只要三分之二的验证者（按其质押权重计算）遵守协议，这些属性就一定能得到保证。未来，若验证者偏离协议，其可能被视为恶意行为并失去所 _质押的资产_ 。

验证者还通过确保链的历史记录始终可用，来维护系统的活性。然而，由于验证者在大多数链上并不提议区块（参见[下一节](../../../zh_CN/developers/advanced_topics/block_creation.md)），因此他们 _无法_ 保证任何特定操作或消息最终会在链上执行。相反，链的所有者决定是否以及何时提议新区块，并决定包含哪些操作和消息。Linera客户端的当前实现会自动将所有传入消息纳入新区块，而操作则是链所有者明确添加的动作（例如转账）。

## 验证者的架构

由于每个链都使用相同的验证者，因此添加更多链时无需新增验证者。相反，需要让每个验证者通过添加更多计算单元（也称为“workers”或“physical shards”）来进行横向扩展。

归根结底，Linera验证者类似于由以下组成的Web2服务。

- 负载均衡器（又称入口/出口），目前通过名为`linera-proxy`的二进制文件实现，

- 多个工作节点，当前通过名为`linera-server`的二进制文件实现，

- 共享数据库，当前通过抽象接口`linera-storage`实现。

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

在验证者内部，各组件通过其内部网络进行通信。值得注意的是，工作节点之间通过直接的远程过程调用（RPC）来传递跨链消息。

需要注意的是，每个验证者的工作节点数量可能有所不同。负载均衡器和共享数据库虽然被表示为单一实体，但在生产环境中需要进行横向扩展。

> 在本地开发测试中，我们目前使用单个工作节点，并通过测试用内存服务作为共享数据库。

<!--
## Configuring Networks, Workers, and Proxies

In [a previous section](../getting_started/hello_linera.md), we used the
`linera net up` command to start a local network. This should be sufficient for
most use cases when you're running a local network.

```bash
linera net up
```

However, it is possible to customize and configure the parameters of the
network.

To do this, you need the `linera-protocol` repository and the
`./scripts/run_local.sh` script.

`run_local.sh` uses the `validator_n.toml` file from the `configuration/`
directory to configure validator number `n`.

```bash
linera-server generate --validators configuration/validator_{1,2,3,4}.toml --committee committee.json
```

generates keys and writes them, together with the options from the TOML files,
to `server_1.json`, ..., `server_4.json`. It also stores the set of the new
validators' public keys in `committee.json`.

```bash
linera --wallet wallet.json --storage rocksdb:linera.db create-genesis-config 10 --genesis genesis.json --initial-funding 10 --committee committee.json
```

creates a configuration for the initial state of the network, `genesis.json`,
with 10 chains, each with a balance of 10. It also creates a `wallet.json` for a
client who owns all those chains and initializes the corresponding local node
`linera.db`.

To start the newly configured network, each validator `n` must start their
proxy:

```bash
linera-proxy server_n.json &
```

And all shards; for shard `i`:

```bash
linera-server run --storage rocksdb:server_n_i.db --server server_n.json --shard i --genesis genesis.json &
```

This will create a separate database file `server_n_i.db` for each shard. In a
production network, these would be running on different machines.

## Changing the Set of Validators

If a new validator wants to start participating, or an old one wants to leave,
all chains must be updated.

The system has one designated _admin chain_, where the validators can join or
leave, and where new _epochs_ are defined. During every epoch, the set of
validators is fixed. If you own the admin chain, you can use the `set-validator`
and `remove-validator` commands to start a new epoch with a modified set of
validators:

```bash
linera --wallet wallet.json set-validator --name 5b611b86cc1f54f73a4abfb4a2167c7327cc85a74cb2a5502431f67b554850b4 --address 127.0.0.1:9100 --votes 3
linera --wallet wallet.json remove-validator --name f65a585f05852f0610e2460a99c23faa3969f3cfce8a519f843a793dbfb4cb84
```

Chain owners must then create a block that receives the `SetCommittees` message
from the admin chain, and have it certified by the old validators. Only the
_next_ block in their chain will be certified by the new validator set!

The _admin chain_ is currently managed by a single user. In the future, it will
be a _public chain_ (i.e. managed by validators). We anticipate that Linera
epochs will change once per day (or less) and that several subsequent epochs
will overlap so that chain owners have enough time to migrate their chains.
(Chain migration may also be delegated to third parties. See
[next section](block_creation.html).)

-->
