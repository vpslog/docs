---
title: "免费获取次级域名 2level.link"
description: ""
excerpt: ""
date: 2023-07-14T08:05:49Z
lastmod: 2023-07-14T08:05:49Z
draft: false
weight: 50
images: ["2level.png"]
categories: ["DEV"]
tags: ["DNS","Domains"]
contributors: ["Gray"]
pinned: false
homepage: false
---

前两篇文章介绍次级域名分发系统的开发，本文来总的介绍一下如何使用 2leve.免费领取次级域名。

## Introduction

次级域名（Subdomain）是在一级域名之下创建的子域名。它是将一级域名进一步划分为更具体的部分。例如 `www.example.com`中，`www`就是一个常见的次级域名，`example.com` 为一级域名，`.com`为顶级域名，或者说域名后缀。

需要注意，在本服务中，`www`，`*`，以及空值都是不允许注册，默认保留的。`*`实际上代表通配符域名，只要某个记录没有解析，就都会被解析到这一记录下。

由于基于 Cloudfalre API 开发，本服务默认支持开启 Cloudflare CDN 加速。

## Usage

访问 [分发系统](https://domains.vpslog.net/)，默认情况下需要进行登录（跳转登录界面）。首先关注[验证 BOT](https://t.me/vpslog_distribute_bot)。发送 `/start` 命令，复制 TG ID 并填入登陆界面中，然后完成人机验证，选择`SEND VERIFY CODE`，从 BOT 中获取验证码完成登录。

![Alt text](image.png)


![Alt text](image-1.png)


![Alt text](image-2.png)


![Alt text](image-3.png)

**注意：本服务仍在测试中，如果有问题请转到 [群组讨论](https://t.me/vpalogchat)**


