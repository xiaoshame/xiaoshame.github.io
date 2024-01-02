---
title : "为博客Giscus评论增加通知机器人" 
date : "2023-12-18T15:34:23+08:00" 
lastmod : "2023-12-18T15:34:23+08:00" 
tags : ["Hugo","代理","JS","cloudflare","Giscus"] 
categories : ["技术"]
draft : false
featuredImage : /images/posts/webhook_proxy/featuredImage.jpg
summary : '文章介绍了如何给Giscus系统增加一个机器人通知功能的方案和实现方式。使用github的webhook和cloudflare的worker服务，可以将新增的discussion和comment信息转发给企业微信或飞书。'
---

## 背景

最近有人对博客内容进行了评论，未收到任何通知，过了几天才发现，不利于沟通交流。因工作中经常使用企业微信，所以准备给Giscus系统增加一个企业微信通知功能。

## 方案

1. google搜索资料,阅读[为Giscus增加邮件通知功能](https://wenchao.ren/posts/%E4%B8%BAGiscus%E5%A2%9E%E5%8A%A0%E9%82%AE%E4%BB%B6%E9%80%9A%E7%9F%A5%E5%8A%9F%E8%83%BD.html)，[拉仇恨！webhook + 企业微信 给同事做了个代码提交监听工具](https://cloud.tencent.com/developer/article/1833773)等文章
2. 方案初步明确：基于企业微信和github的webhook功能，增加一个转发服务即可。
3. 最近使用cloudflare的worker功能，用的非常开心。基于cloudflare的worker服务利用js可以很好完成消息处理和转发。
4. 方案成型:利用github的webhook将新增的discussion和comment信息转发给cloudflare worker中的js服务，js服务对消息处理后通过webhook转发企业微信

## 实现

在github中找到项目[huhuhang/github-wechat-bot](https://github.com/huhuhang/github-wechat-bot/blob/master/README.md)，此项目提供了基于 Cloudflare Workers 部署 API，支持基于 GitHub Webhook 将操作消息推送给企业微信机器人。只用支持Discussions的监控就行。

1. 按照项目的Readme，完成github/企业微信/cloudflare的设置，设置完成后在企业微信可以看到群机器人发送的ping请求。如果没有看到可以在github webhook页面，点击Recent Deliveries,可以重新发送。如果还没有看到可以执行检测设置是否正确。

2. github webhook页面点Edit，然后勾选监控"Discussion comments","Discussions"消息

3. 增加Discussions消息js处理代码，主要通过打印日志进行调试，最终js代码如下
    1. cloudflare在Logs->Real-time Logs->Begin log stream可以看到console.log打印的日志

```JavaScript
/**
 * 
 * @param {JSON} response 处理JSON格式的响应
 * @returns 
 */
async function gatherResponse(response) {
  const { headers } = response
  const contentType = headers.get("content-type") || ""
  if (contentType.includes("application/json")) {
    return JSON.stringify(await response.json())
  }
}

/**
 * 
 * @param {String} botKey 企业微信机器人密钥
 * @param {String} content 需要发送的内容，支持 Markdown 格式
 * @returns 
 */
async function sendMdMsg(botKey, content) {
  const baseUrl = "https://qyapi.weixin.qq.com/cgi-bin/webhook/"
  const url = `${baseUrl}send?key=${botKey}`
  const init = {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      "msgtype": "markdown",
      "markdown": {
        "content": content
      }
    })
  }
  const response = await fetch(url, init)
  return await gatherResponse(response)
}

// 标记事件动作
const actionWords = {
  "opened": "<font color='info'>创建</font>",
  "closed": "<font color='warning'>关闭</font>",
  "deleted": "<font color='info'>删除</font>",
  "reopened": "<font color='info'>重新发起</font>",
  "edited": "<font color='info'>更新</font>",
  "merge": "<font color='warning'>合并</font>",
  "created": "<font color='info'>创建</font>",
  "requested": "<font color='info'>请求</font>",
  "completed": "<font color='warning'>完成</font>",
  "synchronize": "<font color='info'>同步更新</font>"
};

/**
 * 处理 Ping 请求
 * @param {String} botKey 企业微信机器人密钥
 * @param {JSON} reqBody GitHub 传递的请求体
 * @returns 
 */
async function handlePing(botKey, reqBody) {
  const { hook, repository, organization } = reqBody;
  // 判定是组织还是仓库配置 Webhook
  if (hook.type == "Organization") {
    var mdMsg = "成功收到了来自 Github 的 Ping 请求，组织: " + organization.login;
  } else {
    var mdMsg = "成功收到了来自 Github 的 Ping 请求，仓库地址: " + repository.html_url;
  }
  return await sendMdMsg(botKey, mdMsg);
}

/**
 * 处理 PR 请求
 * @param {String} botKey 企业微信机器人密钥
 * @param {JSON} reqBody GitHub 传递的请求体
 * @returns 
 */
async function handlePR(botKey, reqBody) {
  const { action, sender, pull_request, repository } = reqBody;
  if (sender.type !== "Bot") {
    if (action == "opened" || action == "reopened") {
      const mdMsg = `${sender.login} 在 [${repository.full_name}](${repository.html_url}) ${actionWords[action]}了一个 PR:
      > 分支: ${pull_request.head.ref} → ${pull_request.base.ref}
      > 名称: [${pull_request.title}](${pull_request.html_url}) #${pull_request.number}
      > 修改: ${pull_request.changed_files} 个文件 (<font color="info">+ ${pull_request.additions}</font> <font color="warning">- ${pull_request.deletions}</font> 行修改)`;
      return await sendMdMsg(botKey, mdMsg);
    }
    else if (action == "closed" && pull_request.merged) {
      const mdMsg = `${sender.login} 在 [${repository.full_name}](${repository.html_url}) ${actionWords[action]}了一个 PR:
      > 分支: ${pull_request.head.ref} → ${pull_request.base.ref}
      > 名称: [${pull_request.title}](${pull_request.html_url}) #${pull_request.number}
      > 修改: ${pull_request.changed_files} 个文件 (<font color="info">+ ${pull_request.additions}</font> <font color="warning">- ${pull_request.deletions}</font> 行修改)
      > 发起: ${pull_request.user.login} (${pull_request.created_at})
      > 审核: ${pull_request.merged_by.login} (${pull_request.review_comments} 条意见)`;
      return await sendMdMsg(botKey, mdMsg);
    }
    else {
      return `${action} 操作暂时不会被处理`;
    }
  } else {
    return `${sender.type} 操作暂时不会被处理`;
  }
}

/**
 * 处理 Issues 请求
 * @param {String} botKey 企业微信机器人密钥
 * @param {JSON} reqBody GitHub 传递的请求体
 * @returns 
 */
async function handleIssue(botKey, reqBody) {
  const { action, sender, issue, repository } = reqBody;
  if (action == "opened" || action == "closed" || action == "reopened") {
    const mdMsg = `${sender.login}  在 [${repository.full_name}](${repository.html_url}) ${actionWords[action]}了一个 Issues:
    > 名称: [${issue.title}](${issue.html_url})`;
    return await sendMdMsg(botKey, mdMsg);
  }
  else {
    return `${action} 操作暂时不会被处理`;
  }
}

/**
 * 处理 discussion_comment 请求
 * @param {String} botKey 企业微信机器人密钥
 * @param {JSON} reqBody GitHub 传递的请求体
 * @returns 
 */
async function handle_discussion_comment(botKey, reqBody) {
  const { action, sender, comment, discussion } = reqBody;
  if (action == "created" || action == "deleted" || action == "edited") {
    const mdMsg = `${sender.login}  在 ${discussion.title}中${actionWords[action]}评论:${comment.body}`;
    return await sendMdMsg(botKey, mdMsg);
  } else {
    return `${action} 操作暂时不会被处理`;
  }
}

/**
 * 处理 Action 错误请求
 * @param {String} botKey 企业微信机器人密钥
 * @param {JSON} reqBody GitHub 传递的请求体
 * @returns 
 */
async function handleAction(botKey, reqBody) {
  const { action, sender, check_run, repository } = reqBody;
  // 如果状态完成且执行失败，则发送错误信息
  if (action == "completed" && check_run.conclusion == "failure") {
    const mdMsg = `${sender.login}  在 [${repository.full_name}](${repository.html_url}) 中触发的 GitHub Action 执行<font color="warning">失败</font>了:
    > 查看状态: [${check_run.name}](${check_run.html_url})
    > 错误信息: ${check_run.output.summary}`;
    return await sendMdMsg(botKey, mdMsg);
  }
  else {
    return `${action}(${check_run.conclusion}) 暂时不会被处理`;
  }
}

/**
 * 
 * @param {JSON} request GitHub 传递的请求
 * @returns 
 */
async function handleRequest(request) {
  const { searchParams } = new URL(request.url)
  // 从 URL 获取传入的机器人密钥
  let botKey = searchParams.get('key')
  // 从请求中获取消息内容
  var reqBody = await gatherResponse(request)
  // 解析 GitHub 传递的消息类型
  const gitEvent = request.headers.get("X-GitHub-Event")
  console.log(`收到了一个 ${gitEvent} 事件`)
  switch (gitEvent) {
    // 如果是 Ping 事件
    case "ping":
      var results = await handlePing(botKey, JSON.parse(reqBody));
      break;
    // 如果是 PR 事件
    case "pull_request":
      var results = await handlePR(botKey, JSON.parse(reqBody));
      break;
    // 如果是 Issues 事件
    case "issues":
      var results = await handleIssue(botKey, JSON.parse(reqBody));
      break;
    // 如果是 Action 事件
    case "check_run":
      var results = await handleAction(botKey, JSON.parse(reqBody));
      break;
    // 如果是 discussion_comment 事件
    case "discussion_comment":
      var results = await handle_discussion_comment(botKey,JSON.parse(reqBody));
      break;
    // 其他事件暂不支持
    default:
      var results = `暂不支持处理 ${gitEvent} 事件`;
      break;
  }
  return new Response(results)
}

addEventListener("fetch", event => {
  const { request } = event
  // 仅处理 POST 请求
  if (request.method === "POST") {
    return event.respondWith(handleRequest(request))
  }
  else {
    return event.respondWith(new Response("使用方法请参考文档: https://github.com/huhuhang/github-wechat-bot"))
  }
})
```

## 总结

1. 下一阶段要学习下JS，结合cloudflare worker可以做出一些有意思的工具出来
2. 以后看到有意思的产品，在脑海中有意识锻炼下如何通过不同的能力完成对应服务的搭建
3. 最近工作软件切换成飞书，调整hook地址和sendMdMsg中的消息参数后，通知消息切换到飞书机器人
