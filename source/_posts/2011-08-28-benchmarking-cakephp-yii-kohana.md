--- 
categories: 
  - web
  - cakephp
  - php
comments: true
layout: post
title: cakephp、yii、kohana性能比较
---
周末闲来无事，测了下三个php框架的性能，图表如下
{% img /images//uploads/php-framework-bench.png 600 371 php-framework-bench %}

<!--more-->

| framework | reqs | reqs with apc |
| :--------- | :---- | :------------- |
| cake 1.3.11 | 12569.8 | 32786
| yii 1.1.8 | 24382.6 | 115272
| kohana 3.2.0 | 23275.2 | 32784.4

我的测试方法是ab -t 60 -c 10 url，测试页面是输出一个hello world，没有任何附加的组件，纯粹比较框架的自身开销。
当不开启apc时，cake的性能较差，60s内平均完成12569.8次请求，只有yii和kohana的一半，开启apc后，cake的性能和kohana一样了，提升较kohana明显，但是yii却达到了恐怖的115272次，将近5倍，我很好奇它是怎么实现的。
