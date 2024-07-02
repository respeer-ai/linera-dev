# 钱包

与传统的区块链类似，Linera 钱包负责保存用户的私钥。但是，与其签署交易不同，Linera 钱包旨在签署区块并提议将其扩展到其用户拥有的链中。

实际上，钱包包含一个节点，该节点跟踪 Linera 链的子集。在下一节中，我们将看到 Linera 钱包如何运行 GraphQL 服务，将其链的状态暴露给 Web 前端。

> 命令行工具 `linera` 是开发者与 Linera 网络交互并管理本地系统中存在的用户钱包的主要方式。

请注意，这个命令行工具主要用于开发目的。我们的目标是最终使最终用户可以在 [浏览器扩展](overview.html#web3-sdk) 中管理他们的钱包。

## 选择钱包

钱包的私有状态通常存储在 `wallet.json` 文件中，而其节点的状态存储在 `linera.db` 文件中。

要在不同的钱包之间切换，您可以使用 `linera` 工具的 `--wallet` 和 `--storage` 选项，例如： `linera --wallet wallet2.json --storage rocksdb:linera2.db`。

您也可以定义环境变量 `LINERA_STORAGE` 和 `LINERA_WALLET` 以达到同样的效果。例如，`LINERA_STORAGE=$PWD/wallet2.json` 和 `LINERA_WALLET=$PWD/wallet2.json`。

最后，如果为某个编号 `I` 定义了 `LINERA_STORAGE_$I` 和 `LINERA_WALLET_$I`，您可以调用 `linera --with-wallet $I`（或简写为 `linera -w $I`）。

## 链管理

### 列出链

要列出您的钱包中存在的链，您可以使用 `show` 命令：

```bash
linera wallet show
╭──────────────────────────────────────────────────────────────────┬──────────────────────────────────────────────────────────────────────────────────────╮
│ Chain ID                                                         ┆ Latest Block                                                                         │
╞══════════════════════════════════════════════════════════════════╪══════════════════════════════════════════════════════════════════════════════════════╡
│ 668774d6f49d0426f610ad0bfa22d2a06f5f5b7b5c045b84a26286ba6bce93b4 ┆ Public Key:         3812c2bf764e905a3b130a754e7709fe2fc725c0ee346cb15d6d261e4f30b8f1 │
│                                                                  ┆ Owner:              c9a538585667076981abfe99902bac9f4be93714854281b652d07bb6d444cb76 │
│                                                                  ┆ Block Hash:         -                                                                │
│                                                                  ┆ Timestamp:          2023-04-10 13:52:20.820840                                       │
│                                                                  ┆ Next Block Height:  0                                                                │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┤
│ 91c7b394ef500cd000e365807b770d5b76a6e8c9c2f2af8e58c205e521b5f646 ┆ Public Key:         29c19718a26cb0d5c1d28102a2836442f53e3184f33b619ff653447280ccba1a │
│                                                                  ┆ Owner:              efe0f66451f2f15c33a409dfecdf76941cf1e215c5482d632c84a2573a1474e8 │
│                                                                  ┆ Block Hash:         51605cad3f6a210183ac99f7f6ef507d0870d0c3a3858058034cfc0e3e541c13 │
│                                                                  ┆ Timestamp:          2023-04-10 13:52:21.885221                                       │
│                                                                  ┆ Next Block Height:  1                                                                │
╰──────────────────────────────────────────────────────────────────┴──────────────────────────────────────────────────────────────────────────────────────╯

```

每一行表示钱包中存在的一条链。左边是链的唯一标识符，右边是关联的最新区块的元数据。

### 默认链

每个钱包都有一个默认链，所有命令都将应用于该链，除非您在命令行上指定另一个 `--chain`。

默认链在首次向钱包添加链时设置。您可以运行以下命令来检查您钱包的默认链：

```bash
linera wallet show
```

默认链会以绿色文本显示，而不是白色文本。

要更改钱包的默认链，请使用 `set-default` 命令：

```bash
linera wallet set-default <chain-id>
```

### 开放链

Linera 协议定义了创建新链的语义，我们称之为 "开放链"。一条链不能在真空中打开，它需要由网络上的现有链创建。

#### 为自己的钱包开放链

要为自己的钱包开放链，您可以使用 `open-chain` 命令：

```bash
linera open-chain
```

这将使用钱包的默认链创建一个新链，并将其添加到钱包中。使用 `wallet show` 命令查看您现有的链。

#### 为另一个钱包开放链

为另一个 `wallet` 开放链需要额外的两个步骤。首先让我们初始化第二个钱包：

```bash
linera --wallet wallet2.json --storage rocksdb:linera2.db wallet init --genesis target/debug/genesis.json
```

首先，`wallet2` 必须创建一个未分配的密钥对。该密钥对的公共部分然后发送给作为链创建者的 `wallet`。

```bash
linera --wallet wallet2.json keygen
6443634d872afbbfcc3059ac87992c4029fa88e8feb0fff0723ac6c914088888 # this is the public key for the unassigned keypair
```

接下来，使用公钥，`wallet` 可以为 `wallet2` 开放链。

```bash
linera open-chain --to-public-key 6443634d872afbbfcc3059ac87992c4029fa88e8feb0fff0723ac6c914088888
e476187f6ddfeb9d588c7b45d3df334d5501d6499b3f9ad5595cae86cce16a65010000000000000000000000
fc9384defb0bcd8f6e206ffda32599e24ba715f45ec88d4ac81ec47eb84fa111
```

第一行是指定创建新链的跨链消息的消息 ID。第二行是新链的 ID。

最后，要为给定未分配密钥对的 `wallet2` 添加链，我们使用 `assign` 命令：

```bash
 linera --wallet wallet2.json assign --key 6443634d872afbbfcc3059ac87992c4029fa88e8feb0fff0723ac6c914088888 --message-id e476187f6ddfeb9d588c7b45d3df334d5501d6499b3f9ad5595cae86cce16a65010000000000000000000000
```

#### 使用多个用户开放链

`open-chain` 命令是 `open-multi-owner-chain` 的简化版本，后者允许您对新链的所有者集合、类型以及轮次和轮次超时设置进行精细控制。例如，以下命令创建一个具有两个所有者和两个多领导轮次的链：

```bash
linera open-multi-owner-chain \
    --chain-id e476187f6ddfeb9d588c7b45d3df334d5501d6499b3f9ad5595cae86cce16a65010000000000000000000000 \
    --owner-public-keys 6443634d872afbbfcc3059ac87992c4029fa88e8feb0fff0723ac6c914088888 \
                        ca909dcf60df014c166be17eb4a9f6e2f9383314a57510206a54cd841ade455e \
    --multi-leader-rounds 2
```

`change-ownership` 命令为现有链提供了相同的选项，以添加或删除所有者并更改轮次设置。

## 使用 `linera net up` 自动设置额外的钱包

在测试时，与其像上面那样使用 `linera open-chain` 和 `linera assign`，通常更方便的是将选项 `--extra-wallets N` 传递给 `linera net up`。

此选项将创建 `N` 个额外的用户钱包，并输出 Bash 命令来定义环境变量 `LINERA_{WALLET,STORAGE}_$I`，其中 `I` 在 `0..=N` 范围内（`I=0` 是初始链的钱包）。

一旦所有环境变量都定义好，您可以使用 `linera --with-wallet I` 或 `linera -w I` 简写来在不同的钱包之间切换。

## 在 Bash 中自动化

为了在创建本地测试网络后自动设置 `LINERA_WALLET*` 和 `LINERA_STORAGE*` 变量的过程，我们提供了一个 Bash 辅助函数 `linera_spawn_and_read_wallet_variables`。

要在您的 shell 中定义 `linera_spawn_and_read_wallet_variables` 函数，请运行 `source /dev/stdin <<<"$(linera net helper 2>/dev/null)"`。您也可以将 `linera net helper` 的输出添加到您的 `~/.bash_profile` 中，以备将来使用。

一旦函数定义好了，调用 `linera_spawn_and_read_wallet_variables linera net up` 而不是 `linera net up`。

这些是 Linera 钱包和链管理的基本概念和操作。如果您有任何进一步的问题或需要进一步的帮助，请随时告诉我！