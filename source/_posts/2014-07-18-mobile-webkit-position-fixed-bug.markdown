---
layout: post
title: "一个移动端webkit浏览器的bug"
date: 2014-07-18 14:27
comments: true
categories: 
  - web
---
上篇吐槽了IE，这篇说下移动端webkit内核浏览器的一个bug，当你滚动页面时，如果有元素的position属性从fixed变成relative，并修改了top后，这个元素会消失，等滚动结束后，又会出来。这是一个奇葩的bug，需要用奇葩的方式来解决，那就是给这个元素加上以下css样式

``` css
  -webkit-transform: translate3d(0, 0, 0);
```

是不是有点想掀桌？
