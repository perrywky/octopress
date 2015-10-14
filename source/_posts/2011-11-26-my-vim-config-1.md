--- 
layout: post
title: 我的Vim配置（一）
categories: 
  - vim
comments: true
---
使用vim开发有一段时间了，经常会做些配置装些插件，导致.vimrc变得臃肿不堪，.vim文件夹也很混乱，很多配置和插件自己都忘了有什么用处，所以今天打算好好整理下，写篇博客做下总结，因为要配置的东西很多，所以我计划分成几篇来写。
###使用Git来管理vim配置，并使用Dropbox备份
因为vim的配置很多很乱，时间长了难免忘记，所以我打算用版本控制工具来管理，为了避免误删文件夹，我打算用dropbox来将vim配置备份。vim的配置有三部分，.vimrc文件，.gvimrc文件，.vim文件夹，我将他们都设为软连接，指向~/Work/Tools/Vim文件夹下，再将Vim文件夹使用Git来管理（Git是我最爱用的版本控制工具）。再用Dropbox把这个文件夹云备份，这样我就可以高枕无忧的使用vim啦！

###程序员通用的配置
下面我就介绍一些vim的基本配置，这些配置不管你是写什么语言的都会用到
``` vim
syntax enable "语法高亮
set nu "显示行号
set ruler "在右下角显示光标的坐标
set hlsearch "高亮显示搜索结果
set incsearch "边输边搜，即时反馈搜索结果，这个可能看个人喜好
set showcmd "在ruler左边显示当前正在输入的命令，提示性的，避免误操作
set expandtab "将tab键改为空格，默认是8个
set tabstop=4 "将tab键改为4个空格
set cindent "使用C语言的规则自动缩进，当你敲回车时会自动缩进，所有类C语言（PHP，JAVA）都试用，比smartindent更智能
set shiftwidth=4 "自动缩进时，使用4个空格，默认是8个
```
以上就是一些程序员常用的基本配置，我会在后续的博客中介绍更多的配置和插件。
