# UAP 协议 sing-box 实现计划

> **关联技术方案**: [uap-singbox-implementation.md](./uap-singbox-implementation.md)
> **仓库地址**: https://git.uap.io/uap/uap-sing-box
> **最后更新**: 2025-12-22

## 实现阶段

### Step 1: 仓库准备

> 参考: [4. 基于官方 sing-box 实现 UAP](./uap-singbox-implementation.md#4-基于官方-sing-box-实现-uap)

- [x] Fork 官方 sing-box 到 uap-sing-box
- [x] 设置 CI/CD 流程 (Gitea Actions)

### Step 2: 常量定义

> 参考: [4.3.1 常量定义](./uap-singbox-implementation.md#431-常量定义)

- [x] 修改 `constant/proxy.go` 添加 `TypeUAP = "uap"`

### Step 3: 协议实现

> 参考: [4.1 需要新增的文件](./uap-singbox-implementation.md#41-需要新增的文件), [4.3.3 协议实现](./uap-singbox-implementation.md#433-协议实现-从-vless-复制修改)

- [x] 创建 `protocol/uap/` 目录
- [x] 实现 `protocol/uap/constant.go` (TLS 常量、Vision 命令)
- [x] 实现 `protocol/uap/protocol.go` (协议编解码，版本号、UUID、Addons 处理)
- [x] 实现 `protocol/uap/client.go` (客户端连接处理)
- [x] 实现 `protocol/uap/service.go` (服务端连接处理)
- [x] 实现 `protocol/uap/vision.go` (Vision flow 实现)
- [x] 实现 `protocol/uap/vision_reality.go` (Reality 支持)
- [x] 实现 `protocol/uap/vision_utls.go` (uTLS 支持)
- [x] 实现 `protocol/uap/inbound.go` (Inbound 实现)
- [x] 实现 `protocol/uap/outbound.go` (Outbound 实现)

### Step 4: 配置选项

> 参考: [4.3.2 配置选项](./uap-singbox-implementation.md#432-配置选项)

- [x] 创建 `option/uap.go` (UAPInboundOptions, UAPOutboundOptions, UAPUser)

### Step 5: 注册 UAP

> 参考: [4.2 需要修改的文件](./uap-singbox-implementation.md#42-需要修改的文件)
> 注: sing-box v1.13+ 使用注册表模式，所有协议在 `include/registry.go` 中注册

- [x] 修改 `include/registry.go` 导入 UAP protocol 包
- [x] 修改 `include/registry.go` 在 InboundRegistry() 中注册 uap.RegisterInbound
- [x] 修改 `include/registry.go` 在 OutboundRegistry() 中注册 uap.RegisterOutbound

### Step 6: 编译测试

> 参考: [7. 编译与部署](./uap-singbox-implementation.md#7-编译与部署)

- [x] 编译 sing-box (含 UAP): `go build -tags "with_quic,with_utls,with_gvisor,with_wireguard" ./cmd/sing-box`
- [x] 验证 `./sing-box version` 输出 (43MB binary, Tags: with_quic,with_utls,with_gvisor,with_wireguard)
- [x] 验证 UAP 配置解析: `sing-box check` 和 `sing-box format` 正常工作

### Step 7: 功能验证

> 参考: [6. UAP 配置示例](./uap-singbox-implementation.md#6-uap-配置示例)

- [ ] 创建测试 Inbound 配置
- [ ] 创建测试 Outbound 配置
- [ ] 验证连接建立
- [ ] 验证数据传输
- [ ] 验证 Vision flow
- [ ] 验证 Reality 握手

---

## 进度统计

| 步骤 | 任务数 | 完成数 | 进度 |
|------|--------|--------|------|
| Step 1 | 2 | 2 | 100% |
| Step 2 | 1 | 1 | 100% |
| Step 3 | 10 | 10 | 100% |
| Step 4 | 1 | 1 | 100% |
| Step 5 | 3 | 3 | 100% |
| Step 6 | 3 | 3 | 100% |
| Step 7 | 6 | 0 | 0% |
| **总计** | **26** | **20** | **77%** |
