---
layout: post
title: "Docker 容器迁移与环境重构全流程（含CUDA与PyTorch安装）"
date: 2025-04-24 09:30:00 +0800
categories: Docker Linux MLEnv
---

# Docker 容器迁移与环境重构教程

命运多舛的实验环境搭建，防止下次服务器崩，遂写个page供自己参考（求求了不要崩了，这次搞了半个月！！！）
本教程讲解如何将一个已经运行的WSL环境打包导出为镜像，并迁移至其他服务器重新部署，同时在目标环境中安装 CUDA 与 PyTorch，适用于 WSL->Linux 环境。

流程总结：
1. docker export [CONTAINER_ID] > .tar
2. md5sum .tar > .md5
3. scp 传输文件
4. md5sum -c 校验
5. docker import .tar
6. docker run 创建新容器
7. 安装 Miniconda
8. 安装 PyTorch + CUDA
9. 验证环境

## 一、容器导出：从运行中容器生成可迁移文件

### 1.1 查看正在运行的容器

```bash
docker ps
```

示例输出中找到目标容器的 CONTAINER ID，如：

```bash
CONTAINER ID   IMAGE           ...   NAMES
04700a37255e   zy-wsl-ubuntu   ...   gsy_mind
```

### 1.2 导出容器为 .tar 文件
该命令会将整个容器文件系统打包成可传输的归档文件。

```bash
docker export 04700a37255e > zy-wsl-ubuntu.tar
```

### 1.3 校验打包文件完整性

```bash
md5sum zy-wsl-ubuntu.tar > zy-wsl-ubuntu.md5
```

## 二、迁移镜像文件到目标服务器
### 2.1 使用SCP将.tar文件复制至新服务器

```bash
scp zy-wsl-ubuntu.tar user@remote_ip:/home/user/
scp zy-wsl-ubuntu.md5 user@remote_ip:/home/user/
```

### 2.2 在目标服务器验证文件一致性

```bash
md5sum -c zy-wsl-ubuntu.md5
```

## 三、镜像导入与容器创建
### 3.1 加载导出的 tar 文件为 Docker 镜像

```bash
sudo docker import zy-wsl-ubuntu.tar zy-wsl-ubuntu:latest
```

### 3.2 基于导入的镜像创建新的容器
可根据需要添加其他端口或卷挂载

```bash
sudo docker run -itd --privileged -p 10424:22 --name gsy_mind --gpus all -e NVIDIA_DRIVER_CAPABILITYES=compute,utility -e NVIDIA_VISIBLE_DEVICES=all zy-wsl-ubuntu:latest /bin/bash 
```

## 四、安装 CUDA、PyTorch 环境（容器内执行）
### 4.1 进入容器

```bash
sudo docker exec -it gsy_mind /bin/bash
```
### 4.2 下载并执行miniconda安装脚本

```bash
apt update && apt install -y wget
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh

//环境变量生效
source ~/.bashrc 

//检查conda是否安装成功
conda --version
```

### 4.3 创建新的conda环境

```bash
conda create -n mind_env python=3.9 -y
conda activate mind_env
```

### 4.4 安装 PyTorch 和 CUDA 支持

```bash
conda install pytorch torchvision pytorch-cuda=12.4 -c pytorch -c nvidia
```

### 验证 PyTorch 和 GPU 正常使用
启动 Python 交互环境测试：

```bash
python
```
输入以下命令：

```bash
import torch
print(torch.__version__)
print(torch.cuda.is_available())
print(torch.cuda.get_device_name(0))
```


【注意：】zyg的容器中，可以直接使用/home/zy/miniconda3/envs/pencil路径下环境，不再需要创建新的环境。

```bash
conda activate /home/zy/miniconda3/envs/pencil/
```

代码自带的容器端口号需要修改为本服务器的容器端口号！

```bash
//查看SSH配置文件位置
cd /etc/ssh/   
ls

//修改SSHServer配置，指定新端口号、运行root登录
vi /etc/ssh/sshd_config

//重启SSH服务配置生效
service ssh restart

//验证是否在监听
netstat -tunpl
```