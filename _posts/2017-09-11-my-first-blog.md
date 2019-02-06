---
layout: post
title:  "my first blog"
date:   2017-9-12 16:00:23
comments: false
tags:
  - 工具使用
description: 本文记录了github page搭建博客的全过程，从github page项目的创建，到使用，下载jekyll（jekyll是一个简单的免费的Blog生成工具，类似WordPress。但是和WordPress又有很大的不同，原因是jekyll只是一个生成静态网页的工具，不需要数据库
---
## introduction

本文记录了github page搭建博客的全过程，从github page项目的创建，到使用，下载jekyll（jekyll是一个简单的免费的Blog生成工具，类似WordPress。但是和WordPress又有很大的不同，
原因是jekyll只是一个生成静态网页的工具，不需要数据库支持。），选用模板，最后配置上传生成博客。

## github page的优势
优点:
* 轻量级的网站结构，不再有动态网站的沉重。
* 方便的和github pages结合，不仅免费，而且方便。
* 流行又简洁的MarkDown写作语法。
* 大量的模板，使用者不必太关心前端也能搭建很优美的博客。
* 版本控制。
* 可以使用自己的域名。

## 1.github page的创建
- 1.在github创建一个新的repository

![](https://bo07997.github.io/myBlog/styles/images/Blog/firstBlog/1.jpg)  

- 2.repository的命名按如下图规范username.github.io

![](https://bo07997.github.io/myBlog/styles/images/Blog/firstBlog/2.jpg)  

- 3.去到项目下的setting查看github page是否开启(默认开启)

![](https://bo07997.github.io/myBlog/styles/images/Blog/firstBlog/3.jpg)  

![](https://bo07997.github.io/myBlog/styles/images/Blog/firstBlog/4.jpg)  

## 2.安装 Jekyll

什么？你要快速部署一个博客，然后快乐的在本地用markdown写博客？那完全没必要在本地安装jekyll，博主是个做后台的，对前端技术细节不怎么关心，只是单纯的想有个漂亮界面的博客，所以直接去
jekyll的[模板网站](http://jekyllthemes.org/)下载自己喜欢的模板，然后直接提交到我们第一步创建的repository，就可以直接在_post文件夹下写博客，然后去[官方网站](http://jekyll.com.cn/)
看看配置就可以快乐的写博客了。

如果实在要安装Jekyll，在本地使用Jekyll，参见以下步骤
- 1.安装 Ruby
去[官网](http://rubyinstaller.org/downloads/)下载ruby，因为jekyll就是ruby写的。注意：勾选 “Add Ruby executables to your PATH”。
然后 {% highlight markdown %}ruby -v{% endhighlight %} 查看是否安装成功。

- 2.安装 DevKit
去[官网](http://rubyinstaller.org/downloads/)下载DevKit，在命令窗口下切换到DevKit安装目录，并执行以下命令
{% highlight markdown %}
ruby dk.rb init  
ruby dk.rb install
{% endhighlight %} 

- 3.安装 Jekyll  
{% highlight markdown %}  
//更换gem源  
gem sources --remove https://rubygems.org/  
gem sources -a http://gems.ruby-china.org  
	
//查看gem源  
gem sources -l  
	
//更新gem  
gem update --system 
	
//安装jekyll  
gem install jekyll
{% endhighlight %} 
 
注意:淘宝的源https://ruby.taobao.org/ 已经不能用了,必须用https://rubygems.org/  ,"RubyGems 镜像的管理工作以后将交由 Ruby China 负责"，不过这里也有很多坑，我试了很多次http://gems.ruby-china.org，https://gems.ruby-china.org才可以。

## 3.提交博客，访问username.github.io

![](https://bo07997.github.io/myBlog/styles/images/Blog/firstBlog/5.jpg)  

## 待续..

由于域名还在备案，后续再弄绑定域名，评论等功能
