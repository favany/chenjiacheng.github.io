---
layout: post
categories: PHP
tags: [PHP]
---

## cgi

cgi 是一个 web server 与 cgi 程序（这里可以理解为是php解释器）之间进行数据传输的协议，保证了传递的是标准数据。

## php-cgi

php-cgi 是 php 解释器。他自己本身只能解析请求，返回结果，不会管理进程。

## Fastcgi

Fastcgi 是用来提高 cgi程序（php-cgi）性能的方案/协议。

cgi 程序的性能问题在哪呢？"PHP解析器会解析php.ini文件，初始化执行环境"，就是这里了。标准的CGI对每个请求都会执行这些步骤，所以处理的时间会比较长。

Fastcgi 会先启一个 master，解析配置文件，初始化执行环境，然后再启动多个 worker。当请求过来时，master 会传递给一个 worker，然后立即可以接受下一个请求。这样就避免了重复劳动，效率自然提高。而且当 worker 不够用时，master 可以根据配置预先启动几个 worker 等着；当然空闲 worker 太多时，也会停掉一些，这样就提高了性能，也节约了资源。这就是 Fastcgi 的对进程的管理。

## php-fpm

FastCGI 是一个方案或者协议，php-fpm 就是 FastCGI 的后端实现，也就是说，进程分配和管理是 FPM 来做的。官方对 FPM 的解释：

Fastcgi Process Manager（Fastcgi 进程管理器）

php-fpm 的管理对象是 php-cgi，他负责管理一个进程池，来处理来自 Web 服务器的请求。

对于 php.ini 文件的修改，php-cgi 进程是没办法平滑重启的，有了 php-fpm 后，就把平滑重启成为了一种可能，php-fpm 对此的处理机制是新的 worker 用新的配置，已经存在的 worker 处理完手上的活就可以歇着了，通过这种机制来平滑过度的。