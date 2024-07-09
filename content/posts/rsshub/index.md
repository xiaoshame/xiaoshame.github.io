---
title : "RSShub自制路由" 
date : "2024-07-09T10:33:07+08:00" 
lastmod : "2024-07-09T10:33:07+08:00" 
tags : ["RSS","RSShub"] 
categories : ["技术"]
draft : true
featuredImage : /images/posts/rsshub/featuredImage.jpg
summary : 'RSSHub路由和阅读等书源制作，都是对html信息的解析，区别是一个是服务端处理，一个是在客户端处理。'
---

平时主要使用RSS看资讯信息，日常使用RSSHub Radar浏览器插件查询网站RSS链接，很多网站都无法发现RSS链接，一度认为这个插件很鸡肋。最近研究了下，这个插件确实很鸡肋，核心是RSSHub这个开源项目，通过搭建自定义路由，将网站信息转换成RSS信息流。然后使用RSS 阅读器进行订阅阅读。

### 原理

1. 网站内容都是html信息，通过对网站html内容解析，然后按照RSS规范生成RSS文件。即可使用RSS 阅读器订阅阅读。
2. [RSSHub项目地址](https://github.com/DIYgod/RSSHub)，提供基础能力，将网站html内容解析后按照官方文档返回相应的结构。实现自定义路由能力。
3. 借助大模型，参考官方提供的路由，自己很简单完成网站路由制作。
4. 使用RSS阅读网站内容和书源工具阅读小说思想上一致的，一个对源网站数据的处理在服务端进行，一个在客户端进行。

### 制作路由

1. [开发路由官方文档](https://docs.rsshub.app/zh/joinus/)，参考官方文档下载配置好开发环境。
2. 打开想制作的网站F12,查看网站html结构，找到需要解析的元素。
3. 参考下述代码，最重要是handler函数，其他都是配置信息。handler中解析html标签，获取需要的信息。主要就是标题，链接，描述，发布时间。如果想在RSS阅读器中直接阅读完整的内容，可以将文章内容放在描述(description)中。
4. 开发过程中不懂的就问大模型，云厂商都提供了免费服务，能力都差不多。完成相应开发后，将代码部署到服务器上，配置好端口转发就可以通过域名订阅。
5. 如果使用官方提供的RSSHub服务，需要提交PR到RSSHub项目，等待审核通过后才能使用。
6. 可以阅读路由中他人制作的路由，发现感兴趣的网站，查看Route结构，使用官方服务，直接使用。

```js
import { Route } from '@/types';
import { parseDate } from '@/utils/parse-date';
import got from '@/utils/got';
import { load } from 'cheerio';

export const route: Route = {
    path: '/',
    categories: ['blog'],
    example: '/imhcg',
    radar: [
        {
            source: ['https://infos.imhcg.cn/'],
        },
    ],
    name: 'Engineering blogs',
    maintainers: ['xiaoshame'],
    handler,
    url: 'https://infos.imhcg.cn/',
};

async function handler() {
    const url = 'https://infos.imhcg.cn/';
    const response = await got({ method: 'get', url });
    const $ = load(response.data);

    const list = $('li')
        .map((i, e) => {
            const element = $(e);
            const title = element.find('a.title').text();
            const link = element.find('a.title').attr('href');
            const description = element.find('p.text').text().trim();
            const dateraw = element.find('p.time').text();

            return {
                title,
                description,
                link,
                pubDate: parseDate(dateraw, 'YYYY 年 MM 月 DD 日'),
            };
        })
        .get();

    return {
        title: 'Engineering blogs',
        link: url,
        item: list,
    };
}

```


