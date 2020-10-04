---
layout: post
title: cookie与session
date: 2020-10-04
categories: blog
tags: [web,php]
description: 细说cookie与session的原理与作用。
---
[toc]

# Cookie

cookie是一种服务端配置在用户电脑上的小文件，用于保存用户信息。因为http(s)是无连接的，所以服务器没法记住同一个用户，从用户的cookie中获取一定的信息（主要用于识别用户），能够帮助服务端处理请求以及提升用户体验。

PHP能够创建或取回cookie的值

## 创建cookie

setcookie()（或setrawcookie()）用于创建cookie

```php
setcookie(name,value,expire,path,domain);
```

实例1：

```php+HTML
<?php
setcookie('user','Li',time()+3600);	//设置user的值为'Li',在一小时后过期
?>
<html>
  ...
</html>
```

发送cookie时会自动对 cookie值进行url编码，读取时会自动解码，如果不要进行url编码，就可以使用setrawcookie()函数

实例 2:

还可以通过另一种方式设置 cookie 的过期时间。这也许比使用秒表示的方式简单。

```php+HTML
<?php
$expire=time()+60*60*24*30;
setcookie("user", "runoob", $expire);
?>

<html>
  ...  
</html>
```

## 读取cookie值

 $_COOKIE 变量用于取回 cookie 的值。

```php+HTML
<?php
if(isset($_COOKIE['user']))	//用isset()函数来判断是否设置了名为'user'的cookie
{
	$user=$_COOKIE['user'];	//读取user的值
    echo '欢迎'.$user.'！</br>';
}
else
{
    echo '欢迎，游客！'
}
print_r($_COOKIE);	//输出所有cookie值
```

## 删除 Cookie？

当删除 cookie 时，只要使过期日期变更为过去的时间点。

删除的实例：

```php
<?php
// 设置 cookie 过期时间为过去 1 小时
setcookie("user", "Li", time()-3600);
?>
```

# Session

PHP session 变量用于存储关于用户会话（session）的信息，或者更改用户会话（session）的设置。Session 变量存储单一用户的信息，并且对于应用程序中的所有页面都是可用的。

还是由于http(s)无连接的原因，当用户进行了一次操作以后，一般来说，web服务器并不能记住用户，如果用户需要连续，连贯的操作，就会很难受。而通过在服务器上储存session信息来记录用户本次“会话”的临时信息（用户名，身份信息，行为等），可以让服务器知道用户是谁以及做了什么。

Session信息是临时的，在用户离开网站后就会销毁，需要保存的信息只能储存在数据库中

**Session 的工作机制是：为每个访客创建一个唯一的 id (UID)，并基于这个 UID 来存储变量。UID 存储在 cookie 中，或者通过 URL 进行传导。**

## 开始Session

```php+HTML
<?php
session_start(); 
?>
 
<html>
<body>
 
</body>
</html>
```

注意：session_start()函数必须在<html>标签之前。

## 储存和读取session



## 销毁 Session

如果您希望删除某些 session 数据，可以使用 unset() 或 session_destroy() 函数。

unset() 函数用于释放指定的 session 变量：

**实例**

```php
<?php 
    session_start(); 
	if(isset($_SESSION['views']))
    {    
        unset($_SESSION['views']);
    } 
?>
```



也可以通过调用 session_destroy() 函数**彻底销毁** session：

**实例**

```php
<?php 
    session_destroy(); 
?>
```

## Cookie和Session

cookie储存在用户计算机中且可查询可更改，session处在网站服务器中可见但不可随意更改，从这个角度来说，session比cookie更加安全”一些“。

但session不能完全取代cookie的作用，只有session和cookie一起使用才能带来最高的效率。将一些消息保存在用户端以提高便利性，将一些信息保存在服务端以保证安全性和完整性。

一般来说，用cookie来储存session ID，以帮助服务器实别用户。

session ID即服务器中保存的session文件的名字，一般形如这样：sess_4c83638b3b0dbf65583181c2f89168ec（后面是32位编码的随机字符，也正是session ID）。这样，当用户第一次访问服务器，服务器生成一个唯一的随机的session ID，并放进一个cookie里发送给客户端，客户端再访问服务器时就自动带着这个cookie，服务器就能识别用户并找到对应session文件里的信息了（现在还有一种机制叫token，跟这个机制很像，但也有一定的不同，以后单独再开篇文章记录）

以上是客户端开启了cookie服务的情况，如果客户端禁用了cookie，就需要服务器（服务器会判断客户端是否开启了cookie）在url中添加session ID来实现session的传输，

形如 ：www.xxx.com?PHPSESSID=4c83638b3b0dbf65583181c2f89168ec

或者使用隐藏表单来传输session，自动在表单中添加一个属性为"hidden"的<input>,里面提交的是服务器为用户生成的session值。