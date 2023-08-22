---
title: 安装 Docker 后服务器 ping 不通了
date: 2019-10-10 19:40:00
tags: [Docker]
category: 瞎折腾
---
# 背景

这两天遇到一个奇怪的问题：开开心心连上服务器，准备跑上自己的服务。为了方便部署，当然是先安装 Docker 了。开开心心的安装，写 Dockerfile，写完了跑起来！访问，懵逼了，无法访问，直接 ping 不通了。因为是突然发现这个问题，没有太怀疑 Docker 的问题，只是把 Docker 关了后发现依旧不行，就排除 Docker 了。

# 现象

出问题后可以 ssh 登陆服务器，但是服务器 ping 不通，也不能访问。

# 后续

贼心不死，再申请了个服务器，这次学聪明了，先 ping 一下，发现可以 ping 通，然后装上 Docker 再 ping 一下，凉了！问题定位，就是 Docker 的锅！那怎么办呢？

确定是 Docker 的锅就好办了，一番 Google，发现了有相似的问题。

> 一句话解释：原来是 Docker 和宿主机的网段冲突了，改了网段就好了。

Docker 容器网络默认使用的是 bridge 桥接模式，一般容器会使用 `daemon.json` 中定义的虚拟网桥来与宿主机进行通信。

下面分别是 Linux 和 Mac 修改 Docker 默认网段的方法。

# 修改方法

## Linux 修改 Docker 默认网段

### 第一步 删除原有配置

```shell
sudo service docker stop
sudo ip link set dev docker0 down
sudo brctl delbr docker0
sudo iptables -t nat -F POSTROUTING
```

### 第二步 创建新的网桥

```shell
sudo brctl addbr docker0
sudo ip addr add 172.17.10.1/24 dev docker0
sudo ip link set dev docker0 up
```

### 第三步 配置 Docker 的文件

```shell
vi /etc/docker/daemon.json
-bash-4.2$ cat /etc/docker/daemon.json
{
	"bip":"172.17.10.1/24"
}
# 注意就是将 bip 的值改成新设置的网段
```

## Mac 修改

打开 Preferences -> Advanced，修改 Docker subnet 配置 `172.17.10.1/24`，从而避免网段冲突的问题。
