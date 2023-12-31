---
title :  'Hugo+Github零成本搭建博客流程记录'
date : 2023-10-11T10:06:03+08:00
draft : false
tags : [hugo,github,建站]
categories : [技术]
toc : true
summary : '文章介绍了使用Hugo和GitHub搭建博客的流程，包括安装Hugo、配置主题、部署到GitHub、添加评论系统、创建收藏页和增加封面图等。还介绍了如何添加站点运行时间和二级菜单。通过简化操作和利用现有资源，可以实现零成本搭建现代博客。'
---
### 背景

看到一篇文章[零成本搭建现代博客之搭建篇](https://www.bmpi.dev/dev/guide-to-setup-blog-site-with-zero-cost/1/)，目前采用的博客方案存在以下不足：

1. 目前我博客基于infinity搭建，空间只有5G，时间久了空间不足
2. 域名续费，一年成本100+，每年换一个域名太麻烦

### 安装

1. 安装和本地预览博客，参考[Hugo官方文档 quick-start](https://gohugo.io/getting-started/quick-start/)
2. 我选择的LoveIt主题，下载完成后将blog/themes/LoveIt/exampleSite/config.toml 中的内容覆盖blog/hugo.toml文件中的内容
3. 创建blog/static/images目录存放图片，直接相对路径即可使用本地图片,例如："/images/avatar.jpeg"

#### 主题配置

1. 重点介绍我修改的地方
   1. 主题目录地址
   2. 博客标题、博客副标题
   3. 博客logo
   4. 主页显示头像
   5. 关掉社交
2. hugo.toml完整配置如下，可自行对比

```
baseURL = "https://xiaoshame.github.io/"

# theme
# 主题
theme = "LoveIt"
# themes directory
# 主题目录
themesDir = "themes"

# website title
# 网站标题
title = "阿松日常"

# determines default content language ["en", "zh-cn", "fr", "pl", ...]
# 设置默认的语言 ["en", "zh-cn", "fr", "pl", ...]
defaultContentLanguage = "zh-cn"
# language code ["en", "zh-CN", "fr", "pl", ...]
# 网站语言, 仅在这里 CN 大写 ["en", "zh-CN", "fr", "pl", ...]
languageCode = "zh-CN"
# language name ["English", "简体中文", "Français", "Polski", ...]
# 语言名称 ["English", "简体中文", "Français", "Polski", ...]
languageName = "简体中文"
# whether to include Chinese/Japanese/Korean
# 是否包括中日韩文字
hasCJKLanguage = true
summaryLength = 200

# default amount of posts in each pages
# 默认每页列表显示的文章数目
paginate = 10
# copyright description used only for seo schema
# 版权描述，仅仅用于 SEO
copyright = "CC BY-NC 4.0"

# whether to use robots.txt
# 是否使用 robots.txt
enableRobotsTXT = true
# whether to use git commit log
# 是否使用 git 信息
enableGitInfo = true
# whether to use emoji code
# 是否使用 emoji 代码
enableEmoji = true

# ignore some build errors
# 忽略一些构建错误
ignoreErrors = ["error-remote-getjson", "error-missing-instagram-accesstoken"]

# Author config
# 作者配置
[author]
  name = "阿松日常"
  email = "xiaoshame1209@gmail.com"
  link = ""

[params]
  # site default theme ["auto", "light", "dark"]
  # 网站默认主题 ["auto", "light", "dark"]
  defaultTheme = "light"
  # public git repo url only then enableGitInfo is true
  # 公共 git 仓库路径，仅在 enableGitInfo 设为 true 时有效
  gitRepo = "https://github.com/xiaoshame/xiaoshame.github.io.git"
  # which hash function used for SRI, when empty, no SRI is used
  # ["sha256", "sha384", "sha512", "md5"]
  # 哪种哈希函数用来 SRI, 为空时表示不使用 SRI
  # ["sha256", "sha384", "sha512", "md5"]
  fingerprint = ""
  # date format
  # 日期格式
  dateFormat = "2006-01-02"
  # website title for Open Graph and Twitter Cards
  # 网站标题, 用于 Open Graph 和 Twitter Cards
  title = "阿松日常"
  # website description for RSS, SEO, Open Graph and Twitter Cards
  # 网站描述, 用于 RSS, SEO, Open Graph 和 Twitter Cards
  description = "阿松日常的个人博客，奶爸一枚，分享效率提升技巧和带娃日常"
  # website images for Open Graph and Twitter Cards
  # 网站图片, 用于 Open Graph 和 Twitter Cards
  images = "/images/logo.png"

  # Header config
  # 页面头部导航栏配置
  [params.header]
    # desktop header mode ["fixed", "normal", "auto"]
    # 桌面端导航栏模式 ["fixed", "normal", "auto"]
    desktopMode = "fixed"
    # mobile header mode ["fixed", "normal", "auto"]
    # 移动端导航栏模式 ["fixed", "normal", "auto"]
    mobileMode = "auto"
    # Header title config
    # 页面头部导航栏标题配置
    [params.header.title]
      # URL of the LOGO
      # LOGO 的 URL
      logo = "/images/logo.png"
      # title name
      # 标题名称
      name = "阿松日常"
      # you can add extra information before the name (HTML format is supported), such as icons
      # 你可以在名称 (允许 HTML 格式) 之前添加其他信息, 例如图标
      pre = ""
      # you can add extra information after the name (HTML format is supported), such as icons
      # 你可以在名称 (允许 HTML 格式) 之后添加其他信息, 例如图标
      post = ""
      # whether to use typeit animation for title name
      # 是否为标题显示打字机动画
      typeit = false

  # Footer config
  # 页面底部信息配置
  [params.footer]
    enable = true
    # Custom content (HTML format is supported)
    # 自定义内容 (支持 HTML 格式)
    custom = ""
    # whether to show Hugo and theme info
    # 是否显示 Hugo 和主题信息
    hugo = false
    # whether to show copyright info
    # 是否显示版权信息
    copyright = true
    # whether to show the author
    # 是否显示作者
    author = true
    # site creation time
    # 网站创立年份
    since = 2023
    # ICP info only in China (HTML format is supported)
    # ICP 备案信息，仅在中国使用 (支持 HTML 格式)
    icp = ""
    # license info (HTML format is supported)
    # 许可协议信息 (支持 HTML 格式)
    license= '<a rel="license external nofollow noopener noreffer" href="https://creativecommons.org/licenses/by-nc/4.0/" target="_blank">CC BY-NC 4.0</a>'

  # Section (all posts) page config
  # Section (所有文章) 页面配置
  [params.section]
    # special amount of posts in each section page
    # section 页面每页显示文章数量
    paginate = 20
    # date format (month and day)
    # 日期格式 (月和日)
    dateFormat = "01-02"
    # amount of RSS pages
    # RSS 文章数目
    rss = 10

  # List (category or tag) page config
  # List (目录或标签) 页面配置
  [params.list]
    # special amount of posts in each list page
    # list 页面每页显示文章数量
    paginate = 20
    # date format (month and day)
    # 日期格式 (月和日)
    dateFormat = "01-02"
    # amount of RSS pages
    # RSS 文章数目
    rss = 10

  # App icon config
  # 应用图标配置
  [params.app]
    # optional site title override for the app when added to an iOS home screen or Android launcher
    # 当添加到 iOS 主屏幕或者 Android 启动器时的标题, 覆盖默认标题
    title = "阿松日常"
    # whether to omit favicon resource links
    # 是否隐藏网站图标资源链接
    noFavicon = false
    # modern SVG favicon to use in place of older style .png and .ico files
    # 更现代的 SVG 网站图标, 可替代旧的 .png 和 .ico 文件
    svgFavicon = "/images/logo.svg"
    # Android browser theme color
    # Android 浏览器主题色
    themeColor = "#ffffff"
    # Safari mask icon color
    # Safari 图标颜色
    iconColor = "#5bbad5"
    # Windows v8-11 tile color
    # Windows v8-11 磁贴颜色
    tileColor = "#da532c"

  # Search config
  # 搜索配置
  [params.search]
    enable = true
    # type of search engine ["lunr", "algolia"]
    # 搜索引擎的类型 ["lunr", "algolia"]
    type = "algolia"
    # max index length of the chunked content
    # 文章内容最长索引长度
    contentLength = 4000
    # placeholder of the search bar
    # 搜索框的占位提示语
    placeholder = ""
    # max number of results length
    # 最大结果数目
    maxResultLength = 10
    # snippet length of the result
    # 结果内容片段长度
    snippetLength = 30
    # HTML tag name of the highlight part in results
    # 搜索结果中高亮部分的 HTML 标签
    highlightTag = "em"
    # whether to use the absolute URL based on the baseURL in search index
    # 是否在搜索索引中使用基于 baseURL 的绝对路径
    absoluteURL = false
    [params.search.algolia]
      index = ""
      appID = ""
      searchKey = ""
  # Page global config
  # 文章页面全局配置
  [params.page]
    # whether to hide a page from home page
    # 是否在主页隐藏一篇文章
    hiddenFromHomePage = false
    # whether to hide a page from search results
    # 是否在搜索结果中隐藏一篇文章
    hiddenFromSearch = false
    # whether to enable twemoji
    # 是否使用 twemoji
    twemoji = false
    # whether to enable lightgallery
    # 是否使用 lightgallery
    lightgallery = true
    # whether to enable the ruby extended syntax
    # 是否使用 ruby 扩展语法
    ruby = true
    # whether to enable the fraction extended syntax
    # 是否使用 fraction 扩展语法
    fraction = true
    # whether to enable the fontawesome extended syntax
    # 是否使用 fontawesome 扩展语法
    fontawesome = true
    # whether to show link to Raw Markdown content of the content
    # 是否显示原始 Markdown 文档内容的链接
    linkToMarkdown = false
    # whether to show the full text content in RSS
    # 是否在 RSS 中显示全文内容
    rssFullText = false
    # Table of the contents config
    # 目录配置
    [params.page.toc]
      # whether to enable the table of the contents
      # 是否使用目录
      enable = true
      # whether to keep the static table of the contents in front of the post
      # 是否保持使用文章前面的静态目录
      keepStatic = false
      # whether to make the table of the contents in the sidebar automatically collapsed
      # 是否使侧边目录自动折叠展开
      auto = true
    # Code config
    # 代码配置
    [params.page.code]
      # whether to show the copy button of the code block
      # 是否显示代码块的复制按钮
      copy = true
      # the maximum number of lines of displayed code by default
      # 默认展开显示的代码行数
      maxShownLines = 200
    # KaTeX mathematical formulas config (KaTeX https://katex.org/)
    # KaTeX 数学公式配置 (KaTeX https://katex.org/)
    [params.page.math]
      enable = false
      # default inline delimiter is $ ... $ and \( ... \)
      # 默认行内定界符是 $ ... $ 和 \( ... \)
      inlineLeftDelimiter = ""
      inlineRightDelimiter = ""
      # default block delimiter is $$ ... $$, \[ ... \], \begin{equation} ... \end{equation} and some other functions
      # 默认块定界符是 $$ ... $$, \[ ... \],  \begin{equation} ... \end{equation} 和一些其它的函数
      blockLeftDelimiter = ""
      blockRightDelimiter = ""
      # KaTeX extension copy_tex
      # KaTeX 插件 copy_tex
      copyTex = true
      # KaTeX extension mhchem
      # KaTeX 插件 mhchem
      mhchem = true
    # Mapbox GL JS config (Mapbox GL JS https://docs.mapbox.com/mapbox-gl-js)
    # Mapbox GL JS 配置 (Mapbox GL JS https://docs.mapbox.com/mapbox-gl-js)
    [params.page.mapbox]
      # access token of Mapbox GL JS
      # Mapbox GL JS 的 access token
      accessToken = "pk.eyJ1IjoiZGlsbG9uenEiLCJhIjoiY2s2czd2M2x3MDA0NjNmcGxmcjVrZmc2cyJ9.aSjv2BNuZUfARvxRYjSVZQ"
      # style for the light theme
      # 浅色主题的地图样式
      lightStyle = "mapbox://styles/mapbox/light-v10?optimize=true"
      # style for the dark theme
      # 深色主题的地图样式
      darkStyle = "mapbox://styles/mapbox/dark-v10?optimize=true"
      # whether to add NavigationControl (https://docs.mapbox.com/mapbox-gl-js/api/#navigationcontrol)
      # 是否添加 NavigationControl (https://docs.mapbox.com/mapbox-gl-js/api/#navigationcontrol)
      navigation = true
      # whether to add GeolocateControl (https://docs.mapbox.com/mapbox-gl-js/api/#geolocatecontrol)
      # 是否添加 GeolocateControl (https://docs.mapbox.com/mapbox-gl-js/api/#geolocatecontrol)
      geolocate = true
      # whether to add ScaleControl (https://docs.mapbox.com/mapbox-gl-js/api/#scalecontrol)
      # 是否添加 ScaleControl (https://docs.mapbox.com/mapbox-gl-js/api/#scalecontrol)
      scale = true
      # whether to add FullscreenControl (https://docs.mapbox.com/mapbox-gl-js/api/#fullscreencontrol)
      # 是否添加 FullscreenControl (https://docs.mapbox.com/mapbox-gl-js/api/#fullscreencontrol)
      fullscreen = true
    # Social share links in post page
    # 文章页面的分享信息设置
    [params.page.share]
      enable = false
      Twitter = true
      Facebook = true
      HackerNews = true
      Reddit = true
      Line = true
    # Comment config
    # 评论系统设置
    [params.page.comment]
      enable = true
      # giscus comment config (https://giscus.app/)
      # giscus comment 评论系统设置 (https://giscus.app/zh-CN)
      [params.page.comment.giscus]
        # You can refer to the official documentation of giscus to use the following configuration.
        # 你可以参考官方文档来使用下列配置
        enable = true
        repo = "xiaoshame/xiaoshame.github.io"
        repoId = "**"
        category = "Announcements"
        categoryId = "**"
        # automatically adapt the current theme i18n configuration when empty
        # 为空时自动适配当前主题 i18n 配置
        lang = ""
        mapping = "pathname"
        reactionsEnabled = "1"
        emitMetadata = "0"
        inputPosition = "bottom"
        lazyLoading = false
        lightTheme = "light"
        darkTheme = "dark"
    # Third-party library config
    # 第三方库配置
    [params.page.library]
      [params.page.library.css]
        # someCSS = "some.css"
        # located in "assets/" 位于 "assets/"
        # Or 或者
        # someCSS = "https://cdn.example.com/some.css"
      [params.page.library.js]
        # someJavascript = "some.js"
        # located in "assets/" 位于 "assets/"
        # Or 或者
        # someJavascript = "https://cdn.example.com/some.js"
    # Page SEO config
    # 页面 SEO 配置
    [params.page.seo]
      # image URL
      # 图片 URL
      images = ["/images/logo.png"]
      # Publisher info
      # 出版者信息
      [params.page.seo.publisher]
        name = "阿松日常"
        logoUrl = "/images/avatar.png"

  # TypeIt config
  # TypeIt 配置
  [params.typeit]
    # typing speed between each step (measured in milliseconds)
    # 每一步的打字速度 (单位是毫秒)
    speed = 100
    # blinking speed of the cursor (measured in milliseconds)
    # 光标的闪烁速度 (单位是毫秒)
    cursorSpeed = 1000
    # character used for the cursor (HTML format is supported)
    # 光标的字符 (支持 HTML 格式)
    cursorChar = "|"
    # cursor duration after typing finishing (measured in milliseconds, "-1" means unlimited)
    # 打字结束之后光标的持续时间 (单位是毫秒, "-1" 代表无限大)
    duration = -1

  # Site verification code for Google/Bing/Yandex/Pinterest/Baidu
  # 网站验证代码，用于 Google/Bing/Yandex/Pinterest/Baidu
  [params.verification]
    google = "*"
    bing = ""
    yandex = ""
    pinterest = ""
    baidu = ""

  # Site SEO config
  # 网站 SEO 配置
  [params.seo]
    # image URL
    # 图片 URL
    image = "/images/logo.png"
    # thumbnail URL
    # 缩略图 URL
    thumbnailUrl = "/images/logo.png"

  # Analytics config
  # 网站分析配置
  [params.analytics]
    enable = true
    # Google Analytics
    [params.analytics.google]
      id = "G-*"
      # whether to anonymize IP
      # 是否匿名化用户 IP
      anonymizeIP = true
    # Fathom Analytics
    [params.analytics.fathom]
      id = ""
      # server url for your tracker if you're self hosting
      # 自行托管追踪器时的主机路径
      server = ""
    # Plausible Analytics
    [params.analytics.plausible]
      dataDomain = ""
    # Yandex Metrica
    [params.analytics.yandexMetrica]
      id = ""

  # Cookie consent config
  # Cookie 许可配置
  [params.cookieconsent]
    enable = false
    # text strings used for Cookie consent banner
    # 用于 Cookie 许可横幅的文本字符串
    [params.cookieconsent.content]
      message = ""
      dismiss = ""
      link = ""

  # CDN config for third-party library files
  # 第三方库文件的 CDN 设置
  [params.cdn]
    # CDN data file name, disabled by default
    # ["jsdelivr.yml"]
    # located in "themes/LoveIt/assets/data/cdn/" directory
    # you can store your own data files in the same path under your project:
    # "assets/data/cdn/"
    # CDN 数据文件名称, 默认不启用
    # ["jsdelivr.yml"]
    # 位于 "themes/LoveIt/assets/data/cdn/" 目录
    # 可以在你的项目下相同路径存放你自己的数据文件:
    # "assets/data/cdn/"
    data = "jsdelivr.yml"

# Markup related configuration in Hugo
# Hugo 解析文档的配置
[markup]
  # Syntax Highlighting (https://gohugo.io/content-management/syntax-highlighting)
  # 语法高亮设置 (https://gohugo.io/content-management/syntax-highlighting)
  [markup.highlight]
    codeFences = true
    guessSyntax = true
    lineNos = true
    lineNumbersInTable = true
    # false is a necessary configuration (https://github.com/dillonzq/LoveIt/issues/158)
    # false 是必要的设置 (https://github.com/dillonzq/LoveIt/issues/158)
    noClasses = false
  # Goldmark is from Hugo 0.60 the default library used for Markdown
  # Goldmark 是 Hugo 0.60 以来的默认 Markdown 解析库
  [markup.goldmark]
    [markup.goldmark.extensions]
      definitionList = true
      footnote = true
      linkify = true
      strikethrough = true
      table = true
      taskList = true
      typographer = true
    [markup.goldmark.renderer]
      # whether to use HTML tags directly in the document
      # 是否在文档中直接使用 HTML 标签
      unsafe = true
  # Table Of Contents settings
  # 目录设置
  [markup.tableOfContents]
    startLevel = 2
    endLevel = 6

# Sitemap config
# 网站地图配置
[sitemap]
  changefreq = "weekly"
  filename = "sitemap.xml"
  priority = 0.5

# Permalinks config (https://gohugo.io/content-management/urls/#permalinks)
# Permalinks 配置 (https://gohugo.io/content-management/urls/#permalinks)
[Permalinks]
  # posts = ":year/:month/:filename"
  posts = ":filename"

# Options to make output .md files
# 用于输出 Markdown 格式文档的设置
[mediaTypes]
  [mediaTypes."text/plain"]
    suffixes = ["md"]

# Options to make output .md files
# 用于输出 Markdown 格式文档的设置
[outputFormats.MarkDown]
  mediaType = "text/plain"
  isPlainText = true
  isHTML = false

# Options to make hugo output files
# 用于 Hugo 输出文档的设置
[outputs]
  home = ["HTML", "RSS", "JSON"]
  page = ["HTML", "MarkDown"]
  section = ["HTML", "RSS"]
  taxonomy = ["HTML", "RSS"]

# Multilingual
# 多语言
[languages]
  [languages.zh-cn]
    weight = 2
    languageCode = "zh-CN"
    languageName = "简体中文"
    hasCJKLanguage = true
    copyright = "This work is licensed under a Creative Commons Attribution-NonCommercial 4.0 International License."
    [languages.zh-cn.menu]
      [[languages.zh-cn.menu.main]]
        weight = 1
        identifier = "posts"
        pre = "<i class='fa-solid fa-book'></i>"
        post = ""
        name = "所有文章"
        url = "/posts/"
        title = ""
      [[languages.zh-cn.menu.main]]
        weight = 2
        identifier = "favorites"
        pre = "<i class='fa-solid fa-bookmark'></i>"
        post = ""
        name = "收藏夹"
        url = "/favorites/"
        title = ""
      [[languages.zh-cn.menu.main]]
        weight = 3
        identifier = "comments"
        pre = "<i class='fa-solid fa-comment'></i>"
        post = ""
        name = "留言板"
        url = "/comments/"
        title = ""
      [[languages.zh-cn.menu.main]]
        weight = 4
        identifier = "sites"
        pre = "<i class='fa-solid fa-link'></i>"
        post = ""
        name = "友情链接"
        url = "/sites/"
        title = ""  
      # 二级菜单
      [[languages.zh-cn.menu.main]]
        parent = "posts"
        pre = "<i class='fas fa-fw fa-th'></i>"
        name = "分类"
        identifier = "categories"
        url = "/categories/"
        weight = 1
      [[languages.zh-cn.menu.main]]
        parent = "posts"
        identifier = "tags"
        post = ""
        pre = "<i class='fas fa-fw fa-tag'></i>"
        name = "标签"
        url = "/tags/"
        title = ""
        weight = 2
    [languages.zh-cn.params]
      [languages.zh-cn.params.search]
        enable = true
        type = "algolia"
        contentLength = 4000
        placeholder = ""
        maxResultLength = 10
        snippetLength = 50
        highlightTag = "em"
        absoluteURL = false
        [languages.zh-cn.params.search.algolia]
          index = "index.zh-cn"
          appID = "PASDMWALPK"
          searchKey = "b42948e51daaa93df92381c8e2ac0f93"
      [languages.zh-cn.params.home]
        rss = 10
        [languages.zh-cn.params.home.profile]
          enable = true
          gravatarEmail = "xiaoshame1209@gmail.com"
          avatarURL = "/images/avatar.png"
          title = ""
          subtitle = "一位普通的奶爸，一个不出名的程序员，正在努力赚钱和感受生活的酸甜苦辣"
          typeit = true
          social = true
          disclaimer = ""
        [languages.zh-cn.params.social]
          GitHub = "https://github.com/xiaoshame/"
          Email = "xiaoshame1209@gmail.com"
          RSS = true
        [languages.zh-cn.params.home.posts]
          enable = true
          # special amount of posts in each home posts page
          # 主页每页显示文章数量
          paginate = 10
```

### Github部署

1. 参考[Hugo官方github部署](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
2. 官方文档要求将hugo.toml传到git上，其中包含部分key，存在风险
   1. 使用hugo生成pubilc文件，上传到仓库
   2. 修改blog/.github/workflows/hugo.yaml文件,核心是屏蔽hugo在远端构建，只对pubilc文件打包

   3. ```
        # Sample workflow for building and deploying a Hugo site to GitHub Pages
        name: Deploy Hugo site to Pages

        on:
        # Runs on pushes targeting the default branch
        push:
            branches:
            - master

        # Allows you to run this workflow manually from the Actions tab
        workflow_dispatch:

        # Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
        permissions:
        contents: read
        pages: write
        id-token: write

        # Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
        # However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
        concurrency:
        group: "pages"
        cancel-in-progress: false

        # Default to bash
        defaults:
        run:
            shell: bash

        jobs:
        # Build job
        build:
            runs-on: ubuntu-latest
            # env:
            #   HUGO_VERSION: 0.119.0
            steps:
            # - name: Install Hugo CLI
            #   run: |
            #     wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
            #     && sudo dpkg -i ${{ runner.temp }}/hugo.deb  
            - name: Install Dart Sass
                run: sudo snap install dart-sass
            - name: Checkout
                uses: actions/checkout@v3
                with:
                submodules: recursive
                fetch-depth: 0
            - name: Setup Pages
                id: pages
                uses: actions/configure-pages@v3
            - name: Install Node.js dependencies
                run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
            # - name: Build with Hugo
            #   env:
            #     # For maximum backward compatibility with Hugo modules
            #     HUGO_ENVIRONMENT: production
            #     HUGO_ENV: production
            #   run: |
            #     hugo \
            #       --gc \
            #       --minify \
            #       --baseURL "${{ steps.pages.outputs.base_url }}/"  
            - name: Upload artifact
                uses: actions/upload-pages-artifact@v1
                with:
                path: ./public

        # Deployment job
        deploy:
            environment:
            name: github-pages
            url: ${{ steps.deployment.outputs.page_url }}
            runs-on: ubuntu-latest
            needs: build
            steps:
            - name: Deploy to GitHub Pages
                id: deployment
                uses: actions/deploy-pages@v2
      ```

### 配置Giscus评论

1. 对应项目开启GitHub Discussions
2. github上安装[Giscus](https://github.com/apps/giscus)
3. 利用GitHub GraphQL API获取repoId，categoryId，参考[Github 如何获取仓库的 repo_id 和 category_id](https://maling.io/posts/get-github-repo-id/)
4. 修改hugo.toml中repo，repoId，category，categoryId，打开giscus关闭valine，通过enable字段控制，参考安装部分hugo.toml文件
5. `hugo`生成相应代码，同步pubilc文件夹到仓库中

### 收藏页

1. 修改hugo.toml中menu.main配置，添加favorites菜单
2. 生成页面
   1. 使用指令 `hugo new favorites/index.md`，在blog/content目录下生成favorites/index.md文件
   2. 在index.md中编辑内容，draft 值调整为fasle，表示不是草稿
   3. `hugo`生成相应页面代码，同步pubilc文件夹到仓库

### 封面图

本想给文章加个封面图，在查资料解决过程中，看到一个基于LoveIt魔改主题搭建的[博客](https://hakula.xyz/)，文章的样式符合预期，分析自己博客与对方的不同，参考调整

1. 删除原LoveIt主题，删除blog/theme/LoveIt 文件夹
2. 删除blog/.git/config中[submodule "themes/LoveIt"]的配置
3. 添加LoveIt主题，`git submodule add git@github.com:hakula139/LoveIt.git themes/LoveIt`
4. 下载[对方博客内容](https://github.com/hakula139/hakula.xyz)，将hakula.xyz-main/assets中的内容放到自己博客的blog/assets目录中
5. 封面图片可以放到文章同级目录中
6. 修改文章参数设置，参考如下

7. ```
   ---
   title :  'Hugo+Github零成本搭建博客流程记录'
   date : 2023-10-11T10:06:03+08:00
   draft : false
   tags : [hugo,github,博客]
   categories : [技术]
   featuredImage : /hugo_blog/1697027912506.png
   ---
   ```

### 站点运行时间

通过自定义js实现

1. blog/themes/LoveIt/layouts/partials/assets.html 拷贝到blog/layouts/partials/assets.html
2. blog/themes/LoveIt/layouts/partials/footer.html 拷贝到blog/layouts/partials/footer.html
3. 创建blog/static/js/custom.js 文件，粘贴如下代码

   1. ```js
      /* 站点运行时间 */
      function runtime() {
      window.setTimeout("runtime()", 1000);
      /* 请修改这里的起始时间 */
      let startTime = new Date('10/10/2023 15:00:00');
      let endTime = new Date();
      let usedTime = endTime - startTime;
      let days = Math.floor(usedTime / (24 * 3600 * 1000));
      let leavel = usedTime % (24 * 3600 * 1000);
      let hours = Math.floor(leavel / (3600 * 1000));
      let leavel2 = leavel % (3600 * 1000);
      let minutes = Math.floor(leavel2 / (60 * 1000));
      let leavel3 = leavel2 % (60 * 1000);
      let seconds = Math.floor(leavel3 / (1000));
      let runbox = document.getElementById('run-time');
      runbox.innerHTML = '本站已运行`<i class="far fa-clock fa-fw"></i>` '
      + ((days < 10) ? '0' : '') + days + ' 天 '
      + ((hours < 10) ? '0' : '') + hours + ' 时 '
      + ((minutes < 10) ? '0' : '') + minutes + ' 分 '
      + ((seconds < 10) ? '0' : '') + seconds + ' 秒 ';
      }
      runtime();
      ```

4. 修改blog/layouts/partials/assets.html，在最后 `{{- partial "plugin/analytics.html" . -}}`前面加上

   1. ```js
      {{- /* 自定义的js文件 */ -}}
      <script type="text/javascript" src="/js/custom.js"></script>
      ```

5. 修改blog/layouts/partials/footer.html，在 `<divclass="footer-container">`下面加上

   1. ```js
      <div class="footer-line">
          <span id="run-time"></span>
      </div>
      ```

### 二级菜单

1. 调整和相关代码完全参考[Hugo系列(3.2) - LoveIt主题美化与博客功能增强 · 第三章](https://lewky233.top/posts/hugo-3.2.html/#%E8%8F%9C%E5%8D%95%E6%A0%8F%E6%94%AF%E6%8C%81%E5%AD%90%E8%8F%9C%E5%8D%95)
2. 二级菜单调整见上面hugo.toml
3. 菜单logo使用[fontawesome](https://fontawesome.com/),在hugo中配置菜单对应pre参数
