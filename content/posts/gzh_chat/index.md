---
title : "花2毛钱给微信公众号接入阿里云通义千问大模型" 
date : "2024-03-01T19:28:02+08:00" 
lastmod : "2024-03-01T19:28:02+08:00" 
tags : ["大模型"] 
categories : ["技术"]
draft : false
featuredImage : /images/posts/gzh_chat/featuredImage.jpg
summary : '该文指导如何搭建一个微信公众号机器人使用阿里云通义千问模型，步骤包括注册和配置域名、vercel部署代码、设置公众号服务器配置，并提供了相关步骤和所需资源。'
---

## 前期准备

- github账号
- vercel账号
- 微信公众号
- nic.ua账号
- 阿里云账号
- Telegram账号
- cloudflare账号
- 一个外币卡

## 操作流程

### 注册域名

1. 注册nic.ua账号，申请免费域名，注册需要绑定信用卡，验证扣费1乌克兰格里夫纳约0.19人民币
    1. 申请的免费域名需要激活，推荐使用Telegram进行激活，注意Telegram注册的账号与nic.ua注册的手机号一致
    2. 在cloudflare中添加新申请的域名，在nic.ua中修改NS服务器到cloudflare
    3. 参考：[永久免费域名PP.UA最新注册指南](https://zhuanlan.zhihu.com/p/630011467)
2. 嫌麻烦，可以直接在阿里云/Godaddy等网站买个最便宜的域名

### 开通通义千问

1. 注册阿里云账号，开通[模型服务灵积](https://dashscope.console.aliyun.com/overview)服务
2. 服务开通成功后，在左侧菜单找到管理中心->API-KEY管理，创建apiKey备用

### vercel部署

1. 注册github账号后，fork[qw-wechat-vercel](https://github.com/LuhangRui/qw-wechat-vercel)项目到自己仓库
2. 注册vercel账号后，创建项目，选择Import Git Repository从github仓库导入。在Environment Variables选项卡，增加环境变量把下面的变量一项一项的加进去
    | 变量| 说明                             |
    | ---- |  -------------------------------- |
    |API_KEY=sk-xxxx|阿里云中创建的apikey|
    |KEYWORD_REPLAY={"测试":"关键词回复"}|自定义json字符串|
    |API_MODEL=qwen-72b-chat|大模型类型|
    |WX_TOKEN=53acb98d1dac49969b45797129f504f8|32位字符，后续公众号服务器配置会用到|
    |SUBSCRIBE_REPLY=欢迎关注，我已经接入了阿里千问智能AI，对我说句哈喽试试吧|关注自动回复|

3. 待部署完成后，点击ADD Domain，把你的域名填上去就好了，会自动加https。提示添加解析时选择只添加域名
    1. 在cloudflare中对应域名下添加DNS解析，添加A记录将域名指向76.223.126.88。
    2. ![Add A](/images/posts/gzh_chat/1.png)
    3. 参考：[Vercel应用绑定自己的域名](https://blog.tangly1024.com/article/vercel-domain)
    4. 访问https://你的域名/api/spark-wechat 页面输出failed，即为部署成功
    5. 访问https://你的域名 页面404，是因为项目里没有部署页面，所以访问会404

### 配置公众号

1. 注册公众号，个人订阅号就行
2. 后台管理页面上找到设置与开发-基本配置-服务器配置，点修改
3. 服务器地址url：https://你的域名/api/spark-wechat
4. TOKEN为第4步中使用的WX_TOKEN，
5. EncodingAESKey随机生成(不用这一项)
6. 选明文模式就好了，提交会提示token验证成功，然后点启用服务器配置。

## 公众号测试

![hello](/images/posts/gzh_chat/2.jpg)

![Write a poetry](/images/posts/gzh_chat/3.jpg)

![who is Lu Xun](/images/posts/gzh_chat/4.jpg)
