# 3.6. 部署应用

要部署应用，首先需要配置一个钱包。不同的钱包配置决定应用将被部署在什么地方：本地测试网，还是开发网。

## [本地测试网](https://linera-dev.respeer.ai/#/zh_CN/sdk/deploy?id=local-net)

首先，请参考[开始章节](https://linera-dev.respeer.ai/#/zh_CN/getting_started/hello_linera?id=using-the-initial-test-wallet)配置本地测试网。

然后，我们需要设置`LINERA_WALLET`和`LINERA_STORAGE`环境变量，这些变量将会在执行`publish-and-create`命令时用到。此外，我们还需要指定下列参数：

1. 合约字节码路径
2. 服务字节码路径
3. 初始化参数的JSON文本

```bash
linera publish-and-create \
  target/wasm32-unknown-unknown/release/my-counter_{contract,service}.wasm \
  --json-argument "42"
```

## [开发网](https://linera-dev.respeer.ai/#/zh_CN/sdk/deploy?id=devnet)

如果想在开发网上创建微链，可以使用如下的命令配置钱包：

```bash
linera wallet init --with-new-chain --faucet https://faucet.devnet.linera.net
```

开发网水龙头将创建一条新的微链，该微链有一些token，可以用于执行`publish-and-create`命令在开发网部署应用。在开发网部署命令同样需要指定下列参数：

1. 合约字节码路径
2. 服务字节码路径
3. 初始化参数的JSON文本

```bash
linera publish-and-create \
  target/wasm32-unknown-unknown/release/my-counter_{contract,service}.wasm \
  --json-argument "42"
```

## [与应用交互](https://linera-dev.respeer.ai/#/zh_CN/sdk/deploy?id=interacting-with-the-application)

其他用户通过[节点服务](https://linera-dev.respeer.ai/#/zh_CN/core_concepts/node_service)与应用交互。
