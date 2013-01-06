--- 
layout: post
title: 使用Node.js和Redis实现push服务
comments: true
categories: 
  - web
---
push服务是一项很有用处的技术，它能改善交互，提升用户体验。要实现这项服务通常有两种途径，轮询和长连接。轮询就是客户端每隔一段时间就问服务器拿新数据，实现起来很简单但是服务器压力很大，而且大部分请求因为没有新数据都显得很浪费。长连接则是服务器将一个请求挂起，不输出任何内容，直到有新数据产生后才会完成这个请求，浏览器收到响应后则马上再发一个又让服务器挂住，如此反复。这么做的好处是能节省很多无用的请求，但是它不能使用传统的服务端软件，比如apache和php-fpm，客户端多了的话很容易把所有进程占光，这样服务器就没法响应新的请求了。

Node.js让这一切变得简单，它是基于事件和非阻塞I/O的服务器技术，能使用极少的资源响应大量并发的请求，非常适合长连接的要求。但是这样做还存在两个问题。首先你的服务端通常是用另外一套语言和框架做的，有成熟的代码和业务逻辑，为了实现这个push功能，难道又要用javascript来写一套吗？维护起来不嫌麻烦？其次，服务端把请求挂起后，也是不断地重复调用其它服务来获取新数据，这不过是把轮询的代码换个位置而已，本质上没区别，对服务器一样有压力。

有没有什么简单的办法来实现高效的push呢？

答案是有，而且很简单，所需代码不超过20行！

首先我们借助Redis的[Pub/Sub](http://redis.io/topics/pubsub)功能来实现真正的push，其次用JSON来作为客户端和服务端沟通的数据格式。

当Node.js收到请求后，我们将请求挂起，同时实例化一个Redis[客户端](https://github.com/simplegeo/nodejs-redis)，并根据请求里的参数来收听一个特定的频道，原有的服务端代码（比如PHP）处理完业务逻辑后，将新数据用JSON封装下发布到这个频道，Node.js收到消息后将其作为响应传给客户端，这样就完成了一次push。代码如下

``` javascript
var http = require('http');
var url = require('url');
var redis = require('redis');
http.createServer(function (req, res) {
    var query = url.parse(req.url, true).query;
    if(typeof query.channel == 'undefined'){
        res.writeHead(200, {'Content-Type': 'text/plain'});
        res.end('Invalid Request\n');
    }else{
        var client = redis.createClient(6379, '127.0.0.1');
        client.subscribe(query.channel);
        client.on('message', function(channel, message){
            res.writeHead(200, {'Content-Type': 'application/json'});
            res.end(message);
            client.unsubscribe();
            client.end();
        });
    }
}).listen(1337, '127.0.0.1');
```

这个方法简单易用，你只需要定好一个频道和数据关系的协议，然后对现有代码做些简单修改，就能实现一个高效的push服务了！
