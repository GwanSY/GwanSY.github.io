---
layout: post
title: "VS Code Remote-SSH 服务器环境使用 Codex 插件"
date: 2026-06-15 09:30:00 +0800
categories: Other
---

# VS Code Remote-SSH 服务器环境使用 Codex 插件实操与故障排查笔记

- 第一篇：[2026-06-12-GitHub_Pages3](/posts/GitHub_Pages3/)
- 如果你已经完成第一步，可以直接从这里继续排查或补充配置。

## 0. 背景

目标是在本地 VS Code 通过 Remote-SSH 连接远程服务器，并在远程项目中使用 OpenAI Codex 插件。

常见现象包括：

```text
Codex 一直显示云朵图标
Codex 卡在 Thinking...
Token exchange failed: token endpoint returned status 403 Forbidden
listen EACCES: permission denied /tmp/codex-ipc/ipc-1001.sock
failed to initialize sqlite state runtime
migration 1 was previously applied but has been modified
```

本笔记分为三部分：

1. 如何用 VS Code 连接服务器
2. 如何在远程服务器环境开启 Codex
3. 如何按日志排查 Codex 启动失败问题

---

## 1. 在 VS Code 中连接远程服务器

### 1.1 本地安装 Remote-SSH

在本地 VS Code 中安装扩展：

```text
Remote - SSH
```

或者安装扩展包：

```text
Remote Development
```

---

### 1.2 本地配置 SSH

先确认本地终端可以连接服务器：

```bash
ssh user@服务器地址
```

如果使用 SSH config，可以在本地配置文件中添加：

```sshconfig
Host myserver
    HostName 服务器地址
    User user
    Port 22
    IdentityFile ~/.ssh/id_rsa
```

测试：

```bash
ssh myserver
```

确认能正常进入服务器后，再用 VS Code 连接。

---

### 1.3 VS Code 连接服务器

在 VS Code 中：

```text
Ctrl + Shift + P
Remote-SSH: Connect to Host...
选择 myserver
```

连接成功后，VS Code 左下角会显示类似：

```text
SSH: myserver
```

然后打开远程项目目录：

```text
File → Open Folder...
```

例如：

```bash
/home/dm/project_name
```

---

## 2. 在远程 VS Code 中安装 Codex 插件

### 2.1 注意安装位置

Remote-SSH 模式下，VS Code 有本地环境和远程环境之分。

确认左下角已经显示：

```text
SSH: myserver
```

然后在扩展面板中搜索：

```text
Codex
```

安装：

```text
Codex — OpenAI's coding agent
```

如果 VS Code 显示：

```text
Install in SSH: myserver
```

优先选择安装到远程服务器环境。

---

### 2.2 打开 Codex

安装后：

```text
Ctrl + Shift + P
Developer: Reload Window
```

然后在 VS Code 侧边栏打开 Codex 图标。

如果看不到图标，可以查看日志：

```text
View → Output → Codex
```

---

## 3. Codex 登录方案

Codex 在 Remote-SSH 环境中登录时，可能遇到：

```text
Token exchange failed: token endpoint returned status 403 Forbidden
```

或者登录成功后一直停留在：

```text
Thinking...
```

这通常和远程服务器访问 Codex 登录接口、ChatGPT 接口或 OpenAI 相关服务的网络环境有关。

---

## 4. 网络方案选择：先判断是否真的需要 SSH 端口转发

这里分两种情况。

---

### 情况 A：服务器已经配置了代理或 TUN 模式

如果服务器上已经配置了 Clash Meta、TUN 模式或其他系统级代理，并且下面命令可以成功访问：

```bash
curl https://api.openai.com/v1/models
```

返回：

```json
{
  "error": {
    "message": "Missing bearer authentication in header"
  }
}
```

这反而说明 OpenAI API 已经可以访问。

如果执行：

```bash
curl -v --proxy http://127.0.0.1:7897 https://chat.openai.com
```

返回类似：

```text
HTTP/2 308
location: https://chatgpt.com/
```

也说明：

```text
Clash 代理正常
TLS 握手正常
DNS 正常
OpenAI / ChatGPT 链路可访问
```

如果同时：

```bash
env | grep -i proxy
```

没有输出，也不一定是问题。

因为 TUN 模式可能已经接管了服务器流量，不依赖 `HTTP_PROXY` / `HTTPS_PROXY` 环境变量。

这种情况下：

```text
不需要配置 SSH RemoteForward
不需要额外代理转发
不应该把网络问题作为第一怀疑对象
```

继续看：

```text
View → Output → Codex
```

根据 Codex 日志排查。

---

### 情况 B：服务器网络受限，本地电脑能访问，服务器不能访问

如果服务器无法直接访问 OpenAI / ChatGPT / Codex 所需服务，但本地电脑可以访问，可以使用 SSH 端口转发。

整体思路：

```text
先在本地 VS Code 完成 Codex 登录
把登录认证文件复制到服务器
通过 SSH RemoteForward 把服务器端口转发到本地代理端口
在服务器配置代理环境变量
重启 VS Code Remote Server
```

---

## 5. 本地登录 Codex 并同步认证文件

### 5.1 本地 VS Code 登录 Codex

注意：这里不要连接服务器。

在本地 VS Code 中：

```text
安装 Codex 插件
打开 Codex
完成 ChatGPT 登录
```

本地会生成 Codex 配置目录。

Windows 通常是：

```powershell
C:\Users\<你的用户名>\.codex
```

Linux / macOS 通常是：

```bash
~/.codex
```

---

### 5.2 复制认证文件到服务器

知乎方案中是复制整个 `.codex` 目录：

```powershell
scp -r C:\Users\<你的用户名>\.codex user@server:~/
```

但是更稳妥的做法是：

```text
优先只复制 auth.json 和 config.toml
不要优先复制 state_*.sqlite / logs_*.sqlite / goals_*.sqlite / memories_*.sqlite
```

原因是 SQLite 状态库可能和不同平台、不同 Codex 版本的 migration 记录不一致，复制整个目录后可能触发：

```text
migration 1 was previously applied but has been modified
```

推荐方式如下。

在服务器上先创建目录：

```bash
mkdir -p ~/.codex
chmod 700 ~/.codex
```

从 Windows PowerShell 复制：

```powershell
scp "$env:USERPROFILE\.codex\auth.json" user@server:~/.codex/
scp "$env:USERPROFILE\.codex\config.toml" user@server:~/.codex/
```

如果本地没有 `config.toml`，可以只复制 `auth.json`。

服务器上设置权限：

```bash
chmod 600 ~/.codex/auth.json 2>/dev/null || true
chmod 600 ~/.codex/config.toml 2>/dev/null || true
```

然后重新连接 Remote-SSH。

---

## 6. 配置 SSH RemoteForward 端口转发

只有在“服务器不能直接访问，但本地电脑可以访问”的情况下才需要这一步。

如果你的服务器已经通过 Clash Meta TUN 模式正常访问 OpenAI，则跳过本节。

---

### 6.1 本地打开 SSH 配置文件

在 VS Code 中：

```text
Ctrl + Shift + P
Remote-SSH: Open SSH Configuration File...
```

选择本地 SSH config 文件。

例如：

```text
C:\Users\<你的用户名>\.ssh\config
```

或者：

```bash
~/.ssh/config
```

---

### 6.2 添加 RemoteForward

假设本地代理端口是：

```text
127.0.0.1:7897
```

则在对应 Host 下添加：

```sshconfig
Host myserver
    HostName 服务器地址
    User dm
    Port 22
    IdentityFile ~/.ssh/id_rsa

    RemoteForward 7897 127.0.0.1:7897
```

含义是：

```text
服务器上的 127.0.0.1:7897
        ↓
通过 SSH 隧道
        ↓
本地 Windows / macOS / Linux 的 127.0.0.1:7897
        ↓
本地代理软件
        ↓
访问 OpenAI / ChatGPT / Codex 相关服务
```

注意：

```text
RemoteForward 写在本地 SSH config 中
不是写在服务器上的 ~/.ssh/config 中
```

---

### 6.3 常见本地代理端口

不同代理工具端口可能不同：

```text
Clash / Clash Verge / Clash Meta: 7890 / 7897
SOCKS 工具: 1080
其他工具：以实际本地监听端口为准
```

确认本地代理监听端口后，再配置 RemoteForward。

---

### 6.4 避免端口冲突

如果服务器自己已经有服务监听了：

```text
127.0.0.1:7897
```

例如服务器本身已经运行 Clash，那么不要再写：

```sshconfig
RemoteForward 7897 127.0.0.1:7897
```

否则可能出现端口绑定失败或转发不生效。

可换一个服务器端监听端口，例如：

```sshconfig
RemoteForward 17897 127.0.0.1:7897
```

然后服务器环境变量也对应改成：

```bash
export HTTP_PROXY=http://127.0.0.1:17897
export HTTPS_PROXY=http://127.0.0.1:17897
export ALL_PROXY=http://127.0.0.1:17897
```

---

## 7. 在服务器配置代理环境变量

如果使用 RemoteForward，需要在服务器上设置代理环境变量。

编辑服务器上的：

```bash
~/.bashrc
```

添加：

```bash
export HTTP_PROXY=http://127.0.0.1:7897
export HTTPS_PROXY=http://127.0.0.1:7897
export ALL_PROXY=http://127.0.0.1:7897
```

保存后执行：

```bash
source ~/.bashrc
```

验证：

```bash
env | grep -i proxy
```

应该看到：

```text
HTTP_PROXY=http://127.0.0.1:7897
HTTPS_PROXY=http://127.0.0.1:7897
ALL_PROXY=http://127.0.0.1:7897
```

如果使用了自定义端口，例如 `17897`，这里也要同步改成 `17897`。

---

## 8. 重启 VS Code Remote Server

修改 SSH config、RemoteForward、环境变量后，建议重启 VS Code Remote Server。

在 VS Code 中执行：

```text
Ctrl + Shift + P
Remote-SSH: Kill VS Code Server on Host
```

然后重新连接服务器：

```text
Remote-SSH: Connect to Host...
```

---

## 9. 测试网络转发是否成功

在服务器终端执行：

```bash
curl -v --proxy http://127.0.0.1:7897 https://chat.openai.com
```

如果返回类似：

```text
HTTP/2 308
location: https://chatgpt.com/
```

说明代理链路是通的。

也可以测试 OpenAI API：

```bash
curl https://api.openai.com/v1/models
```

如果返回：

```json
{
  "error": {
    "message": "Missing bearer authentication in header"
  }
}
```

说明服务器已经能访问 OpenAI API，只是没有带认证 token。

这是正常结果，不是失败。

---

## 10. Codex 故障排查总入口

Codex 卡住时，不要只看界面状态。

优先打开：

```text
View → Output → Codex
```

找第一条真实错误。

常见错误分为三类：

```text
网络 / 认证错误
IPC socket 权限错误
SQLite migration 状态库错误
```

---

## 11. 问题一：Token exchange failed 403

### 现象

```text
Token exchange failed: token endpoint returned status 403 Forbidden
```

或者登录后一直：

```text
Thinking...
```

### 可能原因

```text
服务器无法访问 Codex 认证服务
远程环境代理配置不完整
VS Code Server 没继承 HTTP_PROXY / HTTPS_PROXY
服务器 DNS / TLS / 代理链路异常
```

### 处理顺序

1. 在本地 VS Code 完成 Codex 登录
2. 只复制 `auth.json` / `config.toml` 到服务器
3. 如果服务器不能直连，则配置 SSH `RemoteForward`
4. 在服务器设置 `HTTP_PROXY` / `HTTPS_PROXY` / `ALL_PROXY`
5. 执行 `Remote-SSH: Kill VS Code Server on Host`
6. 重新连接服务器
7. 重新打开 Codex

---

## 12. 问题二：listen EACCES /tmp/codex-ipc

### 现象

Codex 日志中出现：

```text
listen EACCES: permission denied
/tmp/codex-ipc/ipc-1001.sock
```

### 含义

Codex 启动时需要创建 IPC socket：

```text
/tmp/codex-ipc/ipc-1001.sock
```

但当前用户没有在 `/tmp/codex-ipc` 目录下创建文件的权限。

---

### 检查权限

```bash
ls -ld /tmp/codex-ipc
```

本次看到类似：

```text
drwxrwxr-x 2 wyh wyh ...
```

当前用户是：

```bash
whoami
```

输出：

```text
dm
```

对于用户 `dm` 来说：

```text
other 权限 = r-x
没有 w
```

因此无法创建 socket 文件。

---

### 修复方式

执行：

```bash
sudo chmod 1777 /tmp/codex-ipc
```

如果没有 sudo 权限，需要联系服务器管理员执行。

再次检查：

```bash
ls -ld /tmp/codex-ipc
```

期望看到：

```text
drwxrwxrwt
```

含义：

```text
所有用户都可以创建自己的 socket
sticky bit 生效
不能随便删除别人的文件
适合共享服务器上的临时 IPC 目录
```

---

### 判断是否修复成功

重新打开 Codex 日志：

```text
View → Output → Codex
```

如果：

```text
listen EACCES
```

消失，说明 IPC 权限问题已经解决。

---

## 13. 问题三：SQLite migration 校验失败

### 现象

IPC 权限修复后，日志变成：

```text
failed to initialize sqlite state runtime
migration 1 was previously applied but has been modified
under /home/dm/.codex
```

或类似：

```text
Codex cannot access its local database
Database path:
~/.codex/state_5.sqlite

migration 1 was previously applied but has been modified
```

---

### 含义

Codex 启动时会读取：

```bash
~/.codex/state_5.sqlite
```

并校验 migration 记录。

现在的问题是：

```text
数据库中记录的 migration
和当前 Codex 版本内置的 migration
不一致
```

因此 Codex 主动退出。

---

### 高风险来源

如果你曾经做过这些操作，更容易触发：

```text
从 Windows 复制整个 .codex 到 Linux 服务器
从 WSL / Windows / Desktop App 共用同一个 .codex
不同版本 Codex 共用同一个 state_5.sqlite
复制了 state_*.sqlite / logs_*.sqlite 等状态库
```

所以不要优先复制整个 `.codex` 目录。

更推荐：

```text
只复制 auth.json
必要时复制 config.toml
不要复制 SQLite 状态库
```

---

## 14. 修复 SQLite migration 问题

### 14.1 不要直接删除整个 ~/.codex

不要一上来执行：

```bash
rm -rf ~/.codex
```

因为里面可能有：

```text
auth.json
config.toml
```

删除后可能需要重新登录或重新配置。

---

### 14.2 先备份

关闭 VS Code 远程窗口后，在服务器执行：

```bash
pkill -f codex || true
```

然后备份：

```bash
TS=$(date +%Y%m%d-%H%M%S)
cp -a ~/.codex ~/.codex.backup-$TS
```

确认备份存在：

```bash
ls -ld ~/.codex.backup-$TS
```

---

### 14.3 第一轮：只移动 state_5.sqlite

这是最可疑的文件。

```bash
TS=$(date +%Y%m%d-%H%M%S)
mkdir -p ~/.codex/recover-$TS

mv ~/.codex/state_5.sqlite* ~/.codex/recover-$TS/ 2>/dev/null || true
```

然后重新连接 VS Code Remote-SSH，打开 Codex。

如果成功，说明问题就是 `state_5.sqlite`。

---

### 14.4 第二轮：移动 logs 数据库

如果仍然报：

```text
migration 1 was previously applied but has been modified
```

继续执行：

```bash
TS=$(date +%Y%m%d-%H%M%S)
mkdir -p ~/.codex/recover-$TS

mv ~/.codex/logs_*.sqlite* ~/.codex/recover-$TS/ 2>/dev/null || true
```

然后重启 VS Code Remote-SSH。

---

### 14.5 第三轮：移动所有 SQLite 状态库

如果仍然失败，可以移动所有 SQLite 状态文件，但保留 `auth.json` 和 `config.toml`。

```bash
TS=$(date +%Y%m%d-%H%M%S)
mkdir -p ~/.codex/recover-$TS

mv ~/.codex/*.sqlite* ~/.codex/recover-$TS/ 2>/dev/null || true
```

然后重新启动 Codex。

---

## 15. 当前这次问题的实际结论

根据目前记录，本次不是网络问题。

已经确认：

```text
Clash Meta 正常
TUN 模式正常
OpenAI API 可访问
ChatGPT 入口可访问
TLS 正常
DNS 正常
不需要 SSH RemoteForward
```

执行：

```bash
curl https://api.openai.com/v1/models
```

返回：

```json
{
  "error": {
    "message": "Missing bearer authentication in header"
  }
}
```

说明 API 链路正常。

执行：

```bash
curl -v --proxy http://127.0.0.1:7897 https://chat.openai.com
```

返回：

```text
HTTP/2 308
location: https://chatgpt.com/
```

说明代理链路正常。

---

### 已解决的问题

原始错误：

```text
listen EACCES: permission denied
/tmp/codex-ipc/ipc-1001.sock
```

原因：

```text
/tmp/codex-ipc 属于 wyh:wyh
当前用户 dm 对该目录没有写权限
```

修复：

```bash
sudo chmod 1777 /tmp/codex-ipc
```

修复后：

```text
listen EACCES 消失
```

---

### 当前剩余问题

当前错误：

```text
failed to initialize sqlite state runtime
migration 1 was previously applied but has been modified
under /home/dm/.codex
```

最可疑文件：

```bash
/home/dm/.codex/state_5.sqlite
```

推荐下一步：

```bash
pkill -f codex || true

TS=$(date +%Y%m%d-%H%M%S)
cp -a ~/.codex ~/.codex.backup-$TS
mkdir -p ~/.codex/recover-$TS

mv ~/.codex/state_5.sqlite* ~/.codex/recover-$TS/ 2>/dev/null || true
```

然后：

```text
Ctrl + Shift + P
Remote-SSH: Kill VS Code Server on Host
重新连接服务器
打开 Codex
```

如果仍失败，再执行：

```bash
TS=$(date +%Y%m%d-%H%M%S)
mkdir -p ~/.codex/recover-$TS

mv ~/.codex/logs_*.sqlite* ~/.codex/recover-$TS/ 2>/dev/null || true
```

最后才考虑：

```bash
TS=$(date +%Y%m%d-%H%M%S)
mkdir -p ~/.codex/recover-$TS

mv ~/.codex/*.sqlite* ~/.codex/recover-$TS/ 2>/dev/null || true
```

但始终保留：

```text
auth.json
config.toml
```

---

## 16. 总排查流程图

```text
Codex 卡住 / 云朵图标 / Thinking...
        |
        v
View → Output → Codex
        |
        v
先看第一条真实错误
        |
        +-- Token exchange failed 403？
        |       |
        |       +-- 服务器不能访问，只有本地能访问：
        |       |       本地登录 Codex
        |       |       复制 auth.json / config.toml
        |       |       配置 RemoteForward
        |       |       配置 HTTP_PROXY / HTTPS_PROXY / ALL_PROXY
        |       |       Kill VS Code Server
        |       |
        |       +-- 服务器已经 TUN 接管且 curl 正常：
        |               不要继续纠结代理
        |               继续看后续日志
        |
        +-- listen EACCES /tmp/codex-ipc？
        |       |
        |       +-- 检查：
        |       |       ls -ld /tmp/codex-ipc
        |       |
        |       +-- 修复：
        |               sudo chmod 1777 /tmp/codex-ipc
        |
        +-- failed to initialize sqlite state runtime？
                |
                +-- 备份：
                |       cp -a ~/.codex ~/.codex.backup-$TS
                |
                +-- 第一轮：
                |       mv ~/.codex/state_5.sqlite* ~/.codex/recover-$TS/
                |
                +-- 第二轮：
                |       mv ~/.codex/logs_*.sqlite* ~/.codex/recover-$TS/
                |
                +-- 第三轮：
                        mv ~/.codex/*.sqlite* ~/.codex/recover-$TS/
                        保留 auth.json 和 config.toml
```

---

## 17. 最终一句话总结

在 VS Code Remote-SSH 环境中使用 Codex 时，先区分“网络认证问题”和“本地运行时问题”。

如果服务器网络不通，可以用：

```text
本地登录 Codex
复制 auth.json / config.toml
SSH RemoteForward
服务器代理环境变量
重启 VS Code Remote Server
```

如果服务器网络已经通过 Clash Meta TUN 正常访问 OpenAI，则不要把问题继续归因于代理。

本次实际问题链路是：

```text
第一阶段：
/tmp/codex-ipc 没有写权限
导致 listen EACCES

第二阶段：
~/.codex/state_5.sqlite migration 校验不一致
导致 sqlite state runtime 初始化失败
```

处理顺序应该是：

```text
看 Codex 日志
先修 /tmp/codex-ipc 权限
再处理 ~/.codex 下的 SQLite 状态库
尽量保留 auth.json 和 config.toml
```