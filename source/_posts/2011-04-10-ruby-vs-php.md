--- 
categories: 
  - programming language
  - php
  - ruby
comments: true
layout: post
title: Ruby与PHP的比较
---
最近在学Ruby，觉得这门语言有许多优美的特性，相对于PHP来说，代码更简洁，可读性更强，下面做些简单总结。
###Ruby里所有的变量都是对象
整数、浮点数、字符串还有数组等基本变量类型都是对象
``` ruby ruby
1.object_id    #3
1.5.to_s    #"1.5"
"abc".is_a?(Object)    #true
```
这样设计的一个好处就是，所有类型相关的函数都变成了对象的方法，使得代码更加直观易懂，而不是像PHP那样有成百上千个全局函数，各自的命名规则还不一致：
``` ruby ruby
"a b c".split    #["a", "b", "c"]
"abc".length   #3
["a", "b", "c"].length
```
``` php php
explode(" ", "a b c");
strlen('abc');
count(array('a', 'b', 'c'));
```
###Ruby不需要用分号来结束每行代码
PHP以及很多其它的语言使用分号的目的其实是为了区分表达式。然而，为了代码的可读性，我想没有人会把几行代码写成一行，所以还使用分号来结束每一行代码就显得多余了。
###同样的功能，Ruby让你写的代码更少
简单优美，这是Ruby的设计宗旨，首先体现在方法的命名和操作符上面：
``` ruby ruby
array = [1, 2, 3, 4, 5]
last = array.last    #5
subarr = array[1,3]    #[2,3,4]
```
``` php php
$array = array(1, 2, 3, 4, 5);
$last = $array[count($array)-1]
$subarr = array_slice($array, 1, 3);
```
也体现在变量的声明和互换上面：
``` ruby ruby
a, b = 1, 2    #a=1, b=2
a, b = b, a    #a=2, b=1
```
``` php php
$a = 1;
$b = 2;
$tmp = $a;
$a = $b;
$b = $a;
unset($tmp);
```
还体现在一些循环操作上面：
``` ruby ruby
array = [1, 2, 3]
map = array.map { |item| item + 10 }    #[11, 12, 13]
```
``` php php
$array = array(1, 2, 3);
function plusTen($v){
    return $v + 10;
}
$map = array_map('plusTen', $array);
```
以及一些常用的文件操作：
``` ruby ruby
File.open('example.txt') do |file|
    capital_lines = file.map { |line| line.strip.upcase }
end
```
``` php php
$handle = fopen("example.txt", "r");
$capital_lines = array();
while (($line = fgets($handle)) !== false) {
    $capital_lines[] = strtoupper($line);
}
fclose($handle);
```
