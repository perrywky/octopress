---
layout: post
title: "整理Cacti模板"
date: 2013-04-19 16:30
comments: true
categories: 
    - SA
    - Cacti
---

这两天装了些Cacti的模板，整理下留作备忘

### Mysql

- <http://www.percona.com/doc/percona-monitoring-plugins/cacti/mysql-templates.html>
- 需要创建一个mysql用户，分配SUPER, PROCESS权限
- 它还提供很多其他服务（nginx, memcached, redis）的模板，但都是通过ssh登录后取的，不适用于我的环境

<!--more-->

### PHP-FPM

- <http://slaytanic.blog.51cto.com/2057708/900585>
- 用于统计动态请求的变化

### Nginx

- <http://forums.cacti.net/about26458.html>
- 使用楼主提供的模板，数据收集是一个perl的脚本，需要安装LWP::UserAgent模块，`cpan LWP::UserAgent`， 如果你之前没有用过cpan可能会提示你创建一些目录，一路回车好了

### HAProxy

- 文档<http://haproxy.1wt.eu/download/contrib/netsnmp-perl/README>
- 下载<http://haproxy.1wt.eu/download/contrib/netsnmp-perl/>
- haproxy需要打开stat socket，在global下添加`stats socket /var/run/haproxy.stat mode 666`
- snmp需要把perl模块编译进去，HAProxy和Cacti所在的机器都要重编和配置
- 上面只是用来收集数据，你还需要画图表的模板<http://www.formilux.org/archives/haproxy/0807/1225.html>
- frontend里的session数量取的是总数，然后再根据时间间隔算出每秒的新增数

### Memcached

- <http://dealnews.com/developers/cacti/memcached.html>
- 需要先装一个python的模块，文档里有说明
