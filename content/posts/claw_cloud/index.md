---
title : "使用claw cloud免费部署wewe rss" 
date : "2025-07-08T17:08:56+08:00" 
lastmod : "2025-07-08T17:08:56+08:00" 
tags : ["白嫖","RSS"] 
categories : ["技术"]
draft : false
featuredImage : /images/posts/claw_cloud/featuredImage.jpg
summary : '互联网有很多免费的资源，合理利用可以省时省力省心'
---

本文记录使用[claw cloud](https://us-east-1.run.claw.cloud/)免费资源，部署[wewe_rss](https://github.com/cooderl/wewe-rss)的操作流程。

### 优势
1. claw cloud使用github账号(注册超过 180 天)登录，免费送5美元/月额度
2. 可以使用[dockerhub](https://hub.docker.com/)中镜像进行部署
3. 基于本文操作流程可以部署支持镜像部署的服务，例如：freshrss,qinglong等

### 操作步骤
1. 使用github账号登录，如果无法登录可以切换选择其他区域
2. 在dockerhub中寻找想部署服务的镜像,查看对应服务的启动脚本
```plaintext
docker run -d \
  --name wewe-rss \
  -p 4000:4000 \
  -e DATABASE_TYPE=sqlite \
  -e AUTH_CODE=123567 \
  -v $(pwd)/data:/app/data \
  cooderl/wewe-rss-sqlite:latest
```
3. 从上述脚本可以获取三个信息
    1. 镜像名，cooderl/wewe-rss-sqlite
    2. 环境变量DATABASE_TYPE=sqlite和AUTH_CODE=123567
    3. 挂载目录/data
    4. 端口
4. 在claw cloud中点击App Launchpad -> Create App，然后按照下图将第三点中信息填入相应位置，点击部署即可。
    1. ![部署](/images/posts/claw_cloud/1.png) 
5. cpu和内存根据需要进行调节，左边有一天预估费用，保证月费用不超5美元，即可长时间使用

### 总结
互联网上有很多免费的资源，利用好可以极大方便个人日常使用。推荐几个我使用的服务，vercel/cloudflare/nic.ua/aws.amazon.com。