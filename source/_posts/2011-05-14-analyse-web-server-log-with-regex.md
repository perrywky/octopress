--- 
categories: 
  - web
  - regex
  - log
comments: true
layout: post
title: 使用grep和正则来分析Web服务器日志
---
前两天因为第三方游戏服务器掉线，导致大量用户同时登录我的服务器，将服务器负载瞬间提高到200+，如此恐怖的数字让我不得不考虑增加服务器来抵抗问题重现，然而我的服务器平时负载都很低，0.1都不到，增加服务器来应付这样短暂的风暴未免太过于浪费，于是我决定从日志下手，找到我的网站的瓶颈，希望能通过改善程序来解决这个问题。
###第一步，定位时间
我的日志文件里包含了最近一个礼拜的数据，然而我需要的只是风暴发生时产生的数据，总共不超过20分钟，怎么取呢？因为我的Web服务器采用的是标准的combined格式
```
$remote_addr - $remote_user [$time_local]  "$request" 
$status $body_bytes_sent "$http_referer" "$http_user_agent"
```
而且全部是PHP动态请求，所以我决定从time_local下手，找到并发访问量最高的时间段，这很容易办到：

<!--more-->

```
grep -oP '12\/May\/2011(:\d{2}){3}' access.log | uniq -c | sort -n > time.sort
```
得到如下结果（部分）

{% img /images/uploads/Screenshot-1.png 224 240 time.sort %}

从18点43分开始，我的服务器每秒需要响应120多次动态请求，而18:46:49秒更加变态，211次！看来把我服务器拖垮的，就是18点43分到18点50分这一段。
###第二步， 过滤脚本
定位好了时间，现在要做的，就是取出这一时间段的日志再做分析，使用以下脚本将18:43分到18:50分之间的日志取出来
```
grep -P '12\/May\/2011:18:4[3-9]:\d{2}' access.log > storm.log
```
###第三步， 找出元凶
得到了storm.log，下面我便要找出拖垮我服务器的元凶，及访问数量最高的$request。因为$request都是 "GET /\*\*\* HTTP/1.1" 或 "POST /\*\*\*" 这样的格式， 所以可以通过简单的正则取出$request，如下
```
grep -oP '((?<=GET\s)|(?<=POST\s))[^?\s]+' storm.log | sort \
 | uniq -c | sort -n > request.sort
```
(?<=GET\s)这个正则叫作[零宽断言](http://deerchao.net/tutorials/regex/regex.htm#lookaround)，是将要被匹配文字***前面***的条件，即除非前面有'GET '出现，后面的才匹配。
上面的脚本得到以下结果：

{% img /images/uploads/Screenshot-2.png 333 194 request.sort %}

从这张图我们便可以找出程序方面的瓶颈了，因为上面这些请求大部分都是ajax请求，所以明显的，像'users/getuser'、'games/getservers/sxd'这样的数据请求完全可以被浏览器缓存起来，而'users/logoutService'根据我们的业务逻辑也显得毫无必要，将这三项请求砍掉能节省将近60%的资源！
以下便是通过优化程序代码后13号应对的又一次风暴结果。

{% img /images/uploads/Screenshot-3.png 407 224 %}

看到没，'/users/getuser'请求减少了将近一半，而'/games/getserver/sxd'则减少了近75%，总量减少了近40%！然而'/users/logoutService'却不尽如人意，只少了三成，我们待会再寻找其原因。

通过以上程序的优化和一些系统配置的调整，这次风暴只将我的服务器负载升高到了10+，并在十几秒后就很快地平稳了下来，和前一天的200+相比，可以说成功地解决了短暂风暴的问题。
###第四步，找出来源
上一步遗留了一个问题，即我明明优化了程序，去掉了不必要的'users/logoutService'，为何在风暴中，它依然出现了那么多次，所以我决定分析$http_referer，找出这些请求都是从哪来的。
根据日志的格式，$http_referer前面都有一个$body_bytes_sent、一个空格和一个双引号，例如
```
27.37.113.145 - - [14/May/2011:22:30:47 +0800] "GET /users/logoutService HTTP/1.1"
 200 431 "http://sxd.xd.com/"
 "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)"
```
因为$body_bytes_sent始终是个数字，这样就可以通过以下正则来找出来源
```
grep -P '\/users\/logoutService?' storm.log \
 | grep -oP '(?<=\d\s")[^?"]+' | sort | uniq -c | sort -n > logout.referer.sort
```
先过滤出logout的日志，再通过零宽断言找出来源。得到的结果出乎我意料，来自于一个我在上一步中已经调整过的页面，明明去掉了不必要的'users/logoutService'请求，为何还会重复出现？仔细观察代码后并没发现可疑的地方，于是推测是CDN缓存的问题，这些用户用的JS版本可能还是前一天的，清理缓存，加上版本号，期待下次风暴来验证这一推论！
