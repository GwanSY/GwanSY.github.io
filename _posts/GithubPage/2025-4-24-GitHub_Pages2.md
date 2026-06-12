---
layout: post
title: "Docker 容器迁移与环境重构全流程（含CUDA与PyTorch安装）"
date: 2025-04-24 09:30:00 +0800
categories: Docker Linux MLEnv
---

# Docker 容器迁移与环境重构教程

命运多舛的实验环境搭建，防止下次服务器崩，遂写个 page 供自己参考。

本教程讲解如何将一个已经运行的 WSL / Docker 环境打包导出为镜像，并迁移至其他服务器重新部署，同时在目标环境中安装 CUDA 与 PyTorch，适用于 WSL -> Linux 环境迁移。

适用于以下情况：

- 已经在旧服务器或 WSL 中配置好了实验环境
- 希望将当前 Docker 容器完整迁移到新服务器
- 希望在新服务器上恢复原有环境
- 需要重新安装 Miniconda、PyTorch 与 CUDA
- 需要配置 SSH 端口，方便远程连接容器

---

## 流程总结

1. `docker export [CONTAINER_ID] > .tar`
2. `md5sum .tar > .md5`
3. 使用 `scp` 传输文件
4. `md5sum -c` 校验文件完整性
5. `docker import .tar`
6. `docker run` 创建新容器
7. 安装 Miniconda
8. 安装 PyTorch + CUDA
9. 配置 SSH
10. 验证环境

---

## 一、容器导出：从运行中容器生成可迁移文件

### 1.1 查看正在运行的容器

```bash
docker ps
```

示例输出：

```bash
CONTAINER ID   IMAGE           ...   NAMES
04700a37255e   zy-wsl-ubuntu   ...   gsy_mind
```

其中 `04700a37255e` 即为目标容器的 `CONTAINER ID`。

---

### 1.2 导出容器为 `.tar` 文件

该命令会将整个容器文件系统打包成可传输的归档文件。

```bash
docker export 04700a37255e > zy-wsl-ubuntu.tar
```

---

### 1.3 校验打包文件完整性

为了避免传输过程中出现文件损坏，建议生成 MD5 校验文件。

```bash
md5sum zy-wsl-ubuntu.tar > zy-wsl-ubuntu.md5
```

---

## 二、迁移镜像文件到目标服务器

### 2.1 使用 SCP 将 `.tar` 文件复制至新服务器

```bash
scp zy-wsl-ubuntu.tar user@remote_ip:/home/user/
scp zy-wsl-ubuntu.md5 user@remote_ip:/home/user/
```

其中：

- `user`：目标服务器用户名
- `remote_ip`：目标服务器 IP 地址
- `/home/user/`：目标服务器上的保存路径

---

### 2.2 在目标服务器验证文件一致性

进入目标服务器后执行：

```bash
md5sum -c zy-wsl-ubuntu.md5
```

如果输出类似：

```bash
zy-wsl-ubuntu.tar: OK
```

说明文件传输完整。

---

## 三、镜像导入与容器创建

### 3.1 加载导出的 tar 文件为 Docker 镜像

```bash
sudo docker import zy-wsl-ubuntu.tar zy-wsl-ubuntu:latest
```

其中：

- `zy-wsl-ubuntu.tar`：导出的容器文件
- `zy-wsl-ubuntu:latest`：导入后的镜像名称和标签

---

### 3.2 基于导入的镜像创建新的容器

可根据需要添加端口映射、卷挂载或 GPU 参数。

```bash
sudo docker run -itd \
  --privileged \
  -p 10424:22 \
  --name gsy_mind \
  --gpus all \
  -e NVIDIA_DRIVER_CAPABILITIES=compute,utility \
  -e NVIDIA_VISIBLE_DEVICES=all \
  zy-wsl-ubuntu:latest \
  /bin/bash
```

注意：

- `-p 10424:22` 表示将宿主机的 `10424` 端口映射到容器内的 `22` 端口
- `--gpus all` 表示允许容器使用所有 GPU
- `--privileged` 用于给予容器更高权限
- `NVIDIA_DRIVER_CAPABILITIES` 用于指定 NVIDIA 驱动能力

---

## 四、安装 CUDA、PyTorch 环境

以下操作在容器内执行。

### 4.1 进入容器

```bash
sudo docker exec -it gsy_mind /bin/bash
```

---

### 4.2 下载并执行 Miniconda 安装脚本

```bash
apt update && apt install -y wget

wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

bash Miniconda3-latest-Linux-x86_64.sh
```

安装完成后，使环境变量生效：

```bash
source ~/.bashrc
```

检查 Conda 是否安装成功：

```bash
conda --version
```

---

### 4.3 创建新的 Conda 环境

```bash
conda create -n mind_env python=3.9 -y
conda activate mind_env
```

---

### 4.4 安装 PyTorch 和 CUDA 支持

```bash
conda install pytorch torchvision pytorch-cuda=12.4 -c pytorch -c nvidia
```

---

### 4.5 验证 PyTorch 和 GPU 是否正常使用

启动 Python 交互环境：

```bash
python
```

输入以下命令：

```python
import torch

print(torch.__version__)
print(torch.cuda.is_available())
print(torch.cuda.get_device_name(0))
```

如果 `torch.cuda.is_available()` 输出：

```python
True
```

并且能够正确显示 GPU 名称，则说明 PyTorch 与 CUDA 环境配置成功。

---

## 五、使用已有 Conda 环境

如果容器中已经存在可用环境，例如 zyg 的容器中已有：

```bash
/home/zy/miniconda3/envs/pencil
```

则可以直接激活该环境，不需要重新创建新的 Conda 环境。

```bash
conda activate /home/zy/miniconda3/envs/pencil/
```

注意：

代码中自带的容器端口号需要修改为当前服务器实际使用的容器端口号。

---

## 六、配置 SSH 服务

如果需要通过 SSH 连接容器，需要检查并修改 SSH 服务配置。

### 6.1 查看 SSH 配置文件位置

```bash
cd /etc/ssh/
ls
```

---

### 6.2 修改 SSH Server 配置

```bash
vi /etc/ssh/sshd_config
```

根据需要修改以下内容：

```text
Port 22
PermitRootLogin yes
```

其中：

- `Port 22`：容器内部 SSH 端口
- `PermitRootLogin yes`：允许 root 用户通过 SSH 登录

如果宿主机已经通过 Docker 映射了端口，例如：

```bash
-p 10424:22
```

那么外部连接时应使用宿主机端口 `10424`。

---

### 6.3 重启 SSH 服务

```bash
service ssh restart
```

---

### 6.4 验证端口是否正在监听

```bash
netstat -tunpl
```

如果能看到 SSH 服务正在监听对应端口，则说明 SSH 配置生效。

---

## 七、最终检查

完成迁移后，建议依次检查以下内容：

### 7.1 容器是否正常运行

```bash
sudo docker ps
```

### 7.2 是否可以进入容器

```bash
sudo docker exec -it gsy_mind /bin/bash
```

### 7.3 Conda 是否可用

```bash
conda --version
```

### 7.4 PyTorch 是否可用

```bash
python -c "import torch; print(torch.__version__)"
```

### 7.5 CUDA 是否可用

```bash
python -c "import torch; print(torch.cuda.is_available())"
```

### 7.6 GPU 是否可被 PyTorch 识别

```bash
python -c "import torch; print(torch.cuda.get_device_name(0))"
```

### 7.7 SSH 服务是否正常

```bash
service ssh status
netstat -tunpl
```

---

## 八、常见注意事项

### 8.1 `docker export` 和 `docker save` 的区别

本文使用的是：

```bash
docker export
```

它导出的是容器文件系统，不包含镜像历史、环境变量、CMD、ENTRYPOINT 等镜像元数据。

导入时使用：

```bash
docker import
```

如果希望完整保存镜像历史和元数据，应使用：

```bash
docker save
docker load
```

不过对于“把当前容器文件系统整体搬到新服务器”这一需求，`docker export` / `docker import` 通常已经足够。

---

### 8.2 GPU 参数需要目标服务器支持 NVIDIA Docker

如果使用：

```bash
--gpus all
```

目标服务器需要已经正确安装：

- NVIDIA Driver
- NVIDIA Container Toolkit
- Docker GPU 支持

否则容器可能无法识别 GPU。

---

### 8.3 端口号需要根据实际服务器修改

例如创建容器时使用：

```bash
-p 10424:22
```

则 SSH 连接时应使用：

```bash
ssh root@server_ip -p 10424
```

如果服务器上该端口已经被占用，需要换成其他端口。

---

## 总结

完整迁移流程如下：

```bash
docker ps

docker export 04700a37255e > zy-wsl-ubuntu.tar

md5sum zy-wsl-ubuntu.tar > zy-wsl-ubuntu.md5

scp zy-wsl-ubuntu.tar user@remote_ip:/home/user/
scp zy-wsl-ubuntu.md5 user@remote_ip:/home/user/

md5sum -c zy-wsl-ubuntu.md5

sudo docker import zy-wsl-ubuntu.tar zy-wsl-ubuntu:latest

sudo docker run -itd \
  --privileged \
  -p 10424:22 \
  --name gsy_mind \
  --gpus all \
  -e NVIDIA_DRIVER_CAPABILITIES=compute,utility \
  -e NVIDIA_VISIBLE_DEVICES=all \
  zy-wsl-ubuntu:latest \
  /bin/bash

sudo docker exec -it gsy_mind /bin/bash
```

进入容器后，根据需要安装 Miniconda、PyTorch、CUDA，并配置 SSH 服务即可。