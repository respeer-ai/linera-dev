# 钱包

与传统区块链的钱包相同，Linera钱包也负责管理用户私钥和签名交易，但除此之外，Linera钱包也会创建区块，并将区块提交到验证器来延伸钱包用户管理的微链。

在实现上，Linera钱包将会包含一个节点服务，该节点服务管理Linera微链的一个子集(译者注：只管理当前钱包用户拥有的微链)。[下一节](zh_CN/developers/core_concepts/node_service)中我们将会看到通过Linera钱包提供的GraphQL服务将钱包管理的微链状态提供给Web前端。

> 开发者主要通过命令行工具`linera`与Linera网络交互，并管理本地用户钱包。

命令行工具主要针对开发者，对于最终用户，我们将会提供[浏览器插件](zh_CN/developers/core_concepts/overview.md#Web3-SDK)来管理他们的钱包。

## [选择钱包](zh_CN/developers/core_concepts/wallets.md#选择钱包)

按照惯例，我们将钱包的私有状态存储在`wallet.json`文件，节点的状态存储在`linera.db`文件。

开发者可以使用`linera`工具的`--wallet`和`--storage`选项切换钱包，例如`linera --wallet wallet2.json --storage rocksdb:linera2.db`。

除了通过命令行选项，定义不同的`LINERA_STORAGE`和`LINERA_WALLET`环境变量也可以达到一样的效果，例如`LINERA_STORAGE=$PWD/wallet2.json`和`LINERA_WALLET=$PWD/wallet2.json`。

最后，如果终端中定义了一系列以数字`I`结尾的环境变量`LINERA_STORAGE_$I`和`LINERA_WALLET_$I`，也可以通过执行`linera --with-wallet $I`(或者简写`linera -w $I`)使用不同的钱包。

## [管理微链](zh_CN/developers/core_concepts/wallets.md#管理微链)

### 获取微链清单

使用`show`命令可以获取钱包中的微链清单：

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

上面的清单中每一行代表钱包中的一条微链，左边是微链的标识符，右边是微链最新区块相关的元数据。

### [默认微链](zh_CN/developers/core_concepts/wallets.md#默认微链)
每个钱包都有一条默认微链，如果执行命令时未传递`--chain`参数，命令将使用默认微链。

第一条添加到钱包中的微链将被设置成默认微链，可以通过如下命令查看钱包的默认微链：

```bash
linera wallet show
```

其中微链ID为绿色的行为钱包默认微链。可以通过如下命令改变默认微链：

```bash
linera wallet set-default <chain-id>
```

### 创建微链

Linera协议实现了创建新微链的语义，称为"opening a chain"。微链需要从网络上已经存在的微链创建，不能无中生有。

#### 在你的钱包中创建微链

执行`open-chain`命令可以在你的钱包中创建新微链：

```bash
linera open-chain
```

上面的命令将在默认微链上创建一条新微链，并将其添加到钱包中。执行`linera wallet show`命令可以查看当前所有微链。

#### 在另一个钱包创建微链
在另一个`钱包`创建新的微链需要两个额外的步骤。首先，需要初始化第二个钱包：

```bash
linera --wallet wallet2.json --storage rocksdb:linera2.db wallet init --genesis target/debug/genesis.json
```

`钱包2`需要使用未分配的密钥对创建，该密钥对的公钥将被发送给`钱包1`的创建者。

```bash
linera --wallet wallet2.json keygen
6443634d872afbbfcc3059ac87992c4029fa88e8feb0fff0723ac6c914088888 # 上面提到未分配密钥对的公钥
```

接下来，`钱包1`使用该公钥即可为`钱包2`创建一条微链。

```bash
linera open-chain --to-public-key 6443634d872afbbfcc3059ac87992c4029fa88e8feb0fff0723ac6c914088888
e476187f6ddfeb9d588c7b45d3df334d5501d6499b3f9ad5595cae86cce16a65010000000000000000000000
fc9384defb0bcd8f6e206ffda32599e24ba715f45ec88d4ac81ec47eb84fa111
```

上面的命令中，第一行为命令行输入，第二行和第三行为执行结果。其中，第二行为创建新微链的消息ID，第三行为新微链的ID。

然后，我们使用`assign`命令将新创建的微链添加到`钱包2`，并将其所有者设置为上面提到的公钥：

```bash
 linera --wallet wallet2.json assign --key 6443634d872afbbfcc3059ac87992c4029fa88e8feb0fff0723ac6c914088888 --message-id e476187f6ddfeb9d588c7b45d3df334d5501d6499b3f9ad5595cae86cce16a65010000000000000000000000
```

#### 创建有多个用户的微链

`open-chain` 命令是 `open-multi-owner-chain` 的简化版本，后者允许创建者对链所有者、Round和超时进行更加精细的设定。下面的示例命令将创建一条具有两个所有者及两个多领导者Rounds的微链：

```bash
linera open-multi-owner-chain \
    --chain-id e476187f6ddfeb9d588c7b45d3df334d5501d6499b3f9ad5595cae86cce16a65010000000000000000000000 \
    --owner-public-keys 6443634d872afbbfcc3059ac87992c4029fa88e8feb0fff0723ac6c914088888 \
                        ca909dcf60df014c166be17eb4a9f6e2f9383314a57510206a54cd841ade455e \
    --multi-leader-rounds 2
```

`change-ownership`命令使用相同参数，可以移除运行中的微链所有者，及修改微链的Round设置。

## 使用`linera net up`命令自动创建外部钱包

开发者测试的时候，除了通过上面的`linera open-chain`和`linera assign`创建新钱包和微链，也可以通过更加方便的`linera net up`命令创建测试环境，如果需要多个钱包测试，创建时传递`--extra-wallets N`参数即可。该参数将会创建`N`个额外的用户钱包，并将访问不同钱包的环境变量`LINERA_{WALLET,STORAGE}_$I`打印到终端(`I`的范围为`0..=N`，`I=0`表示管理初始微链的钱包)。

环境都准备就绪后，即可通过执行`linera --with-wallet I`，或者简写的`linera -w I`切换钱包。

## [Bash环境自动设置](zh_CN/developers/core_concepts/wallets.md#Bash环境自动设置)

我们提供了Bash帮助函数`linera_spawn_and_read_wallet_variables`，用于在本地测试网络创建完毕后自动设置`LINERA_WALLET*`和`LINERA_STORAGE*`环境变量。

在终端中执行`source /dev/stdin <<<"$(linera net helper 2>/dev/null)"`，即可在当前终端定义`linera_spawn_and_read_wallet_variables`函数。此外，开发者也可以通过将`linera net helper`添加到`~/.bash_profile`文件中，为将来登录的用户自动设置终端测试环境。

当按照上述步骤定义好帮助函数，即可执行`linera_spawn_and_read_wallet_variables linera net up`来创建测试环境。
