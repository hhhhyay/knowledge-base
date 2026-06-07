---
title: VSCode使用CodeX插件流程
tags:
  - AI
  - LLM
  - Benchmark
model: []
framework: []
issue_type: []
source: feishu
feishu_token: N4hEwhoGPiDNs7khboGcSgxvnyS
feishu_type: DOCX
source_url: https://kcnm6g5dkw5p.feishu.cn/wiki/N4hEwhoGPiDNs7khboGcSgxvnyS
last_synced_at: 2026-06-07T17:07:53.393Z
---

# VSCode使用CodeX插件流程

[在飞书中打开](https://kcnm6g5dkw5p.feishu.cn/wiki/N4hEwhoGPiDNs7khboGcSgxvnyS)

### 本地确认 Clash Verge 端口

在 Clash Verge 里查看 Mixed Port，例如：

<image token="QuSRbWnb0owf9JxlrDfcMGe3nxe" width="1175" height="914" align="center"/>

`Mixed Port: 7897`

本地 PowerShell 测试：
```plaintext {wrap}
curl.exe -x http://127.0.0.1:7897 -I https://api.openai.com
```

看到 HTTP/1.1 200 Connection established 即可。后面出现 421 也不影响。

### 配置本地 SSH config

编辑本地：

`C:\Users\你的用户名\.ssh\config`

加入一个专门用于反向代理的 Host：
```plaintext {wrap}
Host 郑州集群-lvjj1-反向代理
  HostName 42.228.13.241
  Port 65024
  User lvjj1
  RemoteForward 127.0.0.1:18080 127.0.0.1:7897
  ExitOnForwardFailure yes
```

说明：

`18080 = 集群上的代理端口`

`7897  = 本地 Clash Verge 的 Mixed Port`

18080当前已经被占用，建议换成其他端口，例如 28197，后续配置同步替换即可。

### 用 VS Code Remote-SSH 连接

在 VS Code 中连接集群，选择：

`郑州集群-lvjj1-反向代理`

### 在集群远程终端测试代理

VS Code 连上后，打开远程终端，执行：
```plaintext {wrap}
curl -I -x http://127.0.0.1:18080 https://api.openai.com
```

如果看到：

<image token="N0yVbiafFo6r9axC0JpcubhOnnc" width="865" height="263" align="center"/>

说明反向代理成功。

### 配置 VS Code Remote Settings

在已连接集群的 VS Code 窗口中：

`Ctrl + Shift + P`

搜索并打开：

`Preferences: Open Remote Settings (JSON)`

加入：

`{  "http.proxy": "``http://127.0.0.1:18080``",  "http.proxySupport": "override"}`

保存后执行：

`Developer: Reload Window`

### 登录 Codex 插件

重载 VS Code 后，打开 Codex 插件登录。

如果使用 ChatGPT / Google 账号登录，远程插件会通过：

`http://127.0.0.1:18080`

走本地 Clash Verge 出网。

### 常见问题

如果 VS Code 连接时报：

`remote port forwarding failed for listen port 18080`

说明集群上的 18080 被占用。换一个端口，例如：

`RemoteForward 127.0.0.1:28197 127.0.0.1:7897`

然后把 VS Code 设置和测试命令里的 18080 也改成 28197。

### 一句话总结

本地开 Clash Verge，SSH config 里用 RemoteForward 把集群的 127.0.0.1:18080 转发到本地 127.0.0.1:7897，然后让 VS Code Remote 和 Codex 插件使用 http://127.0.0.1:18080 作为代理。

## 关联资源

- 无

## 关联知识

- [[飞书知识图谱/实体/Codex|Codex]]（tool）
- [[飞书知识图谱/实体/PP|PP]]（concept）
- [[飞书知识图谱/实体/TP|TP]]（concept）
- [[飞书知识图谱/实体/VSCode|VSCode]]（tool）
