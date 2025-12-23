# UAP sing-box 部署指南

> **最后更新**: 2025-12-23

## 目录

- [系统要求](#系统要求)
- [从源码构建](#从源码构建)
- [配置示例](#配置示例)
- [部署方式](#部署方式)
- [验证测试](#验证测试)
- [常见问题](#常见问题)

---

## 系统要求

### 编译环境

| 依赖 | 版本要求 |
|------|----------|
| Go | 1.23.1+ |
| Git | 2.0+ |
| Make | 3.0+ (可选) |

### 运行环境

| 平台 | 架构 |
|------|------|
| Linux | amd64, arm64 |
| Windows | amd64 |
| macOS | amd64, arm64 |

---

## 从源码构建

### 1. 克隆仓库

```bash
git clone https://git.uap.io/uap/uap-sing-box.git
cd uap-sing-box
```

### 2. 构建二进制

**推荐方式 (完整功能):**

```bash
go build -tags "with_quic,with_utls,with_gvisor,with_wireguard" -o sing-box ./cmd/sing-box
```

**使用 Makefile:**

```bash
make
# 或指定 tags
TAGS="with_quic with_utls with_gvisor with_wireguard" make
```

**最小构建 (仅 UAP 基础功能):**

```bash
go build -tags "with_utls" -o sing-box ./cmd/sing-box
```

### 3. 构建标签说明

| Tag | 必需 | 说明 |
|-----|------|------|
| `with_utls` | **是** | uTLS 支持，Reality 模式必需 |
| `with_gvisor` | 推荐 | gVisor 网络栈，TUN 模式必需 |
| `with_quic` | 推荐 | QUIC/HTTP3 支持 |
| `with_wireguard` | 可选 | WireGuard 支持 |
| `with_acme` | 可选 | 自动 TLS 证书 |
| `with_clash_api` | 可选 | Clash API 支持 |

### 4. 交叉编译

```bash
# Linux AMD64
GOOS=linux GOARCH=amd64 go build -tags "with_quic,with_utls,with_gvisor,with_wireguard" -o sing-box-linux-amd64 ./cmd/sing-box

# Linux ARM64
GOOS=linux GOARCH=arm64 go build -tags "with_quic,with_utls,with_gvisor,with_wireguard" -o sing-box-linux-arm64 ./cmd/sing-box

# Windows AMD64
GOOS=windows GOARCH=amd64 go build -tags "with_quic,with_utls,with_gvisor,with_wireguard" -o sing-box-windows-amd64.exe ./cmd/sing-box

# macOS ARM64 (Apple Silicon)
GOOS=darwin GOARCH=arm64 go build -tags "with_quic,with_utls,with_gvisor,with_wireguard" -o sing-box-darwin-arm64 ./cmd/sing-box
```

### 5. 验证构建

```bash
./sing-box version
```

预期输出:
```
sing-box version x.x.x

Environment: go1.23.x linux/amd64
Tags: with_quic,with_utls,with_gvisor,with_wireguard
```

---

## 配置示例

### UAP 服务端配置

**基础配置 (无 TLS):**

```json
{
  "log": {
    "level": "info",
    "timestamp": true
  },
  "inbounds": [
    {
      "type": "uap",
      "tag": "uap-in",
      "listen": "0.0.0.0",
      "listen_port": 10086,
      "users": [
        {
          "name": "user1",
          "uuid": "your-uuid-here"
        }
      ]
    }
  ],
  "outbounds": [
    {
      "type": "direct",
      "tag": "direct"
    }
  ]
}
```

**Vision Flow 配置 (TLS):**

```json
{
  "log": {
    "level": "info",
    "timestamp": true
  },
  "inbounds": [
    {
      "type": "uap",
      "tag": "uap-in",
      "listen": "0.0.0.0",
      "listen_port": 443,
      "users": [
        {
          "name": "user1",
          "uuid": "your-uuid-here",
          "flow": "xtls-rprx-vision"
        }
      ],
      "tls": {
        "enabled": true,
        "server_name": "your-domain.com",
        "certificate_path": "/path/to/cert.pem",
        "key_path": "/path/to/key.pem"
      }
    }
  ],
  "outbounds": [
    {
      "type": "direct",
      "tag": "direct"
    }
  ]
}
```

**Reality 配置:**

首先生成密钥对:
```bash
./sing-box generate reality-keypair
```

服务端配置:
```json
{
  "log": {
    "level": "info",
    "timestamp": true
  },
  "inbounds": [
    {
      "type": "uap",
      "tag": "uap-in",
      "listen": "0.0.0.0",
      "listen_port": 443,
      "users": [
        {
          "name": "user1",
          "uuid": "your-uuid-here",
          "flow": "xtls-rprx-vision"
        }
      ],
      "tls": {
        "enabled": true,
        "server_name": "www.microsoft.com",
        "reality": {
          "enabled": true,
          "handshake": {
            "server": "www.microsoft.com",
            "server_port": 443
          },
          "private_key": "your-private-key",
          "short_id": ["abcd1234"]
        }
      }
    }
  ],
  "outbounds": [
    {
      "type": "direct",
      "tag": "direct"
    }
  ]
}
```

### UAP 客户端配置

**基础配置:**

```json
{
  "log": {
    "level": "info",
    "timestamp": true
  },
  "inbounds": [
    {
      "type": "socks",
      "tag": "socks-in",
      "listen": "127.0.0.1",
      "listen_port": 1080
    },
    {
      "type": "http",
      "tag": "http-in",
      "listen": "127.0.0.1",
      "listen_port": 8080
    }
  ],
  "outbounds": [
    {
      "type": "uap",
      "tag": "uap-out",
      "server": "your-server-ip",
      "server_port": 10086,
      "uuid": "your-uuid-here"
    },
    {
      "type": "direct",
      "tag": "direct"
    }
  ]
}
```

**Reality 客户端配置:**

```json
{
  "log": {
    "level": "info",
    "timestamp": true
  },
  "inbounds": [
    {
      "type": "socks",
      "tag": "socks-in",
      "listen": "127.0.0.1",
      "listen_port": 1080
    }
  ],
  "outbounds": [
    {
      "type": "uap",
      "tag": "uap-out",
      "server": "your-server-ip",
      "server_port": 443,
      "uuid": "your-uuid-here",
      "flow": "xtls-rprx-vision",
      "tls": {
        "enabled": true,
        "server_name": "www.microsoft.com",
        "utls": {
          "enabled": true,
          "fingerprint": "chrome"
        },
        "reality": {
          "enabled": true,
          "public_key": "your-public-key",
          "short_id": "abcd1234"
        }
      }
    },
    {
      "type": "direct",
      "tag": "direct"
    }
  ]
}
```

---

## 部署方式

### 方式一: Systemd 服务 (推荐)

**1. 复制二进制文件:**

```bash
sudo cp sing-box /usr/local/bin/
sudo chmod +x /usr/local/bin/sing-box
```

**2. 创建配置目录:**

```bash
sudo mkdir -p /etc/sing-box
sudo cp config.json /etc/sing-box/
```

**3. 创建 systemd 服务文件:**

```bash
sudo tee /etc/systemd/system/sing-box.service > /dev/null <<EOF
[Unit]
Description=sing-box service
Documentation=https://sing-box.sagernet.org
After=network.target nss-lookup.target

[Service]
User=root
WorkingDirectory=/etc/sing-box
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE CAP_NET_RAW CAP_SYS_PTRACE
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE CAP_NET_RAW CAP_SYS_PTRACE
ExecStart=/usr/local/bin/sing-box run -c /etc/sing-box/config.json
ExecReload=/bin/kill -HUP \$MAINPID
Restart=on-failure
RestartSec=10
LimitNOFILE=infinity

[Install]
WantedBy=multi-user.target
EOF
```

**4. 启动服务:**

```bash
sudo systemctl daemon-reload
sudo systemctl enable sing-box
sudo systemctl start sing-box
sudo systemctl status sing-box
```

**5. 查看日志:**

```bash
sudo journalctl -u sing-box -f
```

### 方式二: Docker 部署

**Dockerfile:**

```dockerfile
FROM golang:1.23-alpine AS builder

WORKDIR /app
COPY . .

RUN go build -tags "with_quic,with_utls,with_gvisor,with_wireguard" -o sing-box ./cmd/sing-box

FROM alpine:latest

RUN apk --no-cache add ca-certificates tzdata

COPY --from=builder /app/sing-box /usr/local/bin/

WORKDIR /etc/sing-box
VOLUME /etc/sing-box

ENTRYPOINT ["sing-box"]
CMD ["run", "-c", "/etc/sing-box/config.json"]
```

**构建和运行:**

```bash
# 构建镜像
docker build -t uap-sing-box .

# 运行容器
docker run -d \
  --name sing-box \
  --restart always \
  --network host \
  -v /path/to/config:/etc/sing-box \
  uap-sing-box
```

**Docker Compose:**

```yaml
version: '3.8'

services:
  sing-box:
    build: .
    container_name: sing-box
    restart: always
    network_mode: host
    volumes:
      - ./config:/etc/sing-box
    cap_add:
      - NET_ADMIN
      - NET_RAW
```

### 方式三: 直接运行

**前台运行:**

```bash
./sing-box run -c config.json
```

**后台运行:**

```bash
nohup ./sing-box run -c config.json > sing-box.log 2>&1 &
```

---

## 验证测试

### 1. 配置检查

```bash
./sing-box check -c config.json
```

### 2. 配置格式化

```bash
./sing-box format -c config.json
```

### 3. 连接测试

**服务端启动后，客户端测试:**

```bash
# 通过 SOCKS5 代理测试
curl -x socks5://127.0.0.1:1080 http://httpbin.org/ip

# 通过 HTTP 代理测试
curl -x http://127.0.0.1:8080 http://httpbin.org/ip
```

### 4. 生成 UUID

```bash
./sing-box generate uuid
```

### 5. 生成 Reality 密钥对

```bash
./sing-box generate reality-keypair
```

---

## 常见问题

### Q: 编译报错 `undefined: tun.DefaultNIC`

**原因:** 缺少 `with_gvisor` 构建标签

**解决:**
```bash
go build -tags "with_gvisor" ./cmd/sing-box
```

### Q: Reality 连接失败

**检查:**
1. 确保编译时包含 `with_utls` 标签
2. 确认客户端和服务端的 `public_key`/`private_key` 匹配
3. 确认 `short_id` 一致
4. 检查 handshake server 是否可访问

### Q: Vision flow 不工作

**检查:**
1. 服务端和客户端都配置了 `"flow": "xtls-rprx-vision"`
2. TLS 已启用
3. 用户配置中的 flow 与连接请求匹配

### Q: 端口被占用

```bash
# 查看端口占用
sudo lsof -i :443
sudo netstat -tlnp | grep 443

# 或更换端口
```

### Q: 权限不足

```bash
# 允许绑定低端口
sudo setcap cap_net_bind_service=+ep /usr/local/bin/sing-box

# 或使用 root 运行
sudo ./sing-box run -c config.json
```

---

## 相关文档

- [UAP 技术方案](./uap-singbox-implementation.md)
- [UAP 实现计划](./uap-singbox-implementation-plan.md)
- [sing-box 官方文档](https://sing-box.sagernet.org/)
- [构建指南](./installation/build-from-source.md)
