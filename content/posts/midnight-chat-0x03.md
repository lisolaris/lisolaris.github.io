---
title: "深夜闲谈 0x03"
date: 2025-10-15T16:19:50+08:00
tags: ["2025-10", "杂谈", "深夜闲谈", "Cloudflare", "Hugo", "PaperMod"]
categories: ["深夜闲谈"]
toc: false
draft: true
---

### 0x03

已经很久没有写过博客了。其实也没什么特别想写的，单纯记录一下最近干了些什么（

---

### 使用Cloudflare Workers反向代理Docker Hub

由于某些神秘原因，在国内是无法直连Docker Hub的，并且各个公开镜像站也都消失了。那么如果要访问Docker Hub就有两种方式：为docker本身配置镜像地址，或为守护进程`dockerd`配置代理。

#### 自建Docker Hub反向代理并为docker配置

这种方法的原理是国内可以正常访问到Cloudflare的相关服务（虽然比较慢），而Cloudflare提供了Workers这样的云函数计算服务；但这里我们的计算量并不大，更多地是把Workers当成一个反向代理服务器使用。

如果不想折腾太多，则只需要在Cloudflare上注册开通Workers，之后直接将[cloudflare-docker-proxy](https://github.com/ciiiii/cloudflare-docker-proxy)部署到自己账户下的Workers就可以了。

##### 为反向代理的Workers配置其他路由实现复用

使用上面开源项目的默认配置会占用一整个Workers，如果我们还想用此域名做其他的事有两种方法：一是在Cloudflare后台为Workers配置路由，而是直接在Workers逻辑中实现不同路由访问不同的功能。第一种方法我暂时还没试过，此处介绍一下第二种方法。

首先我们可以将刚刚项目里的`src/index.js`导入到我们的Workers项目中，命名为docker.js;



需要注意的是，对使用免费计划的Workers, Cloudflare会限制此Workers单次连接能够使用的流量大小为100MB，因此我们自建的这个docker反向代理只能缓解有没有的问题，好不好用还是另说（

#### 为守护进程配置代理

如果嫌创建Cloudflare Workers太麻烦，在docker机器所在的主机或局域网内正好有设备开启了代理服务器功能，那么可以直接设置docker使用此代理去连接Docker Hub.

由`systemd`管理运行的服务并不会直接使用当前shell环境的代理环境变量，因此需要自己创建相关的配置文件并设置：

```sh
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo vim /etc/systemd/system/docker.service.d/proxy.conf
```

向配置文件中写入代理服务器设置：

```conf
[Service]
Environment="HTTP_PROXY=http://<username>:<password>@<host>:<port>/"
Environment="HTTPS_PROXY=http://<username>:<password>@<host>:<port>/"
Environment="NO_PROXY=localhost,127.0.0.1,.example.com"
```

记得把上面的`host`, `port`等替换成你自己的，如果代理服务器没有配置用户名和密码可以直接将`<username>:<password>@`删去。

之后需要重载daemon并重启`dockerd`:

```sh
sudo systemctl daemon-reload
sudo systemctl restart docker
```

这时再进行`docker pull`, 应该就没有问题了。

