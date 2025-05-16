# 使用 Docker Compose 运行开发者网络（Devnets）

在本节中，我们使用 Docker Compose 运行一个仅包含单个验证节点的简单开发者网络。

Docker Compose 是一个用于定义和管理多容器 Docker 应用程序的工具。它允许用户通过一个 YAML 文件（docker-compose.yml）描述应用程序的服务、网络和卷，并通过 docker-compose up 和 docker-compose down 等简单命令，以单一单元的形式轻松启动、停止和管理所有容器。

如需更完整的环境配置，建议使用[下一节](../../../zh_CN/operators/devnets/kind.md)将详细介绍的 Kind（Kubernetes in Docker）工具。

## 安装

本节介绍使用 Docker Compose 运行 Linera 网络所需的全部安装内容。

注意：本节内容仅在 Linux 系统上经过测试。

### Docker Compose 安装要求

要安装 Docker Compose，请参阅 Docker [官方文档](https://docs.docker.com/compose/install/) 中的安装说明。

### 安装 Linera 工具链

要安装 Linera 工具链，请参阅[安装章节](../../../zh_CN/developers/getting_started/installation.md#installing-from-github)。

你需要从GitHub安装工具链，因为后续需要通过该仓库来运行Docker Compose验证器服务。

## 使用 Docker Compose 运行

要通过Docker Compose运行本地开发者网络，请进入`linera-protocol`仓库的根目录并运行以下命令：

```bash
cd docker && ./compose.sh
```

此过程可能需要一些时间，因为需要从Linera源代码构建Docker镜像。服务就绪后，临时钱包和数据库将生成在`docker`子目录下。

通过`linera`二进制文件引用这些变量，您即可与开发者网络进行交互：

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


由于网络是临时性的，终止脚本将执行清理操作，销毁与该网络关联的钱包、存储和卷。
