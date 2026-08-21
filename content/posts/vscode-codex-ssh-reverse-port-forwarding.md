---
title: "服务器通过本地代理使用 Codex "
date: 2026-08-21T08:30:00+08:00
draft: false
categories: ["codex"]
summary: "通过 VS Code SSH 反向端口转发，让远程服务器上的 Codex 使用本机代理"
---

本文记录通过 SSH 反向端口转发，让远程服务器上的 Codex 使用本地代理的经验。

## 一、准备

准备：

- 远程服务器已安装 Codex
- 本机已开启代理，并确认 HTTP 代理端口，例如 `7890`
- 已使用 VS Code Remote - SSH 连接远程服务器

本文统一让远程服务器通过 `127.0.0.1:7890` 访问代理。

## 二、修改本机 SSH config

打开本机的 SSH config 文件：

```text
~/.ssh/config
```

在需要连接的服务器配置中加入 `RemoteForward`：

```sshconfig
Host my-server
    HostName <服务器地址>
    User <用户名>
    RemoteForward 47890 127.0.0.1:7890
```

`47890`表示远程服务器的端口， `7890`为本机的代理端口。

## 三、配置远程代理环境变量

在 VS Code 中按 `Ctrl + Shift + P`，输入并打开：

```text
Preferences: Open Remote Settings (JSON)
```

加入以下配置：

```json
{
    "http.proxy": "http://127.0.0.1:7890",
    "http.proxySupport": "on",
    "http.proxyStrictSSL": false,
    "terminal.integrated.env.linux": {
        "http_proxy": "http://127.0.0.1:7890",
        "https_proxy": "http://127.0.0.1:7890",
        "HTTP_PROXY": "http://127.0.0.1:7890",
        "HTTPS_PROXY": "http://127.0.0.1:7890"
    }
}
```

也可以把代理变量写到远程服务器的 `~/.bashrc`：

```bash
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890
```

保存后执行：

```bash
source ~/.bashrc
```


## 四、上传 Codex 登录文件

如果本机已经登录过 Codex，可以把auth.json登录文件复制到远程服务器。

先在远程服务器创建目录：

```bash
mkdir -p ~/.codex
chmod 700 ~/.codex
```

然后在本机终端执行：

```bash
scp ~/.codex/auth.json my-server:~/.codex/auth.json
ssh my-server 'chmod 600 ~/.codex/auth.json'
```

其中 `my-server` 替换为 SSH config 中配置的 Host 名称。

## 五、配置 Codex 代理

打开远程服务器上的 `~/.codex/config.toml`：

```bash
nano ~/.codex/config.toml
```

添加以下配置：

```toml
[proxy]
http_proxy = "http://127.0.0.1:7890"
https_proxy = "http://127.0.0.1:7890"
```

最后重新连接 VS Code Remote - SSH，再在远程终端启动 Codex：

