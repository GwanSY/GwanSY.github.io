---
layout: post
title: "VS Code Codex 插件连接问题解决方案（Windows 版）"
date: 2026-06-12 09:30:00 +0800
categories: Other
---

# VS Code Codex 插件连接问题解决方案（Windows 版）

适用于以下情况：

- 已在终端中成功配置 Codex
- 已登录 ChatGPT/OpenAI 账号
- 命令行中的 Codex 可以正常使用
- Windows 已开启科学上网
- VS Code 中出现：
  - Reconnecting...
  - Connection failed
  - Request timeout

通常原因是 VS Code 未正确使用代理。

---

## 第一步：确认代理端口是否正常监听

打开 CMD：

```cmd
netstat -ano | findstr 7891
```

如果看到类似输出：

```text
TCP    127.0.0.1:7891    0.0.0.0:0    LISTENING
```

说明代理正常运行。

---

## 第二步：测试代理能否访问 OpenAI

打开 PowerShell：

```powershell
curl.exe -x http://127.0.0.1:7891 https://api.openai.com/v1/models
```

如果返回 JSON 数据或 OpenAI 错误信息：

```json
{
  "error": {
    ...
  }
}
```

说明代理链路正常。

如果超时或无法连接，则需要先检查代理软件配置。

---

## 第三步：配置 Windows 环境变量

按：

```text
Win + R
```

输入：

```text
sysdm.cpl
```

进入：

```text
高级
↓
环境变量
```

新增以下用户变量：

### HTTP_PROXY

变量名：

```text
HTTP_PROXY
```

变量值：

```text
http://127.0.0.1:7891
```

### HTTPS_PROXY

变量名：

```text
HTTPS_PROXY
```

变量值：

```text
http://127.0.0.1:7891
```

### ALL_PROXY

变量名：

```text
ALL_PROXY
```

变量值：

```text
http://127.0.0.1:7891
```

保存后重新打开 CMD。

验证：

```cmd
echo %HTTP_PROXY%
```

应输出：

```text
http://127.0.0.1:7891
```

---

## 第四步：配置 VS Code 代理

打开 VS Code：

```text
Ctrl + ,
```

搜索：

```text
proxy
```

修改以下配置：

### Http: Proxy

```text
http://127.0.0.1:7891
```

### Http: Proxy Support

```text
on
```

### Http: System Certificates

```text
√ 勾选
```

### Http: Proxy Strict SSL

如果连接失败：

```text
取消勾选
```

---

## 第五步：直接修改 settings.json（推荐）

按：

```text
Ctrl + Shift + P
```

输入：

```text
Preferences: Open User Settings (JSON)
```

添加：

```json
{
    "http.proxy": "http://127.0.0.1:7891",
    "http.proxySupport": "on",
    "http.systemCertificates": true,
    "http.proxyStrictSSL": false
}
```

保存。

---

## 第六步：检查代理软件设置

如果使用：

- Clash Verge
- Clash for Windows
- Mihomo
- V2RayN

建议开启：

```text
TUN Mode
```

或者：

```text
System Proxy
```

至少开启其中一个。

很多情况下浏览器能正常访问网络，但 VS Code 无法连接 OpenAI，就是因为代理没有接管 VS Code 的网络请求。

---

## 第七步：完全重启 VS Code

不要仅执行 Reload Window。

打开任务管理器：

```text
Ctrl + Shift + Esc
```

结束所有：

```text
Code.exe
```

然后重新启动 VS Code。

---

## 第八步：查看日志排查问题

打开：

```text
View
↓
Output
```

在右上角选择：

```text
OpenAI
```

或者：

```text
Codex
```

查看具体报错信息。

---

## 常见错误说明

### ECONNREFUSED

```text
connect ECONNREFUSED 127.0.0.1:7891
```

原因：

```text
代理未启动
```

---

### ETIMEDOUT

```text
Request timeout
```

原因：

```text
代理规则未正确代理 OpenAI
```

检查规则是否让以下域名走代理：

```text
api.openai.com
chatgpt.com
```

---

### SSL 证书错误

```text
certificate verify failed
```

可尝试关闭 SSL 严格校验：

```json
"http.proxyStrictSSL": false
```

---

## 快速验证命令

查看代理变量：

```cmd
echo %HTTP_PROXY%
echo %HTTPS_PROXY%
```

测试 OpenAI 连通性：

```powershell
curl.exe -x http://127.0.0.1:7891 https://api.openai.com/v1/models
```

检查代理端口：

```cmd
netstat -ano | findstr 7891
```

---

## 推荐最终配置

VS Code `settings.json`：

```json
{
    "http.proxy": "http://127.0.0.1:7891",
    "http.proxySupport": "on",
    "http.systemCertificates": true,
    "http.proxyStrictSSL": false
}
```

完成后按以下顺序执行：

1. 启动 Clash/V2RayN 等代理软件
2. 开启 System Proxy 或 TUN Mode
3. 完全退出 VS Code
4. 重新打开 VS Code
5. 测试 Codex 对话功能

一般情况下，完成上述配置后即可解决 VS Code 中 Codex 插件的连接问题。