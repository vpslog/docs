---
title: "基于 Cloudflare Workers & KV 构建一个评论系统——前端篇"
description: ""
excerpt: ""
date: 2023-06-20T09:08:40Z
lastmod: 2023-06-20T09:08:40Z
draft: false
weight: 50
images: ["frontend.png"]
categories: ["Dev"]
tags: ["Cloudflare","Workers"]
contributors: ["Gray"]
pinned: false
homepage: false
---

本文将继续介绍使用 KV 构建评论系统，主要关注前端开发部分。


## Introduction

对于一个嵌入式的评论系统，开发的原则是风格尽量简单、代码不依赖其他组件、尽量不和原页面冲突、引入尽量简单。理想的方法就是一行 div 加一行 js：

```html
<div class="kv-cmt" endpoint="https://xxx.workers.dev">
</div>
<script src="https://xxx.pages.dev/script.js"></script>
```

然后所有 css 和组件都通过 js 加载。然后获取用 fetch 而非 axios，虽然 Vue 做内容绑定很方便，不过太重了，考虑直接生成一些原生的代码。

## Development

实际开发中也会遇到一些问题，例如老生常谈的 CORS，如何有效地加载评论，以及如何修饰 css。CORS 其实主要是后端处理，需要在 OPTION、GET、POST response 上加上对应 header，加载评论，这里采用了一种比较有意思的方法，即 `append` 每一个 comment 元素时都加上和后端查询出来的 key 相对应 id 值，例如 `kv-1`（此处 `kv-` 前缀主要是为了避免和其他元素冲突），然后对于子元素，例如 key 为 `1|1`，这样就可以直接去掉最后一个数字，然后 `document.getElementById` 把父元素取出来， `append` 即可。最开始是准备用嵌套数组的方式，但是发现这个方法更好。CSS 方面，参考 waline 尽可能用淡一点的颜色、透明度调低，加个简单的缩进就算能用了。毕竟不是做 UI。


JS 代码如下所示，如果需要动态加载可以参考。

```js 
// 加载CSS样式表
function load_css() {
    const cssUrl = './style.css'
    const head = document.head || document.getElementsByTagName('head')[0]
    const style = document.createElement('link')
    style.rel = 'stylesheet'
    style.href = cssUrl
    head.appendChild(style)
}

load_css()

var kv_comment_parent = null

// 获取评论容器和API端点
const container = document.querySelector('.kv-cmt');
const endpoint = container.getAttribute('endpoint')

//添加评论列表容器
const commentsContainer = document.createElement('div');
commentsContainer.classList.add('kv-comments');
commentsContainer.id = 'kv';
container.appendChild(commentsContainer);

//添加新评论容器
const newCommentContainer = document.createElement('div');
newCommentContainer.classList.add('kv-newcomment');
newCommentContainer.innerHTML = `
  <textarea name="kv-comment-value" id="kv-comment-value" placeholder="Comment here" cols="30" rows="10"></textarea>
  <div class="kv-toolbar">
    <span>New Comment / </span><span style="display: none;">New Reply / </span><span>BY </span><input type="text" name="kv-comment-user" id="kv-comment-user" value="匿名">
    <button id="kv-comment-submit" >submit</button>
  </div>
`;
container.appendChild(newCommentContainer);

const replyLabel = '<span class="kv-reply">reply</span>';


//创建表示单个评论的HTML元素
function createCommentElement(comment) {
    const commentElem = document.createElement('div');
    commentElem.id = `kv-${comment.key}`;
    commentElem.classList.add('kv-comment');
    commentElem.innerHTML = `
      <div class="kv-body">
        <div class="kv-stat">BY ${comment.user} / AT ${new Date(comment.time).toLocaleString()} ${replyLabel}</div>
        <div class="kv-content">${comment.content}</div>
        <hr class="kv-hr">
      </div>
    `;
    return commentElem;
}

//获取评论列表
fetch(`${endpoint}/?url=${window.location.href}`)
    .then(response => response.json())
    .then(comments => {
        //遍历JSON数组并为每个评论创建HTML元素
        comments.forEach(comment => {
            const commentElem = createCommentElement(comment);
            if (!comment.key.includes('|')) {
                commentsContainer.appendChild(commentElem);
            } else {
                const id = comment.key;
                const parentId = id.substring(0, id.lastIndexOf('|')); // 获取父级评论的 ID
                const parentComment = document.getElementById(`kv-${parentId}`); // 查找元素
                parentComment.appendChild(commentElem)
            }
        });
    });


//为“回复”标签添加事件监听器
commentsContainer.addEventListener('click', event => {
    if (event.target.classList.contains('kv-reply')) {
        const commentElem = event.target.closest('.kv-comment');
        kv_comment_parent = commentElem.id.split('-')[1]
        const toolbar = newCommentContainer.querySelector('.kv-toolbar');
        toolbar.children[0].style.display = 'none';
        toolbar.children[1].style.display = '';
    }
});

//为提交增加监听器
const submitBtn = document.getElementById('kv-comment-submit');
submitBtn.addEventListener('click', async () => {
    const commentValue = document.getElementById('kv-comment-value').value;
    const commentUser = document.getElementById('kv-comment-user').value;

    // 构造请求体
    const requestBody = {
        url: window.location.href,
        content: commentValue,
        user: commentUser,
        parent: kv_comment_parent, // 假设已知父级评论的 ID
    };

    try {
        const response = await fetch(`${endpoint}`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(requestBody),
        });

        if (response.ok) {
            const responseBody = await response.json();
            alert(responseBody.message);
        } else {
            console.error(`Failed to post comment. Status code: ${response.status}`);
        }
    } catch (error) {
        console.error(`Failed to post comment. Error: ${error}`);
    }
});
```


## Conclusion

其实这一套我一直想加一点人机验证，防止刷评论、刷接口访问。用 CF 的托管质询，似乎不好处理 POST；用 [Turnsite](https://developers.cloudflare.com/turnstile/)，似乎没地方加组件。不过类似这种数据应该被攻击了问题也不大。另外就是这个 KV 的更新速率真的是，，一言难尽。基本上是不能做到即时反馈。不清楚 CF 自己是怎么做数据库更新的。