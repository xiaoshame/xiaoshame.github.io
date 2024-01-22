---
title : "使用feedly订阅公众号" 
date : "2024-01-18T17:40:30+08:00" 
lastmod : "2024-01-18T17:40:30+08:00" 
tags : ["rss","python","博客","MinIO"] 
categories : ["技术"]
draft : false
featuredImage : /images/posts/gzh_to_rss/featuredImage.jpg
summary : '使用python编写的脚本可以将公众号的最新文章保存到本地，但本地阅读并不方便。作者通过使用feedly来订阅文章的方式解决了这个问题。'
---

## 背景

在公众号发布过一个文章[python批量获取最新博客](https://mp.weixin.qq.com/s?__biz=Mzg2ODg5MTk3MQ==&mid=2247483963&idx=1&sn=35a16a0a6e6308b2c566629cda770593&chksm=cea42e17f9d3a7016925cf2bbb975ad81c42f9d89353e2677fd8aba3b84b4e68c35c43b2ea93#rd)，实现将一系列公众号最新发布的文章保存到本地方便阅读。实际使用过程中，发现在本地阅读非常不方便，导致使用几次后就放弃。平时我使用feedly订阅RSS阅读，所以想到能不能将这些内容使用feedly进行订阅方便阅读。

## 解决思路

1. feedly订阅的是一个网站的rss.xml文件，所以只需构造一个rss.xml文件放到服务器,对应的文件支持通过http访问，即可在feedly进行订阅
2. 使用服务器定时任务，定时更新最近发布的公众号文章信息到rss.xml文件
3. minio服务可以对外提供文件服务

## 具体操作

### 获取一个xml模板

1. 我直接下载了博客网站的[rss.xml文件](https://xiaoshame.github.io/rss.xml)
2. 删除rss.xml文件中的文章信息
3. 修改rss.xml文件中的title,description,link,href等信息

```xml
<?xml version='1.0' encoding='UTF-8'?>
<rss xmlns:ns0="http://purl.org/rss/1.0/modules/content/" version="2.0">
    <channel>
        <title>公众号综合</title>
        <link>https://xiaoqian713.live/rss/rss.xml</link>
        <description>公众号聚合</description>
        <copyright>asong</copyright>
        <pubDate>Mon, 22 Jan 2024 16:28:02 </pubDate>
        <lastBuildDate>Mon, 22 Jan 2024 16:28:02 </lastBuildDate>
    </channel>
</rss>
```

### 修改python代码

1. 将本地保存文章调整为将获取到的RSS信息中的文章内容保存到xml中
2. 使用docker部署了minio对象存储服务，用于存储rss.xml文件
3. 将代码和rss.xml放到服务器中，配置python相关环境
4. 测试代码确定运行无误，rss.xml可自行上传到minio中
5. 在feedly中订阅对应的文件地址
    1. 大家想订阅这个rss,订阅地址为：`https://xiaoqian713.live/rss/rss.xml`

```python
import concurrent.futures
import xml.etree.ElementTree as ET
from datetime import datetime, timedelta
import threading
import requests
import minio
from bs4 import BeautifulSoup

# 定义一个锁
lock = threading.Lock()

def get_list_name_url(html):
    # 创建BeautifulSoup对象bito.ai
    soup = BeautifulSoup(html, 'html.parser')

    # 提取博客名称和RSS地址内容
    name_list = []
    url_list = []
    paragraph = soup.find_all('div', class_='post-content')
    for parent in paragraph:
        span_a_content = parent.find_all(name='p')
        for a_content in span_a_content:
            name_list.append(a_content.find('a').text)
            url_list.append(a_content.find('a')['href'])
    return name_list, url_list

def get_recent_article(url,channel,start_time,end_time):
    ## 获取对应XML信息
    try:
        response = requests.get(url=url, headers={})
        content_type = response.headers.get('Content-Type')

        # 检查是否有charset=utf-8
        if 'charset=utf-8' in content_type:
            xml = response.text
        else:
        # 如果不是UTF-8，可以尝试手动转换编码
            xml = response.content.decode('utf-8', errors='ignore')
    except requests.exceptions.SSLError as err:
        print('SSL Error. Adding custom certs to Certifi store...')
    # 加载XML文件,获取根元素
    try:
        # 查找最新的文章
        new_rss = ET.fromstring(xml)
        # new_root = new_rss.getroot()
        # # mew_channel = ET.SubElement(new_rss, 'channel')
        new_channel = new_rss.find('channel')
        if new_channel is None:
            return url + " fail"
        # 使用锁来同步访问
        with lock:
            for item in new_channel.findall('item'):
                node = item.find('pubDate')
                if node is not None and node.text != "":
                    # 将日期时间字符串转换为时间对象
                    if node.text.endswith("GMT"):
                        pubDate = datetime.strptime(node.text, "%a, %d %b %Y %H:%M:%S %Z").replace(tzinfo=None)
                    else:
                        pubDate = datetime.strptime(node.text, "%a, %d %b %Y %H:%M:%S %z").replace(tzinfo=None)
                    ## 将内容写入文件中
                    if pubDate > start_time and pubDate < end_time:
                        first_item = channel.find('item')
                        if first_item is not None:
                            channel.insert(list(channel).index(first_item), item)
                        else:
                        # 如果没有其他item元素，直接追加到channel元素
                            channel.append(item)
            return url + " successe"
    except ET.ParseError:
        return url + " fail"

def get_blog_list(url,start_time,end_time):
    ## 获取对应网页html
    try:
        html = requests.get(url=url, headers='').text
    except requests.exceptions.SSLError as err:
        print('SSL Error. Adding custom certs to Certifi store...')
    ## 获取不同博客名和对应RSS订阅地址
    name_list,url_list = get_list_name_url(html)
    # 创建或加载新的RSS文件树和根元素
    rss = ET.parse(r'D:\workspace\script\gzh_to_rss\rss.xml')
    root = rss.getroot()
    channel = root.find('channel')
    last_build_date = channel.find('lastBuildDate')
    pubDate = channel.find('pubDate')
    if last_build_date is not None:
        last_build_date.text = end_time.strftime('%a, %d %b %Y %H:%M:%S %z')
    if pubDate is not None:
        pubDate.text = end_time.strftime('%a, %d %b %Y %H:%M:%S %z')
    # channel = ET.SubElement(rss, 'channel')
    # 提交任务给线程池，并获取Future对象
    futures = []
    # 创建线程池
    executor = concurrent.futures.ThreadPoolExecutor()
    for i in range(len(url_list)):
        future = executor.submit(get_recent_article,url_list[i],channel,start_time,end_time)
        futures.append(future)

    # 获取任务的返回值
    # results = []
    for future in concurrent.futures.as_completed(futures):
        result = future.result()
        print(result)
        # results.append(result)
    # 关闭线程池
    executor.shutdown()
    #将最终结果写入文件
    with lock:
        rss.write(r'D:\workspace\script\gzh_to_rss\rss.xml', encoding='UTF-8', xml_declaration=True)
    print("write rss done")

def up_data_minio(bucket: str):
    minio_conf = {
        'endpoint': '127.0.0.1:9000',
        'access_key': 'xxxx',
        'secret_key': 'xxxx',
        'secure': False
    }
    client = minio.Minio(**minio_conf)
    client.fput_object(bucket_name=bucket, object_name='rss.xml',file_path='rss.xml',content_type='text/xml')
    print("syn rss.xml done")

if __name__ == "__main__":
    end_time = datetime.now()
    ## 定时查询
    start_time = end_time - timedelta(hours=4)
    ## 获取最近4H内发表的文章
    get_blog_list("https://wechat2rss.xlab.app/posts/list/",start_time,end_time)
    up_data_minio('rss')

```

### 配置定时任务

使用crontab设置定时任务

```plaintext
### ubuntu 系统检查 cron 包是否安装
dpkg -l cron 

### 未安装 cron，先安装 cron 软件包
apt-get install cron

###验证 cron 服务是否正在运行
service cron status

### cron服务未启动，启动
service cron start

### 编辑当前用户cron服务，我选择vi编辑模式
crontab -e

### 每天9点定时运行python脚本
0 */4 * * * python /data/talebook/books/extract/603/blog_list_rss.py

### 为crontab增加日志
0 */4 * * * python /data/talebook/books/extract/603/blog_list_rss.py >> $HOME/for_crontab/mylog.log 2>&1
```

### minio部署

参考文章：

1. [部署私有对象存储服务: Minio](https://ysicing.me/tools/minio-deploy/)
2. [自建对象存储服务MinIO进行文件备份](https://blog.littlefox.me/archives/695)
3. [Docker搭建minio文件服务器](https://developer.aliyun.com/article/1329720)

```plaintext
### 下载镜像
1. docker pull minio/minio

```

## 最后

1. 这种方式具备可行性的基础是开源项目[wechat2rss](https://github.com/ttttmr/wechat2rss)持续的维护，谢谢ttttmr大佬
2. [公众号订阅list](https://wechat2rss.xlab.app/posts/list/)中包含众多公众号，可以自行选择想订阅的公众号
3. 可基于本文方案调整为只生成喜欢的公众号文章
