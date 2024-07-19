# 加入测试网

本章我们将使用Docker Compose运行一个验证者节点，并接入运行中的测试网。

## 安装

本小节涵盖了使用Docker Compose运行Linera验证节点的所有内容。

> 注意：下列步骤仅在Linux系统下测试通过。

### Docker Compose需求

参考[安装Docker Compose](https://docs.docker.com/compose/install/)安装Docker Compose

### 安装Linera工具链

参考[Github安装章节](../../developers/getting_started/installation.md#installing-from-github)安装Linera工具链。

Docker Compose脚本位于Linera代码库内，因此需要通过Github安装Linera工具链。

## 设置Linera验证者

以下章节将在`linera-protocol`仓库的`docker`子目录中操作。

### 基础设施设置

通过Docker Compose运行的验证者节点并未预装执行TLS终止所需的负载均衡器(与Kubernetes部署不同)。因此，需要验证者运营商提供TLS终止并支持Linera通知系统正常运行所需的HTTP/2长连接。

### 创建验证者配置

验证者可以使用以下模板设置：

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

创世配置描述了网络创建时的验证者委员会和链，验证者需要这些配置才能开始工作。

测试网络的创世配置将存储在由Linera Protocol核心团队管理的公共存储桶中。

下面是一个测试网创世配置的例子：

```bash
wget "https://storage.cloud.google.com/linera-io-dev-public/devnet-2024-05-07/genesis.json"
```

### 创建私钥

现在[验证者配置](joining.md#creating-your-validator-configuration)已经创建好了，并且[创世配置](joining.md#genesis-configuration)可用，可以生成验证者私钥。

下列命令将生成私钥：

```bash
linera-server generate --validators /path/to/validator/configuration.toml
```

验证者运行所需的信息在`server.json`文件中，该文件由上面的命令生成，包含密码学密钥对，其公钥将打印在屏幕上：

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

此时，`92f`开头的公钥拥有者应该与Linera Protocol核心团队沟通，告知其主机名，以便团队在下一个Epoch将该验证者添加到测试网络。

> 注意：下一个Epoch之前，验证者节点将不会从现有用户那里接收任何流量。

### 构建Linera Docker镜像

在`linera-protocol`存储库的根目录运行以下命令构建镜像：

```bash
$ docker build -f docker/Dockerfile . -t linera
```

这一步需要几分钟时间。

### 运行一个验证者节点

现在，通过在`docker`目录内运行以下命令来启动验证者，该验证者将使用`docker/genesis.json`中的创世配置，和`docker/server.json`中的服务器配置：

```bash
cd docker && docker compose up -d
```

上面的命令将以分离模式运行Docker Compose部署，这一步可能需要几分钟来下载并启动ScyllaDB镜像。
