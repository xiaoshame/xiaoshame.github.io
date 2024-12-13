---
title : "Android Flutter APP逆向分析" 
date : "2024-12-13T17:20:06+08:00" 
lastmod : "2024-12-13T17:20:06+08:00" 
tags : ["Flutter","逆向","Android","blutter","frida"] 
categories : ["技术"]
draft : flase
featuredImage : /images/posts/inverse_analysis/flutte/1/featuredImage.jpg
summary : '利用blutter和frida分析Flutter APP获取加密key'
---

### 相关工具
1. [blutter](https://github.com/worawit/blutter)，flutter APP 静态分析辅助工具
    1. 按照官方文档进行安装
        1. 安装 git and python 3
        2. git clone git@github.com:worawit/blutter.git
        3. 安装 Visual Studio时，勾选"Desktop development with C++" 和 "windows C++ CMake tools"
        4. 在blutter代码目录执行python scripts\init_env_win.py (安装libcapstone and libicu4c库)
2. frida，hook工具
    1. pip install frida -i https://pypi.tuna.tsinghua.edu.cn/simple
    2. pip install frida-tools -i https://pypi.tuna.tsinghua.edu.cn/simple
3. root android手机(尝试使用模拟器，hook so时报错)
4. adb 组件，android 调试工具
5. ida，查看汇编内容
6. WinHex ，查看文件二进制
7. 代理工具，方便blutter编译

### 分析流程
#### blutter获取APP函数符号

使用flutter 开发的APP，程序代码都编译到libapp.so中，使用ida打开函数名经过混淆不利于分析，blutter通过编译Dart AOT 运行时来分析应用程序获取相应的函数名。

1. 在本文编写时间点，blutter只持分析Dart 3.6.0以下版本，dart 3.6.0-0.0.dev 以上版本编译报错
2. 小技巧:如果待分析的APP使用dart 3.6.0以上的版本编译，dart3.6.0 版本2024年7月2号发布，寻找此时间点前APP的历史版本进行分析，可以规避此问题
3. 管理员权限打开x64 Native Tools Command Prompt
    1. 执行python .\blutter.py D:\workspace\app\lib\arm64-v8a D:\workspace\app\lib\arm64-v8a\output
    2. D:\workspace\app\lib\arm64-v8a 中存放apk解压后，lib 中的libapp.so和libflutter.so
    3. D:\workspace\app\lib\arm64-v8a\output中为blutter分析完成后内容
    4. 全程使用代理

#### ida 静态分析

1. ida打开libapp.so
2. file -> Script file 加载blutter环节获得的output\ida_script\addNames.py脚本，还原混淆后的函数名
3. 根据需要分析的内容，猜测可能使用的函数名，进行分析

#### frida hook

1. blutter 分析完成后，会生成一个hook脚本output\blutter_frida.js
2. ida静态分析环节确定需要hook的函数地址，修改blutter_frida.js中onLibappLoaded函数fn_addr值即可
3. 手机开启开发模式，打开adb调试，非常重要
4. 查看frida版本和查看手机架构,下载[frida_server](https://github.com/frida/frida/releases)
    1. 我下载的frida-server-16.5.9-android-arm64.xz解压获得frida-server-16.5.9-android-arm64
```
frida --version    #我使用的版本16.5.9 
adb shell
su
getprop ro.product.cpu.abi   #查看手机架构，真机一般是arm64-v8a，模拟器一般是x86_64，

```
5. 手机上启动frida_server服务
```
## 将frida-server-16.5.9-android-arm64放入手机磁盘
adb push ./frida-server-16.5.9-android-arm64 /data/local/tmp
## 启动frida-server-16.5.9-android-arm64服务
adb shell
su
cd /data/local/tmp
mv frida-server-16.5.9-android-arm64 frida-server
chmod 777 frida-server
./frida-server &   
```
6. 使用frida脚本hook 函数，查看信息
    1. 附加成功后，手机上操作APP,触发hook函数执行，即可看到hook打印的信息
```
1. 查看应用进程ID，frida-ps -U
2. 查看应用的包名和进程ID,frida-ps -Uai
方法一: 
frida -U -f 包名 -l blutter_frida.js
-f 启动APP,会导致已启动的APP重启，libapp.app 没有加载成功，会导致hook失败
方法二：
1. 启动APP
2. frida -U -n 应用名 -l blutter_frida.js
3. -N 可能会失败，原因未分析
方法三：
1. 启动APP
2. frida -U -p 进程ID -l blutter_frida.js
3. 我采用的此方法
```
#### 打印hook函数信息

1. blutter_frida.js中增加onLeave，调用dumpArgs函数用于打印函数返回信息
2. hook的函数如果有多个入参，可以再blutter_frida.js中onEnter代码进行打印
```js
function dumpArgs(step, address, bufSize) {

    var buf = Memory.readByteArray(address, bufSize)

    console.log('Argument ' + step + ' address ' + address.toString() + ' ' + 'buffer: ' + bufSize.toString() + '\n\n Value:\n' +hexdump(buf, {
        offset: 0,
        length: bufSize,
        header: false,
        ansi: false
    }));

    console.log("Trying interpret that arg is pointer")
    console.log("=====================================")
    try{

    console.log(Memory.readCString(ptr(address)));
    console.log(ptr(address).readCString());
    console.log(hexdump(ptr(address)));
    }catch(e){
        console.log(e);
    }


    console.log('')
    console.log('----------------------------------------------------')
    console.log('')
}

function onLibappLoaded() {
    const fn_addr = 0x966e24;
    Interceptor.attach(libapp.add(fn_addr), {
        onEnter: function () {
            init(this.context);
            ## 打印第一个入参
            let objPtr = getArg(this.context, 0);
            const [tptr, cls, values] = getTaggedObjectValue(objPtr);
            console.log(`${cls.name}@${tptr.toString().slice(2)} =`, JSON.stringify(values, null, 2));
            ## 打印第二个入参
            let objPtr1 = getArg(this.context, 1);
            const [tptr1, cls1, values1] = getTaggedObjectValue(objPtr1);
            console.log(`${cls1.name}@${tptr1.toString().slice(2)} =`, JSON.stringify(values1, null, 2));
        },
        onLeave: function(retval){
            ## 打印函数返回信息
            dumpArgs(0,retval,500);
        }
    });
}
```
### 总结

1. 本次分析APP的目标是获取加密函数的KEY,相对简单，因而只使用frida 进行hook就可以达到目的，如果分析的内容较为复杂需要搭配ida调试功能
2. 最初使用模拟器进行hook，始终无法获取到libapp.so的地址，原因是模拟器开辟了一片新空间存储arm的so文件
3. 尝试在x86_64 windows 电脑上运行arm架构的模拟器，最终以失败结束，直接使用手机可以提高幸福指数

### 参考文档
[frida安装正确流程](https://www.cnblogs.com/fuxuqiannian/p/17930851.html#)
[【flutter对抗】blutter使用+ACTF习题](https://juejin.cn/post/7311254319323889699)
[CTT2023 Hflag — 200 pts](https://medium.com/@fnnnr/ctt2023-hflag-200-pts-4be08927769f#id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6IjJjOGEyMGFmN2ZjOThmOTdmNDRiMTQyYjRkNWQwODg0ZWIwOTM3YzQiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJhenAiOiIyMTYyOTYwMzU4MzQtazFrNnFlMDYwczJ0cDJhMmphbTRsamRjbXMwMHN0dGcuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJhdWQiOiIyMTYyOTYwMzU4MzQtazFrNnFlMDYwczJ0cDJhMmphbTRsamRjbXMwMHN0dGcuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJzdWIiOiIxMTM4MTY1NTI1OTMwNjI5NjYzNjgiLCJlbWFpbCI6InhpYW9zaGFtZTEyMDlAZ21haWwuY29tIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsIm5iZiI6MTczMzgzMTAwNywibmFtZSI6ImFzb25nIiwicGljdHVyZSI6Imh0dHBzOi8vbGgzLmdvb2dsZXVzZXJjb250ZW50LmNvbS9hL0FDZzhvY0xoVWttRkg1UkFfRWFUM0o5NXBMV2t3UDJzWjFfUXJqNnZDQ2o2MER3MzZwYjVMcDQ9czk2LWMiLCJnaXZlbl9uYW1lIjoiYXNvbmciLCJpYXQiOjE3MzM4MzEzMDcsImV4cCI6MTczMzgzNDkwNywianRpIjoiODVmMjMzNmQ2MzAyNzcwNTQ4MTk1NDM5MTdkNzU3NDc1MTAxMWIxNyJ9.hRuecCI2zq_thf__SNbMZ_7J-cbThCFMG7sCEqGvPQC5cOEc4fUU0GILpUgVmzoWWlxUZ-7b6ucV6Kce4ncP1sbsMN9XehDjPZankfabrGW0ZZ5xPLLiFz4TpxPSF8AYPDCPz5-RY7kV2fhA35Oaj3RK91U-z0mmj09mg1htK9d1YhpLTbt4RmH_oQ5ciua-TvP-4xrP7otw1whNhYe7EoPEWxXD5KUYFuz4OFh9BJ08qm3V9rsL_t0IH4GgF4cZ5-jbRsRJJG6w1YGKhr3bC1P2fq4Rghlv4T1g63sEGaXMO-8sRfiElvSLzEq7h18SYMCctXekffvtEcZ5Bw1KJw)
[解决无法在x86模拟器上frida-hook掉arm的Native层方法的问题](https://blog.csdn.net/qq_65474192/article/details/138916083)