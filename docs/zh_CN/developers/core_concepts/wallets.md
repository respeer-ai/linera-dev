# 钱包

与传统区块链类似，Linera钱包负责管理用户私钥。但不同于交易签名机制，Linera钱包的核心功能是区块签名与链扩展提案——通过向所属用户的链提交新区块实现链上数据延伸。

实践中，钱包内嵌节点用于追踪用户授权访问的Linera链子集。[后续章节](../../../zh_CN/developers/core_concepts/node_service.md)将详述如何通过GraphQL服务公开链上状态，实现前端网页与链状态的实时交互。

> 命令行工具`linera`是开发者与Linera网络交互及管理本地系统部署的开发者钱包的核心途径。

需注意，此命令行工具主要面向开发场景使用。我们的目标在于，终端用户最终可通过浏览器扩展实现钱包自主管理。

## 创建开发者钱包创建开发者钱包

使用`linera`命令行工具创建钱包的最简方法是请运行以下命令：

```bash
linera wallet init --with-new-chain --faucet $FAUCET_URL
```

其中 `$FAUCET_URL` 表示网络水龙头的访问地址（参见[前一节](../../../zh_CN/developers/getting_started/hello_linera.md)）

## 选择钱包

钱包的私有状态默认存储于 `wallet.json` 文件，而节点状态则存储于 `linera.db` 文件。

可通过 `linera` 工具的 `--wallet` 和 `--storage` 参数切换钱包，例如：
`linera --wallet wallet2.json --storage rocksdb:linera2.db`

亦可定义环境变量 `LINERA_STORAGE` 和 `LINERA_WALLET` 实现相同效果，例如：
`LINERA_STORAGE=$PWD/wallet2.json`和`LINERA_WALLET=$PWD/wallet2.json`

若为索引 $I 定义了 LINERA_STORAGE_$I 和 LINERA_WALLET_$I，可通过以下方式调用：
`linera --with-wallet $I  # 或简写为 linera -w $I`

## 链管理

### 链列表

要查看您钱包中的链列表，可使用 `show` 命令：

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

每行对应钱包中的一条链。左侧为链的唯一标识符，右侧为该链最新区块关联的元数据。

### 默认链

每个钱包均存在一个默认链，所有命令默认作用于该链，除非您在命令行中另行指定其他 `--chain` 参数。

默认链的设置发生于钱包首次添加链时。您可通过运行以下命令查看当前钱包的默认链：

```bash
linera wallet show
```

以绿色文本显示（而非白色）的链ID即为您的默认链。

要更改钱包的默认链，请使用 `set-default` 命令：

```bash
linera wallet set-default <chain-id>
```

### 创建链

在Linera协议中，链通常通过现有链的交易创建。

#### ​为您的钱包从现有链创建新链​​

要从钱包的默认链创建新链，可使用 `open-chain` 命令：

```bash
linera open-chain
```

此操作将创建新链并将其添加至钱包。使用 `wallet show` 命令查看现有链列表。

#### 为另一个钱包从现有链创建新链​​

为其他`钱包`创建链需额外执行两个步骤。下面初始化第二个钱包：

```bash
linera --wallet wallet2.json --storage rocksdb:linera2.db wallet init --faucet $FAUCET_URL
```

First `wallet2` must create an unassigned keypair. The public part of that
keypair is then sent to the `wallet` who is the chain creator.
首先，`wallet2`需创建未分配的密钥对。随后将该密钥对的公钥部分发送至作为链创建者的`钱包`。

```bash
linera --wallet wallet2.json keygen
6443634d872afbbfcc3059ac87992c4029fa88e8feb0fff0723ac6c914088888 # this is the public key for the unassigned keypair
```

接下来，使用公钥，`钱包`可为`wallet2`创建链。

```bash
linera open-chain --to-public-key 6443634d872afbbfcc3059ac87992c4029fa88e8feb0fff0723ac6c914088888
e476187f6ddfeb9d588c7b45d3df334d5501d6499b3f9ad5595cae86cce16a65010000000000000000000000
fc9384defb0bcd8f6e206ffda32599e24ba715f45ec88d4ac81ec47eb84fa111
```

首行是用于指定创建新链的跨链消息的消息ID。次行为新链的链ID。

最终，要将链添加至`wallet2`并关联指定的未分配密钥，我们使用`assign`命令：

```bash
 linera --wallet wallet2.json assign --key 6443634d872afbbfcc3059ac87992c4029fa88e8feb0fff0723ac6c914088888 --message-id e476187f6ddfeb9d588c7b45d3df334d5501d6499b3f9ad5595cae86cce16a65010000000000000000000000
```

需注意，在配备水龙头的测试网络中，新钱包与新链亦可直接通过水龙头使用以下方式创建：

```bash
linera --wallet wallet2.json --storage rocksdb:linera2.db wallet init --with-new-chain --faucet $FAUCET_URL
```

#### 为多用户开设链​​

`open-chain` 命令是 `open-multi-owner-chain` 的简化版本，可对新建链的所有者集合、轮次类型及轮次超时设置进行细粒度控制。例如：此命令可创建包含两个所有者和两个多领导者轮次的链。

```bash
linera open-multi-owner-chain \
    --chain-id e476187f6ddfeb9d588c7b45d3df334d5501d6499b3f9ad5595cae86cce16a65010000000000000000000000 \
    --owner-public-keys 6443634d872afbbfcc3059ac87992c4029fa88e8feb0fff0723ac6c914088888 \
                        ca909dcf60df014c166be17eb4a9f6e2f9383314a57510206a54cd841ade455e \
    --multi-leader-rounds 2
```

`change-ownership` 命令提供与 open-chain 相同的功能选项，支持对现有链进行所有者增删操作及轮次参数调整。