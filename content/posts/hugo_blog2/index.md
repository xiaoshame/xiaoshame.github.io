---
title : 'Hugo+Github零成本搭建博客流程记录2'
date : 2023-11-08T11:38:20+08:00
draft : false
tags : [hugo,CI,github]
categories : [技术]
toc : true
summary : '文章提到了创建博客过程中对敏感信息的保护方法。hugo.toml中的敏感信息使用私有仓库和Actions来改进。git提交记录中的敏感信息更改配置实现。对自动投稿公众号的key保护，通过系统环境变量来保存敏感信息'
---
### 背景

创建博客到现在，使用过程中存在以下问题

1. hugo.toml中有敏感信息，每次提交代码时都额外需要注意
2. git 提交记录中包含敏感信息
3. 公众号APP_ID和APP_SECRET保护

### 解决方案

#### hugo.toml

1. 参考[hugo 使用 github actions 保存源码和自动化构建](https://www.wingoftime.cn/p/setup-blog-second/)
2. 新建一个私有仓库blog，存储博客源文件
3. 设置github 'Personal access tokens',在私有仓库blog中绑定用于后续自动推送
4. 在私有仓库blog中创建自动化流程，脚本放在项目根目录 .github/workflows/deploy.yml,最后的文件名自行定义
5. 文章内容推送到私有仓库blog，通过Action进行hugo编译，然后将blog内容同步到xiaoshame.github.io(博客仓库)
6. xiaoshame.github.io 仓库中部署模式需要调整，setting->Pages->Build and deployment选'Deploy from a branch'，选择对应分支保存即可

```yaml
name: deploy

on:
    push:
        branches:
            - main
    workflow_dispatch:

jobs:
    build:
        runs-on: ubuntu-latest
        steps:
            - name: Checkout
                uses: actions/checkout@v3
                with:
                    submodules: true
                    fetch-depth: 0
                    ref: main # 这里选择你触发部署的分支！默认是 master

            - name: Setup Hugo
                uses: peaceiris/actions-hugo@v2
                with:
                    hugo-version: "latest"
                    extended: true # 用 stack 主题需要加这个配置

            - name: Build Web
                run: hugo --minify

            - name: Deploy Web
                uses: peaceiris/actions-gh-pages@v3
                with:
                    PERSONAL_TOKEN: ${{ secrets.PERSONAL_TOKEN }}
                    EXTERNAL_REPOSITORY: xiaoshame/xiaoshame.github.io # 改成你的仓库
                    PUBLISH_BRANCH: main
                    PUBLISH_DIR: ./public
                    commit_message: ${{ github.event.head_commit.message }}
```

#### Git

1. git 的配置分为三级别，`System` -> `Global` ->`Local`，优先级是 `Local`> `Global`> `System`。
2. 直接在项目下设置用户名和邮箱,注意其他的项目还是用的 `Global`中的配置

   ```plaintext
   git config --local user.name "jitwxs"
   git config --local user.email "jitwxs@foxmail.com
   ```

#### 公众号

1. 通过系统环境变量保存，windows 我的电脑->属性->高级系统设置->环境变量->系统变量，PATH中设置
2. adfa

```plaintext
    robot.config["APP_ID"] = os.environ.get("WX_GZH_APP_ID")
    robot.config["APP_SECRET"] = os.environ.get("WX_GZH_APP_SECRET")
```

### 待完成

目前获取文章摘要 和 自动推送微信公众号还需要单独运行两个脚本，后续计划将这两个操作合并到Action操作中，有一个问题是怎么控制微信公众号推送的时机
