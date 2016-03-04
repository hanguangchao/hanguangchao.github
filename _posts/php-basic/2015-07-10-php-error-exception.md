---
layout: post
title: "PHP错误和异常"
date: 2015-07-10 16:03
category: php-basic
tags: [php-error]
---
{% include JB/setup %}

#PHP错误和异常

>`天有不测风云，月有阴晴圆缺` ,正如生活中的麻烦不断，程序出错也是家常便饭。如何快速定位程序错误异常并解决Bug,是考验一名程序员素质的重要环节。

> 下面就说一说PHP程序中， `Error` 和 `Exception` 的知识和一般处理的办法。

***

##关于 `Error` `Exception`

**错误级别**

- `Notice` 运行时通知，一般为变量未声明之类的提示信息， 发生`Notice`错误， 程序仍会继续执行。
- `Warning` 运行时警告 (非致命错误)。仅给出提示信息，但是脚本不会终止运行。
- `Error` 运行时错我（致命错误），程序会在发生错误行终止运行。

**错误报告**

错误报告是PHP的配置项。以下为建议的设置。

在开发环境：
```
display_errors= On
error_reporting = E_ALL
```

在生产环境
```
display_errors= Off
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
```

##关于调试程序和检查程序问题

1. 设置错误报告， 在浏览器直接查看发送的错误内容。
```php
error_reporting(E_ALL);
ini_set('display_errors', true);
```

2. 在程序中直接打断点。这个方法也是经常使用的。
```php
$a = 'abc';
$b = $_GET['b'] . $a;
//`var_dump()` 
var_dump($b);
exit();
//或 `error_log()` 写到日志文件
error_log(var_export($b, true), 3, '/tmp/php-error.log');
```
3. `Xdebug`扩展，单步调试。
4. `Xhprof`扩展，检查`PHP`程序函数调用和执行效率。
5. `strace`命令， 检查`PHP`进程系统调用。要懂操作系统底层调用相关知识，否则strace内容是看不懂的。
6. `gdb`调试， 直接跟踪PHP执行，假如检查到某php进程异常， 可以使用 ```gdb -p pid ```命令进入进程调试。

##收集PHP程序错误

**错误处理器**
使用`set_error_handler()`收集错误 
```php
//实用代码
error_reporting( 0 );
//不显示错误报告,将错误收集到错误日志
ini_set('display_errors', false);

function myErrorHandler($errno, $errstr, $errfile, $errline)
{
    switch ($errno) {
        case E_ERROR:
        case E_PARSE:
        case E_CORE_ERROR:
        case E_COMPILE_ERROR:       
        case E_USER_ERROR:
        case E_STRICT:
        case E_WARNING:
        #case E_NOTICE:
        $str = "Error:" . date("Y-m-d H:i:s", time()) . "\t";
        $str .= "Errno:" . $errno . "\t";
        $str .= "File:" . $errfile . "\t";
        $str .= "SERVER:" . $_SERVER['SERVER_ADDR'] . "\t";
        $str .= "URI:" . $_SERVER['HTTP_HOST'] . $_SERVER['REQUEST_URI'] . "\t";
        $str .= "USER:" . $_SESSION['_sys_zname'] . "\t";
        $str .= "Message:" . $errstr . "\t";
        $str .= "Line:" . $errline . PHP_EOL;
        error_log($str, 3,  '/tmp/php_log/' . date("Ymd").'.log');
            break;
    }
    
}

function myfatalError(){
    if ($e = error_get_last()) {
        myErrorHandler($e['type'], $e['message'], $e['file'], $e['line']);
    }
}
set_error_handler("myErrorHandler");
register_shutdown_function('myfatalError');
```
**异常处理器**
```php
//设置异常处理器
function myexception($e){
   echo $e->getMessage();
} 
set_exception_handler("myexception");
```

> 总结：通过错误收集和调试的方法，可以很方便的应对PHP错误处理。

***
