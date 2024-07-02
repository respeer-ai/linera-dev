# 你好，Linera

本节将介绍如何与Devnet进行交互，运行本地开发网络，然后从头开始编译和部署你的第一个应用程序。

到本节结束时，你将在Devnet上和/或你的本地网络上拥有一个[microchain](../core_concepts/microchains.md)，以及一个可以使用GraphQL查询的工作应用程序。

## 使用Devnet

Linera Devnet是Linera协议的一个部署，对开发者非常有用。请注意，它不稳定，并且随时可以从干净状态和新的创世纪重新启动。

要与Devnet交互，需要一些代币。提供了一个水龙头服务，用于创建新的microchain并获取一些测试代币。在初始化钱包时配置如下：

```bash
linera wallet init --with-new-chain --faucet https://faucet.{{#include ../../../RELEASE_DOMAIN}}.linera.net
```

> 这将在Devnet上创建一个新的microchain，并提供一些初始的测试代币，该链会自动添加到新创建的钱包中。
>
> > 确保使用与当前Devnet兼容的Linera工具链 [compatible with the current Devnet](installation.md#installing-from-cratesio).

## 启动本地测试网络

另一个选项是启动你自己的本地开发网络。开发网络由多个[验证者](../advanced_topics/validators.md)组成，每个验证者包括入口代理（即“负载均衡器”）和若干工作节点（即“物理分片”）。

要启动本地网络，请运行以下命令：

```bash
linera net up
```

这将启动一个验证者，使用默认数量的分片，并创建一个临时目录来存储整个网络状态。

这将设置一些初始链，并创建一个用于操作这些链的初始钱包。

### 使用初始测试钱包

`linera net up`会在其标准输出中打印Bash语句，帮助你配置终端以使用新测试网络的初始钱包，例如：

```bash
export LINERA_WALLET="/var/folders/3d/406tbklx3zx2p3_hzzpfqdbc0000gn/T/.tmpvJ6lJI/wallet.json"
export LINERA_STORAGE="rocksdb:/var/folders/3d/406tbklx3zx2p3_hzzpfqdbc0000gn/T/.tmpvJ6lJI/linera.db"
```

这个钱包只在单个网络的生命周期内有效。每次重新启动本地网络时，都需要重新配置钱包。

## 与网络交互

> 在以下示例中，我们假设钱包已初始化以与Devnet进行交互，或者变量 `LINERA_WALLET` 和 `LINERA_STORAGE` 都已设置并指向运行中本地网络的初始钱包。

与网络和部署应用程序的主要方式是使用`linera`客户端。

要检查网络是否正常工作，可以同步你的[默认链](../core_concepts/wallets.md)与网络的其余部分，并显示链上的余额，如下所示：

```bash
linera sync
linera query-balance
```

你应该会看到一个数字输出，例如 `10`。

## 构建示例应用程序

在Linera上运行的应用程序是[Wasm](https://webassembly.org/)字节码。每个验证者和客户端都有一个内置的Wasm虚拟机（VM），可以执行字节码。

让我们从`examples/`子目录构建`counter`应用程序：

```bash
cd examples/counter && cargo build --release --target wasm32-unknown-unknown
```

1. ## 发布你的应用程序

   你可以使用`linera`客户端的`publish-and-create`命令发布字节码，并在你的本地网络上使用它创建一个应用程序，并提供：

   1. 合约字节码的位置
   2. 服务字节码的位置
   3. JSON编码的初始化参数

```bash
linera publish-and-create \
  ../target/wasm32-unknown-unknown/release/counter_{contract,service}.wasm \
  --json-argument "42"
```

恭喜你！你已经在Linera上发布了你的第一个应用程序！

## 查询你的应用程序

现在让我们查询你的应用程序，以获取当前计数器值。为此，我们需要使用运行在[_service_模式](../core_concepts/node_service.md)的客户端。这将在本地暴露一堆API，我们可以用它们与网络上的应用程序交互。

```bash
linera service
```

<!-- TODO: add graphiql image here -->

在浏览器中导航到 `http://localhost:8080` 访问GraphiQL，即[GraphQL](https://graphql.org) IDE。我们将在稍后的章节中详细讨论这个功能。 现在，通过运行以下查询来列出你的默认链上部署的应用程序 e476…：

```gql
query {
  applications(
    chainId: "e476187f6ddfeb9d588c7b45d3df334d5501d6499b3f9ad5595cae86cce16a65"
  ) {
    id
    description
    link
  }
}
```

因为我们只部署了一个应用程序，所以返回的结果中只有一个条目。

在返回的JSON的底部有一个 `link` 字段。要与你的应用程序交互，复制并粘贴链接到新的浏览器标签页中。

最后，要查询计数器的值，运行：

```gql
query {
  value
}
```

这将返回一个值 `42`，这是我们在部署应用程序时指定的初始化参数。