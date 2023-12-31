---
title : '文章自动同步到公众号'
date : 2023-10-24T19:09:53+08:00
draft : false
tags : [CI,效率]
categories : [技术]
toc : true
summary : '通过将Markdown文件转换为HTML文件，利用微信开发者API将文章同步到公众号的草稿箱中。文章还提到了一些操作记录和运行前提，但是存在的不足是无法直接发布文章，只能通过手动群发让粉丝看到。'
---
### 背景

受[如何无痛苦更新公众号](https://catcoding.me/p/publish-to-wechat/)影响，开始思考如何应用到我的日常文章编写中。

### 操作记录

完整代码已同步[个人仓库](https://github.com/xiaoshame/script)，修正pickle运行问题，代码高亮后无滚动条问题。

脚本原理：

1. 利用markdown库实现md文件转换成html文件
2. 利用codehilite 实现代码高亮
3. 利用微信开发者API，基于werobot库完成图片资源上传和文章同步到草稿箱

运行前提：

1. 在公众号设置与开发-基本配置，获取个人APP_ID和APP_SECRET填入脚本
2. 在公众号设置与开发-基本配置，将运行脚本的机器出口IP加入公众号白名单
3. 修改blog_path为需要同步的文章地址

不足：

可以对文章进行发布，腾讯公众号发布的定义是粉丝无法看到文章。文章群发后粉丝才能看到，但是腾讯开发者API未提供群发接口。
