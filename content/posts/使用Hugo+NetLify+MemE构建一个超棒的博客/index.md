---
title: "使用Hugo+NetLify+MemE构建一个超棒的博客"
date: 2020-09-13T15:56:05+08:00
tags : ["doubt"]
---

# Abstrct
构建一个基于Hugo博客的想法来自于三天前，那天我想给“deep-graph-matching-consensus”的代码进行注释，以理解里面很多概念以便于自己写代码。  
虽然过去的一周我已经通过MacBook的Keynote设计了很好的笔记系统，可以用来学习概念或者阅读论文。但是这种笔记系统更像是在白板上记录自己的学习笔记，而且缺乏很多比如归档，代码，标签等功能（以后我可能会想办法自己实现这个功能），并不是重复造轮子，而是没有合适的轮子开我这张车。  
我之前在疫情期间，自己还是搞了一个博客的，即Hexo，这个博客还算比较成熟。最近，我无意间在网上看到关于Hugo的介绍，据说这是用Golang写的，且是目前最快的静态博客，且安装非常容易。我就心里痒痒想开始瞎搞了！（不折腾自己不舒服司机）。  
本篇blog将介绍如何使用Hugo + NetLify + MemE构建一个超棒的个人博客。  为什么放着已有Hexo不用，且使用NetLify 的CDN网站，以及MemE主题，大概有以下原因：
1. Hexo部署起来实在是太慢了。
2. 我其实是想把博客作为一个私人成长的空间，不太想这么快就被同学朋友发现，但是我在更新我的Hexo博客时，被我同学（强哥）发现了。
3. 我的Hexo博客的LaTeX公式无法正确使用。
4. Hugo有一个优点，即可以直接用Git来管理网站的部署。
5. 为了不宕掉我的Hexo博客，于是我打算在NetLify上部署我的网站。
6. MemE是我在知乎上搜索Hugo时发现的一个女生自己编写的Hugo主题，即简洁，又满足我对字体等博客优美性的考虑，且这个主题是非移植的，独占在Hugo上，因此会更focus在Hugo上。  

综上考虑，我开始在Hugo上重新搭建了静态博客，从此开始书写我的技术人生。

# 安装Hugo
环境：Ubuntu 18.04  
我采用的是Snap安装，这个“包管理器”还挺神奇的，比如：
```
snap version
=>snap    2.45.3.1
=>snapd   2.45.3.1
=>series  16
=>ubuntu  18.04
=>kernel  5.4.0-42-generic
```
使用Snap安装extended版本的Hugo如下：
```
snap install hugo --channel=extended
```
extended版本的Hugo可以支持Sass/SCSS，这个到底有什么用，暂时【存疑】。
安装完，验证Hugo版本：
```
hugo version
=> Hugo Static Site Generator v0.74.3/extended linux/amd64 BuildDate: 2020-07-23T20:39:21Z
```
已知目前最新的Hugo版本为**v0.74.3**，说明这种安装方式没啥问题。我还看到有通过源码安装，并配置PATH路径的方法，还没试过，暂且不表（我喜欢这两种安装方式）。

# 部署MemE主题的本地Hugo网站
基本上是参照，MemE的文档。
```
hugo new site blog_hugo
```
Hugo是没有默认主题的，所以一定要主动安装主题，我这里安装的是MemE主题。
```
~ $ cd blog_hugo
~/blog_hugo $ git init
~/blog_hugo $ git submodule add --depth 1 https://github.com/reuixiy/hugo-theme-meme.git themes/meme
```
## 定期更新MemE主题
```
~/blog_hugo $ git submodule update --rebase --remote
```
## 更换配置文件
需要将MemE的配置文件覆盖默认的配置文件，首先删除Hugo博客根目录下的**config.toml**文件，再将`themes/meme/config-examples/zh-cn/`目录下，MemE 提供的 **config.toml** 复制回站点根目录下。也可以直接使用命令行实现：
```
~/blog_hugo $ rm config.toml && cp themes/meme/config-examples/zh-cn/config.toml config.toml
```

## ...
(to be continue)

## 在Hugo上方便的插入图片
默认的插入图片方式为
```
![图片路径](图片描述)
```
图片路径可以是网上的超链接，也可以是本地相对路径的静态资源，这里选择相对路径的静态资源。为了更清晰的管理目录，这里使用了Page Bundles来新建博文。具体的方式就是新建post的时候，直接新建一个文件夹（文件夹目录可以随意指定，我使用文章名字来作为目录名），然后文章的 **.md** 文件就以 **index.md** 来命名。可以参考https://gohugo.io/content-management/organization/ 以及 https://alison.rbind.io/post/2019-02-21-hugo-page-bundles/ 来获取详细的信息。

由于保存图片，然后再配置插入图片的代码，实在太麻烦了。所以这里使用了一个VS code的插件： **paste Image**. 看下里面的插件介绍，介绍的还算详细。  
这里将VS code 的 **setting.json**文件修改下，加入以下两行：
```
    "pasteImage.path": "${currentFileDir}/",
    "pasteImage.insertPattern": "![${imageFilePath}](${imageFilePath})"
```
大概意思是，图片的插入路径在文章所在的路径下（`${currentFileDir}`）， 而插入文本行的代码则格式为`![${imageFilePath}](${imageFilePath})`. 想了解细节的再看看paste Image的介绍。反正这样就很方便可以插入图片了，只需要截图，然后在想插入图片的位置使用快捷键 “**Ctrl+Alt+V (Cmd+Alt+V on Mac)**”即可！


## 在不同电脑上更新博客
按我的理解，Hugo在异地迁移，不同平台共同写作等方面表现卓越，可以很方便的更新博客。因为Hugo的包是类似二进制文件挂在博客目录下的，因此只需要在不同的电脑上装上Hugo，并用Git clone下远端的源码即可。这里简单介绍下，我是如何从另一台Ubuntu上部署和更新博客的。  
首先，另一台Ubuntu需要安装Hugo. 最好各个系统上的Hugo版本一致。然后从我的GitHub上clone博客的git: 
```
git clone git@github.com:timewaitsfor/blog_hugo.git
```
这个时候，发现一个神奇的事情，即`thems/meme`目录下竟然什么都没有，这里**存疑**。不太清楚为什么，于是我把meme的文件手动复制到了blog_hugo/目录下，运行
```
hugo server -D
```
博客的重新部署，完成！