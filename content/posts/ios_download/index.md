---
title : "苹果手机离线下载方案" 
date : "2024-03-15T11:14:27+08:00" 
lastmod : "2024-03-15T11:14:27+08:00" 
tags : ["快捷指令","aria2"] 
categories : ["技术"]
draft : false
featuredImage : /images/posts/ios_download/featuredImage.jpg
summary : '本文章提供了一个在苹果手机上实现离线下载和实时观看电影的方案'
---

## 背景

平时手机不方便在线观看电影，最近看到一个方案可以解决我的问题，手动部署下。

整体方案：

1. 快捷指令发送RPC请求到aria2服务器实现离线下载
2. filebrowser实现下载文件管理，通过filebrowser分享功能获取分享链接
3. 手机上使用VLC通过分享链接观看视频

## 准备工作

1. 1台服务器，系统用的ubuntu 22.04
2. 1个域名,配置好dns
    1. ![dns](/images/posts/ios_download/dns.png)
    2. 我使用cloudflare管理域名，配置好dns后在SSL/TLS选择中选择Full模式
3. 安装docker和docker-compose
    1. apt-get update
    2. apt install docker.io docker-compose
4. 安装caddy2
    1. sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
    2. curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
    3. curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
    4. sudo apt update
    5. sudo apt install caddy
    6. sudo systemctl enable --now caddy
    7. [caddy2官方安装文档](https://caddy2.dengxiaolong.com/docs/install)

## 安装filebrowser

- 基于开源项目[filebrowser-docker](https://github.com/hurlenko/filebrowser-docker)采用docker-compose.yml安装，配置如下：

```plaintext
version: "3"

services:
  filebrowser:
    image: hurlenko/filebrowser
    ports:
      - 8080:8080
    volumes:
      - ./data:/data
      - ./config:/config
    environment:
      - FB_BASEURL=/filebrowser
    restart: always
```

- 在docker-compose.yml 同级目录创建data，config 目录
- docker-compose up -d运行
- 配置域名，编辑/etc/caddy/Caddyfile文件，添加如下内容后，systemctl restart caddy重启caddy

```plaintext
file.你的域名 {
    reverse_proxy 127.0.0.1:8080
}
```

- 在网页中访问file.你的域名 即可访问，登录密码为admin/admin 自行修改密码

到此filebrower安装完成，接下来安装aria2和ariang

## 安装aria2和ariang

- 基于开源方案[Aria2-Pro-Docker](https://github.com/P3TERX/Aria2-Pro-Docker)采用docker-compose.yml安装, 配置文件如下

```plaintext
version: "3.8"

services:

  Aria2-Pro:
    container_name: aria2-pro
    image: p3terx/aria2-pro
    environment:
      - PUID=65534
      - PGID=65534
      - UMASK_SET=022
      - RPC_PORT=6800
      - LISTEN_PORT=6888
      - DISK_CACHE=64M
      - IPV6_MODE=false
      - UPDATE_TRACKERS=true
      - CUSTOM_TRACKER_URL=
      - TZ=Asia/Shanghai
      - RPC_SECRET=设置你的KEY
    volumes:
      - ./aria2-config:/config
      - /home/ubuntu/filebrowser/data:/data/downloads   ## 此处/home/ubuntu/filebrowser/data修改为你的filebrower绑定的data路径
# If you use host network mode, then no port mapping is required.
# This is the easiest way to use IPv6 networks.
#    network_mode: host
    network_mode: bridge
    ports:
      - 6800:6800
      - 6888:6888
      - 6888:6888/udp
    restart: unless-stopped
# Since Aria2 will continue to generate logs, limit the log size to 1M to prevent your hard disk from running out of space.
    logging:
      driver: json-file
      options:
        max-size: 1m

# AriaNg is just a static web page, usually you only need to deploy on a single host.
  AriaNg:
    container_name: ariang
    image: p3terx/ariang
    command: --port 6880 --ipv6
#    network_mode: host
    network_mode: bridge
    ports:
      - 6880:6880
    restart: unless-stopped
    logging:
      driver: json-file
      options:
        max-size: 1m
```

- docker-compose up -d运行
- 配置域名，编辑/etc/caddy/Caddyfile文件，添加如下内容后，systemctl restart caddy重启caddy

```plaintext
aria2.你的域名 {
    reverse_proxy 127.0.0.1:6880
}
rpc.你的域名 {
    reverse_proxy 127.0.0.1:6800
}

```

- 通过aria2.你的域名即可在网页中访问ariang
- 在AiraNg设置-RPC中修改配置
    1. Aria2 RPC地址修改为：rpc.你的域名，端口改成443
    2. Aria2 RPC 密钥修改为：docker-compose.yml中RPC_SECRET配置的值

到此aria2和ariang 安装完成。

## 配置快捷指令

- 手机网页打开[RPC 请求快捷指令](https://www.icloud.com/shortcuts/7483b8cec7484c0f98b72882d0f1e3e2)安装指令
- 配置RPC请求域名和密钥
