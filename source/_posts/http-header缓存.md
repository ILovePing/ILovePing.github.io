---
title: http-header缓存
date: 2019-03-25 18:24:57
tags:
categories:
---
# Header缓存

## 强缓存

Q：强缓存没有过失效时间的情况下，如何重新从服务端加载资源？
A：更新页面中的资源地址，比如static.js?_v=1f6d7f

### Expires（http1.0规范）

绝对时间的GMT格式的时间字符串，如Mon, 10 Jun 2015 21:31:12 GMT

### cache-control（http1.1规范）

优先级高于Expires

- max-age=<number>

  相对时间，同第一次请求的时间相加计算出过期时间同当前时间比较

- no-cache

  不使用本地强缓存

- no-store

  禁止使用浏览器缓存，每次请求资源都需要从服务端取得。

- public

  可以被所有用户缓存，包括终端用户和cdn等中间代理服务器。

- private

  只能被终端用户缓存，cdn等中间服务器不能缓存。

## 协商缓存

值都是成对出现的，例如：第一次服务端在Header里返回Last-modified字段，那么下次请求发出的时候会带上if-modified-since字段，值就是首次响应头的last-modified值。

### if-modified-since:<Last-modified>

浏览器发起请求，请求头加上If-Modified-Since：<Last-modified>，服务端接受到之后会拿该值同资源在服务器上的最后修改时间作对比，如果没有变化，就返回304，不返回资源，告诉浏览器使用本地的缓存即可；同时因为资源并没有变化，那么响应头并不会返回last-modified，因为同第一次的last-modified相同。如果资源变化了，那么就返回200，资源重新下载，同时返回最新的last-modified。

### if-none-match: <Etag>

值为资源的一个唯一标识字符串，具体通信策略上同。
唯一不同的是304的时候Etag也会返回，即使值相同。
同时存在的情况下Etag优先。
Q：有last-modified了，为什么还要有etag？
A：
有些资源变更频率是毫秒级更甚，last-modified颗粒度就到秒为止。
有些资源周期性变更，修改时间变了，但是可能文件内容没变，这时候etag相同，返回304，使用本地缓存即可。
有些服务器不能精确拿到资源修改时间。
