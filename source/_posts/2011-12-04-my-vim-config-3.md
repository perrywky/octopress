--- 
categories: 
  - vim
comments: true
layout: post
title: 我的Vim配置（三）
---
继续之前的Vim配置系列，这次我将介绍两个插件和PHP相关的配置
###使用taglist来实现IDE中的outline功能
以前使用IDE时，有个很贴心的功能我很喜欢，就是把当前文件的类函数全部列出来，这样当你的函数很多时，能很方便的找到自己想要的，而不需要在代码里翻来翻去。[taglist](http://www.vim.org/scripts/script.php?script_id=273)就是这样的一个插件，它使用ctags动态生成当前文件的tag并在一个新窗口里显示出来，你只需点击这个窗口里的函数名，就能快速定位到某个函数的开始位置，非常好用。安装也很简单，跟以前一样将这个插件放到bundle目录下，就能使用了。因为我主要使用MacVim，所以我添加了以下配置信息到.gimvrc里，使用起来更加方便。

<!--more-->

``` vim
autocmd VimEnter * TlistToggle "启动时强制打开taglist窗口
let Tlist_Show_One_File=1 "一次只显示一个文件的tag，默认会显示所有打开过的
let Tlist_Use_Right_Window=1 "将taglist窗口放到右边，因为左边放了NERDTree，所以这条很有用！
let Tlist_Compact_Format=1 "紧凑显示，在有限的窗口里可以多显示几行内容
```
###使用snipMate减少你的敲代码字数
对于程序员来说，[snipMate](http://www.vim.org/scripts/script.php?script_id=2540)可是不可或缺的利器，它可以帮你自动完成一些代码片段，省去你不少敲击代码的时间，尤其是当你手动编辑html标签时，会觉得如释重负！比如当你输入div再按下tab键，一个完整的div标签就输入完整了，不需要一堆尖括号反斜杠等等重复烦人的输入，而且光标还会自动停留在id上等你来修改！
snipMate支持几乎所有主流语言，PHP，HTML自然不在话下，你可以放心使用！使用这个插件注意要在.vimrc中加入以下这行
``` vim
filetype plugin indent on     "自动检测文件类型以调用相应的语法
```
###让Vim能检查PHP语法错误
Vim默认带了PHP的语法高亮，但这还不够，它不如IDE那样能帮你自动检测语法错误，但要实现这功能也不难，在.vimrc文件里添加以下配置，就能使用CTRL-L键来检查语法，如果有错误，它会告诉你在哪一行。注意你的PHP路径可能和我的不一样。
``` vim
autocmd FileType php noremap <C-L> :!/usr/local/bin/php -l %<CR> " 使用(CTRL-L)命令来检查PHP语法
```
###让taglist能正确显示PHP的类函数
当使用taglist时，我发现它不能很好的支持PHP文件，每个方法和变量都显示了两次，在网上搜了下，发现这是ctags的问题，添加以下配置到.ctags文件，就能解决，注意这个文件是ctags的配置，和Vim没有关系。
``` bash
-f ./tags
-R
--totals=yes
--tag-relative=yes
--PHP-kinds=+cf-v
--regex-PHP
--regex-PHP=/(^|^[^*]+[[:blank:]])class[[:blank:]]+([[:alpha:]][[:alnum:]_]*)/\2/c/
--regex-PHP=/(^|^[^*]+[[:blank:]])function[[:blank:]]+&?[[:blank:]]*([[:alpha:]][[:alnum:]_]*)/\2/f/
```
###让snipMate识别模版文件
snipMate会自动根据文件名的后缀来判断文件类型，但是我使用的cakephp框架里有自定义的.ctp模版文件，我需要它能同时支持html和php的snip，节省我敲代码的数量。很简单，在.vimrc中添加以下配置，将.ctp文件的类型同时设为php和html就可以了
``` vim
au BufRead,BufNewFile *.ctp    set filetype=php.html    "将.ctp文件的类型同时设为php和html
```
我已经更新了[github](https://github.com/perrywky/dotfiles)，并把之前的.ctags文件也加了进来，去看看吧 :)
