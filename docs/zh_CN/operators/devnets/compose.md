# 使用 Docker Compose 运行开发网络

在这一部分，我们使用 Docker Compose 来运行一个简单的开发网络，只包含一个验证者。

Docker Compose 是一个用于定义和管理多容器 Docker 应用程序的工具。它允许您在一个单独的 YAML 文件（docker-compose.yml）中描述应用程序的服务、网络和卷。使用 Docker Compose，您可以轻松地启动、停止和管理应用程序中的所有容器，只需使用类似 `docker-compose up` 和 `docker-compose down` 的简单命令。

对于更完整的设置，请考虑使用 Kind，如下一节所述。

## 安装

本节涵盖了运行 Linera 网络所需的所有 Docker Compose 安装步骤。

注意：此部分仅在 Linux 环境下进行过测试

### Docker Compose 要求

要安装 Docker Compose，请参阅 Docker 文档中的 [安装 Docker Compose](https://docs.docker.com/compose/install/) 部分。

### 安装 Linera 工具链

要安装 Linera 工具链，请参考 [安装部分](../../developers/getting_started/installation.md#installing-from-github)。

您需要从 GitHub 安装工具链，因为将使用该存储库来运行 Docker Compose 验证者服务。

## 使用 Docker Compose 运行

要使用 Docker Compose 在本地运行开发网络，请导航到 `linera-protocol` 存储库的根目录，并执行以下命令：

```bash
cd docker && ./compose.sh
```

这将花费一些时间，因为需要从 Linera 源代码构建 Docker 镜像。当服务准备就绪时，临时钱包和数据库将位于 `docker` 子目录下。

使用 `linera` 二进制文件与开发网络进行交互：

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

由于网络是临时的，因此停止脚本会执行清理操作，销毁与网络相关的钱包、存储和卷。