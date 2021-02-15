---
title: 使用chromeDevTools Audits 优化web应用实践总结
date: 2021-02-14 21:17:02
tags:
categories: 技术
---

提高首屏渲染速度

1. Tree shake -> remove unused code

2. Code split(webpack `import` / `require.ensure`) -> load critical code

3. Script 使用`async`/`defer`防止脚本加载会阻塞 dom 渲染（async/defer 异同点，两者都是异步加载，async 是乱序的加载好就马上执行，defer 加载好之后在 html 解析之后按顺序执行，具体见下图）
   例子：`jquery.js`可以使用 async 加载，因为它不依赖其他 js，而`$(“#id1”).hide()`这种依赖 jquery 的 js 应该使用 defer 在 html parse 完成之后加载执行。
   ![484D2E9E-00C9-4C03-B179-3DBB6C5F8A60](https://user-images.githubusercontent.com/13430709/62349670-01394200-b533-11e9-9899-b3f9ac5b5c3a.png)
   ![CE2D2F28-F6C5-4520-AD5A-C03FEFD3CE4C](https://user-images.githubusercontent.com/13430709/62349681-07c7b980-b533-11e9-96d1-b748e7c0edec.png)
   `preload`和`prefetch`这两个是预加载资源到 Http 缓存中，相当于上图里 script fetch 这一步，仔细别搞混淆。

> 我们可以明确 DOMContentLoaded 所计算的时间，当文档中没有脚本时，浏览器解析完文档便能触发 DOMContentLoaded 事件；如果文档中包含脚本，则脚本会阻塞文档的解析，而脚本需要等位于脚本前面的 css 加载完才能执行。在任何情况下，DOMContentLoaded 的触发不需要等待图片等其他资源加载完成。

4.  Optimize encoding and transfer size of text-based assets: enable text compression，nginx 静态资源开启 gzip，服务端接口返回数据使用`gzip`或`brotli`(Brotli 压缩比优于 gzip)等压缩 response 正文。
    可通过 Network 里查看请求响应头里的`content-encoding`有没有`gzip/deflate/br`字段，和压缩前后 size 大小比较：
    ![525E22B0-A478-43D0-92C1-0956B7BAB780](https://user-images.githubusercontent.com/13430709/62349882-93414a80-b533-11e9-8c0e-17ece1bec786.png)

        ![BB43FCD4-CC5A-4903-8580-B01E6C8990F7](https://user-images.githubusercontent.com/13430709/62349829-7a389980-b533-11e9-8c43-ad8acd8682ac.png)

5.Defer critical css ,页面只适合被关键 css（这种关键 css 适合 inline 到 html 里面，减少资源请求的时间）加载阻塞，其他与当前页面展示无关的 css 适合使用 defer 放在。
在 Coverage 里可以看到哪些是 critical 资源：
![5DC8DFFC-D660-4A8A-838F-FDBCE2D5DBAD](https://user-images.githubusercontent.com/13430709/62349945-c7b50680-b533-11e9-9aa2-0e96d378b8ac.png)

6. 尽量保持主线程空闲，不要长时间阻塞，能及时处理用户输入。

7. 使用服务线程缓存资源，进行计算等，a.防止主线程长时间占用，b.以 httpCache 能力为基准作渐进式增强资源预缓存能力，理想状态无 http 请求往返。
   使用`workbox-webpack-plugin` -> use service work to generate thread to load resourses cached. （发布到线上要用 https），具体代码如下：
   ![image](https://user-images.githubusercontent.com/13430709/62350083-33976f00-b534-11e9-854c-4c6d002afeb1.png)
   ![image](https://user-images.githubusercontent.com/13430709/62350090-3d20d700-b534-11e9-8716-c6de5a1b5d61.png)

8. 减少 redirect 次数，降低多次的 http 往返开销

9. 用户不可见图片不用提前加载；使用适当图片格式，svg 格式是一种挺好的尝试，可以任意 resize 而且 svgcode 作为 text 可以 compress 和 minify；图片使用工具适当压缩；使用多个大小、格式版本的图片资源适配不同场景（<picture> srcset mediaQuery）

10. 图片格式还可以使用最新的 webp，jpeg 2000，jpeg XR，就当作是渐进式增强咯。

11. TTI -> time to interactive 减少页面加载到可以交互处理用户输入事件的时间长度，防止阻塞页面渲染主要还是防止首屏加载非必要资源，预加载和最小化（压缩混淆）必要资源，预缓存 assets，懒加载其他（未来的）navigation。

12. 运用 Http Cache 的能力。
    ![image](https://user-images.githubusercontent.com/13430709/62350152-6e010c00-b534-11e9-9498-f5ad8fa2d212.png)
