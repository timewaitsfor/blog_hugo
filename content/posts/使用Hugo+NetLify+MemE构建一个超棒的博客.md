---
title: "使用Hugo+NetLify+MemE构建一个超棒的博客"
date: 2020-09-13T15:56:05+08:00
tags : ["doubt"]
---

# abstrct
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
需要将MemE的配置文件覆盖默认的配置文件，首先删除Hugo博客根目录下的**config.toml**文件，再将**themes/meme/config-examples/zh-**cn/目录下，MemE 提供的 **config.toml** 复制回站点根目录下。也可以直接使用命令行实现：
```
~/blog_hugo $ rm config.toml && cp themes/meme/config-examples/zh-cn/config.toml config.toml
```

