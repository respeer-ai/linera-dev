# 部署应用

The first step to deploy your application is to configure a wallet. This will
determine where the application will be deployed: either to a local net or to
the public deployment (i.e. a devnet or a testnet).
部署应用程序的第一步需完成​​钱包配置​​，该配置将决定应用的部署目标环境：
本地网络​​或者    ​​公共网络​​（例如开发网络（devnet）或测试网络（testnet））。


## 本地网络

请按照​​[入门指南](../../../zh_CN/developers/getting_started/hello_linera.md#using-the-initial-test-wallet)​中的步骤完成本地网络配置。


完成本地网络配置后，需先设置 `LINERA_WALLET` 与 `LINERA_STORAGE` 环境变量，即可在执行 `publish-and-create` 命令部署应用时同步指定：

1. 合约字节码存储路径
2. 服务字节码存储路径
3. JSON编码初始化参数

```bash
linera publish-and-create \
  target/wasm32-unknown-unknown/release/my-counter_{contract,service}.wasm \
  --json-argument "42"
```

## 开发网络和测试网

在创建新微链时配置当前测试网钱包，可使用如下命令：

```bash
linera wallet init --with-new-chain --faucet https://faucet.{{#include ../../../RELEASE_DOMAIN}}.linera.net
```

Faucet 将为新创建的微链提供初始代币，这些代币可用于通过 `publish-and-create` 命令部署应用。需指定以下参数：

1. 合约字节码存储路径
2. 服务字节码存储路径
3. JSON编码初始化参数

```bash
linera publish-and-create \
  target/wasm32-unknown-unknown/release/my-counter_{contract,service}.wasm \
  --json-argument "42"
```

## 与应用交互

要与已部署的应用程序进行交互，需通过[node service](../../../zh_CN/developers/core_concepts/node_service.md)实现。
