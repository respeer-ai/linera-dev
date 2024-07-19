# Oracles 和 Ethereum

> Oracles 是一种机制，允许应用程序从尚未在链上的外部世界获取信息。一个例子是从 Linera 应用程序中访问以太坊智能合约的状态。

合约运行时目前具有两种 Oracle 方法：

- `query_service` 允许合约调用应用程序自己的服务代码。服务可以访问一些链下信息，因此不能保证每次调用时都返回相同的结果。
- `http_post` 发出一个 HTTP POST 请求并返回响应。

应用程序应仅以非常有可能让所有验证者看到相同结果的方式使用这些方法，否则运行该应用程序代码的任何区块提案都可能失败，损害用户链的活跃性。（此外，在快速轮次中完全禁止 Oracle 方法；请参阅[链所有权语义](../core_concepts/microchains.md#chain-ownership-semantics)。）

Linera SDK 使用 `http_post` 实现了 `EthereumClient` 类型，提供以下函数来查询以太坊节点：

- `get_balance` 用于在特定区块号下访问以太坊账户的余额。
- `read_events` 用于从指定以太坊智能合约中读取事件，从一个块到最后一个块。
- `non_executive_call` 用于在特定区块上执行以太坊智能合约中的函数，其结果不会被包含在链上。

这些函数都不会改变以太坊区块链的状态，而是观察链的情况。所有这些函数都接受一个特定的块号作为输入，否则不同的验证者可能会读取不同数量的块。

智能合约 `ethereum-tracker` 提供了一个这样的合约示例，端到端测试 `test_wasm_end_to_end_ethereum_tracker` 展示了实际的交互方式。
