---
layout: post
title:  "openResty开发2 Nginx请求处理阶段"
date:   2017-11-30 10:00:23
comments: false
categories: Nginx
tag: Nginx
description: 本文主要记录Nginx处理请求的11个阶段，主要是为了后面使用lua代码插入Nginx这些阶段做准备                                                      
---
* content
{:toc}
## introduction
读取完请求头后，`nginx`进入请求的处理阶段。简单的情况下，客户端发送过的统一资源定位符(`url`)对应服务器上某一路径上的资源，`web`服务器需要做的仅仅是将`url`映射到本地文件系统的路径，然后读取相应文件并返回给客户端。但这仅仅是最初的互联网的需求，而如今互联网出现了各种各样复杂的需求，要求`web`服务器能够处理诸如安全及权限控制，多媒体内容和动态网页等等问题。这些复杂的需求导致`web`服务器不再是一个短小的程序，而变成了一个必须经过仔细设计，模块化的系统。`nginx`良好的模块化特性体现在其对请求处理流程的多阶段划分当中，多阶段处理流程就好像一条流水线，一个`nginx`进程可以并发的处理处于不同阶段的多个请求。`nginx`允许开发者在处理流程的任意阶段注册模块，在启动阶段，`nginx`会把各个阶段注册的所有模块处理函数按序的组织成一条执行链。

`nginx`实际把请求处理流程划分为了11个阶段，这样划分的原因是将请求的执行逻辑细分，各阶段按照处理时机定义了清晰的执行语义，开发者可以很容易分辨自己需要开发的模块应该定义在什么阶段，其定义在`http`/`ngx`_http_core_module.h中有定义：
## Nginx请求处理阶段 
### 1.post-read(`NGX`_`HTTP`_`POST`_`READ`_`PHASE`)

`Nginx` 读取并解析完请求头（`request` `headers`）之后就立即开始运行，它位于uri重写之前，实际上很少有模块会注册在该阶段，默认的情况下，该阶段被跳过。
### 2.server-rewrite(`NGX`_`HTTP`_`SERVER`_`REWRITE`_`PHASE`) 

`server`级别的`uri`重写阶段，也就是该阶段执行处于`server`块内，location块外的重写指令，在读取请求头的过程中nginx会根据host及端口找到对应的虚拟主机配置。
### 3.find-config(`NGX`_`HTTP`_`FIND`_`CONFIG`_`PHASE`)

寻找`location`配置阶段，该阶段使用重写之后的`uri`来查找对应的location，完成当前请求与location配置块配对，值得注意的是该阶段可能会被执行多次，因为也可能有location级别的重写指令。
### 4.rewrite(`NGX`_`HTTP`_`REWRITE`_`PHASE`) 

`location`级别的uri重写阶段，该阶段执行location基本的重写指令，也可能会被执行多次，当ngx_rewrite指令用于location中，就是再这个阶段运行的。
### 5.post-rewrite(NGX_HTTP_POST_REWRITE_PHASE)

location级别重写的后一阶段，用来检查上阶段是否有uri重写，并根据结果跳转到合适的阶段。
### 6.PREACCESS(NGX_HTTP_PREACCESS_PHASE)

访问权限控制的前一阶段，该阶段在权限控制阶段之前，一般也用于访问控制，比如限制访问频率，链接数等。
### 7.access(`NGX`_`HTTP`_`ACCESS`_PHASE)

访问权限控制阶段，ngx_access在这个阶段运行，配置指令多是执行访问控制相关的任务，比如基于ip黑白名单的权限控制，基于用户名密码的权限控制等。
### 8.post-access(`NGX`_`HTTP`_`POST`_`ACCESS`_`PHASE`)

访问权限控制的后一阶段，主要用于配合 access 阶段实现标准 ngx_http_core 模块提供的配置指令 satisfy 的功能,该阶段根据权限控制阶段的执行结果进行相应处理。
### 9.try-files(NGX_HTTP_TRY_FILES_PHASE)

try_files指令的处理阶段，如果没有配置try_files指令，则该阶段被跳过。
### 10.content(NGX_HTTP_CONTENT_PHASE)

内容生成阶段，是所有请求处理阶段中最为重要的阶段,该阶段产生响应，并发送到客户端。
### 11.`log`(`NGX`_`HTTP`_`LOG`_`PHASE`)

日志记录阶段，该阶段记录访问日志。
	
参考:
[`NGINX`处理请求的几个阶段](`https`://`github`.`com`/`bingbo`/`blog`/`wiki`/`NGINX`%`E`5%`A`4%84%`E`7%90%86%`E`8%`AF`%`B`7%`E`6%`B`1%82%`E`7%9`A`%84%`E`5%87%`A`0%`E`4%`B`8%`AA`%`E`9%98%`B`6%`E`6%`AE`%`B`5)

&`ensp`;&`ensp`;&`ensp`;&`ensp`;&`ensp`;[`nginx-lua` 运行阶段](`http`://`blog`.`csdn`.`net`/`fb`408487792/`article`/`details`/53610140)
	
	
	
