---
title : '写个视频下载器送给老婆'
date : 2023-11-21T15:09:07+08:00
draft : false
tags : [python,下载,gui]
categories : [技术]
toc : true
summary : '为自己的老婆开发了一款支持下载B站和好看视频的GUI软件。基于一个开源项目，核心下载器是yt-dlp.exe，并使用PyInstaller将程序打包成exe文件'
---

### 背景

临近老婆生日思考该送什么，经常看到别人自己写个软件送个老婆，程序员的浪漫永远这么的耿直。

### 整体流程

#### 确定方向

日常老婆经常让我帮她下载B站和好看视频的内容，同时老婆对计算机一窍不通，只会点点点。最终确定给老婆写个支持下载B站和好看视频带GUI的软件。

#### 确定方案

1. 本人日常有使用yt-dlp下载youtube和b站视频，就这样敲定了下载的核心
2. 本人很少使用python写gui程序，因而需要找一个yt-dlp的gui开源项目，减少开发难度
    1. 经过寻找开源项目[youtube-dl-gui](https://github.com/MrS0m30n3/youtube-dl-gui)很适合，虽然已经停止维护6年
3. 好看视频直接爬取网页中视频链接，然后对视频链接进行下载

#### youtube-dl-gui改造

1. 主要调整为将python2调整为python3,根据调试报错进行针对性修正，修正过程不赘述
    1. 可以使用代码对比工具自行对比[yt-dlp-gui](https://github.com/xiaoshame/yt-dlp-gui)与[youtube-dl-gui](https://github.com/MrS0m30n3/youtube-dl-gui)

2. 将下载器调整为yt-dlp.exe,参数YOUTUBEDL_BIN最初调整为本地yt-dlp.exe目录，因为实时下载功能未完成，将yt-dlp.exe放到/data/exe/目录下方便后续打包，同时YOUTUBEDL_BIN也调整为获取运行目录加上相对路径

    ```python
    YOUTUBEDL_BIN = os.getcwd()

    if os.name == 'nt':
        YOUTUBEDL_BIN += '/_internal/exe/yt-dlp.exe'
    ```

#### python打包成exe

1. 主要参考[PyInstaller打包Python项目详解](https://www.cnblogs.com/bbiu/p/13209612.html)
2. 操作流程

    ``` plaintext
    # 1.执行命令，__main__.py为程序入口文件

    pyinstall -D __main__.py

    # 2.删除生成的bulid和dist文件夹,仅保留__main__.spec文件

    # 3.修改__main__.spec文件,素材和yt-dlp.exe加入打包
    # -*- mode: python ; coding: utf-8 -*-
    a = Analysis(
        ['__main__.py'],
        pathex=['D\\workspace\\youtube_dl_gui'],
        binaries=[('D:\\workspace\\youtube_dl_gui\\data\\exe\\yt-dlp.exe','exe')],
        datas=[('D:\\workspace\\youtube_dl_gui\\data\\icons\\hicolor\\128x128\\apps\\youtube-dl-gui.png','data\\icons\\hicolor'),
        ('D:\\workspace\\youtube_dl_gui\\data\\icons\\hicolor\\16x16\\apps\\youtube-dl-gui.png','data\\icons\\hicolor'),
        ('D:\\workspace\\youtube_dl_gui\\data\\icons\\hicolor\\256x256\\apps\\youtube-dl-gui.png','data\\icons\\hicolor'),
        ('D:\\workspace\\youtube_dl_gui\\data\\icons\\hicolor\\32x32\\apps\\youtube-dl-gui.png','data\\icons\\hicolor'),
        ('D:\\workspace\\youtube_dl_gui\\data\\icons\\hicolor\\48x48\\apps\\youtube-dl-gui.png','data\\icons\\hicolor'),
        ('D:\\workspace\\youtube_dl_gui\\data\\icons\\hicolor\\64x64\\apps\\youtube-dl-gui.png','data\\icons\\hicolor'),
        ('D:\\workspace\\youtube_dl_gui\\data\\pixmaps\\arrow_down_32px.png','data\\pixmaps'),
        ('D:\\workspace\\youtube_dl_gui\\data\\pixmaps\\arrow_up_32px.png','data\\pixmaps'),
        ('D:\\workspace\\youtube_dl_gui\\data\\pixmaps\\camera_32px.png','data\\pixmaps'),
        ('D:\\workspace\\youtube_dl_gui\\data\\pixmaps\\cloud_download_32px.png','data\\pixmaps'),
        ('D:\\workspace\\youtube_dl_gui\\data\\pixmaps\\delete_32px.png','data\\pixmaps'),
        ('D:\\workspace\\youtube_dl_gui\\data\\pixmaps\\folder_32px.png','data\\pixmaps'),
        ('D:\\workspace\\youtube_dl_gui\\data\\pixmaps\\icons-license','data\\pixmaps'),
        ('D:\\workspace\\youtube_dl_gui\\data\\pixmaps\\pause_32px.png','data\\pixmaps'),
        ('D:\\workspace\\youtube_dl_gui\\data\\pixmaps\\play_arrow_32px.png','data\\pixmaps'),
        ('D:\\workspace\\youtube_dl_gui\\data\\pixmaps\\reload_32px.png','data\\pixmaps'),
        ('D:\\workspace\\youtube_dl_gui\\data\\pixmaps\\settings_20px.png','data\\pixmaps'),
        ('D:\\workspace\\youtube_dl_gui\\data\\pixmaps\\stop_32px.png','data\\pixmaps'),
        ('D:\\workspace\\youtube_dl_gui\\data\\pixmaps\\youtube-dl-gui.ico','data\\pixmaps'),
        ('D:\\workspace\\youtube_dl_gui\\data\\pixmaps\\youtube-dl-gui.png','data\\pixmaps')],
        hiddenimports=[],
        hookspath=[],
        hooksconfig={},
        runtime_hooks=[],
        excludes=[],
        noarchive=False,
    )
    pyz = PYZ(a.pure)

    exe = EXE(
        pyz,
        a.scripts,
        [],
        exclude_binaries=True,
        name='youtube_dl_gui',
        debug=False,
        bootloader_ignore_signals=False,
        strip=False,
        upx=True,
        console=False,
        disable_windowed_traceback=False,
        argv_emulation=False,
        target_arch=None,
        codesign_identity=None,
        entitlements_file=None,
    )
    coll = COLLECT(
        exe,
        a.binaries,
        a.datas,
        strip=False,
        upx=True,
        upx_exclude=[],
        name='__main__',
    )

    # 4.执行命令

    pyinstaller __main__.spec

    # 5.去dist文件夹下找youtube_dl_gui.exe文件,运行自测

    # 6.运行成功，删除临时文件目录build；dist目录为打包的结果，可执行文件和其它程序运行的关联文件都在这个目录下
    ```

#### 支持好看视频下载

```python
def get_video(self):
        base_url = self.bv

        headers = {
            "user-agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1"
        }
        try:
            response = requests.get(url=base_url, headers=headers, timeout=5)
            if response.status_code == 200:    
                # soup = BeautifulSoup(response.text, 'html.parser')
                html = response.text
                start = html.find("window.__PRELOADED_STATE__ = ")
                end = html.find("}};", start)
                json_str = html[start+len("window.__PRELOADED_STATE__ = "):end+2]
                json_data = json.loads(json_str)
                title= json_data['curVideoMeta']['title']
                videoInfo = json_data['curVideoMeta']['clarityUrl']
                url = ''
                for item in videoInfo:
                    if item['key'] == 'sc':
                        url = item['url']
                return title,url
        except Exception as e:
            return '',''
```

1. 测试上述代码获取对应页面视频链接无误
2. 在downloaders.py中添加HaoKanDownloader类，添加下载和下载进度代码
3. 重新打包后，运行exe提示``OPENSSL_Uplink{00007FFE7BF17068,08}: no OPENSSL_Applink``
4. 尝试[issue](https://github.com/python/cpython/issues/108687)中的方法，使用官方__ssl.pyd覆盖codna DLLs中的文件，解决此报错提示另外报错，将conda DLLs中所有文件全部用官方DLLs中文件替换后运行正常无报错，但是下载B站视频失败
5. 回滚第4部操作，尝试将requests替换为urllib.request，返回的数据中视频地址被加密
6. 找到项目[crawler](https://github.com/litaolemo/crawler)中支持好看视频信息获取，发起请求同样使用requests，调整为urllib.request.Request后验证可以正常使用
7. 再次打包成exe,运行正常无报错，下载B站和好看视频正常，设置代理后可以下载youtube视频

### 总结

1. 后续修改开源项目过程中，程序报错需要顺手截图留存，方便后续文章编写使用
2. 通过此项目熟悉了wxPython基础，后续可以依葫芦画瓢
3. 此前python基本作为脚本语言使用，后续可以在小型项目使用
