---
title: "从零开始构建一个个人影音服务器"
description: ""
excerpt: ""
date: 2023-07-14T13:39:32Z
lastmod: 2023-07-14T13:39:32Z
draft: false
weight: 50
images: []
categories: ["Blog"]
tags: ["Emby","NAS"]
contributors: ["Gray"]
pinned: false
homepage: false
---


最近购入了一些 4T 硬盘无限流量的大盘鸡，最近又没有什么好项目，就先拿着跑一个影音服务器。

暂定计划是 qbit+alist+emby。之后再看能不能加其他的。


## 基础准备

格式化硬盘，创建、挂载硬盘到 `/data` 文件夹。

```
fdisk -l
df -a
mkfs.ext4 /dev/vda
mkdir /data
mount /dev/vda /data
```

## qbit

```
docker run -d --name=qbittorrent -e WEBUI_PORT=8080 -p 8080:8080 -p 6881:6881 -p 6881:6881/udp   -v /data:/downloads lscr.io/linuxserver/qbittorrent:latest
```
访问地址 8080，默认账号 admin 密码 adminadmin。建议是改一个密码。默认将文件下载到宿主机 `/data` 文件夹下。

![Alt text](image.png)


![Alt text](image-1.png)


![Alt text](image-2.png)

![Alt text](image-3.png)

## alist

``` 
docker run -d --restart=always -v /etc/alist:/opt/alist/data  -v /data:/share -p 5244:5244 --name="alist" xhofe/alist:latest
# 查看密码
docker logs alist
```

端口 `5244`。

复制下面密码到界面。

![Alt text](image-4.png)

## emby



```
docker run -d  --name embyserver  -v /data:/mnt/share -p 8096:8096  -p 8920:8920  emby/embyserver:latest
```

端口 `8096`。需要初始化账号密码一级存储库（`/mnt/share`）
