---
title: html5新标签兼容IE6/7/8
date: 2016-11-29 13:38:01
categories: 技术
tags: [前端,兼容]
---
html5的新标签header footer nav aside section article 能让页面代码语义化更加直观，同时还有利于SEO优化。所以如何让低版本的IE浏览器兼容新标签就成了我们要解决的问题了。
### solution 1: 通过js自定义标签
```javascript
<!--[if lt IE9]>
<script>
   (function() {
     if (!
     /*@cc_on!@*/
     0) return;
     var e = "abbr, article, aside, audio, canvas, datalist, details, dialog, eventsource, figure, footer, header, hgroup, mark, menu, meter, nav, output, progress, section, time, video".split(', ');
     var i= e.length;
     while (i--){
         document.createElement(e[i])
     }
})()
</script>
<![endif]-->
```

### solution2:使用Google的html5shiv包（推荐）
```
<!--[if lt IE9]>
<script src="http://html5shiv.googlecode.com/svn/trunk/html5.js"></script>
<![endif]-->
```
不管使用以上哪种方法,都要初始化新标签的CSS.因为HTML5在默认情况下表现为内联元素，对这些元素进行布局我们需要利用CSS手工把它们转为块状元素方便布局
/*html5*/
article,aside,dialog,footer,header,section,footer,nav,figure,menu{display:block}

ie6/7/8 禁用脚本的用户 建议参考facebook的做法：
```
<!--[if lte IE 8]> 
<noscript>
     <style>.html5-wrappers{display:none!important;}</style>
     <div class="ie-noscript-warning">您的浏览器禁用了脚本，请<a href="">查看这里</a>来启用脚本!或者<a href="/?noscript=1">继续访问</a>.
     </div>
</noscript>
<![endif]-->
```