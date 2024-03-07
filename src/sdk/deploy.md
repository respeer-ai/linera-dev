# 3.6. Deploying the Application

The first step to deploy your application is to configure a wallet. This will determine where the application will be deployed: either to a local net or to the devnet.

## [Local Net](https://linera.dev/sdk/deploy.html#local-net)

To configure the local network, follow the steps in the [Getting Started section](https://linera.dev/getting_started/hello_linera.html#using-the-initial-test-wallet).

Afterwards, the `LINERA_WALLET` and the `LINERA_STORAGE` environment variables should be set and can be used in the `publish-and-create` command to deploy the application while also specifying:

1. The location of the contract bytecode
2. The location of the service bytecode
3. The JSON encoded initialization arguments

```bash
linera publish-and-create \
  target/wasm32-unknown-unknown/release/my-counter_{contract,service}.wasm \
  --json-argument "42"
```

## [Devnet](https://linera.dev/sdk/deploy.html#devnet)

To configure the wallet for the devnet while creating a new microchain, the following command can be used:

```bash
linera wallet init --with-new-chain --faucet https://faucet.devnet.linera.net
```

The Faucet will provide the new chain with some tokens, which can then be used to deploy the application with the `publish-and-create` command. It requires specifying:

1. The location of the contract bytecode
2. The location of the service bytecode
3. The JSON encoded initialization arguments

```bash
linera publish-and-create \
  target/wasm32-unknown-unknown/release/my-counter_{contract,service}.wasm \
  --json-argument "42"
```

## [Interacting with the Application](https://linera.dev/sdk/deploy.html#interacting-with-the-application)

To interact with the deployed application, a [node service](https://linera.dev/core_concepts/node_service.html) must be used.