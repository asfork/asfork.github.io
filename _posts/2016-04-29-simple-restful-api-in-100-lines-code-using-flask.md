---
layout: post
title: "用 Flask 和 100 行代码写一套 REST API"
date: 2016-04-29 11:22:33 +800
categories: python
---

最近在写后台时用到了 Flask 框架。
作为 Python 刚接触不久的初学者，Flask 带来了极大的满足感。

首先是小巧简洁，对于需求只是搭建一小套 API 来说，用上全功能的 Django 框架未免太“重了。
组件搭配灵活，给开发者充分的自由度。蓝图(Blueprints)设计也提供了更大的扩展潜力。

当然你也可以把上述两条看作缺点，毕竟面对众多 Flask 模块也有一定的试错成本。
另外 Flask 的中文资料十分稀少且落后，一定要阅读英文文档。

本文涉及的主题并不高深，仅仅是强调减轻开发负担的最佳实践和模式。尽量避免啰嗦官方文档中提到的内容。如果你想尝试它们，一定会有对阅读官方文档的急迫需求（这不错吧？）。

## 组织模式
*单一模块*
由于项目功能比较简单，只是实现一套小型 API，大部分代码都放到了单一文件 `app.py` 中。对于一些微项目来说这恰到好处，毕竟只需要处理几个路由（route）并且只有百来行代码。

整个项目的结构看起来像这样：

```
app.py
config.py
schedule.py
requirements.txt
static/
templates/
```

如果需要开发一个更加复杂的项目，可以考虑采用以包或者蓝图(Blueprints)的方式组织你的项目。

## 模块 Modules
我的项目 `requirements.txt` 是这个样子：

```
Flask==0.10.1
Flask-RESTful==0.3.5
Flask-SQLAlchemy==2.1
Flask-Marshmallow==0.6.2
Marshmallow-Sqlalchemy==0.8.1
PyMySQL==0.7.2
Gunicorn==19.4.5
APScheduler==3.0.5
```

### [Flask](http://flask.pocoo.org/)
Flask 不必多说，当前的最新版本 0.10.1。同时支持 Python 3 和 Python 2。不过 Armin Ronacher, Flask 的作者原话如下：

> 我自己现在并不用它，我也不会向别人推荐自己都不相信的东西，所以我不会推荐 Python 3.
如果你打算将 Flask 应用到生产环境并且不想面对开发了几个月后发现自己不能使用包 X, Y, Z 的话，还是采用 Python2 环境比较好。
但是，秉着“要啥自行车，先整个最新的”态度，我还是使用了 Pyhton 3...  因为项目用到的几个模块都已经官方支持 Python 3。

### [Flask-RESTful](http://flask-restful-cn.readthedocs.io/zh/latest/)
Flask-RESTful 是 FLask 的一个扩展，使用它能快速的构建 REST APIs。同类型的模块还有好几个，都是帮助你少写一些重复代码，我随便挑了一个。

官方文档提供了中英双版本，勉强算一个优点吧。

### [Flask-SQLAlchemy](http://flask-sqlalchemy.pocoo.org/), [Flask-Marshmallow](https://flask-marshmallow.readthedocs.io/) and [Marshmallow-Sqlalchemy](https://marshmallow-sqlalchemy.readthedocs.io)
这三个放在一起讲不是因为不重要，恰恰相反，它们搭配起来能大大简化数据的持久化、序列化等工作。三者联系紧密，你一定不想错过它们。

*Flask-SQLAlchemy*
SQLAlchemy 是 Flask 上使用最广泛的 ORM 模块。官方更新勤快，功能强大，性能稳定优宜。你还有什么理由不用它？
Flask-SQLAlchemy 为 Flask 提供了扩展支持，使你更轻松的使用 SQLAlchemy。它提供了对 SQLAlchemy 的一些默认配置，帮助你少写些代码。

*Flask-Marshmallow*

Marshmallow 一个轻量级的序列化/反序列化库，用于将复杂对象与 Python 数据类型相互转换。
而 Flask-Marshmallow 增加了一些扩展特性，包括支持 HATEOAS-ready APIs 的 `URL` 和 `Hyperlinks` fields(不知这个词该怎么翻译...)。

*Marshmallow-Sqlalchemy*

Marshmallow-Sqlalchemy 能帮 Sqlalchemy 自动生成 Marshmallow 视图。同样也是为了少写些代码(懒人的境界)。

### [PyMySQL](https://pypi.python.org/pypi/PyMySQL)
Python 实现的 Mysql 的驱动。
Flask 支持绝大部分主流数据库，关系型数据库 MySQL, PostgreSQL等等，非关系型数据库如 MongoDB, Redis 等等。

我这里使用了 Mysql。

### [Gunicorn](http://gunicorn.org/)
虽然 Flask 自带着 WSGI server, 但是性能不太好，更多的是测试用途。发布时则使用生产环境的 WSGI server 或者是联合 Nginx 做 Uwsgi 。

Gunicorn 是一个 Python WSGI UNIX 的 HTTP 服务器。与各种Web框架兼容，只要简单配置执行，轻量级的资源消耗，就能运行的相当迅速。

下面是我用到的网络框架模型(图片来源：[峰云，就她了](http://www.tuicool.com/articles/aiami2))，由 Nginx pass_proxy 到 app 的前端口，然后用 Gunicorn 来协同处理 。
Network framework

### [APScheduler](https://apscheduler.readthedocs.io)
APScheduler 定时任务框架，实现了Quartz的所有功能。提供了基于日期、固定时间间隔以及 Crontab 类型的任务，并且可以持久化任务。

我用来跑一些定时任务，发统计数据到 Slack, 导出备份数据库，启动爬虫之类。

## 代码
Talk is cheap 对不对，最后把 `app.py` 文件贴出来。
其实业务非常简单，仔细阅读过官方文档的话理解起来很轻松。

Gist: [app.py](https://gist.github.com/uhy/2037f8bb89682d31dd1f02a4203618a1)
