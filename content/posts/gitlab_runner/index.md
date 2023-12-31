---
title : '解决存储迁移导致的Gitlab Runner问题'
date : 2023-10-19T15:02:51+08:00
draft : false
tags : [gitlab,cppcheck,cpplint,CI,docker]
categories : [技术]
toc : true
summary : '总结了使用github-runner进行代码规范检测碰到的问题和排查、解决的过程。问题出在镜像文件找不到，通过恢复镜像和注册Gitlab-Runner解决。文章提供了详细的步骤和参考资料供读者参考。'
---
### 背景

组内使用github-runner进行代码规范检测，最近运维进行存储切换导致代码检测运行失败，之前找其他人安装的，年久无人维护只能自己上，记录整个过程。本文默认大家了解gitlab-runner的基础知识，不了解的可以先阅读参考资料中的文章。

### 问题排查

我们代码检测主要用的cppcheck和cpplint，查看Github-Runner jobs报错，发现是对应的镜像文件找不到，首先恢复镜像

```text
Preparing the "docker" executor
00:48
Using Docker executor with image 10.1.107.12:5000/dy/cppcheck ...
ERROR: Preparation failed: adding cache volume: set volume permissions: running permission container "a192958483eb385c5a19432a82b4bfd20d54a7a7cd28a35e3c3f85938bc8ab31" for volume "runner-bcvnjjb9-project-11377-concurrent-0-cache-3c3f060a0374fc8bc39395164f415a70": starting permission container: Error response from daemon: error evaluating symlinks from mount source "/dy_video1/docker-data/volumes/runner-bcvnjjb9-project-11377-concurrent-0-cache-3c3f060a0374fc8bc39395164f415a70/_data": lstat /dy_video1/docker-data/volumes/runner-bcvnjjb9-project-11377-concurrent-0-cache-3c3f060a0374fc8bc39395164f415a70: no such file or directory (linux_set.go:105:5s)
```

#### 恢复镜像

通过centos镜像安装cppcheck和cpplint,安装完成后导出镜像，重新导入并修改名称

```bash
### 下载centos镜像
docker pull centos:7
### 启动centos容器，进入安装环境cppcheck cpplint
docker run -it centos:7 /bin/bash
### 安装基础编译工具
yum install wget
yum install gcc
yum install gcc-c++
yum install make
yum install cmake
### 下载cppcheck源码编译
wget -O cppcheck.tar.gz https://github.com/danmar/cppcheck/archive/refs/tags/2.12.1.tar.gz
tar -xvzf cppcheck.tar.gz
cd cppcheck-2.12.1 && \
    mkdir build && \
    cd build && \
    cmake .. && \
    cmake --build . && \
    make install SRCDIR={解压的cppcheck路径}/cppcheck-2.12.1/build CFGDIR={解压的cppcheck路径}/cppcheck-2.12.1/cfg FILESDIR=/usr/bin
    sudo ln -s /usr/local/bin/cppcheck /usr/bin/cppcheck
## 安装cpplint
yum install python3
rm -rf /usr/bin/python
sudo ln -s /usr/bin/python3 /usr/bin/python
pip3 install cpplint -i https://pypi.tuna.tsinghua.edu.cn/simple
### 退出容器
exit
### 容器导出
docker export centos > container.tar
### tar导入为镜像命名为10.1.107.12:5000/dy/cppcheck
docker import container.tar 10.1.107.12:5000/dy/cppcheck
```

#### Gitlab-Runner 注册

在下载centos镜像操作中，走了很多弯路，最大的弯路是设置docker代理，提高docker pull速度，由于误操作，导致GitLab-Runner镜像丢失

##### docker运行Gitlab-Runner

```bash

### 下载gitlab-runner 镜像，alpine小一点
docker pull gitlab/gitlab-runner:alpine 
### 使用本地系统卷挂载，启动 Runner 容器
docker run -d --name gitlab-runner --restart always   -v /srv/gitlab-runner/config:/etc/gitlab-runner   -v /var/run/docker.sock:/var/run/docker.sock   gitlab/gitlab-runner:alpine
### 进入容器执行注册
docker exec -it gitlab-runner bash
### 重新注册,git地址和REGISTRATION_TOKEN 在gitlab设置Settings CI/CD中查看
sudo gitlab-runner register --url https://XXXX/ --registration-token $REGISTRATION_TOKEN
> Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com ) ###直接回车
> Please enter the gitlab-ci token for this runner #### 直接回车
> Please enter the gitlab-ci description for this runner ### 随意写个描述
> Please enter the gitlab-ci tags for this runner (comma separated): ### 定义个tag，我用的
ai-bus1
> Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
docker
> Please enter the Docker image (eg. ruby:2.1):
alpine:latest
### 退出容器
exit 

```

#### 再次报错

1. 勾选Indicates whether this runner can pick jobs without tags
   1. ![Run untagged jobs](/images/posts/gitlab_runner/1.png)
2. project拉取新分支测试,测试报错

```text
WARNING: Failed to pull image with policy "always": Error response from daemon: Get "http://10.1.107.12:5000/v2/": dial tcp 10.1.107.12:5000: connect: connection refused (manager.go:237:0s)
ERROR: Job failed: failed to pull image "10.1.107.12:5000/dy/cppcheck" with specified policies [always]: Error response from daemon: Get "http://10.1.107.12:5000/v2/": dial tcp 10.1.107.12:5000: connect: connection refused (manager.go:237:0s)

预计是docker运行gitlab-runner未绑定端口导致，查看.gitlab-ci.yml文件中image字段是10.1.107.12:5000/dy/cppcheck,我采用解决方案是修改镜像名

#### 修改gitlab-ci.yml文件中image字段
image:codecheck:v1
#### 删除10.1.107.12:5000/dy/cppcheck 镜像
docker rmi 10.1.107.12:5000/dy/cppcheck
#### 导入container.tar命名为codecheck:v1
docker import container.tar codecheck:v1
```

#### 继续报错

```text
WARNING: Failed to pull image with policy "always": Error response from daemon: pull access denied for codecheck, repository does not exist or may require 'docker login': denied: requested access to the resource is denied (manager.go:237:2s)
ERROR: Job failed: failed to pull image "codecheck:v1" with specified policies [always]: Error response from daemon: pull access denied for codecheck, repository does not exist or may require 'docker login': denied: requested access to the resource is denied (manager.go:237:2s)

### 修改宿主机中gitlab-runner配置
vim /etc/gitlab-runner/config.toml  ### 增加pull_policy = "never"
#### 修改容器中gitlab-runner配置
docker exec -it gitlab-runner bash
vim /etc/gitlab-runner/config.toml ### 增加pull_policy = "never"
#### 修改参考
[[runners]]
  name = "XXXX"
  url = "https://gitlab.com/"
  id = 412
  token = "XXXX"
  token_obtained_at = 2023-10-19T02:08:04Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
    pull_policy = "never"
### pull_policy参数作用
pull_policy = “never”      ### 只能使用 Runner 所在的 Docker 主机上提取过的本地镜像
pull_policy = “if-not-present”  ### Runner 将首先检查映像是否在本地存在。如果是，则使用图像的本地版本
pull_policy = “always”     ### 默认拉取策略 (未设置 pull_policy 执行默认拉取策略)，去拉取公网上的镜像
```

再次测试，测试通过，问题修复

### 后续

写完本文后，下班回到家洗澡时，思考这个问题的原因，因运行以前注册的runner提示的错误，与倒数第二个错误类似，怀疑本次问题修复，主要原因是镜像的缺失和运行未绑定端口导致。关闭gitlab-runner docker 容器和gitlab中ai-bus1 runner，重新测试测试运行无误，说明只重新安装cpplint/cppcheck镜像和修改image字段镜像名，即可修复此问题。

### 参考资料

* [搭建一个自己的 Gitlab CI Runner](https://chee5e.space/gitlab-runner-in-docker/)
* [在容器中运行极狐GitLab Runner](https://docs.gitlab.cn/runner/install/docker.html)
* [注册 Runner](https://docs.gitlab.cn/runner/register/index.html#docker)
* [Docker-从入门到实践-导出和导入](https://yeasy.gitbook.io/docker_practice/container/import_export)
