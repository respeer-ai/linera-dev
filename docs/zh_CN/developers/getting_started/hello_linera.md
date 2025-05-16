# Hello, Linera

本节将指导您完成开发者钱包初始化，测试网交互，本地开发网络搭建，应用编译部署的全流程操作。

本节结束时，您将在测试网或本地网络部署就绪[微链](../../../zh_CN/developers/core_concepts/microchains.md)，并拥有支持GraphQL查询的可用应用。

## 在最新测试网创建钱包

要在最新测试网进行交互，您需要开发者钱包、新微链及测试代币。这些资源可通过测试网**水龙头**服务一次性获取，具体操作如下：

```bash
linera wallet init --with-new-chain --faucet https://faucet.{{#include ../../../RELEASE_DOMAIN}}.linera.net
```

若出现错误提示，请确保使用的[Linera工具链与当前测试网版本兼容](../../../zh_CN/developers/getting_started/installation.md#installing-from-cratesio)。

> Linera测试网是Linera协议的测试部署环境。该部署包含多个[验证节点](../../../zh_CN/developers/advanced_topics/validators.md)，每个节点运行：
> 一个前端服务（亦称`linera-proxy`），
> 工作节点集群（亦称`linera-server`），
> 共享数据库（默认使用`linera-storage-service`）。

## 本地测试网络

另一个选项是启动本地开发网络。要这样做，请运行以下命令：

```bash
linera net up --with-faucet --faucet-port 8080
```

这将启动验证节点（默认分片数量）并开启水龙头服务。

现在，我们准备通过在独立终端中执行以下命令创建开发者钱包：

```bash
linera wallet init --with-new-chain --faucet localhost:8080
```

```admonish warn
钱包仅在所属网络的生命周期内有效。每次重启本地网络时，需删除原有钱包并重新创建新钱包。
```

## 同时操作多个开发者钱包及多网络环境

默认情况下，`linera`命令会在操作系统决定的配置路径中查找钱包文件。若需自定义钱包文件存储位置，可按如下方式设置环境变量`LINERA_WALLET`与`LINERA_STORAGE`：

```bash
DIR=$HOME/my_directory
mkdir -p $DIR
export LINERA_WALLET=$DIR/wallet.json"
export LINERA_STORAGE="rocksdb:$DIR/linera.db"
```

选择此类目录有助于管理多网络环境，因为钱包始终仅绑定于其创建时所在的网络。

> 我们之所以将通过`linera`命令行工具创建的钱包称为"开发者钱包"，是因为它们通过开发者工具操作且仅面向测试及开发环境使用。

> 生产级用户钱包通常通过浏览器扩展、移动应用程序或硬件设备进行管理。


## 与Linera网络进行交互

为验证网络运行状态，可执行链同步操作并查看链上余额，具体方法如下：

```bash
linera sync
linera query-balance
```

.您应当看到输出数值，例如：10。

## 构建示例应用

Linera链上应用均为[Wasm](https://webassembly.org/)字节码形态。每个验证节点及客户端均内置Wasm虚拟机（VM），可执行字节码指令。

让我们从[Linera测试网分支](https://github.com/linera-io/linera-protocol/tree/testnet_babbage)的`examples/`子目录构建`counter`应用：

```bash
cd examples/counter && cargo build --release --target wasm32-unknown-unknown
```

## 部署您的应用

您可通过`Linera`客户端的`publish-and-create`命令，在本地网络部署字节码并创建应用，具体操作如下：

1. 合约字节码存储路径
2. 服务字节码存储路径
3. JSON编码的初始化参数

```bash
linera publish-and-create \
  ../target/wasm32-unknown-unknown/release/counter_{contract,service}.wasm \
  --json-argument "42"
```

恭喜！您已在Linera上成功部署首个应用！

## 查询应用

现在让我们通过查询应用获取当前计数器数值。为此，需使用运行于[服务模式](../../../zh_CN/developers/core_concepts/node_service.md)的客户端。该模式将在本地暴露一组API，可用于与网络上的应用进行交互。

```bash
linera service
```

<!-- TODO: add graphiql image here -->

在浏览器中访问 `http://localhost:8080` 以使用 [GraphiQL](https://graphql.org)（GraphQL 集成开发环境）。该工具将在[后续章节](../../../zh_CN/developers/core_concepts/node_service.md#graphiql-ide)详细说明；当前请通过运行以下命令，列出默认链上已部署的应用：

```gql
query {
  applications(chainId: "...") {
    id
    description
    link
  }
}
```

其中`...`需替换为通过`linera wallet show`命令显示的链ID。

由于我们仅部署了一个应用，返回结果将包含单个条目。

在返回的JSON底部存在`link`字段。要与应用交互，请将此链接复制并粘贴至新浏览器标签页。

最后，若需查询计数器数值，请运行以下命令：

```gql
query {
  value
}
```

这将返回数值`42`，即我们部署应用时设置的初始化参数。
