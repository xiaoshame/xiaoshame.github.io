---
title : "Hugo使用小结" 
description : "" 
date : "2023-12-09T11:11:06+08:00" 
lastmod : "2023-01-23T20:16:00+08:00" 
tags : ["hugo","tips"] 
categories : ["技术"]
draft : false
featuredImage : /images/posts/hugo_tips/featuredImage.jpg
summary : '本文总结了使用hugo建立博客需要的前端和hugo知识。同时，还给出了三个具体的案例，分别是使用Giscus进行评论、修复Lunr搜索不支持中文的问题和如何在博客中添加说说功能的步骤。本文重点强调阐述解决问题的思维方式。'
---

### 背景

使用hugo创建博客接近2个月，一直走在折腾样式的路上，随着不停的折腾，建立了对hugo初步的认识。本文主要用于梳理个人对hugo和前端知识的理解，同时希望能帮助对hugo和前端知识不熟悉的人，建立相关的概念。

{{< notice notice-info >}}
内容仅供参考，本人对前端知识了解不多，部分内容的理解必然存在问题。如果您发现了问题，欢迎在评论区留言指正。
{{< /notice >}}

### 基础知识

首先建立hugo的工作原理，了解hugo的目录结构、shortcode使用、html的基础信息方便自定义修改。推荐阅读资料[hugo官方文档](https://gohugo.io/getting-started/)

#### hugo工作原理

- hugo工作原理：我的理解是按照指定的规则生成html文件。
  - 主题就是已经写好的转换规则，基于主题的自定义，就是修改转化规则
  - 文章是将markdown文件，按照主题预设的规则转换成html
  - 页面是将主题预设的规则和自定义规则，拼凑成html
  - 自定义规则一般不会直接修改主题，而是在博客目录的assets、layouts、static/js中添加内容进行修改
- hugo 目录结构
  - assets 用于存放scss文件，主要用于样式的渲染
  - content 用于存放.md文件，主要是文章内容和自定义页面
  - data 存放一些数据文件，自定义的结构化数据，比如JSON、YAML或TOML格式的文件。
  - layouts用来存放html元素文件，在页面指定的位置添加元素内容，定义页面布局
  - static 不需要经过处理的静态文件，比如图片、样式表和脚本文件。
    - js 用于放js文件，用于动态的修改展示的信息
  - themes 目录包含自定义主题的文件，如果你使用了自定义主题的话。
  - hugo.toml 是网站的配置文件，用于定义网站的各种设置。
- shortcode 按照模板在指定位置插入数据，常用语句如下
  - {{ .Get "class" }},取key为class的值
  - {{ .Inner }},取shortcode标签对之间的内容
  - {{ with .Get "class" }} class="{{ . }}"{{ end }}，如果存在key为class 输出class="对应值"，否则什么都不输出
  - {{ if (.Get "title")}} class="xxx" {{else}} class="yyy" {{ end }},如果存在key为title,class为"xxx"否则为"yyy"
  - {{ .Get 0 }},按位置访问参数,取第一个参数的值
  - [Hugo Shortcodes 示例](https://blog.yandaojiang.com/posts/others/hugo_shortcodes%E7%A4%BA%E4%BE%8B)
  - [自定义 Hugo Shortcodes 简码](https://guanqr.com/tech/website/hugo-shortcodes-customization/)

#### HTML

- html内容基本可以划分为以下几个部分:
  - content: 这不是HTML标准元素，但可以是自定义元素。在Web组件中，可能会使用<content>标签来插入组件内的内容。
  - footer: 通常用于网页底部，放置版权信息、联系方式、链接等内容。
  - head: 在HTML文档中表示头部部分，包含了页面的元数据和链接引用等信息，如标题、样式表链接、脚本引用等。
  - header: 通常用于网页顶部，包含网站的标题、导航栏等内容。
  - post-meta: 这同样不是HTML标准元素，可以是自定义元素。在博客或新闻网站中，可能会使用<post-meta>标签来展示文章的元信息，比如作者、发布日期等。
  - script: 用于在网页中嵌入JavaScript代码的标签，可以用于引入外部脚本或直接书写JavaScript代码。
  - 此处内容详情由GPT3生成。

### 案例

#### 评论使用Giscus

{{< notice notice-tip >}}
meme主题最新版本已支持Giscus，在hugo.toml中添加配置即可使用。实现方案与本文内容类似
{{< /notice >}}

具体操作流程参考文章[安裝Giscus作為Hugo網站的留言板，支援轉換Gitalk/Utterances的留言](https://ivonblog.com/posts/hugo-giscus-comment/)
主要操作点如下：

1. 在github中启用Giscus功能
2. 在`\layouts\partials\comments`中添加giscus.html文件，粘贴[Giscus官网](https://giscus.app/zh-CN)中获取的Giscus代码
3. 复制`themes\meme\layouts\partials\pages\post.html`放到`\layouts\partials\pages\`中
4. 修改`\layouts\partials\pages\post.html` 中`{{ partial "components/comments.html" . }}`为`{{ partial "comments/giscus.html" . }}`
5. 评论无法动态切换成暗黑模式，修改`\themes\meme\assets\js\dark-mode.js`中changeMode函数，增加下面代码

```js
    const giscusIframe = document.querySelector('.giscus-frame');
    if (giscusIframe) {
        // 检测当前主题是否为暗黑模式
        if (isDark) {
            giscusIframe.src = giscusIframe.src.replace('theme=light', 'theme=dark');
        } else {
            giscusIframe.src = giscusIframe.src.replace('theme=dark', 'theme=light');
        }
    }
```

#### Lunr搜索支持中文

整体修复流程如下：

1. lunr搜索官方不支持中文,google 找到了开源lurn扩展项目[lunr-languages](https://github.com/MihaiValentin/lunr-languages)
2. 在[cdn.jsdelivr.net](https://www.jsdelivr.com/)找到扩展项目对应lunr-zh.js地址，修改hugo.toml中lunr_lang参数`lunr_lang="/npm/lunr-languages-zh@1.4.0/lunr.zh.js"`，在网页中测试不支持中文，查看网页html源码发现未引用lunr-zh.js
3. 阅读主题源码是blog\themes\meme\layouts\partials\third-party\lunr-search.html中`if in $supported $lang` 过滤zh导致，在$supported中添加"zh",查阅页面html发现lunr-zh.js已引用，测试搜索仍然不支持中文，同时网页控制台报错`未捕获的类型错误：nodejieba.cut 不是函数`
4. 查看[lunr-languages issues](https://github.com/MihaiValentin/lunr-languages/issues/91)提示原因是需要node运行，同时贴出了解决项目方案[mochi-cards/lunr-languages](https://github.com/mochi-cards/lunr-languages/tree/mochi/zh-novel-segment)
5. 修复版本的lurn-zh.js无法通过在线地址方式导入，调整为本地导入。下载mochi-cards/lunr-languages项目中的lunr-zh.js文件放入/static/js中
6. 在`layouts/partials/custom`创建script.html文件，填入`<script src="/js/lunr.zh.js" defer></script>`
7. 修改lunr-search.html在线导入代码，`{{- $scripts = union $scripts (slice $srcLang) -}}`调整为`{{- if eq $lang "zh" -}}{{- else -}}{{- $scripts = union $scripts (slice $srcLang) -}}{{- end -}}`
8. 运行测试控制台提示'Lunr is not present. Please include / require Lunr before this script.'，查看lurn-zh.js源码是
`'undefined' === typeof lunr`导致
9. 因对前端知识不了解，只能通过控制变量进行测试定位问题。下载lurn-de.js文件，将引入lurn-zh.js文件调整为引入lurn-de.js文件，运行无报错。对比两个文件源码，lurn-zh.js中`(this, function(Segment) {` 比lurn-de.js多Segment参数，确定Segment未使用删除Segment参数，再次测试运行正常
10. 在[cdn.jsdelivr.net](https://www.jsdelivr.com/)中找到支持中文的lurn.js,修改hugo.toml中`lunr = "/npm/lunr-zh-cn@0.7.1/lunr.min.js"`,测试后对中文的搜索支持的更好。

整个修复流程使我对hugo的结构有了更一步的理解，同时也对meme主题有了更深的认知。

#### 创建说说页面

一直想在博客中增加说说功能，在对shortcode的使用慢慢熟悉后，开始动手操作将主题[hugo-theme-moments](https://github.com/FarseaSH/hugo-theme-moments)的样式集成到博客中。

解决思路：新建博客使用hugo-theme-moments主题，分析博客网页说说内容部分的html元素，抽象为shortcode。将说说部分对应的css、js文件集成博客中，最终使用shortcode发说说。

hugo-theme-moments主题生成的网页中使用到的css和js信息：

```javascript
<script type="text/javascript" src="https://unpkg.com/jquery@3.3.1/dist/jquery.min.js"></script>
<script type="text/javascript" src="https://cdn.bootcss.com/jqueryui/1.12.1/jquery-ui.min.js"></script>
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/@fancyapps/ui/dist/fancybox.umd.js"></script>
<link
  rel="stylesheet"
  href="https://cdn.jsdelivr.net/npm/@fancyapps/ui/dist/fancybox.css"
/>
<link
  rel="stylesheet"
  href="https://unpkg.com/purecss@2.0.6/build/grids-min.css"
/>
<link
  rel="stylesheet"
  href="https://unpkg.com/purecss@2.0.6/build/grids-responsive-min.css"
/>
<link
  rel="stylesheet"
  href="https://farseash.github.io/demo-hugo-theme-moments/style-refractored.min.f763c606ea7f621cac55d5419e71c55d50792a0b67ae61e88a8de81fa2cc2b3d.css"
/>
<link
  rel="stylesheet"
  href="https://cdn.jsdelivr.net/npm/@fancyapps/ui/dist/fancybox.css"
/>
```

抽象后的shortcode：
  
```javascript
// 文本内容shortcode
<div class="moment-row">
    <div class="pure-g">
        <div class="pure-u-1 pure-u-sm-1-3">
            <div class="time">
                <div class="month-day">{{.Get "date"}}</div>
                <div class="year">{{.Get "year"}}</div>
            </div>
        </div>
        <div class="pure-u-1 pure-u-sm-2-3">
            <div class="moment-content">
                <div style="padding-left: 5px; padding-right: 5px;">
                    <div class="context">
                        <p>{{.Get "content"}}</p>
                    </div>
                    <div class="pic-grid" style="width: 53%">
                        <div class="pure-g">
                        {{ .Inner }}
                        </div>
                    </div>
                    <div class="share-link-block">
                        {{with .Get "url"}}<p class="url"><a href = {{ . }}>{{ . }}</a></p>{{end}}
                    </div>
                </div>
                <hr>
            </div>
        </div>
    </div>
</div>

// 图片内容shortcode
<div class="pure-u-1-2">
    <div class="add-padding">
        <div class="img-container">
            <a data-src="{{ .Get "url" }}" data-fancybox="gallery" {{ with .Get "caption" }}data-caption="{{ . }}"{{end}}>
                <img src="{{ .Get "url" }}" alt="pictures" />
            </a>
        </div>
    </div>
</div>

```

1. 将用到的css信息下载下来放到assets/scss中，文件尾缀修改为scss
2. layouts/partials/custom/script.html中增加上面贴出的js文件
3. layouts/shortcodes中新建moment.html和moment_image.html，将上面shortcode复制进去
4. 为了只在说说页面使用相关的scss文件，在layouts/shortcodes中新建moments.html
5. 新建页面和添加菜单操作不赘述，发说说使用如下语句即可

```markdown
{{</* moments */>}}

{{</* moment year = "2023" date = "12.06" content = "更换博客主题，折腾博客比写文章快乐!" */>}}
{{</* /moment */>}}

{{</* moment year = "2023" date = "12.05" content = "第一条" */>}}
{{</* moment_image url = "/moments/laugh.png" */>}}
{{</* /moment */>}}

{{</* moments */>}}
```

```javascript
//moments.html内容
{{ $options := (dict "targetPath" "/css/style-refractored.css" "outputStyle" "compressed" "enableSourceMap" true) }}
{{ $style := resources.Get "/scss/style-refractored.scss" | resources.ToCSS $options }}

{{ $options1 := (dict "targetPath" "/css/grids-min.css" "outputStyle" "compressed" "enableSourceMap" true) }}
{{ $style1 := resources.Get "/scss/grids-min.scss" | resources.ToCSS $options1 }}

{{ $options2 := (dict "targetPath" "/css/grids-responsive-min.css" "outputStyle" "compressed" "enableSourceMap" true) }}
{{ $style2 := resources.Get "/scss/grids-responsive-min.scss" | resources.ToCSS $options2 }}

{{ $options3 := (dict "targetPath" "/css/fancybox.css" "outputStyle" "compressed" "enableSourceMap" true) }}
{{ $style3 := resources.Get "/scss/fancybox.scss" | resources.ToCSS $options3 }}

<link rel="stylesheet" href="{{ $style.RelPermalink }}">
<link rel="stylesheet" href="{{ $style1.RelPermalink }}">
<link rel="stylesheet" href="{{ $style2.RelPermalink }}">
<link rel="stylesheet" href="{{ $style3.RelPermalink }}">

{{ .Get 0 }}
{{ range split .Inner "\n" }}
{{ $line := replace . " " " " }}
{{ printf "%s" $line | safeHTML }}
{{ end }}
```

### bug修复

1. 手机上搜索框不展示
    1. 页面上调试发现搜索框与其他菜单class值不同，搜索为`class="menu-item search-item"`， 与其他菜单为`class="menu-item"`
    2. 手动删除search-item显示正常，在代码中搜素search-item，删除menu.html中search-item即可修复此问题
2. 说说页面无暗黑模式
    1. 将`\assets\scss\style-refractored.scss`中color相关的代码删除，复用主题的色彩

### 总结

1. 本文最想告知读者和自我总结的是:解决问题的思维方式。希望读者可以感受到这点，也希望本文可以帮助读者解决一些问题。
2. 间隔1个月后再次阅读本文，整体文章内容偏多,结构不清晰，表述能力需要加强
