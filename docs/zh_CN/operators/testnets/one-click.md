## 一键部署

在下载 `linera-protocol` 代码仓库并切换到 `testnet_babbage` 测试分支后，可通过运行命令 `scripts/deploy-validator.sh <hostname>` 部署 Linera 验证节点。

例如：

```bash
$ git fetch origin
$ git checkout -t origin/testnet_babbage
$ scripts/deploy-validator.sh linera.mydomain.com --remote-image
```

部署将自动监听新的镜像更新并自动拉取。

> 注：如果您不指定 --remote-image，系统将默认从源代码构建镜像。

命令执行完成后，系统会输出公钥和账户密钥，例如：

```bash
$ scripts/deploy-validator.sh linera.mydomain.com --remote-image
...
Public Key: 02a580bbda90f0ab10f015422d450b3e873166703af05abd77d8880852a3504e4d,009b2ecc5d39645e81ff01cfe4ceeca5ec207d822762f43b35ef77b2367666a7f8
```

在这种情况下，公钥和账户密钥分别以 `02a` 和 `009` 开头，必须连同所选的主机名一起提交给 Linera 协议核心团队，以便在下一个 epoch 加入网络。

如需更定制化的部署方案，请参考下方手动安装说明。


> 注意：如果之前已部署过验证节点，可能需要删除旧的Docker卷（`docker_linera-scylla-data` 和 `docker_linera-shared`）。
