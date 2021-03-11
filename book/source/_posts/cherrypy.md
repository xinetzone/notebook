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

首先了解一些基础知识。

### 基础教程

下面的示例演示了用 CherryPy 编写的最基本的应用程序。它启动一个服务器并托管一个应用程序，该应用程序将在请求时提供服务到 <http://127.0.0.1:8080/>。

```python
import cherrypy

class HelloWorld:
    @cherrypy.expose
    def index(self):
        return "你好世界！"

cherrypy.quickstart(HelloWorld())
```

此时，终端可以看到：

![](hello.png)

这告诉你一些事情。前三行表示服务器将处理 [`signal`](https://docs.python.org/3/library/signal.html#module-signal "(在 Python v3.8)") 。下一行告诉您服务器的当前状态，因为此时服务器处于 `STARTING` 阶段。然后，系统会通知您的应用程序没有为其设置特定的配置。接下来，服务器启动一些内部实用程序，稍后我们将对此进行解释。最后，服务器表明它现在已经准备好接受传入的通信，因为它正在监听地址 `127.0.0.1:8080`。换句话说，在那个阶段，您的应用程序就可以使用了。

然后，在浏览器打开输入网址： <http://127.0.0.1:8080/>，可以看到：

![](hello1.png)

#### [不同的函数产生不同的 URL](https://docs.cherrypy.org/en/latest/tutorials.html#id5)

显然，您的应用程序可能会处理多个 URL。假设您有一个应用程序，每次调用它时都会生成一个随机字符串：

```python
import random
import string

import cherrypy


class StringGenerator:
    @cherrypy.expose
    def index(self):
        return "你好世界！"

    @cherrypy.expose
    def generate(self):
        return ''.join(random.sample(string.hexdigits, 8))


if __name__ == '__main__':
    cherrypy.quickstart(StringGenerator())
```

现在转到 `http://localhost:8080/generate`，您的浏览器将显示一个随机字符串。

![](urls.png)

让我们花一分钟时间来分解这里发生的事情。这是您在浏览器中键入的 URL: [http://localhost:8080/generate](http://localhost:8080/generate)

此 URL 包含多个部分：

*   `http://` 这大致表明它是一个使用 HTTP 协议的 URL（请参见 [**RFC 2616**](https://tools.ietf.org/html/rfc2616.html) ）

*   `localhost:8080` 是服务器的地址。它由主机名和端口组成。

*   `/generate` 它是 URL 的路径段。这就是 CherryPy 用来定位 [exposed](https://docs.cherrypy.org/en/latest/glossary.html#term-exposed) 响应的函数或方法。

在这里，CherryPy 使用 [`index()`](https://docs.cherrypy.org/en/latest/pkg/cherrypy.lib.covercp.html#cherrypy.lib.covercp.CoverStats.index "cherrypy.lib.covercp.CoverStats.index") 方法处理 `/` 以及 `generate()` 方法处理 `/generate`。

当然也可以是中文作为方法名称，比如：

```python
import random
import string

import cherrypy


class StringGenerator:
    @cherrypy.expose
    def index(self):
        return "你好世界！"

    @cherrypy.expose
    def 生成(self):
        return ''.join(random.sample(string.hexdigits, 8))


if __name__ == '__main__':
    cherrypy.quickstart(StringGenerator())
```

此时，将创建子路径 `http://localhost:8080/生成`（不推荐使用中文）。

#### [设置 URL 的参数](https://docs.cherrypy.org/en/latest/tutorials.html#tutorial-3-my-urls-have-parameters "Permalink to this headline")

在前面的教程中，我们已经了解如何创建可以生成随机字符串的应用程序。现在假设您希望动态地指示该字符串的长度。

```python
import string
import random
import cherrypy


class StringGenerator:
    @cherrypy.expose
    def index(self):
        return "你好世界！"

    @cherrypy.expose
    def generate(self, length=8):
        return ''.join(random.sample(string.hexdigits, int(length)))


if __name__ == '__main__':
    cherrypy.quickstart(StringGenerator())
```

现在转到 `http://localhost:8080/generate?length=16`，浏览器将显示长度为 16 的生成字符串。请注意，我们也可以从 Python 的默认参数值中获益，以支持 URL，如 `http://localhost:8080/generate`。

在这样的 URL 中，后面的部分 `?` 称为查询字符串。一般地，查询字符串通过传递一组（键、值）对来将 URL 上下文化（contextualize）。它们的格式是 `key=value`，且使用 `&` 分隔。

注意我们必须转换给定的 `length` 值为整数。实际上，值是作为字符串从客户端发送到服务器的。

与 CherryPy 将 URL 路径段映射到公开的函数很相似，查询字符串键映射到公开的函数参数。

下面介绍一些进阶是教程。

### 简单的配置

在继续之前，让我们讨论一下关于配置的消息。默认情况下，CherryPy 有一个特性，它将检查配置应用程序时可以提供的设置的语法正确性。如果没有提供，日志中将显示警告消息。那根原木是无害的，不会阻止 CherryPy 工作。你可以参考 [配置的文档](https://docs.cherrypy.org/en/latest/basics.html#perappconf) 了解如何设置配置。

函数 `quickstart(root=None, script_name='', config=None)`，利用参数 `config` 可以修改一些配置参数。

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

config = 'configs/base-server.toml' # 这里也可以是 dict 形式
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
    def __init__(self, **kw):
        super().__init__(**kw)
        self.__dict__ = self


def load_option(path):
    opt = toml.load(path)
    return Bunch(**opt)


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

    @property
    def config(self):
        return self._configure

    def __repr__(self):
        return self.template.substitute(self.config)

    def encode(self, encoding='utf-8'):
        return repr(self).encode(encoding)

    def update(self, config):
        self.config.update(config)
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
httpd.update(d) # 更新 HTML
cherrypy.quickstart(httpd, config=config)
```

### 提交表单

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

    def quickstart(self, script_name='', app_config=None):
        cherrypy.quickstart(self, script_name, app_config)


template_path = 'templates/form.html'
config_path = 'configs/form.toml'
config = 'configs/base-server.toml'
httpd = TemplateHtml(template_path, config_path)
httpd.quickstart(app_config=config)
```

请注意，在此示例中，表单使用 `GET` 方法，并且当您按下 `Give it now!` 按钮，使用与上一教程相同的 URL 发送表单。HTML 表单还支持 `POST` 方法，在这种情况下，查询字符串不会附加到 URL，而是作为客户端请求主体发送给服务器的。但是，这不会更改您应用程序的 `exposed` 方法，因为 CherryPy 以相同的方式处理这两种情况，并使用 `exposed` 的处理程序参数来处理查询字符串 `(key, value)` 对。

### [跟踪最终用户的活动](https://docs.cherrypy.org/en/latest/tutorials.html#id8)

应用程序需要一段时间跟踪用户的活动并不少见。通常的机制是使用在用户和您的应用程序之间的对话过程中携带的 [会话标识符](http://en.wikipedia.org/wiki/Session_(computer_science)#HTTP_session_token)。
