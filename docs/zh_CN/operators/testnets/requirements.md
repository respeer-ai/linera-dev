# 需求

本节介绍部署Linera验证节点及加入测试网络的相关要求。验证节点部署方案已在**Linux**系统下经过全面测试，但目前尚未对MacOS或Windows系统提供官方支持。

## 安装Linera工具链

> 在安装 Linera 工具链时，必须检出 `testnet_babbage` 分支。

要安装 Linera 工具链，请参阅[安装章节](../../../zh_CN/developers/getting_started/installation.md#installing-from-github)。

你需要从GitHub安装工具链，因为后续需要通过该仓库来运行Docker Compose验证器服务。

## Docker Compose需求

Linera 验证节点通过 Docker Compose 运行。

要安装 Docker Compose，请参阅 [Docker 官方文档](https://docs.docker.com/compose/install/)中的安装说明。

## Key 管理

目前，Linera 的密钥存储在本地文件系统的 JSON 文件中。出于便利性考虑，这些密钥目前以明文形式存储。该文件通常命名为 `server.json`，位于核心协议仓库的 `docker/` 目录下。

一旦生成密钥，请务必进行备份，因为如果密钥丢失，目前无法恢复。

## 基础设施需求

通过 Docker Compose 运行的验证节点未配备用于执行 TLS 终止的预装负载均衡器（与在 Kubernetes 上运行的验证节点不同）。

负载均衡器配置**必须**包含以下属性：

1. 支持 HTTP/2 连接。
2. 支持 gRPC 连接。
3. 支持 long-lived HTTP/2 连接。
4. 支持最大请求体大小不超过20 MB。
5. 提供使用由知名证书颁发机构（CA）签发的证书进行 TLS 终止。

最后，执行 TLS 终止的负载均衡器必须将流量从 `443` 端口重定向到 `19100` 端口（即代理暴露的端口）。

### 使用Nginx

最低支持版本：1.18.0。

以下是一个符合 `/etc/nginx/sites-available/default` 中所述基础设施要求的 Nginx 配置示例：

```ignore
server {
        listen 80 http2;

        location / {
                grpc_pass grpc://127.0.0.1:19100;
        }
}

server {
    listen 443 ssl http2;
    server_name <hostname>; # e.g. my-subdomain.my-domain.net

    # SSL certificates
    ssl_certificate <ssl-cert-path>; # e.g. /etc/letsencrypt/live/my-subdomain.my-domain.net/fullchain.pem
    ssl_certificate_key <ssl-key-path>; # e.g. /etc/letsencrypt/live/my-subdomain.my-domain.net/privkey.pem;

    # Proxy traffic to the service running on port 19100.
    location / {
        grpc_pass grpc://127.0.0.1:19100;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    keepalive_timeout 10m 60s;
    grpc_read_timeout 10m;
    grpc_send_timeout 10m;

    client_header_timeout 10m;
    client_body_timeout 10m;
}
```

### 使用Caddy

最低支持版本：v2.4.3

以下是一个符合 `/etc/caddy/Caddyfile` 中定义的基础设施要求的 Caddy 配置示例：

```ignore
example.com {
  reverse_proxy localhost:19100 {
    transport http {
      versions h2c
      read_timeout 10m
      write_timeout 10m
    }
  }
}
```

### ScyllaDB 配置

ScyllaDB 是一种开源分布式 NoSQL 数据库，专为高性能和低延迟而设计。Linera 验证者节点使用 ScyllaDB 作为其持久化存储。

ScyllaDB 可能需要修改内核参数才能正常运行。具体来说，这涉及调整异步 I/O 上下文中允许的事件数量。

要设置此参数，请运行：

```bash
echo 1048576 > /proc/sys/fs/aio-max-nr
```
