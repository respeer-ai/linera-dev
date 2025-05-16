# 验证安装

要验证安装是否成功，可使用 `linera query-validator` 命令。例如：

```bash
$ linera wallet init --with-new-chain --faucet https://faucet.{{#include ../../../TESTNET_DOMAIN}}.linera.net
$ linera query-validator grpcs:my-domain.com:443

RPC API hash: kd/Ru73B4ZZjXYkFqqSzoWzqpWi+NX+8IJLXOODjSko
GraphQL API hash: eZqzuBlLT0bcoQUjOCPf2j22NfZUWG95id4pdlUmhgs
WIT API hash: 4/gsw8G+47OUoEWK6hJRGt9R69RanU/OidmX7OKhqfk
Source code: https://github.com/linera-io/linera-protocol/tree/

0cd20d06af5262540535347d4cc6e5952a921d1a6a7f6dd0982159c9311cfb3e
```

最后一行是网络创世配置的哈希值。如果此命令成功执行，说明您的验证节点已开始运行并准备好加入网络。
