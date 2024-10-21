---
title : "自建FreshRSS和RSSHub服务"
date : "2024-10-21T10:50:07+08:00" 
lastmod : "2024-10-21T10:50:07+08:00" 
tags : ["RSS","RSSHub","FreshRSS"] 
categories : ["技术"]
draft : false
featuredImage : /images/posts/rss/featuredImage.jpg
summary : '自建FreshRSS服务，使用vercel部署RSSHub服务提升使用体验'
---

之前使用他人提供的[FreshRSS](https://github.com/FreshRSS/FreshRSS)服务，因为网站证书过期导致使用不便。有一台可以使用的服务器，就考虑自建FreshRSS服务。同时日常使用官方[RSSHub](https://github.com/DIYgod/RSSHub)服务过程中，订阅源非常不稳定，经常不可用。发现vercel可以部署RSSHub，完成相应部署后，整体使用感受大幅提升，记录下过程，方便以后使用。 

## FreshRSS dockercompose 部署
1. 自备一个服务器
2. 安装docker 和 docker-compose, 配置docker-compose.yml
3. 启动指令docker-compose up -d
4. 配置caddy配置文件caddyfile,方便使用域名访问
5. 通过域名访问freshRSS,创建管理员账户和配置数据库
    1. 数据库信息对应Docker Compose 配置文件中的 POSTGRES_USER、POSTGRES_PASSWORD、POSTGRES_DB、表前缀任意填
    2. 主机名要稍微注意一下，要用容器的 IP(docker inspect <container id>)
6. 手机上使用Reeder登录FreshRSS
    1. freshRSS 设置->认证->允许API访问
    2. freshRSS 设置->账户->API管理->设置AIP密码
    3. Reeder中使用API管理下的地址+用户名+API密码进行登录
7. [Freshrss参考文档](https://blog.ichr.me/post/docker-freshrss-setup/)

```yaml
version: "3"

services:
  freshrss-db:
    image: postgres:latest
    container_name: freshrss-db
    hostname: freshrss-db
    restart: unless-stopped
    volumes:
      - freshrss-db:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: freshrss
      POSTGRES_PASSWORD: freshrss
      POSTGRES_DB: freshrss

  freshrss-app:
    image: freshrss/freshrss:latest
    container_name: freshrss-app
    hostname: freshrss-app
    restart: unless-stopped
    ports:
      - "7080:80"
    depends_on:
      - freshrss-db
    volumes:
      - ./data:/var/www/FreshRSS/data
      - ./extensions:/var/www/FreshRSS/extensions
    environment:
      CRON_MIN: '*/45'
      TZ: Asia/Shanghai

volumes:
  freshrss-db:
```

## Vercel RSSHub 部署
1. 因为vercel nodejs 最高版本只能选择 20.x，导致无法使用RSSHub master分支部署，使用legacy 分支进行部署,操作流程见参考文档
2. legacy 相比master分支缺少很多路由，需要自己按需添加路由
    1. 在master分支找到缺少的路由，复制到legacy分支，修改文件名尾缀ts为js,参考其他路由调整代码格式
    2. 在\RSSHub\lib\router.js中添加新增的路由地址
    3. 如果需要使用环境变量，在\RSSHub\lib\config.js中添加，并在vercel 对应的项目->设置->Environment Variables添加相同环境变量
3. 使用vercel 部署的好处有
    1. 国内外网站的信息都可以直接抓取
    2. 无需服务器可以免费使用
4. [vercel rsshub部署参考文档](https://cloud.tencent.com/developer/article/2432561)

## 其他
1. [caddy的安装参考文档](https://xiaoshame.github.io/posts/ios_download/)
2. [永久免费域名PP.UA最新注册指南](https://zhuanlan.zhihu.com/p/630011467)
