---
title: CherryPy：极简主义的 Python Web 框架
lang: zh-CN
tags: CherryPy
categories: 教程
abbrlink: b4158442
date: 2021-03-10 13:36:49
description:
updated:
---

## 简介

[CherryPy](http://www.cherrypy.org/) 是一个 pythonic 的，面向对象的 Web 框架。CherryPy 允许开发人员以与构建任何其他面向对象的 Python 程序相同的方式构建 Web 应用程序。可以在更短的时间内开发出更小的源代码。

CherryPy 应用程序通常如下所示：

```python
import cherrypy

class HelloWorld:
    # CherryPy 绝不会发布未将 `expose` 属性设置为 `True` 的方法
    # 在 Web 中公开 index 方法
    @cherrypy.expose
    def index(self):
        # CherryPy 将为根 URI（"/"）调用此方法，并将其返回值发送给客户端
        return "Hello World!"

# 在尝试将请求 URI 映射到对象时，CherryPy 始终以 `app.root` 开始，
# 因此我们需要挂载请求处理程序根。 对 `'/'` 的请求将映射到 `HelloWorld().index()`。
cherrypy.quickstart(HelloWorld())
```

为了充分利用 CherryPy，您应该从[教程](https://docs.cherrypy.org/en/latest/tutorials.html#tutorials)开始，它将指导您完成框架的最常见方面。完成后，您可能需要浏览[基础](https://docs.cherrypy.org/en/latest/basics.html#basics)和[高级](https://docs.cherrypy.org/en/latest/advanced.html#advanced)部分，以演示如何实现某些操作。最后，您将需要仔细阅读配置和[扩展](https://docs.cherrypy.org/en/latest/extend.html#extend)部分，那是有关框架提供的强大功能的更深入的内容。

## 架构解析

CherryPy 不是由一层组成，而是由四个独立的 API 层组成。

<article>
    <link rel="stylesheet" href="https://xinetzone.github.io/w3css/4/w3.css">
    <link rel="stylesheet" href="https://xinetzone.github.io/xinet-css/tabs.css">
    <div class="tab-set w3-light-grey">
        <input checked="True" id="tab-set--0-input--1" name="tab-set--0" type="radio">
        <label for="tab-set--0-input--1">应用层</label>
        <div class="tab-content w3-padding">
            <p>
                <strong>应用层</strong>是最简单的。CherryPy 应用程序被编写为类和方法的树，其中树中的每个分支都对应于 URL 路径中的一个分支。每个方法都是一个“page
                handler”，它接收 GET 和 POST 参数作为关键字参数，并返回或产生 response
                的（HTML）主体（body）。特殊方法名称“index”用于以斜杠结尾的路径，特殊方法名称“default”用于通过单个处理程序处理多个路径。该层还包括：
            <ul>
                <li><code>exposed</code> 属性 (和 <code>cherrypy.expose</code>)</li>
                <li><code>cherrypy.quickstart()</code></li>
                <li><code>_cp_config attributes</code></li>
                <li><code>cherrypy.tools</code> (包括 <code>cherrypy.session</code>)</li>
                <li><code>cherrypy.url()</code></li>
            </ul>
            </p>
        </div>
        <input id="tab-set--0-input--2" name="tab-set--0" type="radio">
        <label for="tab-set--0-input--2">环境层</label>
        <div class="tab-content w3-padding">
            <p>
                开发人员在各个级别都使用“<strong>环境层</strong>”。它通过一组（默认）顶级（top-level）对象提供有关当前请求和响应以及应用程序和服务器环境的信息：
            <ul>
                <li><code>cherrypy.request</code></li>
                <li><code>cherrypy.response</code></li>
                <li><code>cherrypy.engine</code></li>
                <li><code>cherrypy.server</code></li>
                <li><code>cherrypy.tree</code></li>
                <li><code>cherrypy.config</code></li>
                <li><code>cherrypy.thread_data</code></li>
                <li><code>cherrypy.log</code></li>
                <li><code>cherrypy.HTTPError</code>, <code>NotFound</code>, 和 <code>HTTPRedirect</code></li>
                <li><code>cherrypy.lib</code></li>
            </ul>
            </p>
        </div>
        <input id="tab-set--0-input--3" name="tab-set--0" type="radio">
        <label for="tab-set--0-input--3">扩展层</label>
        <div class="tab-content w3-padding">
            <p>
                <strong>扩展层</strong>允许高级用户构造和共享他们自己的插件。它包括：
            <ul>
                <li><code>Hook</code> API</li>
                <li><code>Tool</code> API</li>
                <li><code>Toolbox</code> API</li>
                <li><code>Dispatch</code> API</li>
                <li>Config Namespace API</li>
            </ul>
            </p>
        </div>
        <input id="tab-set--0-input--4" name="tab-set--0" type="radio">
        <label for="tab-set--0-input--4">核心层</label>
        <div class="tab-content w3-padding">
            <p>
                <strong>核心层</strong> 使用核心 API 来构造可在更高层使用的默认组件。您可以将默认组件视为 CherryPy
                的“参考实现”。Megaframeworks（和高级用户）可以用自定义或扩展组件替换默认组件。核心 API 是：
            <ul>
                <li>Application API</li>
                <li>Engine API</li>
                <li>Request API</li>
                <li>Server API</li>
                <li>WSGI API</li>
            </ul>
            </p>
        </div>
    </div>
</article>

这些 API 在 [CherryPy 规范](https://github.com/cherrypy/cherrypy/wiki/CherryPySpec) 中进行了描述。

## 使用手册

一些基础教程可以查阅：[CherryPy 教程@简书](https://www.jianshu.com/p/16dc6e4dc556)。下面介绍一些进阶是教程。

### 简单的配置

函数 `quickstart(root=None, script_name='', config=None)`，有应该参数 `config` 可以修改一些配置参数。

`config` 可以是包含应用程序配置的文件或字典（`dict`）。如果包含 `[global]` 部分，这些配置项将在（站点范围内） `global` 中使用配置。下面创建一个配置文件 `base-server.toml`，内容如下：

```toml
[global]
server.socket_host = "127.0.0.1"
server.socket_port = 9999
server.thread_pool = 10
```

这样，便可以修改默认的端口为 `9999`：

```python
import cherrypy

class HelloWorld:
    # CherryPy 绝不会发布未将 `expose` 属性设置为 `True` 的方法
    # 在 Web 中公开 index 方法
    @cherrypy.expose
    def index(self):
        # CherryPy 将为根 URI（"/"）调用此方法，并将其返回值发送给客户端
        return "Hello World!"

config = 'configs/base-server.toml'
cherrypy.quickstart(HelloWorld(), config=config)
```

### 创建 HTML 模板

CherryPy 是用于构建 Web 应用程序的 Web 框架。应用程序采用的最传统的形式是通过与 CherryPy 服务器通信的 HTML 用户界面。为此，可以创建一个载入 HTML 模板，加快软件开发效率。

1. 我们借助 [`string`](https://docs.python.org/3/library/string.html).`Template` 创建一个基础的 HTML 模板 `base.html`：

```html
<!DOCTYPE html>
<html lang="${lang}">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>${title}</title>
</head>

<body>
    ${body}
</body>

</html>
```

2. 接着，创建该模板的初始化配置文件 `base.toml`：

```toml
lang = 'zh-CN'
title = 'Web 模板'
body = '<h1>环艺</h1>'
```

3. 创建可以载入模板 HTML 和配置文件的函数：

```python
from string import Template
import toml


class Bunch(dict):
    def __init__(self, *args, **kw):
        super().__init__(*args, **kw)
        self.__dict__ = self


def load_option(path):
    opt = toml.load(path)
    return Bunch(opt)


def load_template(path):
    with open(path, encoding='utf-8') as fp:
        h5 = fp.read()
    return Template(h5)
```

4. 创建解析 HTML 模板的类和函数：

```python
class Template:
    def __get__(self, obj, objtype=None):
        path = obj.template_path
        if path:
            return load_template(path)
        else:
            return {}


class Html:
    template = Template()

    def __init__(self, template_path, config_path):
        self.template_path = template_path
        self._config_path = config_path
        self._configure = self.reset()

    def reset(self):
        path = self._config_path
        if path:
            return load_option(path)
        else:
            return {}

    def __repr__(self):
        return self.template.substitute(self.configure)

    def encode(self, encoding='utf-8'):
        return repr(self).encode(encoding)

    @property
    def configure(self):
        return self._configure

    def update_content(self, config):
        self.configure.update(config)
```

自此，便完成对 HTML 的解析，使用如下：

```python
template_path = 'templates/base.html'
config_path = 'configs/base.toml'

html = Html(template_path, config_path)
html
```

输出：

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Web 模板</title>
</head>

<body>
    
</body>

</html>
```

5. 调用服务器：

```python
import cherrypy
from app.htmlx import Html


class TemplateHtml(Html):
    def __init__(self, template_path, config_path):
        super().__init__(template_path, config_path)

    @cherrypy.expose
    def index(self):
        yield self.encode('utf-8')

template_path = 'templates/base.html'
config_path = 'configs/base.toml'
config = 'configs/base-server.toml'
httpd = TemplateHtml(template_path, config_path)
d = {'body': '''<h1>欢迎进入 Web 世界 <h1>
<p>CherryPy</p>
'''}
httpd.update_content(d) # 更新 HTML
cherrypy.quickstart(httpd, config=config)
```

### GET 请求

编写一个 GET 请求：

```python
import random
import string

import cherrypy
from app.htmlx import Html


class TemplateHtml(Html):
    def __init__(self, template_path, config_path):
        super().__init__(template_path, config_path)

    @cherrypy.expose
    def index(self):
        yield self.encode('utf-8')

    @cherrypy.expose
    def generate(self, length=8):
        return ''.join(random.sample(string.hexdigits, int(length)))


template_path = 'templates/form.html'
config_path = 'configs/form.toml'
config = 'configs/base-server.toml'
httpd = TemplateHtml(template_path, config_path)
cherrypy.quickstart(httpd, config=config)
```