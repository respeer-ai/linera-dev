# 加入现有的测试网

在本节中，我们使用 Docker Compose 运行一个验证者节点，并加入一个现有的测试网。

## 安装

本节涵盖了您需要安装以使用 Docker Compose 运行 Linera 验证节点的所有内容。

> 注意：本章节仅在Linux系统下测试通过。

### Docker Compose Requirements

安装Docker Compose请参考下列步骤
[installing Docker Compose](https://docs.docker.com/compose/install/) 

### 安装Linera工具链

要安装 Linera 工具链，请参考下列步骤
[installation section](../../developers/getting_started/installation.md#installing-from-github).

您想要从 GitHub 上安装工具链，因为您将使用该存储库来运行 Docker Compose 验证者服务。

## 设置 Linera 验证者

在下一节中，我们将在 `linera-protocol` 存储库的 `docker` 子目录中进行操作。

### 基础设施设置

通过 Docker Compose 运行的验证者节点并不附带预打包的负载均衡器来执行 TLS 终止（与在 Kubernetes 上运行的验证者节点不同）。

因此，需要验证者运营商提供TLS终止和支持 Linera 通知系统正常运行所需的长连接。

### 创建您的验证者配置

验证者是使用 TOML 文件进行配置的。 您可以使用以下模板来设置您自己的验证者配置：

```toml
server_config_path = "server.json"
host = "<your-host>" # e.g. my-subdomain.my-domain.net
port = 443
metrics_host = "proxy"
metrics_port = 21100
internal_host = "proxy"
internal_port = 20100
[external_protocol]
Grpc = "Tls"
[internal_protocol]
Grpc = "ClearText"

[[shards]]
host = "shard"
port = 19100
metrics_host = "shard"
metrics_port = 21100

```

### 创世配置

创世配置描述了网络创建时的验证者委员会和链。 验证者的功能需要它。

最初，每个测试网络的创世配置将存储在由 Linera Protocol 核心团队管理的公共存储桶中。

可以参考这个示例：

```bash
wget "https://storage.cloud.google.com/linera-io-dev-public/{{#include ../../../RELEASE_DOMAIN}}/genesis.json"
```

### Creating Your Keys

现在，已经创建了[验证者配置](joining.md#creating-your-validator-configuration)，并且[创世配置](joining.md#genesis-configuration)可用，可以生成验证者私钥。

要生成私钥，使用 `linera-server` 二进制文件：

```bash
linera-server generate --validators /path/to/validator/configuration.toml
```

这将生成一个名为 `server.json` 的文件，其中包含验证者运行所需的信息，包括密码学密钥对。

命令执行完成后，将打印公钥，例如：

```bash
$ linera-server generate --validators /path/to/validator/configuration.toml
2024-07-01T16:51:32.881255Z  INFO linera_version::version_info: Linera protocol: v0.12.0
2024-07-01T16:51:32.881273Z  INFO linera_version::version_info: RPC API hash: p//G+L8e12ZRwUdWoGHWYvWA/03kO0n6gtgKS4D4Q0o
2024-07-01T16:51:32.881274Z  INFO linera_version::version_info: GraphQL API hash: KcS5z1lEg+L9QjcP99l5vNSc7LfCwnwEsfDvMZGJ/PM
2024-07-01T16:51:32.881277Z  INFO linera_version::version_info: WIT API hash: p//G+L8e12ZRwUdWoGHWYvWA/03kO0n6gtgKS4D4Q0o
2024-07-01T16:51:32.881279Z  INFO linera_version::version_info: Source code: https://github.com/linera-io/linera-protocol/tree/44b3e1ab15 (dirty)
2024-07-01T16:51:32.881519Z  INFO linera_server: Wrote server config server.json
92f934525762a9ed99fcc3e3d3e35a825235dae133f2682b78fe22a742bac196 # <- Public Key
```

The public key, in this case beginning with `92f`, must be communicated to the
Linera Protocol core team along with the chosen host name for onboarding in the
next epoch.
在这种情况下以 `92f` 开头的公钥必须与选择的主机名一起在下一个时期的入职中与 Linera Protocol 核心团队沟通。

> 注意：在被列入下一个时期之前，验证者节点将不会从现有用户那里接收任何流量。

### 构建Linera Docker镜像

要构建Linera Docker镜像，请从`linera-protocol`存储库的根目录运行以下命令：

```bash
$ docker build -f docker/Dockerfile . -t linera
```

这一步需要几分钟时间。

### 运行一个验证者节点

现在，创世配置在`docker/genesis.json`中可用，服务器配置在`docker/server.json`中可用，可以通过在`docker`目录内运行以下命令来启动验证者：

```bash
cd docker && docker compose up -d
```

这将以分离模式运行Docker Compose部署。可能需要几分钟来下载并启动ScyllaDB镜像。
