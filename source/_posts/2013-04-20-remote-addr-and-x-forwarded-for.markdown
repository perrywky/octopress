---
layout: post
title: "怎样正确设置remote_addr和x_forwarded_for"
date: 2013-04-20 17:09
comments: true
categories: 
  - SA
---
做网站时经常会用到`remote_addr`和`x_forwarded_for`这两个头信息来获取客户端的IP，然而当有反向代理或者CDN的情况下，这两个值就不够准确了，需要调整一些配置。

### 什么是remote_addr

remote_addr代表客户端的IP，但它的值不是由客户端提供的，而是服务端根据客户端的ip指定的，当你的浏览器访问某个网站时，假设中间没有任何代理，那么网站的web服务器（Nginx，Apache等）就会把remote_addr设为你的机器IP，如果你用了某个代理，那么你的浏览器会先访问这个代理，然后再由这个代理转发到网站，这样web服务器就会把remote_addr设为这台代理机器的IP。

### 什么是x_forwarded_for

正如上面所述，当你使用了代理时，web服务器就不知道你的真实IP了，为了避免这个情况，代理服务器通常会增加一个叫做x_forwarded_for的头信息，把连接它的客户端IP（即你的上网机器IP）加到这个头信息里，这样就能保证网站的web服务器能获取到真实IP

### 使用HAProxy做反向代理

通常网站为了支撑更大的访问量，会增加很多web服务器，并在这些服务器前面增加一个反向代理（如HAProxy），它可以把负载均匀的分布到这些机器上。你的浏览器访问的首先是这台反向代理，它再把你的请求转发到后面的web服务器，这就使得web服务器会把remote_addr设为这台反向代理的IP，为了能让你的程序获取到真实的客户端IP，你需要给HAProxy增加以下配置

    option forwardfor

它的作用就像上面说的，增加一个x_forwarded_for的头信息，把你上网机器的ip添加进去

### 使用Nginx的realip模块

当Nginx处在HAProxy后面时，就会把remote_addr设为HAProxy的IP，这个值其实是毫无意义的，你可以通过nginx的[realip](http://wiki.nginx.org/HttpRealipModule)模块，让它使用x_forwarded_for里的值。使用这个模块需要重新编译Nginx，增加`--with-http_realip_module`参数

    set_real_ip_from   10.1.10.0/24;
    real_ip_header     X-Forwarded-For;

上面的配置就是把从10.1.10这一网段过来的请求全部使用X-Forwarded-For里的头信息作为remote_addr

### 将Nginx架在HAProxy前面做HTTPS代理

网站为了安全考虑通常会使用https连接来传输敏感信息，https使用了ssl加密，HAProxy没法直接解析，所以要在HAProxy前面先架台Nginx解密，再转发到HAProxy做负载均衡。这样在Web服务器前面就存在了两个代理，为了能让它获取到真实的客户端IP，需要做以下配置。

首先要在Nginx的代理规则里设定

    proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;

这样会让Nginx的https代理增加x_forwarded_for头信息，保存客户的真实IP。

其次修改HAProxy的配置

    option     forwardfor except 10.1.10.0/24

这个配置和之前设定的差不多，只是多了个内网的IP段，表示如果HAProxy收到的请求是由内网传过来的话（https代理机器），就不会设定x_forwarded_for的值，保证后面的web服务器拿到的就是前面https代理传过来的。

### 为什么PHP里的HTTP_X_FORWARDED_FOR和Nginx的不一样

当你的网站使用了CDN后，用户会先访问CDN，如果CDN没有缓存，则回源站（即你的反向代理）取数据。CDN在回源站时，会先添加x_forwarded_for头信息，保存用户的真实IP，而你的反向代理也会设定这个值，不过它不会覆盖，而是把CDN服务器的IP（即当前remote_addr）添加到x_forwarded_for的后面，这样x_forwarded_for里就会存在两个值。Nginx会使用这些值里的第一个，即客户的真实IP，而PHP则会使用第二个，即CDN的地址。为了能让PHP也使用第一个值，你需要添加以下fastcgi的配置。

    fastcgi_param HTTP_X_FORWARDED_FOR $http_x_forwarded_for;
                            
它会把nginx使用的值（即第一个IP）传给PHP，这样PHP拿到的x_forwarded_for里其实就只有一个值了，也就不会用第二个CDN的IP了。

### 忽略x_forwarded_for

其实，当你使用了Nginx的realip模块后，就已经保证了remote_addr里设定的就是客户端的真实IP，再看下这个配置

    set_real_ip_from   10.1.10.0/24;
    real_ip_header     X-Forwarded-For;

它就是把x_forwarded_for设为remote_addr，而nginx里的x_forwarded_for取的就是其中第一个IP。

使用这些设置就能保证你的remote_addr里设定的一直都是客户端的真实IP，而x_forwarded_for则可以忽略了:)
