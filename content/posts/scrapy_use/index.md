---
title : "搭建个人书城" 
date : "2024-01-02T10:15:29+08:00" 
lastmod : "2024-01-02T10:15:29+08:00" 
tags : ["scrapy","xpath","python"] 
categories : ["技术"]
draft : false
featuredImage : /images/posts/scrapy_use/featuredImage.jpg
summary : '搭建个人书城，可选择使用talebook或copyBook开源项目，前者简洁支持多种格式上传，后者类似网络小说网站，需要将章节信息写入数据库。爬虫复用copyBook项目，添加解析文件即可，还可添加代理和在python中运行js'
---

## 背景

最近准备看看《一只特立独行的猪》这本书，在找书过程中看到了很多有意思的开源项目和网站。考虑到有个服务器一直闲置，然后目的由找书变成了搭建个人的书城。

## 资源

1. [网站搭建talebook](https://github.com/talebook/talebook)
2. [网站搭建copyBook](https://github.com/hahaha108/copyBook)
3. [24h搜书网](https://24hbook.com/)
4. [备胎书屋](https://beitai.cc/)
5. [必读推荐](https://www.bidutuijian.com/)

## 网站搭建

1. talebook 支持epub,txt等格式，支持批量上传，搭建的网站简洁
2. copyBook 需要通过爬虫脚本将章节信息写入数据库，搭建的网站类似网络小说在线阅读网站
3. 采用talebook 搭建网站，步骤很简单，按照说明文档进行部署就行。我采用docker进行的部署
    1. [docker-compose 安装](https://cloud.tencent.com/developer/article/1855291)
    2. docker-compose.yml中`/tmp/demo:/data`调整为`/data/talebook:/data`
    3. 部署指令：`docker-compose -f docker-compose.yml  up -d`
4. 闲置的机器上采用caddy2进行反向代理
    1. talebook部署完成后，修改/etc/caddy/Caddyfile新增`reverse_proxy 127.0.0.1:8080`
    2. 8080为docker-compose.yml中填写的端口
    3. XXXX为对应的域名
    4. 使用对应域名即可访问网站
5. 通过爬虫获取不同线上书城的书籍，上传到服务器指定的目录`/data/talebook/book/imports`，在网页通过批量同步方式导入网站
6. 手机上下载kybook APP添加opds地址，支持搜索，书籍下载后阅读

```plaintext
XXXX {
    handle_path /XXXX {
        reverse_proxy 127.0.0.1:28727
    }
    reverse_proxy 127.0.0.1:8080
}
import sites/*
```

## 爬虫常用技能

### scrapy爬虫

1. 整体爬虫代码框架完全复用copyBook这个项目,只需在`bookspider/bookspider/spider`中添加解析文件
2. 添加解析xx.py文件，并修改name参数为xx，编写好想要解析代码
    1. 修改`copyBook\bookspider\start.py`为`cmdline.execute('scrapy crawl xx'.split())` 即可运行
3. `yield scrapy.Request(bookUrl,meta={"categoryName":categoryName},callback=self.getBooks)`
    1. bookUrl为完整的地址,getBooks为回调函数
    2. getBooks中response为请求返回数据，如果地址是文件地址，则response.body即文件数据
4. allowed_domains 中为允许访问的域名，根据爬虫需要进行添加

### scrapy使用代理

参考网络文章，修改`copyBook\bookspider\bookspider\settings.py`未成功，直接在爬虫类中添加如下代码，验证代理通过

```python
os.environ["HTTP_PROXY"] = "http://127.0.0.1:1081"
os.environ["HTTPS_PROXY"] = "http://127.0.0.1:1081"
```

### python运行js

```python
def decode(self,data):
    path = os.path.split(os.path.realpath(__file__))[0] + '/xxx.js'
    with open(path, 'r', encoding='UTF-8') as f:
        js_code = f.read()
        context = execjs.compile(js_code)
        result = context.call("decode", data)
        return result
```

1. 安装PyExecJS，`pip install PyExecJS`
2. context.call中decode为xxx.js中函数名，data为函数需要的参数
3. PyExecJS的运行依赖node.js

### xpath

```python
def parse(self, response):
        authors_table = response.xpath("//article[@class='markdown-body']/table")
        authors_ul = response.xpath("//article[@class='markdown-body']/ul")
        for author in authors_table:
            author_name = ""
            author_url = ""
            if len(author.xpath("thead/tr/th/a/text()").extract()) == 0 or len(author.xpath("thead/tr/th/a/@href").extract()) == 0:
                continue
            else:
                author_name = author.xpath("thead/tr/th/a/text()").extract()[0]
                author_url = author.xpath("thead/tr/th/a/@href").extract()[0]
                author_url = author_url.replace("..","")
                yield scrapy.Request(self.base_url + author_url,meta={"authorName":author_name},callback=self.getBooks)
            
            authors_body = author.xpath("tbody/tr")
            for author_body in authors_body:
                author_body_name = ""
                author_body_url = ""
                if len(author_body.xpath("td/a/text()").extract()) == 0 or len(author_body.xpath("td/a/@href").extract()) == 0:
                    continue
                else:
                    author_body_name = author_body.xpath("td/a/text()").extract()[0]
                    author_body_url = author_body.xpath("td/a/@href").extract()[0]
                    author_body_url = author_body_url.replace("..","")
                    yield scrapy.Request(self.base_url +author_body_url,meta={"authorName":author_body_name},callback=self.getBooks)

        for author in authors_ul:
            author_name = ""
            author_url = ""
            if len(author.xpath("li/a/text()").extract()) == 0 or len(author.xpath("li/a/@href").extract()) == 0:
                continue
            else:
                author_name = author.xpath("li/a/text()").extract()[0]
                author_url = author.xpath("li/a/@href").extract()[0]
                author_url = author_url.replace("..","")
                yield scrapy.Request(self.base_url + author_url,meta={"authorName":author_name},callback=self.getBooks)
```

1. `len(author.xpath("thead/tr/th/a/text()").extract())`通过长度判断是否有值
2. `author_name = author.xpath("thead/tr/th/a/text()").extract()[0]`取值
3. `book.xpath("td[1]/text()").extract()` 列表索引从1开始

## 总结

1. github 真是效率工具，需要的各种工具都可以找到
2. scrapy 只使用了常规的能力，代理池、增量更新等还未涉及
