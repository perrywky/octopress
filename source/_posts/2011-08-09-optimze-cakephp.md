--- 
categories: 
  - web
  - cakephp
comments: true
layout: post
title: cakephp优化总结
---
对于网站来说，响应速度是一件非常重要的事情，虽然打开一个页面的80%的时间是耗费在网络传输上，服务器本身的响应速度也不容忽视，当服务器负载上升后，响应速度直接影响到了服务器的吞吐率。假设一个PHP脚本的平均执行时间是100毫秒，那么在只有一个CPU的服务器上，每秒钟最多也就能响应10次请求，如果代码写的不好，执行时间需要500毫秒，那么服务器每秒钟则只能响应2个请求。

减少代码的执行时间，关键在于减少重复的操作，尽可能的利用缓存。下面我便介绍一下我在使用cakephp这个开源框架中总结的优化经验。

<!--more-->

###将索引文件缓存到memcached
当开启缓存后，cakephp会建立一个文件和目录的索引，大幅降低加载Component和Model的时间。这些索引通常是保存在app/tmp/cache/persistent目录下的，但是当你使用memcached来作为默认缓存引擎时，这些文件则无法缓存，索引则不能保存下来，这丝毫不能降低加载代码的时间。解决的办法其实很简单，在core.php下，定义一个名为'\_cake_core\_'的缓存配置，如下
``` php
Cache::config('_cake_core_', array(
    'engine' => 'Memcache',
    'duration' => '+30 day',
    'servers' => array('127.0.0.1:11211'),
    'compress' => false
));
```
这样，这些文件就能成功被缓存。
###将数据库表结构缓存
原理同上面的文件索引一样，cakephp可以把数据库的表结构缓存起来，免去重复读取和处理表结构的操作，当开启memcached的时候，这些缓存也不能直接缓存，需要定义以下规则
``` php
Cache::config('_cake_model_', array(
    'engine' => 'Memcache',
    'duration' => '+30 day',
    'servers' => array('127.0.0.1:11211'),
    'compress' => false
));
```
###将Controller需要加载的Model序列化
在MVC框架中，Controller的定位是处理请求并返回操作结果，具体的业务逻辑则交给Model来做，这样Controller在处理请求的过程中，通常会加载很多Model，与其每次都重复引入文件，解析类，再生成对象，不如将这些常用的Model序列化成字符串保存起来，需要时再反序列化成对象，可以直接使用，节省大量的加载时间。这样做有一个前提，那就是在反序列化的过程中，类的构造函数__construct不会被执行，所以写代码时要注意不要在构造函数里保存状态。实现也很简单，在Controller里定义一个公共属性
``` php
public $persistModel = true;
```
我会在AppController里定义这个变量，这样所有的Controller都默认继承了这一优化。
###代码更新时要即使清空缓存
使用以上这三个缓存技巧时，要注意每当有代码更新时，比如新加了一个Component，或者Model的代码有更新，或者数据库表的结构有变化时，要及时清理相应的缓存，不然会产生找不到文件或代码未更新的bug，具体做法是删除以下缓存
``` php
$Cache = Cache::getInstance();
//清理文件索引
$Cache->delete('dir_map', '_cake_core_');
$Cache->delete('file_map', '_cake_core_');
$Cache->delete('object_map', '_cake_core_');

//清理表结构
App::import('Core', 'ConnectionManager');
$dataSource = ConnectionManager::getDataSource('default');
if($dataSource){
    $db_name = $dataSource->config['database'];
    $table_list_file = 'default_'.$db_name.'_list';
    $table_files = $Cache->read($table_list_file, '_cake_model_');
    if($table_files){
        foreach($table_files as $table_file){
            $Cache->delete('default_'.$table_file, '_cake_model_');
        }
        $Cache->delete($table_list_file, '_cake_model_');
    }
}

//清理序列化Model
$persist_files = CACHE.'persistent'.DS.'*';
exec("rm -rf $persist_files");
```
###省去Model间的关联
上面解决的是减少加载时间的问题，下面介绍的是怎样省去不必要的加载。
当我们使用cakephp脚手架生成User和Group这两个Model时，会在User类加上这样一行代码
``` php
var $belongsTo = array('Group');
```
它的作用是在加载User这个Model时同时加载Group，这样能使得你能很方便的通过外键来访问其他Model，如果你的应用并不需要这一功能，那么你就可以将这行代码删除，换取更快的加载时间。
###不使用Controller的默认加载
Controller有两个属性，$uses和$components，可以让你的Controller自动加载Model和Component，然而相较于Controller的loadModel方法，$uses的开销更大，而且当你的Action不需要默认的Model时，自动加载的Model也显得多余，成为负担，解决的办法是在Controller里定义$uses为空的数组
``` php
public $uses = array();
```
components原理同样，虽然Cookie，Session，Auth，Acl这几个Component是属于比较常用的，但是也不见得所有的Controller和Action都会使用到，所以明智的做法是在Controller的constructClasses()方法里再根据action选择性的引用，如
``` php
function constructClasses(){
    if(in_array($this->action, array('index'))){
        $this->components[] = 'Session';
    }
    parent::constructClasses();
}
```
###总结
总结以上的优化，关键就在于两点，缓存和减肥，将重复性的操作缓存起来，提高效率，将不必要的代码去掉，减少负担，通过以上的优化，我的服务器的平均响应时间已从350毫秒降到了50ms左右，减少了6倍！
