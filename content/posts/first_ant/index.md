---
title : "windwos初探ant" 
date : "2024-02-22T10:34:04+08:00" 
lastmod : "2024-02-22T10:34:04+08:00" 
tags : ["游戏","引擎"] 
categories : ["技术"]
draft : true
featuredImage :
summary : 'window编译ant教程'
---

### 编译ant

相关指令默认在msys2终端中运行，部分git更新代码指令在系统终端中运行

#### 安装msys2

- 下载并安装 [msys2](https://www.msys2.org/)
- 找到 msys2 安装目录，用 mingw64.exe 打开 msys2 的终端
- 在 msys2 的终端中修改镜像服务器

``` bash
echo "Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/i686/" > /etc/pacman.d/mirrorlist.mingw32
echo "Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/x86_64/" > /etc/pacman.d/mirrorlist.mingw64
echo "Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/msys/\$arch/" > /etc/pacman.d/mirrorlist.msys
```

- 把 ming64 的路径加到环境变量

``` bash
echo "export MINGW=/mingw64" >> ~/.bash_profile
echo "export PATH=\$MINGW/bin:\$PATH" >> ~/.bash_profile
```

- 首先更新

``` bash
pacman -Syu
```

- 安装 gcc/ninja

``` bash
pacman -Syu mingw-w64-x86_64-gcc mingw-w64-x86_64-ninja
```

<!-- - 安装make

``` bash
pacman -S make
```

- 安装[lua](https://www.lua.org/download.html)

``` bash
curl -L -R -O https://www.lua.org/ftp/lua-5.4.6.tar.gz
tar -zxvf lua-5.4.6.tar.gz
cd lua-5.4.6/
make mingw
cd src
rm -rf *.o
cp luac.exe  lua54.dll lua.exe ../../../../mingw64/bin
cp lauxlib.h lua.h lua.hpp lua.hpp lualib.h  ../../../../mingw64/include
cp liblua.a ../../../../mingw64/lib

``` -->

### hello word

#### 编译构建工具 luamake

``` bash
git clone https://github.com/actboy168/luamake
cd luamake
git submodule update --init
./compile/install.sh (mingw/linux/macos)  ## msys2终端中运行
```

#### 编译runtime

``` bash
git clone git@github.com:ejoy/ant.git
cd ant
## 部分模块使用https协议，修改.git/config，将模块的地址中https://github.com/ 替换为git@github.com:，调整为git协议
git submodule update --init
luamake ## msys2终端中运行
```

#### 编译tools

tools包含：shaderc, texturec, gltf2ozz，release模式会快一个数量级（debug模式下的tools可以不编译）

``` bash
luamake -mode release tools
```

#### 编译选项

``` bash
luamake [target] -mode [debug/release] #-mode默认是debug
```

#### 运行

运行一个最简单windwos的示例

``` bash
bin/mingw/debug/lua.exe test/simple/main.lua
```

#### 启动编辑器

```bash
bin/mingw/debug/lua.exe tools/editor/main.lua test/simple
```

{{< notice notice-tip >}}
文章更新后无法同步到公众号，如果您是在公众号阅读的本文，可以点击阅读原文查看更新后的内容。
{{< /notice >}}
