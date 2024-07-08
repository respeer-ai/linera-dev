1. # 部署应用程序

   部署应用程序的第一步是配置钱包。这将决定应用程序部署到本地网络还是开发网络（devnet）。

   ## 本地网络

   要配置本地网络，请按照[入门指南部分](../getting_started/hello_linera.html#using-the-initial-test-wallet)中的步骤进行操作。

   配置完成后，需要设置 `LINERA_WALLET` 和 `LINERA_STORAGE` 环境变量，并且可以在 `publish-and-create` 命令中使用这些变量来部署应用程序，同时指定以下内容：

   1. 合约字节码的位置
   2. 服务字节码的位置
   3. JSON 编码的初始化参数

```bash
linera publish-and-create \
  target/wasm32-unknown-unknown/release/my-counter_{contract,service}.wasm \
  --json-argument "42"
```

## 开发网络（Devnet）

要为 devnet 配置钱包并创建新的微链，可以使用以下命令：

```bash
linera wallet init --with-new-chain --faucet https://faucet.{{#include ../../../RELEASE_DOMAIN}}.linera.net
```

1. Faucet 将为新的链提供一些代币，这些代币可以用于使用 `publish-and-create` 命令部署应用程序。在部署时需要指定以下内容：
   1. 合约字节码的位置
   2. 服务字节码的位置
   3. JSON 编码的初始化参数

```bash
linera publish-and-create \
  target/wasm32-unknown-unknown/release/my-counter_{contract,service}.wasm \
  --json-argument "42"
```

## 与应用程序交互

要与部署的应用程序进行交互，必须使用[node service](../core_concepts/node_service.html)。