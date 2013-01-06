--- 
categories: 
  - web
comments: true
layout: post
title: Apache的rewrite要点
---
Apache的rewrite模块功能强大，但是想要把它用顺需要注意几点，不然你会很痛苦！
###RewriteCond在RewriteRule条件之后判断
apache的rewrite规则是，先判断是否匹配RewriteRule的条件，成功后再判断是否匹配RewriteCond。假设我们想要把不存在的页面redirect到另外一个地址，我们可以用这样的规则：
```
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ http://www.example2.com/$1 [R,L]
```
这里首先用^(.*)$这一正则表达式来匹配所有的url，然后再判断这个url指向的文件是否存在。
###DocumentRoot设定后‘/’永远存在
继续上面的例子，假设我的一个域名没有首页，我希望把针对这个域名的访问也转发到另外一个地址，比如www.example.com跳转到www.example2.com，那么上面的规则就不够了，因为‘/’目录是存在的，无法通过条件
```
RewriteCond %{REQUEST_FILENAME} !-d
```
我们需要在后面再加一条规则
```
RewriteRule ^$ http://www.example2.com [R,L]
```
这样我们就能成功地把www.example.com下的首页和不存在的页面都转发到www.example2.com下了。
###.htaccess里的规则不能直接放到httpd.conf或VirtualHost里
.htaccess的规则只是针对其文件所在目录下而言的，其效果和放在Directory里一样，然而当把这些规则挪到httpd.conf或VirtualHost里时，就需要做出调整了。继续上面的例子，假设我们把DocumentRoot设为"/var/www"，那么上面介绍的两个规则放在/var/www/.htaccess文件里，或者放在httpd.conf或VirtualHost的\<Directory /var/www\>中，效果一样。然而这两个规则其实都是Server级的，我们完全可以把它放到VirtualHost下，而不是\<Directory /var/www\>中，新的规则如下
```
RewriteCond %{LA-U:REQUEST_FILENAME} !-f
RewriteCond %{LA-U:REQUEST_FILENAME} !-d
RewriteRule ^/(.*)$ http://www.example2.com/$1 [R,L]

RewriteRule ^/$ http://local.sgj [R,L]
```
这里有两点区别需要注意，首先REQUEST_FILENAME前加了‘LA-U:’，这是因为当处理VirtualHost下的Rewrite规则时，apache还处在一个url对应文件的翻译阶段（URL-to-filename translation），apache这个时候还不知道该访问哪个文件或目录，自然也就无从得知这个文件或目录是否存在了，这里的‘LA-U:’就是让apache先去获取这个变量，知道自己该访问哪个文件或目录，然后才能判断其是否存在。
第二点是两个正则表达式里都多了一个‘/’，这是因为VirtualHost和\<Directory /var/www\>不同，它提供给RewriteRule判断的是完整的url，及www.example.com/index.html里的‘/index.html’，而\<Directory /var/www\>下提供的是目录‘/’之后的内容，及‘index.html’。以此类推，www.example.com/abc/index.html，在VirtualHost下判断的是‘/abc/index.html’，在\<Directory /var/www\>下则是‘abc/index.html’，而在\<Directory /var/www/abc\>下则是‘index.html’了。
