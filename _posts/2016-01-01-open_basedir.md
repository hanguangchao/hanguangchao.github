---
layout: post
title: "PHP选项 open_basedir"
date: 2016-01-01 16:03
category: php-basic
tags: []
---

#PHP文件包含目录 open_basedir


#open_basedir

> 这几天在一个vhost做测试，php报包含文件错误，刚开始检查了php.ini , 并没有发现包含目录设置，有点诡异， 最终检查到和PHP配置项`open_basedir` 有关，排查过程中检查了几个配置方法，做个记录和总结。 

先来看下 `open_basedir` 官方介绍

`open_basedir` `string`
将 PHP 所能打开的文件限制在指定的目录树，包括文件本身。本指令不受安全模式打开或者关闭的影响。
当一个脚本试图用例如 fopen() 或者 gzopen() 打开一个文件时，该文件的位置将被检查。
当文件在指定的目录树之外时 PHP 将拒绝打开它。
所有的符号连接都会被解析，所以不可能通过符号连接来避开此限制。
特殊值 . 指明脚本的工作目录将被作为基准目录。但这有些危险，因为脚本的工作目录可以轻易被 chdir() 而改变。

在 httpd.conf 文件中中，open_basedir 可以像其它任何配置选项一样用“php_admin_value open_basedir none”的方法关闭（例如某些虚拟主机中）。

在 Windows 中，用分号分隔目录。在任何其它系统中用冒号分隔目录。作为 Apache 模块时，父目录中的 open_basedir 路径自动被继承。

*用 open_basedir 指定的限制实际上是前缀，不是目录名*。
也就是说“open_basedir = /dir/incl”也会允许访问“/dir/include”和“/dir/incls”，如果它们存在的话。
如果要将访问限制在仅为指定的目录，用斜线结束路径名。例如：“open_basedir = /dir/incl/”。
> Note: 支持多个目录是 3.0.7 加入的。
默认是允许打开所有文件。


## 接下来总结下， 可以有几种方式设置限制包含目录

1. php.ini  open_basedir = /home/wwwroot/
2. ini_set   注意：PHP >5.2.3+ PHP_INI_ALL ，不建议使用，这么设置太随意了。 
3. apache httpd.conf中Directory配置 

```
"php_admin_value open_basedir none" #关闭 
php_admin_value open_basedir "/home/wwwroot/:/tmp/:/var/tmp/:/proc/"
```

httpd.conf中VirtualHost 

```
php_admin_value open_basedir "/home/wwwroot/:/tmp/:/var/tmp/:/proc/"
```

4. nginx fastcgi.conf   

```
fastcgi_param  PHP_VALUE  "open_basedir=$document_root:/tmp/"; 
```
            
5. `.user.ini` 文件
    设置方法同 1 .


了解到这么多方式可以限制目录，妈妈再也不担心找不到PHP包含文件了。

