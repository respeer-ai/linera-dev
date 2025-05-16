# 手动安装

如果您不想使用提供的部署脚本，可以选择手动部署验证节点。

## 搭建 Linera 验证节点

下一部分内容将涉及操作 `linera-protocol` 仓库中的 `docker` 子目录。

### 创建验证节点配置

验证节点通过 TOML 文件进行配置。您可以使用以下模板来创建自己的验证节点配置：

```toml
server_config_path = "server.json"
host = "<your-host>" # e.g. my-subdomain.my-domain.net
port = 19100
metrics_host = "proxy"
metrics_port = 21100
internal_host = "proxy"
internal_port = 20100
[external_protocol]
Grpc = "ClearText" # Depending on your load balancer you may need "Tls" here.
[internal_protocol]
Grpc = "ClearText"

# Adjust depending on the number of shards you have
[[shards]]
host = "docker-shard-1"
port = 19100
metrics_port = 21100

[[shards]]
host = "docker-shard-2"
port = 19100
metrics_port = 21100

[[shards]]
host = "docker-shard-3"
port = 19100
metrics_port = 21100

[[shards]]
host = "docker-shard-4"
port = 19100
metrics_port = 21100

```

### 创世配置

创世配置描述了网络创建时验证节点委员会和链的配置情况，是验证节点正常运行的必要条件。

最初，每个测试网络的创世配置将存放在由 Linera 协议核心团队管理的公共存储桶中。

示例可在此处找到：

```bash
wget "https://storage.googleapis.com/linera-io-dev-public/{{#include ../../../TESTNET_DOMAIN}}/genesis.json"
```

### 创建您的密钥

在[验证节点配置](../../../zh_CN/operators/testnets/manual-installation.md#creating-your-validator-configuration)已创建且[创世配置](../../../zh_CN/operators/testnets/manual-installation.md#genesis-configuration)可用后，即可生成验证节点的私钥。


要生成私钥，需使用 `linera-server` 二进制文件：

```bash
linera-server generate --validators /path/to/validator/configuration.toml
```

这将生成一个名为 `server.json` 的文件，其中包含验证节点运行所需的信息，包括加密密钥对。

命令执行完成后，系统会输出公钥，例如：

```bash
$ linera-server generate --validators /path/to/validator/configuration.toml
2024-07-01T16:51:32.881519Z  INFO linera_server: Wrote server config server.json
02a580bbda90f0ab10f015422d450b3e873166703af05abd77d8880852a3504e4d,009b2ecc5d39645e81ff01cfe4ceeca5ec207d822762f43b35ef77b2367666a7f8
```

在这种情况下，公钥和账户密钥分别以 `02a` 和 `009` 开头，必须连同所选的主机名一起提交给 Linera 协议核心团队，以便在下一个 epoch 加入网络。

> 注意：验证节点在加入下一个纪元之前，将不会收到现有用户的任何流量。

### 构建 Linera Docker 镜像

要在 `linera-protocol` 仓库的根目录下构建 Linera 的 Docker 镜像，请运行以下命令：

```bash
docker build --build-arg git_commit="$(git rev-parse --short HEAD)" -f docker/Dockerfile . -t linera
```

此过程可能需要几分钟时间。

### 运行验证节点

现在，创世配置文件位于 `docker/genesis.json`，服务器配置文件位于 `docker/server.json`。请在 `docker` 目录下运行以下命令启动验证节点：

```bash
cd docker && docker compose up -d
```

这将以分离模式运行 Docker Compose 部署。ScyllaDB 镜像的下载和启动可能需要几分钟时间。
