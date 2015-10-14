---
layout: post
title: "强制IE重绘"
date: 2014-07-18 14:14
comments: true
categories: 
  - web
---
这两天做chufaba.me的页面时，碰到一个很奇怪的bug，在IE浏览器下修改position为fixed时，页面会错乱，花了我两天时间都没搞定，感觉整个人都不好了。

后来在css3-mediaqueries.js里看到一个强制ie重绘的代码，灵机一动拿过来试试，果然有效，真想跪舔下作者！

``` javascript
  document.documentElement.style.display = 'block';
  setTimeout(function () {
    document.documentElement.style.display = '';
  }, 0);
```

问题虽然解决了，但我还是希望IE毁灭，如果我有超能力的话。
