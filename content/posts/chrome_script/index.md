---
title : "使用大模型编写chrome插件"
date : "2024-11-05T13:52:52+08:00" 
lastmod : "2024-11-05T13:52:52+08:00" 
tags : ["chrome","scrite","大模型"] 
categories : ["技术"]
draft : false
featuredImage : /images/posts/chrome_script/featuredImage.jpg
summary : '使用大模型编写chrome插件，删除无效书签'
---

### 背景

日积月累浏览器中收藏的书签网页越来越多，同时失效的书签也越来越多，如何快速删除无效的书签？一直想写个chrome插件，趁着这个机会，练练手。

### 思路
大模型的出现真是大大降低了简单需求实现的成本，本人没有系统学习过js相关知识，平时可以看懂简单的js代码，复杂点的代码需要连蒙带猜才能看懂。

为方便大模型理解，我把自己的需求拆解为3个步骤：
  1. 编写chrome插件读取浏览器书签信息，并展示
  2. 对书签地址进行检测，只展示无法访问的书签信息
  3. 在无法访问的书签信息后面增加删除按钮，点击按钮删除浏览器中对应的无效书签

首先使用的是文心一言的文心快码，给出的代码无法正常运行。换成GPT4o，给出的代码可以直接使用，非常方便，点赞。

### 实践

#### 与大模型沟通
按照步骤输入大模型，获得阶段代码后，验证正确性。确认相关代码无误后开展下一步，一步一步调整后，最终代码实现如下：

manifest.json 内容如下：
```json
{
  "manifest_version": 3,
  "name": "Bookmark Viewer",
  "version": "1.0",
  "description": "清理无效书签的Chrome插件",
  "permissions": [
    "bookmarks",
    "webRequest",
    "webRequestBlocking",
    "<all_urls>"
  ],
  "host_permissions": [
    "<all_urls>"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html",
    "default_title": "View Bookmarks",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  },
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  }
}
```

background.js 内容如下：
```javascript
// 用于初始化的后台脚本，当前不需要特别功能，可留空
console.log("Bookmark Viewer Background Script Running.");
```

popup.js 内容如下：
```javascript
document.addEventListener('DOMContentLoaded', function () {
  const bookmarksList = document.getElementById('bookmarks');

  function createBookmarkNode(bookmark, isValid) {
    if (!isValid) {
      // 创建一个列表项和锚点以显示书签信息
      const li = document.createElement('li');
      const anchor = document.createElement('a');
      anchor.href = bookmark.url;
      anchor.textContent = `${bookmark.title} (${bookmark.url})`;
      anchor.target = '_blank';
      li.appendChild(anchor);

      // 创建删除按钮
      const deleteBtn = document.createElement('button');
      deleteBtn.textContent = 'Delete';
      deleteBtn.onclick = () => deleteBookmark(bookmark.id, li);
      li.appendChild(deleteBtn);

      return li;
    }
    return null;
  }

  function deleteBookmark(bookmarkId, li) {
    // 调用 Chrome 的书签 API 删除书签
    chrome.bookmarks.remove(bookmarkId, function() {
      bookmarksList.removeChild(li);
    });
  }

  function checkBookmarkValidity(url, callback) {
    fetch(url, { method: 'HEAD' })
      .then(response => {
        if (response.status == 404 || response.status == 500) {
          callback(false);
        }else{
          callback(true);
        }
      })
      .catch(() => {
        callback(false); // 请求失败认为无效
      });
  }

  function traverseBookmarks(bookmarkTreeNodes, parentElement) {
    bookmarkTreeNodes.forEach((bookmark) => {
      if (bookmark.children) {
        // 创建目录节点
        const folderLi = document.createElement('li');
        folderLi.textContent = bookmark.title;
        const ul = document.createElement('ul');
        folderLi.appendChild(ul);
        parentElement.appendChild(folderLi);
        // 递归遍历子书签
        traverseBookmarks(bookmark.children, ul);
      } else if (bookmark.url) {
        checkBookmarkValidity(bookmark.url, (isValid) => {
          const bookmarkNode = createBookmarkNode(bookmark, isValid);
          if (bookmarkNode) {
            parentElement.appendChild(bookmarkNode);
          }
        });
      }
    });
  }

  chrome.bookmarks.getTree(function(bookmarkTreeNodes) {
    traverseBookmarks(bookmarkTreeNodes, bookmarksList);
  });
});
```

popup.html 内容如下：
```html
<!DOCTYPE html>
<html>
<head>
  <title>Invalid Bookmarks</title>
  <style>
    ul { list-style-type: none; }
    li { margin: 5px 0; }
    button { margin-left: 10px; }
  </style>
</head>
<body>
  <h1>Invalid Bookmarks</h1>
  <ul id="bookmarks"></ul>
  <script src="popup.js"></script>
</body>
</html>
```

popup.css 内容如下：
```css
body {
    width: 300px;
    font-family: Arial, sans-serif;
  }
  ul {
    list-style-type: none;
    padding: 0;
  }
  li {
    margin: 5px 0;
  }
  a {
    text-decoration: none;
    color: blue;
  }
  a:hover {
    text-decoration: underline;
  }
```

#### 安装扩展

相关代码按照目录结构放在文件中，然后在chrome浏览器->设置->扩展程序->管理扩展程序->加载已解压的扩展程序，加载该目录即可安装插件。

目录结构：
![tree](/images/posts/chrome_script/tree.png)

#### 调试扩展

![tree](/images/posts/chrome_script/debug.png)

插件点右键->审查弹出内容
  1. console，即可看到日志输出
  2. Sources, 增加断点，调试js代码

[完整代码地址](https://github.com/xiaoshame/script/tree/main/url_check)

### 其他

本打算将插件上传chrome官方商店，但是注册开发者账号需要5美金，所以放弃了。需要的可以从github下载，本地安装使用。