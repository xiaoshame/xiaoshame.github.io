---
title : '文章自动同步到公众号2'
date : 2023-10-30T15:38:16+08:00
draft : false
tags : [CI,效率]
categories : [技术]
toc : true
summary : '介绍了在将文章同步到微信公众号时出现的排版问题，并提出了解决方案。作者将markdown文件转化为html的项目与文章同步项目进行了整合，并对部分代码进行了修改。最后，作者总结了使用过程中的注意事项和一些使用技巧。完整的代码已上传到GitHub。'
---
### 背景

[文章自动同步到公众号](https://xiaoshame.github.io/article_auto_push_gzh/)上传到公众号草稿箱后，因公众号接口原因导致排版存在问题，无法做到直接发布。排版问题如下：

1. 代码无法展示完整
2. 有序列表前面的数字无法展示
3. 无须列表多了几列
4. 三级/四级标题样式固化
5. 最后的参考内容数字列表格式错乱

### 解决方案

查阅资料找到markdown文件转html开源项目[hzwz-markdown-wx](https://github.com/coder-pig/hzwz-markdown-wx)。hzwz-markdown-wx项目存在如下问题：

1. 生成的html文件复制到微信公众号操作，实操失败
2. 生成的html文件无法自动投稿到公众号

   1. 使用highlight对code高亮生成的html文件上传公众号报错

   2. ```plaintext
      Invalid JSON: Expecting value: line 1 column 1 (char 0)
      <!DOCTYPE html><html><head><script>var i=location.href;var v=window.btoa?window.btoa(window.encodeURIComponent(i)):"";window.location.href="https://waf.tencent.com/501page.html?u="+location.origin+"&id=1b5cfa348bdfe54f018dcc316bcbf1cf-1697099164938977-85119-139672863803136-76234149314225168&st=01&v="+v;</script></head></html>
      ```

3. 整体方案是将[文章自动同步到公众号](https://xiaoshame.github.io/article_auto_push_gzh/)中的markdown-to-wechat方案与hzwz-markdown-wx融合。

### 核心改动介绍

1. 修改sync.py中upload_media_news函数，只保留发布草稿箱功能

   1. ```python
      def upload_media_news(content,baseinfo,token):
          """
          上传到微信公众号
          """
          articles = {
              'articles':
              [
                  {
                      "title": baseinfo.TITLE,
                      "thumb_media_id": baseinfo.THUMB_MEDIA_ID,
                      "author": baseinfo.AUTHOR,
                      "digest": baseinfo.digest,
                      "show_cover_pic": 1,
                      "content": content,
                      "content_source_url": baseinfo.CONTENT_SOURCE_URL
                  }
                  # 若新增的是多图文素材，则此处应有几段articles结构，最多8段
              ]
          }

          headers={'Content-type': 'text/plain; charset=utf-8'}
          datas = json.dumps(articles, ensure_ascii=False).encode('utf-8')

          # 发布草稿箱
          postUrl = "https://api.weixin.qq.com/cgi-bin/draft/add?access_token=%s" % token
          r = requests.post(postUrl, data=datas, headers=headers)
          try:
              resp = json.loads(r.text)
              media_id = resp['media_id']
              print(media_id)
              # ### 发布
              # media_params = {
              #     "media_id": media_id
              # }
              # postUrl = "https://api.weixin.qq.com/cgi-bin/freepublish/submit?access_token=%s" % token
              # datas = json.dumps(media_params, ensure_ascii=False).encode('utf-8')
              # r = requests.post(postUrl,data=datas, headers=headers)
              # resp = json.loads(r.text)
              # print(resp)
              return True
          except json.JSONDecodeError as e:
              # 捕获JSON解码错误
              print("Invalid JSON:", e)
              print(r.text)
              return False
          except KeyError as e:
              # 捕获键错误
              print("Key Error:", e)
              print(r.text)
              return False
      ```

2. app.py增加同步图片素材、获取文章基本信息、文章投稿等操作

   1. ```python
      if __name__ == '__main__':
          print()
          # 相关文件夹初始化
          if cp_utils.is_dir_existed(md_dir) or cp_utils.is_dir_existed(out_dir):
              print("目录不存在")
              exit(0)
          if cp_utils.is_dir_existed(styles_dir) or cp_utils.is_dir_existed(template_dir):
              print("目录不存在")
              exit(0)
          # 文件检查/
          md_file_path_list = cp_utils.filter_file_type(md_dir, '.md')
          if len(md_file_path_list) == 0:
              print("当前目录无md文件，请检查后重试！" + md_dir)
              exit(0)
          theme_file_path_list = cp_utils.filter_file_type(styles_dir, '.ini')
          if len(theme_file_path_list) == 0:
              print("当前目录无样式配置文件，请检查后重试！" + styles_dir)
              exit(0)

          print("begin sync to wechat")
          start_time = time.time() # 开始时间
          sync.init_cache()
          client, token = sync.Client()
          for md_file_path in md_file_path_list:
              split_list = md_file_path.split(os.sep)
              if len(split_list) > 0:
                  file_name = split_list[-1]
                  print("读取文件 →", file_name)
                  file_content = cp_utils.read_file_content(md_file_path)
                  ## 图片资源上传微信公众号
                  file_content,imageId = sync.update_images_urls(file_content,client)
                  ## 获取文章基础信息
                  baseinfo = sync.BaseInfo(file_content,imageId,md_file_path)
                  for theme_file_path in theme_file_path_list:
                      theme_name = theme_file_path.split(os.sep)[-1][:-4]
                      print("应用样式 →", theme_name)
                      renderer_content = render_article(file_content, theme_file_path, template_dir)
                      out_file_path = os.path.join(out_dir, file_name.replace(".md", "_{}.html".format(theme_name)))
                      print("输出文件 →", out_file_path)
                      cp_utils.write_file(renderer_content, out_file_path)
                      if sync.upload_media_news(renderer_content,baseinfo,token):
                          sync.cache_update(md_file_path)
                          print("sync " + md_file_path + " to wechat successful")
      ```

3. 修改styles_renderer.py增加去掉md中文章描述信息，修改block_code函数

   1. ```python
      # 代码块
          def block_code(self, code, info=None):
              if self.mac_window_template is not None:
                  highlight_result = renderer_by_node(code, self.codestyle, info)
                  return self.mac_window_template.render(text=highlight_result)
              else:
                  exts = ['markdown.extensions.extra',
                  'markdown.extensions.tables',
                  'markdown.extensions.toc',
                  'markdown.extensions.sane_lists',
                  codehilite.makeExtension(
                      guess_lang=False,
                      noclasses=True,
                      pygments_style='friendly'
                  ),]
                  if info is not None:
                      info = info.strip()
                  lang = ''
                  if info:
                      lang = info.split(None, 1)[0]
                  code = "```"+ lang + '\n' + code + "```"
                  html = markdown.markdown(code, extensions=exts)
                  return replace_return(html)
      ```

   2. 对html了解不多，利用markdown.markdown完成代码高亮，规避使用highlight进行代码高亮无法投稿的问题

### 使用总结

1. blog\push_to_gzh\config.ini中控制相关资源目录地址
2. blog\push_to_gzh\styles\custom\cp_article_wx.ini 配置文章样式
   1. codespan和codestyle配置不可用，会导致文章介绍的投稿失败问题。
3. css_beautify函数可以美化html文件，使用blog\push_to_gzh\template\author\assets中的文件实现
4. 完整的代码已上传[github](https://github.com/xiaoshame/script)可自行查看
