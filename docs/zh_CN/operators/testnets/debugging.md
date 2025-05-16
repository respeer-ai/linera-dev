# 调试

本节介绍验证节点部署中可能遇到的潜在问题及其解决方法。

## 常见问题

以下列举了 Linera 鉴权器部署中常见的几种情况及其解决方法。

### `shard-init`进程卡住

`shard-init` 进程负责初始化数据库和分片

验证节点内置的 ScyllaDB 数据库在初始化时需要进行性能检查并根据底层硬件进行自我调优，这一过程约需10分钟。

如果 shard-init 进程在等待 10 分钟后仍然卡住，通常与以下原因相关：

1. 异步I/O上下文中允许的事件数量不足，解决方案请参阅[此处](../../../zh_CN/operators/testnets/requirements.md#scylladb-configuration)。
2. 旧部署产生的残留卷不会被 Docker 自动清理（即使通过 `docker compose down` 或 `docker system prune -a` 删除旧部署），需手动通过 `docker volume rm ...` 命令移除。

如果上述两种修复方法均未能解决问题，则需进一步检查日志。

### `​拉取访问被拒绝​​`

在部署验证节点时，你可以自行构建Docker镜像，或使用Linera团队提供的预构建远程镜像。

Docker Compose清单文件会查找`LINERA_IMAGE`环境变量（通常由默认脚本设置），若未找到则默认使用`linera`值，默认假设该镜像已存在于本地。

要解决此问题，要么明确指定 `LINERA_IMAGE` 环境变量，要么确保该[镜像已在本地构建](../../../zh_CN/operators/testnets/manual-installation.md#building-the-linera-docker-image)。

### `对 genesis.json 的访问被拒绝`

当创世配置URL因字符串格式错误导致格式不正确时会出现此问题。部署脚本使用当前分支名称生成URL，请确保已检出 `testnet_babbage` 分支。

## 支持

有关支持及与核心团队的沟通，请通过 [Linera Discord](https://discord.com/invite/linera) 中的 `#validator` 私有频道进行。

如果通过查阅本文档仍未能解决任何未决问题，请通过此处联系我们。
