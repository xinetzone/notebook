---
title: WSGI 实用程序和参考实现
lang: zh-CN
tags:
  - Python
  - WSGI
categories: 手册
abbrlink: c16a919d
date: 2021-02-20 12:58:50
description:
updated:
---

WSGI（Web Server Gateway Interface, Web 服务器网关接口）是 Web 服务器软件和用 Python 编写的 Web 应用程序之间的标准接口，此接口的目的是使得 Web 应用在不同 Web 服务器之间具有可移植性。具有标准的界面可以轻松地将支持 WSGI 的应用程序与许多不同的 Web 服务器一起使用。

只有 Web 服务器和编程框架的作者才需要了解 WSGI 设计的每个细节和特殊情况。您无需了解 WSGI 的每个细节，仅安装 WSGI 应用程序或使用现有框架编写 Web 应用程序即可。

[wsgiref](https://docs.python.org/zh-cn/3.10/library/wsgiref.html#module-wsgiref) 是 WSGI 规范的参考实现，可用于将 WSGI 支持添加到 Web 服务器或框架。它提供了用于处理 WSGI 环境变量和响应标头的实用程序，用于实现 WSGI 服务器的基类，为 WSGI 应用程序提供服务的演示 HTTP 服务器以及用于检查 WSGI 服务器和应用程序是否符合 WSGI 规范的验证工具（[PEP 3333](https://www.python.org/dev/peps/pep-3333)）。

有关 WSGI 的更多信息，请参见 [wsgi.readthedocs.io](https://wsgi.readthedocs.io/)，以及教程和其他资源的链接。

Python 目前拥有大量的 Web 框架，比如 Zope, Quixote, Webware, SkunkWeb, PSO, 和 Twisted Web。大量的选择使得新手无所适从，因为总得来说，框架的选择都会限制 Web 服务器的选择。

WSGI 的目的是使得 Web 框架和 Web 服务器之间轻松互连，而不是创建一套新的 Web 框架。

## 概述

参考 [PEP 333：Python Web服务器Gateway接口 v1.0](https://www.biaodianfu.com/pep-333-python-web-server-gateway-interface.html)|[PEP 3333：Python Web 服务器网关接口v1.0.1](https://www.biaodianfu.com/pep-3333.html)

WSGI 接口有服务端和应用端两部分，服务端也可以叫网关端，应用端也叫框架端。服务端调用一个由应用端提供的可调用对象。如何提供这个对象，由服务端决定。例如某些服务器或者网关需要应用的部署者写一段脚本，以创建服务器或者网关的实例，并且为这个实例提供一个应用实例。另一些服务器或者网关则可能使用配置文件或其他方法以指定应用实例应该从哪里导入或获取。

除了这些比较纯的服务器/网关 和应用/框架，我们还可以创建实现了此规范两端的中间件组件。这个中间件对服务器来说像是应用，对它所包含的应用来说像是服务器。中间件可以用来提供扩展 API、内容转换、导航等其他有用的功能。

纵观整个规范，术语『可调用对象』可能代表函数、方法、类或者实现了 `__call__` 方法的实例。这取决于服务器、网关或者应用选择哪种实现技术。相反地，调用这个可调用对象的服务器、网关或者应用不能依赖提供给它的是哪种可调用对象。可调用对象仅仅是被调用，不会内省自己。

## 字符串类型

一般来说，HTTP 处理的是字节，这意味着 WSGI 规范也要处理字节。然而，这些字节内容往往有某种文本解释。在 Python 中，字符串是处理文本最方便的方式。

但是在很多 Python 版本和实现中，字符串是 Unicode，不是字节。这就需要小心平衡在 HTTP 的上下文中如何正确转换字节和文本，并提供有用的 API。尤其是需要支持 Python 实现中不同 `str` 类型的转换代码。

因此 WSGI 定义了两种字符串：

- 原生字符串（总是使用 `str` 来实现）用于 请求/响应 的头部（headers）和元数据（metadata）
- 字节字符串（在 Python3 中用 `bytes` 实现）用于请求/响应的数据部分（如 POST/PUT 的输入数据，HTML 的输出内容等）。


不要弄混了：即使 Python 的 `str` 实际上是 unicode 字符，原生字符串内容也必须支持通过 Latin-1 转换成字节码。

## 应用/框架端

应用对象（application object）就是一个简单的接受两个参数的可调用对象。不要混淆术语"object"就真的是一个对象实例。Python 中的函数、方法、类、实现了 `__call__` 的实例都是可以接受的。应用对象必须可以被多次调用，因为实际上所有服务器/网关(除了 CGI 网关)都会重复地调用它。

注意：我们总是讲 应用对象，不要误解为应用开发者需要使用 WSGI 作为 web 编程 API！应用开发者可以继续使用已经存在的、高级框架服务去开发他们的应用。WSGI 是一个为框架开发者和服务器开发者准备的工具，应用开发者不需要直接使用 WSGI。

应用对象必须：

1. accept two positional parameters:
    - A dictionary containing CGI like variables; and
    - a callback function that will be used by the application to send HTTP status code/message and HTTP headers to the server.
2. return the response body to the server as strings wrapped in an iterable.

比如：

```python
def simple_app(environ, start_response):
    """最简单的应用对象
    参数
    =======
    environ: 指向包含类似于 CGI 的环境变量的字典，该字典由服务器针对客户端收到的每个请求填充
    start_response: 服务器提供的回调函数，它以 HTTP 状态和标头为参数
    """
    # 使用提供的 environ 字典构建响应主体
    response_body = f'Request method: {environ['REQUEST_METHOD']}'
    # HTTP 响应代码和消息
    status = '200 OK'
    # 客户端期望的 HTTP 标头必须将它们包装为元组对的列表：[(Header name, Header value)]
    response_headers = [('Content-type', 'text/plain'),
                        ('Content-Length', str(len(response_body)))]
    # 使用提供的函数将它们发送到服务器
    start_response(status, response_headers)
    # 返回响应主体。请注意，尽管它可以迭代，但它包装在列表中。
    return [response_body]
```

下面是两个应用对象(application object)的示例。一个是函数(function)，一个是类(class)：

```python
HELLO_WORLD = b"Hello world!\n"


def simple_app(environ, start_response):
    """最简单的应用对象"""
    status = '200 OK'
    response_headers = [('Content-type', 'text/plain')]
    start_response(status, response_headers)
    return [HELLO_WORLD]


class AppClass:
    """产生相同的输出，但是用类实现。

    (Note: 'AppClass' is the "application" here, so calling it
    returns an instance of 'AppClass', which is then the iterable
    return value of the "application callable" as required by
    the spec.

    If we wanted to use *instances* of 'AppClass' as application
    objects instead, we would have to implement a '__call__'
    method, which would be invoked to execute the application,
    and we would need to create an instance for use by the
    server or gateway.
    """

    def __init__(self, environ, start_response):
        self.environ = environ
        self.start = start_response

    def __iter__(self):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        self.start(status, response_headers)
        yield HELLO_WORLD
```

## 服务器/网关端

服务器或者网关每次从 HTTP 客户端收到一个请求，就调用一次应用对象。为了描述方便，以下是一个简单的 CGI 网关，用 Python 函数实现，接收应用对象。注意这个简单的示例在错误处理方面相当简单，因为默认情况下，未捕获的异常会被 `dump` 到 `sys.stderr`，并且被 Web 服务器记入日志。

```python
import os
import sys

enc, esc = sys.getfilesystemencoding(), 'surrogateescape'


def unicode_to_wsgi(u):
    # Convert an environment variable to a WSGI "bytes-as-unicode"
    # string
    return u.encode(enc, esc).decode('iso-8859-1')


def wsgi_to_bytes(s):
    return s.encode('iso-8859-1')


def run_with_cgi(application):
    environ = {k: unicode_to_wsgi(v) for k, v in os.environ.items()}
    environ['wsgi.input'] = sys.stdin.buffer
    environ['wsgi.errors'] = sys.stderr
    environ['wsgi.version'] = (1, 0)
    environ['wsgi.multithread'] = False
    environ['wsgi.multiprocess'] = True
    environ['wsgi.run_once'] = True

    if environ.get('HTTPS', 'off') in ('on', '1'):
        environ['wsgi.url_scheme'] = 'https'
    else:
        environ['wsgi.url_scheme'] = 'http'

    headers_set = []
    headers_sent = []

    def write(data):
        out = sys.stdout.buffer

        if not headers_set:
            raise AssertionError("write() before start_response()")

        elif not headers_sent:
            # Before the first output, send the stored headers
            status, response_headers = headers_sent[:] = headers_set
            out.write(wsgi_to_bytes('Status: %s\r\n' % status))
            for header in response_headers:
                out.write(wsgi_to_bytes('%s: %s\r\n' % header))
            out.write(wsgi_to_bytes('\r\n'))

        out.write(data)
        out.flush()

    def start_response(status, response_headers, exc_info=None):
        if exc_info:
            try:
                if headers_sent:
                    # Re-raise original exception if headers sent
                    raise exc_info[1].with_traceback(exc_info[2])
            finally:
                exc_info = None     # avoid dangling circular ref
        elif headers_set:
            raise AssertionError("Headers already set!")

        headers_set[:] = [status, response_headers]

        # Note: error checking on the headers should happen here,
        # *after* the headers are set.  That way, if an error
        # occurs, start_response can only be re-called with
        # exc_info set.

        return write

    result = application(environ, start_response)
    try:
        for data in result:
            if data:    # don't send headers until body appears
                write(data)
        if not headers_sent:
            write('')   # send headers now if body was empty
    finally:
        if hasattr(result, 'close'):
            result.close()
```

## 中间件：可以与两端交互的组件

中间件（Middleware）就是一个简单对象：既可以作为服务端角色，响应应用对象；也可以作为应用对象，与服务器交互。除此之外，还有一些其他功能：

- 重写 environ，然后基于 URL，将请求对象路由给不同的应用对象。
- 支持多个应用或者框架顺序地运行于同一个进程中。
- 通过转发请求和响应，支持负载均衡和远程处理。
- 支持对内容做后处理(postprocessing)，比如处理一个 XSL 样式表文件。


中间件的灵魂是：对 WSGI 接口的服务器/网关端和 应用/框架端是透明的，不需要其他条件。

希望将中间件合并进应用的用户，将这个中间件传递给服务器即可，就好像这个中间件是一个应用对象；或者让中间件去调用应用对象，好像这个中间件就是服务器。当然，被中间件包装(wrap)的应用对象，实际上可能是另一个包装了另一个应用的中间件，以此类推，就创建了一个中间件栈（middleware stack）。

最重要的，中间件必须同时满足服务端和应用端的限制和条件。然而，在有些情况下，中间件需要的条件比单纯的服务端或者应用端更严格，这些点会在下面予以说明。

以下是一个中间件示例。他用 Joe Strout 的 `piglatin.py` 将 `text/plain` 的响应转换成 pig latin（注意：真正的中间件应该使用更加健壮的方式——应该检查内容的类型和内容的编码，同样这个简单的例子还忽略了一个单词可能被分割到一个块边界的可能性）。

```python
from foo_app import foo_app
from piglatin import piglatin


class LatinIter:

    """Transform iterated output to piglatin, if it's okay to do so

    Note that the "okayness" can change until the application yields
    its first non-empty bytestring, so 'transform_ok' has to be a mutable
    truth value.
    """

    def __init__(self, result, transform_ok):
        if hasattr(result, 'close'):
            self.close = result.close
        self._next = iter(result).__next__
        self.transform_ok = transform_ok

    def __iter__(self):
        return self

    def __next__(self):
        if self.transform_ok:
            return piglatin(self._next())   # call must be byte-safe on Py3
        else:
            return self._next()


class Latinator:

    # by default, don't transform output
    transform = False

    def __init__(self, application):
        self.application = application

    def __call__(self, environ, start_response):

        transform_ok = []

        def start_latin(status, response_headers, exc_info=None):

            # Reset ok flag, in case this is a repeat call
            del transform_ok[:]

            for name, value in response_headers:
                if name.lower() == 'content-type' and value == 'text/plain':
                    transform_ok.append(True)
                    # Strip content-length if present, else it'll be wrong
                    response_headers = [(name, value)
                                        for name, value in response_headers
                                        if name.lower() != 'content-length'
                                        ]
                    break

            write = start_response(status, response_headers, exc_info)

            if transform_ok:
                def write_latin(data):
                    write(piglatin(data))   # call must be byte-safe on Py3
                return write_latin
            else:
                return write

        return LatinIter(self.application(environ, start_latin), transform_ok)


# Run foo_app under a Latinator's control, using the example CGI gateway
run_with_cgi(Latinator(foo_app))
```

## [wsgiref.util](https://docs.python.org/zh-cn/3.10/library/wsgiref.html#module-wsgiref.util) -- WSGI environment utilities

该模块提供了用于 WSGI 环境的各种实用程序函数。WSGI 环境是一个包含 HTTP 请求变量的字典，如 [PEP 3333](https://www.python.org/dev/peps/pep-3333) 中所述。所有带有 `environ` 参数的函数都希望提供符合 WSGI 的字典；请参阅 PEP 3333 了解详细规格。

- `wsgiref.util.guess_scheme(environ)`：通过检查 `environ` 字典中的 `HTTPS` 环境变量，返回有关 `wsgi.url_scheme` 应为 `"http"` 还是 `"https"` 的猜测。返回值是字符串。当创建包装 CGI 或类似 CGI 的协议（如 FastCGI）的网关时，此功能很有用。通常，当通过 SSL 接收到请求时，提供此类协议的服务器将包含一个 `HTTPS` 变量，其值为 `"1"`, `"yes"`, 或者 `"on"`。因此，如果找到该值，此函数将返回 `"https"`，否则返回 `"http"`。
- `wsgiref.util.application_uri(environ)`：返回应用程序的基本 URI（无 `PATH_INFO` 或 `QUERY_STRING`）
- `wsgiref.util.request_uri(environ, include_query=True)`：使用 PEP 3333 的 "URL Reconstruction" 部分中找到的算法，返回完整的请求 URI（可选包括查询字符串）。如果 `include_query` 为 `false`，则查询字符串不包含在结果 URI 中。
- `wsgiref.util.shift_path_info(environ)`：将单个名称从 `PATH_INFO` 移至 `SCRIPT_NAME`，然后返回该名称。`environ` 字典就地修改；如果您需要保留原始的 `PATH_INFO` 或 `SCRIPT_NAME`，请使用 `copy`。如果 `PATH_INFO` 中没有剩余的路径段，则返回 `None`。通常，此例程（routine）用于处理请求 URI 路径的每个部分，例如，将该路径视为一系列字典键。该例程修改了传入的环境，使其适合于调用位于目标 URI 处的另一个 WSGI 应用程序。例如，如果在 `/foo` 处有一个 WSGI 应用程序，并且请求 URI 路径是 `/foo/bar/baz`，在 `/foo` 处的 WSGI 应用程序调用 `shift_path_info` 函数，它将接收字符串 "bar"，并且环境将被更新以适合传递到 `/foo/bar/` 的 WSGI 应用程序。也就是说，`SCRIPT_NAME` 将从 `/foo` 更改为 `/foo/bar/`，`PATH_INFO` 将从 `/bar/baz` 更改为 `/baz`。当 `PATH_INFO` 只是 `"/"` 时，此例程将返回一个空字符串，并在 `SCRIPT_NAME` 后面附加一个斜杠，即使通常忽略空路径段，并且 `SCRIPT_NAME` 通常也不以斜杠结尾。这是故意的行为，以确保应用程序在使用此例程进行对象遍历时可以区分以 `/x` 结尾的 URI 与以 `/x/` 结尾的 URI 之间的区别。
- `wsgiref.util.setup_testing_defaults(environ)`：为了测试目的，使用简单的默认值更新环境。此例程添加了 WSGI 所需的各种参数，包括 `HTTP_HOST`，`SERVER_NAME`，`SERVER_PORT`，`REQUEST_METHOD`，`SCRIPT_NAME`，`PATH_INFO` 以及所有 PEP 3333 定义的 `wsgi.*` 变量。它仅提供默认值，并且不替换这些变量的任何现有设置。该例程旨在使 WSGI 服务器和应用程序的单元测试更容易设置虚拟环境。实际的 WSGI 服务器或应用程序都不应使用它，因为数据是伪造的！
用法示例：

```python
from wsgiref.util import setup_testing_defaults
from wsgiref.simple_server import make_server

# A relatively simple WSGI application. It's going to print out the
# environment dictionary after being updated by setup_testing_defaults


def simple_app(environ, start_response):
    setup_testing_defaults(environ)

    status = '200 OK'
    headers = [('Content-type', 'text/plain; charset=utf-8')]

    start_response(status, headers)

    ret = [(f"{key}: {value}\n").encode("utf-8")
           for key, value in environ.items()]
    return ret


with make_server('', 8000, simple_app) as httpd:
    print("Serving on port 8000...")
    httpd.serve_forever()
```

- `wsgiref.util.is_hop_by_hop(header_name)`：如果 `'header_name'` 是 [RFC 2616](https://tools.ietf.org/html/rfc2616.html) 定义的 HTTP/1.1 "Hop-by-Hop" Header，则返回 `True`。
- `class wsgiref.util.FileWrapper(filelike, blksize=8192)`：用于将类似文件的对象转换为迭代器的包装器（wrapper）。生成的对象支持 `__getitem __()` 和 `__iter __()` 迭代样式，以与 Python 2.1 和 Jython 兼容。当对象被迭代时，可选的 `blksize` 参数将反复传递给类似文件的对象的 `read()` 方法，以获取要产生的字节串。当 `read()` 返回一个空字节串时，迭代结束且不可恢复。如果 filelike 具有 `close` 方法，则返回的对象也将具有 `close` 方法，并且在调用时将调用该 Filelike 对象的 `close` 方法。

用法示例：

```python
from io import StringIO
from wsgiref.util import FileWrapper

# We're using a StringIO-buffer for as the file-like object
filelike = StringIO("This is an example file-like object"*10)
wrapper = FileWrapper(filelike, blksize=5)

for chunk in wrapper:
    print(chunk)
```

## [wsgiref.headers](https://docs.python.org/zh-cn/3.10/library/wsgiref.html#module-wsgiref.headers) -- WSGI response header tools

该模块提供了一个单独的类 `Headers`，用于使用类似于映射的接口方便地操作 WSGI 响应标头。

`class wsgiref.headers.Headers([headers])`：创建一个类似于映射的对象包装标头，该标头必须是标头 **名称/值** 元组的列表，如 PEP 3333 中所述。标头的默认值为空列表。标头对象支持典型的映射操作，包括 `__getitem__()`, `get()`, `__setitem__()`, `setdefault()`, `__delitem__()` 和 `__contains__()`。对于这些方法中的每一个，键都是标头名称（不区分大小写地对待），并且值是与该标头名称关联的第一个值。设置 header 会删除该 header 的所有现有值，然后在包装的 header 列表的末尾添加一个新值。通常保留 header 的现有顺序，将新的 header 添加到包装列表的末尾。

与字典不同，当您尝试获取或删除包装的标头列表中没有的键时，标头对象不会引发错误。获取不存在的标头只会返回 `None`，而删除不存在的标头则不会执行任何操作。

`Headers` 对象还支持 `keys`，`values` 和 `items` 方法。如果存在多值标头，则 `keys` 和 `items` 方法返回的列表可以多次包含同一个键。`Headers` 对象的 `len` 方法与其 `items` 方法的长度相同，也与包装的 haeder 列表的长度相同。实际上，`items` 方法仅返回包装的 header 列表的副本。