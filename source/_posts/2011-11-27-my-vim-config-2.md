--- 
categories: 
  - vim
comments: true
layout: post
title: 我的Vim配置（二）
---
上次讲了一些Vim的基本配置，这次我将介绍三个必装的Vim插件
###使用pathogen来管理你的插件
安装vim插件是件很麻烦的事情，首先要把\*.vim文件放到plugin目录里，其次要把文档放到doc目录里，而很多插件还自带了一些文件和目录，久而久之，.vim文件夹就显得臃肿不堪了，***你都不知道哪个文件是属于哪个插件的！***[pathogen](https://github.com/tpope/vim-pathogen)就是一个专门对付这种问题的插件。装了此插件后，你其他所有的插件都可以放到.vim/bundle目录下，每个插件对应的是一个单独的目录，这样插件自带的文件就不会混在一起，管理起来就轻松多了！
pathogen的安装我就不多说了，链接里介绍的很清楚。

<!--more-->

###NERDTree
[NERDTree](http://www.vim.org/scripts/script.php?script_id=1658)是我装过的第一个插件，它可以在vim里显示目录和文件的结构，类似IDE里的Navigator功能，方便你找到文件，非常好用，安装也很简单，参照链接里的指示就行了。不过我建议你安装[Github](https://github.com/scrooloose/nerdtree)里的最新版本，命令如下
``` bash
git submodule add https://github.com/scrooloose/nerdtree.git .vim/bundle/nerdtree
```
它将nerdtree的git repository作为一个子项目下载到bundle目录里，并可以通过以下命令更新
``` bash
git submodule foreach git pull origin master
```
这样你就可以方便的更新你的vim插件，并同时管理好你自己的vim配置了！
此外，NERDTree的书签功能是我的最爱，我将常用的几个项目地址设为书签，这样就可以很方便的在不同项目间导航！
为了更方便的使用，你可能还需要下面3行配置
``` vim
autocmd VimEnter * NERDTree "启动Vim时自动打开nerdtree
let NERDTreeShowBookmarks=1 "一直显示书签
let NERDTreeChDirMode=2 "打开书签时，自动将Vim的pwd设为打开的目录，如果你的项目有tags文件，你会发现这个命令很有帮助
```
因为我使用的是[MacVim](https://github.com/b4winckler/macvim)，所以我将以上三行放到了.gvimrc里。放在.vimrc里的话，NERDTree会占用很大的空间，而通常终端窗口都很小，所以不推荐。
###Command-T
[Command-T](https://wincent.com/products/command-t)是一个比NERDTree更好用的插件！使用NERDTree打开文件你还需要从目录树里层层打开文件夹，找到文件再按o打开，使用Command-T，你只需要输几个简单的字符，它就能帮你定位到pwd目录下最匹配的那个文件！类似你在使用google搜索时，下面实时反馈的搜索建议。具体演示可看链接里的第一个视频，非常强大！
然而使用Command-T有一个条件，就是你的vim得支持ruby，通常原生的vim是不带这个这个功能的，不过MacVim没有这个问题，大可以直接安装这个插件。Command-T的安装过程比NERDTree稍微复杂点，需要编译，不过[这个文档](http://git.wincent.com/command-t.git/blob_plain/HEAD:/doc/command-t.txt)说得也很清楚了，注意第四节，使用pathogen。
安装好了后就可以直接使用了，添加以下两个快捷键可以让你用的更加顺手！
``` vim
nmap ,t :CommandT<CR>
nmap ,b :CommandTBuffer<CR>
```
###Github
我将我的vim配置上传到[github](https://github.com/perrywky/dotfiles)上了，感兴趣的可以去看看 :)
