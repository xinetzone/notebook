---
title: CherryPy 基础
lang: zh-CN
abbrlink: 5b871e48
date: 2021-03-12 09:11:05
description:
updated:
tags: CherryPy
categories: 教程
---

翻译自 [CherryPy Basics](https://docs.cherrypy.org/en/latest/basics.html#id3)

以下各节将引导您完成 CherryPy 应用程序的基础知识，并介绍一些基本概念。

## 1 一分钟的应用示例

您可以用 CherryPy 编写的最基本的应用程序几乎涉及其所有核心概念。

```python
import cherrypy


class Root(object):
    @cherrypy.expose
    def index(self):
        return "Hello World!"


if __name__ == '__main__':
    cherrypy.quickstart(Root(), '/')
```

首先，对于大多数任务，您将只需要第 1 行中所示的单个 `import` 语句即可。在讨论这些内容之前，让我们跳到第 11 行，该行显示如何使用 CherryPy 服务器应用程序托管您的应用程序，以及如何在 `/` 路径中将其与内置的 HTTP 服务器一起使用。

现在回到实际的应用程序。即使 CherryPy 没有强制要求，大多数时候您的应用程序仍将被编写为 Python 类。这些类的方法将由 CherryPy 调用以响应客户端请求。但是，CherryPy 需要意识到可以使用这种方法，我们说该方法需要公开。这正是 `cherrypy.expose` 装饰器在第 5 行中所做的。

执行此程序，在你的浏览器定位到：`http://127.0.0.1:8080` 可以预览效果。

<article class="w3-card w3-padding w3-light-grey">
<p class="w3-text-yellow w3-wide w3-large">注意</p>
<p>
CherryPy 是一个小型框架，专注于一项任务：接收 HTTP 请求并找到与请求的 URL 匹配的最合适的 Python 函数或方法。与其他知名框架不同，CherryPy 不提供对数据库访问，HTML 模板或任何其他中间件漂亮功能的内置支持。
</p>
<p>
简而言之，一旦 CherryPy 找到并调用了公开的方法，作为开发人员，您就应自行提供工具来实现应用程序的逻辑。
</p>
<p>
CherryPy 认为您（开发人员）最了解。
</p>
</article>

<article class="w3-card w3-padding w3-margin-top w3-pale-red">
<p class="w3-text-yellow w3-wide w3-large">警告</p>
<p>
前面的示例演示了 CherryPy 接口的简单性，但是您的应用程序可能还会包含其他一些细节：静态服务，更复杂的结构，数据库访问等。这将在教程部分中进行开发。
<p>
</article>

CherryPy 是一个微型框架，但不是一个裸露的框架，它带有一些基本工具来涵盖您期望的常用用法。

## 2 [托管一个或多个应用程序](https://docs.cherrypy.org/en/latest/basics.html#id5)

Web 应用程序需要访问 HTTP 服务器。 CherryPy 提供了自己的，可投入生产的 HTTP 服务器。有两种方法来托管应用程序。

### 2.1 单一应用

最直接的方法是使用 `cherrypy.quickstart` 函数。它需要至少一个参数，即要托管的应用程序实例。另外两个设置是可选的。首先，可以从中访问应用程序的基本路径。其次，使用配置字典或文件来配置您的应用程序。

```python
cherrypy.quickstart(Blog())
cherrypy.quickstart(Blog(), '/blog')
cherrypy.quickstart(Blog(), '/blog', {'/': {'tools.gzip.on': True}})
```

第一个意味着您的应用程序将在 ` http://hostname:port/` 上可用，而另两个将使您的博客应用程序在 `http://hostname:port/blog` 上可用。此外，最后一个为应用程序提供了特定的设置。

<article class="w3-card w3-padding w3-light-grey">
<p class="w3-text-yellow w3-wide w3-large">注意</p>
<p>
注意在第三种情况下，设置如何仍然相对于应用程序，而不是在何处可用，因此使用 {'/': ... } 而不是 {'/blog': ... }。
</p>
</article>

### 2.2 多元应用

`cherrypy.quickstart` 方法适用于单个应用程序，但缺乏使用服务器托管多个应用程序的能力。 为此，必须使用 `cherrypy.tree.mount` 函数，如下所示：

```python
cherrypy.tree.mount(Blog(), '/blog', blog_conf)
cherrypy.tree.mount(Forum(), '/forum', forum_conf)

cherrypy.engine.start()
cherrypy.engine.block()
```

本质上，`cherrypy.tree.mount` 具有与 `cherrypy.quickstart` 相同的参数：应用程序，托管路径段和配置。最后两行只是启动应用程序服务器。

<article class="w3-card w3-padding w3-light-grey">
<p class="w3-text-yellow w3-wide w3-large">重要</p>
<p>
    <code>cherrypy.quickstart</code> 和 <code>cherrypy.tree.mount</code> 不是唯一的。例如，前几行可以写成：
</p>

```python
cherrypy.tree.mount(Blog(), '/blog', blog_conf)
cherrypy.quickstart(Forum(), '/forum', forum_conf)
```
</article>

<article class="w3-card w3-padding w3-margin-top w3-light-grey">
<p class="w3-text-yellow w3-wide w3-large">注意</p>
<p>
您也可以 <a href="https://docs.cherrypy.org/en/latest/advanced.html#hostwsgiapp"> 托管外部 WSGI 应用程序</a>。
</p>
</article>

## 3 [Logging](https://docs.cherrypy.org/en/latest/basics.html#id8)

日志记录（Logging）是任何应用程序中的重要任务。CherryPy 将记录所有传入的请求以及协议错误。

为此，CherryPy 管理着两个记录器：

1. 记录每个传入请求的访问权限
2. 跟踪错误或其他应用程序级别消息的应用程序/错误日志

您的应用程序可以通过调用 `cherrypy.log` 来利用第二个记录器。

```python
cherrypy.log("hello there")
```

您还可以记录异常：

```python
try:
   ...
except Exception:
   cherrypy.log("kaboom!", traceback=True)
```

这两个日志都将写入由配置中的以下键标识的文件：

- 使用[通用日志格式](http://en.wikipedia.org/wiki/Common_Log_Format)的传入请求的 `log.access_file`
- 其他日志的 `log.error_file`

<article class="w3-card w3-padding w3-margin-top w3-light-grey">
<p class="w3-text-yellow w3-wide w3-large">也可以参考</p>
<p>
有关 CherryPy 的日志记录体系结构的更多详细信息，请参阅 <a href="https://docs.cherrypy.org/en/latest/pkg/cherrypy._cplogging.html#module-cherrypy._cplogging">cherrypy._cplogging</a> 模块。
</p>
</article>

### 3.1 [Disable logging](https://docs.cherrypy.org/en/latest/basics.html#id9)

您可能有兴趣禁用某个日志。

要禁用文件日志记录，只需在[全局配置](https://docs.cherrypy.org/en/latest/basics.html#globalsettings)中为 `log.access_file` 或 `log.error_file` 键对应更多值设置一个空字符串。

要禁用控制台日志记录，请将 `log.screen` 设置为 `False`。

```python
cherrypy.config.update({'log.screen': False,
                        'log.access_file': '',
                        'log.error_file': ''})
```

### 3.2 [与您的其他记录器一起玩](https://docs.cherrypy.org/en/latest/basics.html#id10)

您的应用程序可能显然已经在使用日志记录模块来跟踪应用程序级别的消息。下面是一个简单的设置示例。

```python
import logging
import logging.config

import cherrypy

logger = logging.getLogger()
db_logger = logging.getLogger('db')

LOG_CONF = {
    'version': 1,

    'formatters': {
        'void': {
            'format': ''
        },
        'standard': {
            'format': '%(asctime)s [%(levelname)s] %(name)s: %(message)s'
        },
    },
    'handlers': {
        'default': {
            'level':'INFO',
            'class':'logging.StreamHandler',
            'formatter': 'standard',
            'stream': 'ext://sys.stdout'
        },
        'cherrypy_console': {
            'level':'INFO',
            'class':'logging.StreamHandler',
            'formatter': 'void',
            'stream': 'ext://sys.stdout'
        },
        'cherrypy_access': {
            'level':'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'formatter': 'void',
            'filename': 'access.log',
            'maxBytes': 10485760,
            'backupCount': 20,
            'encoding': 'utf8'
        },
        'cherrypy_error': {
            'level':'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'formatter': 'void',
            'filename': 'errors.log',
            'maxBytes': 10485760,
            'backupCount': 20,
            'encoding': 'utf8'
        },
    },
    'loggers': {
        '': {
            'handlers': ['default'],
            'level': 'INFO'
        },
        'db': {
            'handlers': ['default'],
            'level': 'INFO' ,
            'propagate': False
        },
        'cherrypy.access': {
            'handlers': ['cherrypy_access'],
            'level': 'INFO',
            'propagate': False
        },
        'cherrypy.error': {
            'handlers': ['cherrypy_console', 'cherrypy_error'],
            'level': 'INFO',
            'propagate': False
        },
    }
}

class Root(object):
    @cherrypy.expose
    def index(self):

        logger.info("boom")
        db_logger.info("bam")
        cherrypy.log("bang")

        return "hello world"

if __name__ == '__main__':
    cherrypy.config.update({'log.screen': False,
                            'log.access_file': '',
                            'log.error_file': ''})
cherrypy.engine.unsubscribe('graceful', cherrypy.log.reopen_files)
    logging.config.dictConfig(LOG_CONF)
    cherrypy.quickstart(Root())
```

在此代码段中，我们创建一个[配置字典](https://docs.python.org/2/library/logging.config.html#logging.config.dictConfig)，然后将其传递到 `logging` 模块以配置记录器：

- 默认的根记录器与单个流处理程序关联
- db 后端的记录器，还有一个流处理程序

另外，我们重新配置 CherryPy 记录器：

- 顶级 `cherrypy.access` 记录器，将请求记录到文件中
- `cherrypy.error` 记录器，将其他所有内容记录到文件中并登录到控制台

当自动重新加载程序启动时，我们还阻止 CherryPy 尝试打开其日志文件。由于我们甚至都不让 CherryPy 首先打开它们，因此这不是严格要求的。但是，这样可以避免浪费时间在无用的东西上。

## 4 [Configuring](https://docs.cherrypy.org/en/latest/basics.html#id11)

CherryPy 带有细粒度的配置机制，可以在各种级别上进行设置。

<article class="w3-card w3-padding w3-margin-top w3-light-grey">
<p class="w3-text-yellow w3-wide w3-large">也可以参考</p>
<p>
复习了基础知识后，请参考有关配置的<a href="https://docs.cherrypy.org/en/latest/config.html#configindepth">深入讨论</a>。
</p>
</article>

### 4.1 [Global server configuration](https://docs.cherrypy.org/en/latest/basics.html#id12)

