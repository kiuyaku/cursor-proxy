# Cursor Proxy

**仓库地址**：[https://github.com/kiuyaku/cursor-proxy](https://github.com/kiuyaku/cursor-proxy)

让 Cursor 在不开启 TUN 模式的情况下走 SOCKS5/HTTP 代理。基于 [antigravity-proxy](https://github.com/yuaotian/antigravity-proxy) 的 DLL 流量劫持方案，将 Cursor 相关进程的网络请求透明重定向到本地代理。

## 原理说明

Cursor 不读取系统代理设置，直接连接外网时会被阻断。本方案通过 **version.dll 劫持** 实现：

1. **DLL 劫持**：Cursor 启动时加载同目录的 `version.dll`，该 DLL 来自 antigravity-proxy，会 Hook Winsock API（connect、getaddrinfo 等）
2. **流量重定向**：当 Cursor 及其子进程（如 `language_server_windows`）发起网络连接时，请求被拦截并转发到 `config.json` 中配置的 SOCKS5/HTTP 代理
3. **进程级代理**：仅 Cursor 相关进程走代理，其他应用不受影响，无需 TUN 模式或管理员权限

## 快速开始

### 1. 准备代理

确保本机有可用的 SOCKS5 或 HTTP 代理（Clash / V2Ray / Mihomo 等），例如 `127.0.0.1:7890`。

### 2. 下载并部署

1. 从本仓库 [Releases](https://github.com/kiuyaku/cursor-proxy/releases) 或 [antigravity-proxy Releases](https://github.com/yuaotian/antigravity-proxy/releases) 下载 `version.dll` 和 `config.json`
2. 将两者复制到 **Cursor 安装目录**（与 `Cursor.exe` 同级）
3. 按需编辑 `config.json` 中的 `proxy.host`、`proxy.port`、`proxy.type`
4. 启动 Cursor

### 3. Cursor 安装目录

默认：`%LOCALAPPDATA%\Programs\cursor\`  
或右键 Cursor 图标 → **打开文件所在的位置**

## 配置说明

`config.json` 中 `target_processes` 已配置为 `["language_server_windows", "Cursor.exe"]`，与 antigravity-proxy 逻辑一致，仅对上述进程进行流量劫持代理。修改 `proxy` 段以匹配本地代理：

```json
{
    "proxy": {
        "host": "127.0.0.1",
        "port": 7890,
        "type": "socks5"
    }
}
```

## 致谢与许可

本方案基于 [antigravity-proxy](https://github.com/yuaotian/antigravity-proxy) 实现，BSD-2-Clause 许可。
