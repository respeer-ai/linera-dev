# 使用Docker Compose运行开发网络

本章我们将介绍怎么使用Docker Compose运行只包含一个验证者的本地开发网络。

Docker Compose是一个用于编排和管理多容器Docker应用的工具。开发者可以在一个YAML文件(docker-compose.yaml)中描述应用使用的服务、网络和存储卷。在Docker Compose的帮助下，开发者可以通过像`docker-compose up`和`docker-compose down`这样的简单命令统一启动、停止和管理应用的所有容器。

如果开发者需要更加完整的设置，请考虑使用下一章介绍的Kind。

## 安装

本节涵盖了运行Linera网络的所有Docker Compose步骤。

注意：本部分仅在Linux环境下测试通过。

### Docker Compose要求

安装Docker Compose参阅Docker文档的[安装 Docker Compose](https://docs.docker.com/compose/install/)部分。

### 安装Linera工具链

安装Linera工具链参阅本文档[安装部分](../../developers/getting_started/installation.md#installing-from-github)。

由于Docker Compose的安装脚本包含在Linera的Github仓库中，开发者需要从Github安装Linera工具链。

## 使用Docker Compose运行

使用Docker Compose运行本地开发网络需要先clone linera-protocol源码库，然后进入源码clone目录，执行以下命令：

```bash
## 应该确保主机安装了protoc，rust等工具链
cargo install --path linera-service
## 参考[Docker Compose error](https://github.com/docker/compose/issues/8630#issuecomment-1073166114)解决docker compose报错
cd docker && ./compose.sh
```

从源码构建Docker镜像将会会费一些时间。当服务就绪后，将在docker子目录创建临时钱包和数据库。

开发者可以通过`linera`命令行和开发网络交互：

```bash
$ linera --wallet wallet.json --storage rocksdb:linera.db sync
2024-06-07T14:19:32.751359Z  INFO linera: Synchronizing chain information
2024-06-07T14:19:32.771842Z  INFO linera::client_context: Saved user chain states
2024-06-07T14:19:32.771850Z  INFO linera: Synchronized chain information in 20 ms
$ linera --wallet wallet.json --storage rocksdb:linera.db query-balance
2024-06-07T14:19:36.958149Z  INFO linera: Evaluating the local balance of e476187f6ddfeb9d588c7b45d3df334d5501d6499b3f9ad5595cae86cce16a65 by staging execution of known incoming messages
2024-06-07T14:19:36.959481Z  INFO linera: Balance obtained after 1 ms
10.
```

本章部署的开发网络是临时的，停止脚本将会清理与网络相关的钱包、数据库和存储卷。

