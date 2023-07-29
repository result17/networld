---
author: cosin
pubDatetime: 2023-07-26T20:23:39+08:00 
title: browser 浏览器面试题
featured: true
draft: false
tags:
  - color-schemes
  - docs
description:
  面试题记录
---

## sessionStorage 与 localStorage有何区别
sessionStorage 存储的数据在浏览器窗口关闭时就会消失，而 localStorage 存储的数据会在浏览器关闭后仍然存在。

sessionStorage 通常用于存储在会话期间需要访问的数据，例如用户名和密码。localStorage 通常用于存储在不同会话之间需要访问的数据，例如书签和偏好设置。

两者都只能保存字符串。

- sessionStorage 的最大容量为 5 MB，而 localStorage 的最大容量为 10 MB。
- sessionStorage 的数据在不同浏览器窗口之间是隔离的，而 localStorage 的数据在所有浏览器窗口之间是共享的。
- sessionStorage 的数据在浏览器禁用 cookies 时仍然可以访问，而 localStorage 的数据在浏览器禁用 cookies 时无法访问。

## cookie 有哪些字段
Cookie 有以下属性

Domain
Path
Expire/MaxAge
HttpOnly: 是否允许被 JavaScript 操作
Secure: 只能在 HTTPS 连接中配置
SameSite

cookie 如果没有设置maxAge，过期时间为会话时间。

## Cookie SameSite
None: 任何情况下都会向第三方网站请求发送 Cookie
Lax: 只有导航到第三方网站的 Get 链接会发送 Cookie，跨域的图片、iframe、form表单都不会发送 Cookie
Strict: 任何情况下都不会向第三方网站请求发送Cookie

## 浏览器中监听事件函数 addEventListener 第三个参数有那些值
- capture。监听器会在时间捕获阶段传播到 event.target 时触发。
- passive。监听器不会调用 preventDefault()。
- once。监听器只会执行一次，执行后移除。
- singal。调用 abort()移除监听器。

## 什么是事件委托
事件委托指当有大量子元素触发事件时，将事件监听器绑定在父元素进行监听，此时数百个事件监听器变为了一个监听器，提升了网页性能。

另外，React17 把所有事件委托在 Root Element，用以提升性能。
v16：出于对冒泡的了解，我们直接在按钮事件上加e.stopPropagation()，这样就不会冒泡到document，isShowText 也不会被置为false了。但由于v16版本的事件委托是绑在document上的，它的事件源跟document就是同级了，而不是上下级，所以e.stopPropagation()并没有起作用。如果要阻止冒泡，可以使用原生的
e.nativeEvent.stopImmediatePropagation()阻止同级冒泡，这样文字就可以显示了。

v17：由于事件委托到根目录root节点，与document属于上下级关系，所以可以直接使用e.stopPropagation()阻止

## 如何取消请求的发送
- XHR 使用 xhr.abort()
- fetch 使用 AbortController

## requestIdleCallback
requestIdleCallback 维护一个队列，将在浏览器空闲时间内执行。它属于 Background Tasks API，你可以使用 setTimeout 来模拟实现

## JSONP 的原理是什么，如何实现
JSONP，全称 JSON with Padding，为了解决跨域的问题而出现。虽然它只能处理 GET请求。

## 异步加载 JS 脚本时，async 与 defer 有何区别
JS 的脚本分为加载、解析、执行几个步骤，简单对应到图中就是 fetch (加载) 和 execution (解析并执行)
JS 的脚本加载(fetch)且执行(execution)会阻塞 DOM 的渲染，因此 JS 一般放到最后头
而 defer 与 async 的区别如下:

相同点: 异步加载 (fetch)
不同点:
async 加载(fetch)完成后立即执行 (execution)，因此可能会阻塞 DOM 解析；
defer 加载(fetch)完成后延迟到 DOM 解析完成后才会执行(execution)**，但会在事件 DomContentLoaded 之前

## React/Vue 中的 router 实现原理如何
history API

通过 history.pushState() 跳转路由
通过 popstate event 监听路由变化，但无法监听到 history.pushState() 时的路由变化
hash

通过 location.hash 跳转路由
通过 hashchange event 监听路由变化