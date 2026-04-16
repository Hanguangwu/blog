---
title: Ubuntu24.04LTS安装Docker
description: 本文介绍Ubuntu24.04LTS安装Docker及配置阿里云源。
date: 2025-04-16T12:34:25-08:00
draft: false
categories:
- APP
tags:
- Docker
- Ubuntu
---

# Ubuntu 24.04.1 LTS 安装 Docker（国内源版，全程可访问）

https://www.cnblogs.com/boradviews/p/19329796

以下是适配 Ubuntu 24.04 的 Docker 安装命令，全程使用阿里云国内源，避免官方源访问慢/超时问题，步骤简洁且可直接复制执行：

## 一、前置准备：卸载旧版本（避免冲突）

若之前装过 Docker 旧版本，先清理：

```bash
sudo apt remove -y docker docker-engine docker.io containerd runc
sudo rm -rf /var/lib/docker /var/lib/containerd
```

## 二、安装必要依赖

```bash
# 更新系统包索引
sudo apt update && sudo apt upgrade -y

# 安装证书、curl、gnupg 等基础依赖
sudo apt install -y ca-certificates curl gnupg lsb-release
```

## 三、添加阿里云 Docker 官方 GPG 密钥

```bash
# 创建密钥存储目录
sudo mkdir -p /etc/apt/trusted.gpg.d

# 导入阿里云 Docker GPG 密钥（避免签名验证失败）
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg
```

## 四、添加阿里云 Docker 国内源

```bash
# 添加适配 Ubuntu 24.04（noble）的阿里云 Docker 源
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/trusted.gpg.d/docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 再次更新包索引（加载新添加的 Docker 源）
sudo apt update
```

## 五、安装 Docker 引擎（最新稳定版）

```bash
# 安装 Docker CE、Containerd、Docker Compose 插件
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 六、启动并设置开机自启

```bash
# 启动 Docker 服务
sudo systemctl start docker

# 设置开机自启
sudo systemctl enable docker

# 验证 Docker 状态（输出 active (running) 即正常）
sudo systemctl status docker
```

## 七、可选：配置用户权限（避免每次用 sudo）

默认只有 root/ sudo 用户能操作 Docker，将当前用户加入 docker 组：

```bash
# 将当前用户加入 docker 组
sudo usermod -aG docker $USER

# 生效组配置（无需重启，重新登录终端即可）
newgrp docker

# 验证权限（无需 sudo 能执行即成功）
docker ps
```

## 八、关键优化：配置 Docker 镜像加速（国内拉取镜像快）

编辑 Docker 配置文件，添加阿里云镜像加速器：

```bash
# 创建配置目录
sudo mkdir -p /etc/docker

# 写入加速配置（阿里云加速器，也可替换为网易/腾讯源）
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com",
    "https://mirror.aliyuncs.com"
  ]
}
EOF

# 重启 Docker 使加速配置生效
sudo systemctl daemon-reload
sudo systemctl restart docker

# 验证加速器是否生效
docker info | grep -i mirror
```

输出中能看到配置的镜像地址，说明加速生效。

## 九、验证安装成功

```bash
# 运行测试容器（hello-world），验证 Docker 功能正常
docker run hello-world
```

若输出 “Hello from Docker!” 相关内容，说明安装+配置全部成功。

## 常见问题解决

1. **安装时报 “签名无效”**：  
    重新执行第三步的 GPG 密钥导入命令，确保密钥文件正确生成。
2. **docker ps 提示 “permission denied”**：  
    确认已执行第七步的用户组配置，或重新登录终端（如 SSH 连接需重新连接）。
3. **启动 Docker 失败**：  
    检查 containerd 状态：`sudo systemctl status containerd`，若未启动则执行 `sudo systemctl start containerd` 后再重启 Docker。














