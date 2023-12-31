---
title : '懒人必备，自动发小红书'
date : 2023-11-24T13:53:21+08:00
draft : false
tags : [python,自动化]
categories : [技术]
toc : true
summary : '通过将文章保存为html文件，然后利用selenium将html转换为图片，再通过selenium模拟点击小红书的发布流程，实现了将文章自动发布到小红书的功能。'
---

### 背景

日常写文章同步公众号已经有自动化工具辅助，但是发布到小红书还是需要经过重新编辑非常不方便。前段时间听播客的时候，听到一个方案是将html转换成图片，然后在小红书上只发布图片，解决了重新编辑和文章字数限制。目前适合我的脚本已编写完成，记录下整体流程。

### 方案

1. 在同步公众号的脚本中将markdown文件转换成了html文件，利用selenium加载html文件全文截图，然后按照标题坐标将不同段落转换成图片。之后再借助selenium模拟点击小红书发布流程，实现全自动化文章发布。
2. 首先在github上查找是否有合适的脚本，找到一个[自动发布小红书视频脚本](https://github.com/menhuan/notes/blob/b5e24e7e5184df538c9c4755bf70eb8202e6c75e/python/douyin/upload_xiaohongshu.py)

### html转图片

1. 最开始试用了很多库例如html2image,imgkit,pyppeteer等，因为可扩展性和不熟悉放弃，最终选用了PIL和selenium

2. 最初采用设置窗口宽高方案截图，但是无法便捷的裁剪指定区域。调整为对整个文章进行截图，然后根据标题左上角的坐标确定裁剪区域

3. 阅读webdriver.py API发现可以使用**set_window_rect**设置截图的坐标实现相应功能，读者可自行实验

```python
import os.path

from PIL import Image
from selenium import webdriver
from selenium.webdriver.common.by import By
import time

def html_to_png(html_path,out_path): 
    # options = webdriver.ChromeOptions()
    options = webdriver.EdgeOptions()
    options.add_argument('--headless')  # 不知为啥只能在无头模式执行才能截全屏
    options.add_argument('--disable-gpu')
    driver = webdriver.Edge(options=options)
    # driver = webdriver.Chrome(options=options)
    driver.maximize_window()

    try:
        driver.get(html_path)
        sections = driver.find_elements(By.XPATH, "//h3 | //h4")
        h3_height_list = []
        for section in sections:    
            # 使用JavaScript执行脚本来获取章节元素在网页中的位置信息
            location = driver.execute_script("return arguments[0].getBoundingClientRect();", section)
            h3_height_list.append((location['left'],location['top']))

        index = 0
        size = []
        scroll_width = driver.execute_script('return document.body.parentNode.scrollWidth')
        scroll_height = driver.execute_script('return document.body.parentNode.scrollHeight')
        driver.set_window_size(scroll_width, scroll_height)
        driver.get_screenshot_as_file(out_path + "screenshot.png")
        for item in h3_height_list:
            if index > 0:
                screenshot = Image.open(out_path + "screenshot.png")
                # 裁剪出感兴趣的位置
                cropped_image = screenshot.crop((0, int(size[1]), scroll_width, int(item[1])))
                # 保存裁剪后的图片
                cropped_image.save(out_path + "%d.png" % index)
            size = item
            index += 1
        cropped_image = screenshot.crop((0, int(size[1]), scroll_width, scroll_height))
        # 保存裁剪后的图片
        cropped_image.save(out_path + "%d.png" % index)
        os.remove(out_path + "screenshot.png")
        driver.quit()
    except Exception as e:
        print(e)



if __name__ == '__main__':
    _HTML = 'D:\\workspace\\script\\push_to_gzh\\article\\out\\index_cp_article_wx.html'
    _OUTFILE = 'D:\\workspace\\script\\push_to_xiaohongshu\\out\\'

    # 首先创建一个保存截图的文件夹
    if not os.path.isdir(_OUTFILE):
        # 判断文件夹是否存在，如果不存在就创建一个
        os.makedirs(_OUTFILE)
    # 将html转换成png
    html_to_png("file:///"+_HTML,_OUTFILE)
```

### 自动发图文

1. [自动发布小红书视频脚本](https://github.com/menhuan/notes/blob/b5e24e7e5184df538c9c4755bf70eb8202e6c75e/python/douyin/upload_xiaohongshu.py)功能很完善，微微调整就可以使用
2. **vidoe.send_keys(mp4[0])**中mp4[0]调整为图片地址
3. **for label in ["#虐文","#知乎文","#小说推荐","#知乎小说","#爽文"]:** 标签通过参数传递
4. **get_publish_date** 函数简化，不像定时发布可注释相应代码
5. 安装selenium后，第一次执行webdriver.EdgeOptions()时会下载相应驱动，耗时会有点久
6. 自动发布视频函数未经测试，如有bug可自行调整

```python
import json
import os
import time
import traceback
from datetime import datetime,timedelta

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys

XIAOHONGSHU_COOKING = r'D:\workspace\script\push_to_xiaohongshu\out\config.json'

def get_driver():
    options = webdriver.EdgeOptions()
    # options.add_argument('--headless')  # 不知为啥只能在无头模式执行才能截全屏
    # options.add_argument('--disable-gpu')
    driver = webdriver.Edge(options=options)
    driver.maximize_window()

    return driver

def xiaohongshu_login(driver):
    if (os.path.exists(XIAOHONGSHU_COOKING)):
        print("cookies存在")
        with open(XIAOHONGSHU_COOKING) as f:
            cookies = json.loads(f.read())
            driver.get("https://creator.xiaohongshu.com/creator/post")
            driver.implicitly_wait(10)
            driver.delete_all_cookies()
            time.sleep(4)
            # 遍历cook
            print("加载cookie")
            for cookie in cookies:
                if 'expiry' in cookie:
                    del cookie["expiry"]
                # 添加cook
                driver.add_cookie(cookie)
            # 刷新
            print("开始刷新")
            driver.refresh()
            driver.get("https://creator.xiaohongshu.com/publish/publish")
            time.sleep(5)
    else:
        print("cookies不存在")
        driver.get('https://creator.xiaohongshu.com/creator/post')
        # driver.find_element(
        #     "xpath", '//*[@placeholder="请输入手机号"]').send_keys("")
        # # driver.find_element(
        # #     "xpath", '//*[@placeholder="请输入密码"]').send_keys("")
        # driver.find_element("xpath", '//button[text()="登录"]').click()
        print("等待登录")
        time.sleep(30)
        print("登录完毕")
        cookies = driver.get_cookies()
        with open(XIAOHONGSHU_COOKING, 'w') as f:
            f.write(json.dumps(cookies))
        print(cookies)
        time.sleep(1)

def get_publish_date():
    tomorrow = now = datetime.today()
    if(now.hour > 20):
        tomorrow = now + timedelta(days = 1)
        tomorrow = tomorrow.replace(hour=20)
    else:
        tomorrow = tomorrow.replace(hour=20,minute=0)
    return tomorrow.strftime("%Y-%m-%d %H:%M")

def publish_xiaohongshu_video(driver, mp4, index):
    driver.find_element("xpath", '//*[text()="发布笔记"]').click()
    print("开始上传文件", mp4[0])
    time.sleep(3)
    # ### 上传视频
    vidoe = driver.find_element("xpath", '//input[@type="file"]')
    vidoe.send_keys(mp4[0])

    # 填写标题
    content = mp4[1].replace('.mp4', '')
    driver.find_element(
        "xpath", '//*[@placeholder="填写标题，可能会有更多赞哦～"]').send_keys(content)

    time.sleep(1)
    # 填写描述
    content_clink = driver.find_element(
        "xpath", '//*[@placeholder="填写更全面的描述信息，让更多的人看到你吧！"]')
    content_clink.send_keys(content)

    time.sleep(3)
    # #虐文推荐 #知乎小说 #知乎文
    for label in ["#虐文","#知乎文","#小说推荐","#知乎小说","#爽文"]:
        content_clink.send_keys(label)
        time.sleep(1)
        data_indexs = driver.find_elements(
            "class name", "publish-topic-item")
        try:
            for data_index in data_indexs:
                if(label in data_index.text):
                    print("点击标签",label)
                    data_index.click()
                    break
        except Exception:
            traceback.print_exc()
        time.sleep(1)

    # 定时发布
    dingshi = driver.find_elements(
        "xpath", '//*[@class="css-1v54vzp"]')
    time.sleep(4)
    print("点击定时发布")
    dingshi[3].click()
    time.sleep(5)
    input_data = driver.find_element("xpath", '//*[@placeholder="请选择日期"]')
    input_data.send_keys(Keys.CONTROL,'a')     #全选
    # input_data.send_keys(Keys.DELETE)
    input_data.send_keys(get_publish_date())    
    time.sleep(3)
    # driver.find_element("xpath", '//*[text()="确定"]').click()

    # 等待视频上传完成
    while True:
        time.sleep(10)
        try:
            driver.find_element("xpath",'//*[@id="publish-container"]/div/div[2]/div[2]/div[6]/div/div/div[1]//*[contains(text(),"重新上传")]')
            break
        except Exception as e:
            traceback.print_exc()
            print("视频还在上传中···")
    
    print("视频已上传完成！")
    time.sleep(3)
    # 发布
    driver.find_element("xpath", '//*[text()="发布"]').click()
    print("视频发布完成！")
    time.sleep(10)

def publish_xiaohongshu_image(driver, image_path,title,describe,keywords):
    time.sleep(3)
    driver.find_element("xpath", '//*[text()="发布笔记"]').click()
    print("开始上传图片")
    time.sleep(3)
    # ### 上传图片
    driver.find_element("xpath", '//*[text()="上传图文"]').click()
    push_file = driver.find_element("xpath", '//input[@type="file"]')
    file_names = os.listdir(image_path)
    # 打印所有文件名
    for file_name in file_names:
        push_file.send_keys(image_path + "\\"+file_name)

    # 填写标题
    driver.find_element(
        "xpath", '//*[@placeholder="填写标题，可能会有更多赞哦～"]').send_keys(title)

    time.sleep(1)
    # 填写描述
    content_clink = driver.find_element(
        "xpath", '//*[@placeholder="填写更全面的描述信息，让更多的人看到你吧！"]')
    content_clink.send_keys(describe)

    time.sleep(3)
    # #虐文推荐 #知乎小说 #知乎文
    for label in keywords:
        content_clink.send_keys(label)
        time.sleep(1)
        data_indexs = driver.find_elements(
            "class name", "publish-topic-item")
        try:
            for data_index in data_indexs:
                if(label in data_index.text):
                    print("点击标签",label)
                    data_index.click()
                    break
        except Exception:
            traceback.print_exc()
        time.sleep(1)

    # 定时发布
    dingshi = driver.find_elements(
        "xpath", '//*[@class="css-1v54vzp"]')
    time.sleep(4)
    print("点击定时发布")
    dingshi[3].click()
    time.sleep(5)
    input_data = driver.find_element("xpath", '//*[@placeholder="请选择日期"]')
    input_data.send_keys(Keys.CONTROL,'a')     #全选
    # input_data.send_keys(Keys.DELETE)
    input_data.send_keys(get_publish_date())
    time.sleep(3)
    # driver.find_element("xpath", '//*[text()="确定"]').click()

    # 发布
    driver.find_element("xpath", '//*[text()="发布"]').click()
    print("图文发布完成！")
    time.sleep(10)

if __name__ == "__main__":
    try:
        title = "测试下"
        describe = ['#python','#礼物']
        driver = get_driver()
        xiaohongshu_login(driver=driver)
        # publish_xiaohongshu_video(driver, r"D:\workspace\script\push_to_xiaohongshu\out\9.png", 1)
        publish_xiaohongshu_image(driver, r"D:\workspace\script\push_to_xiaohongshu\out", title,describe)
    finally:
        driver.quit()
```

### 总结

selenium是个非常强大的库，可以实现很多自动化的工作。第一次使用被惊艳到，这也是代码的迷人之处，期待后续使用selenium做的小工具。
