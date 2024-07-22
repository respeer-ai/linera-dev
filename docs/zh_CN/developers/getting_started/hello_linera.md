# 1.2. Hello, Linera


本节我们将介绍Linera Devnet交互方式和运行本地测试网的步骤，然后从0开始编译并部署第一个Linera应用。

本小节结束后，你将能够在Linera Devnet和你自己的本地测试网上创建一条[微链](zh_CN/developers/core_concepts/microchains.md)，并运行一个可以使用GraphQL访问的应用。

## [使用Devnet](zh_CN/developers/getting_started/hello_linera.md#使用Devnet)

Linera Devnet是部署给开发者使用，开发者需要知道Devnet是不稳定的，随时可能使用新的创世设置重启。

为了能够与Devnet交互，我们需要一些Token。执行以下命令初始化并设置钱包后，我们可以从水龙头服务领取一些测试Token：

```bash
linera wallet init --with-new-chain --faucet https://faucet.devnet-2024-05-07.linera.net
```

上述命令将在Devnet上创建一条微链，该微链持有一定数量的测试Token，并且自动被加入到新创建的钱包。

> 确定你使用的Linera工具链[与当前的Devnet兼容](zh_CN/developers/getting_started/installation.md#从cratesio安装)。

## [启动本地测试网](zh_CN/developers/getting_started/hello_linera.md#启动本地测试网)

你也可以在本地环境启动本地测试网络。测试网络包含一些[验证器](zh_CN/developers/advanced_topics/validators.md)，每个验证器都包含一个入口代理(即负载均衡器)(译者注：原文为ingress，等同于微服务集群中的网关代理，用于将请求根据预设规则路由到对应的服务)和一些工作节点(即物理分片)。

执行以下命令启动本地测试网：

```bash
linera net up
```

上述命令将会启动一个具有默认分片设置的验证器，并创建一个临时目录来存储网络状态。

同时，该命令会创建一些初始微链，并创建一个可以操作这些微链的钱包。

### [使用测试网络初始钱包](zh_CN/developers/getting_started/hello_linera.md#使用测试网络初始钱包)

命令`linera net up`在终端中打印了Bash设置(如下面的例子)，这些设置可以帮助开发者设置他们的终端，以使用测试网络的初始钱包。

```bash
export LINERA_WALLET="/var/folders/3d/406tbklx3zx2p3_hzzpfqdbc0000gn/T/.tmpvJ6lJI/wallet.json"
export LINERA_STORAGE="rocksdb:/var/folders/3d/406tbklx3zx2p3_hzzpfqdbc0000gn/T/.tmpvJ6lJI/linera.db"
```

该钱包仅仅在测试网络执行期间有效，每次重启测试网络，该钱包都将被重新配置。

## [与网络交互](zh_CN/developers/getting_started/hello_linera.md#与网络交互)

> 在后面的例子中，我们假设终端已经将钱包设置好直接访问Devnet，或者`LINERA_WALLET`和`LINERA_STORAGE`环境变量已经设置为本地测试网络的初始钱包。

`linera`客户端是与Linera网络和应用交互的基本方式。

你可以通过如下命令同步[默认微链](zh_CN/developers/core_concepts/wallets.md)并显示微链余额来确认网络是否正常工作：

```bash
linera sync
linera query-balance
```

如果一切正常，你将会看到一个数字，例如`10`。

## [构建示例应用程序](zh_CN/developers/getting_started/hello_linera.md#构建示例应用程序)

运行在Linera网络上的应用是[Wasm](https://webassembly.org/)字节码。每个验证器和客户端上都会运行一个内置的Wasm虚拟机(VM)实例，用来执行字节码。

下面我们将构建`examples/`子目录中的`counter`应用：

```bash
cd examples/counter && cargo build --release --target wasm32-unknown-unknown
```

## [发布应用](zh_CN/developers/getting_started/hello_linera.md#发布应用)

使用`linera`客户端中的`publish-and-create`命令， 你可以发布字节码，同时创建一个Linera应用，该命令需要以下参数：

1. 合约字节码的路径
2. 服务字节码的路径
3. 初始化参数的JSON文本

```bash
linera publish-and-create \
  ../target/wasm32-unknown-unknown/release/counter_{contract,service}.wasm \
  --json-argument "42"
```

恭喜你！到此你已经成功发布了第一个Linera应用！

## [查询应用](zh_CN/developers/getting_started/hello_linera.md#查询应用)

现在我们可以查询上面部署的应用，获取当前计数值。查询应用需要使用客户端的[_服务_ 模式](zh_CN/developers/core_concepts/node_service.md)运行一个Linera节点服务，我们将通过节点服务提供的一系列APIs与应用交互。

```bash
linera service
```

打开你的浏览器，在地址栏输入`http://localhost:8080`访问上面运行的节点服务器内置的GraphiQL用户界面([GraphQL](https://graphql.org/) IDE)。[后面的章节](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/node_service?id=graphiql-ide)我们会提供更多GraphiQL的使用细节，现在，我们通过下面的命令获得默认微链(e476…)上的应用列表：

```graphql
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

鉴于我们只部署了一个应用，上面的请求只返回一个结果。

上面的查询请求返回的应用列表中，每一个应用都包含`link`字段。将`link`字段的值填入到浏览器地址栏并确定，我们就可以与对应的应用交互了！

最后，执行下面的查询获得计数值：

```graphql
query {
  value
}
```

上面的查询请求将会返回`42`，这是我们在部署`counter`应用的时候设置的初始参数。
