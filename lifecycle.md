# Laravel 的请求生命周期

- [介绍](#introduction)
- [请求生命周期概述](#lifecycle-overview)
- [聚焦服务提供者](#focus-on-service-providers)

<a name="introduction"></a>
## 介绍

当使用任何「现实世界」中的任何工具时，如果理解这个工具的运作原理，那么你会更加得心应手的使用这个工具。应用开发也是这样。当你明白你的开发工具如何运行的，你就会对它们的使用用游刃有余。

这篇文档的目的是让你更好的理解 Laravel 框架如何进行工作及它的工作原理。通过对框架进行全面的了解，一切都不会那么「神奇」，也将会让你更有自信的构建你的应用。如果你不能理解所有的这些术语，也不要丧失信心！只要对现在提到的东西有个基本概念，随着对本文档和其他章节的不断探索，你对它们的理解会不断提升。

<a name="lifecycle-overview"></a>
## 生命周期概述

### 第一件事

一个 Laravel 应用的所有请求的入口都是 `public/index.php` 文件。 通过网页服务器 (Apache / Nginx) 所有请求都会导向这个文件。 `index.php` 文件没有太多的代码，只是加载框架其他部分的一个起点。

`index.php` 文件载入 Composer 生成的自动加载器定义，并从 `bootstrap/app.php` 脚本获取到 Laravel 应用实例。Laravel 的第一个动作就是创建一个自身应用实例 / [服务容器](/docs/{{version}}/container)。

### HTTP / Console 内核

接下来，传入的请求会被发送给 HTTP 内核或者 console 内核，这根据进入应用的请求的类型而定。这两个内核服务是所有请求都经过的中枢。现在，让我们只关注位于 `app/Http/Kernel.php` 的 HTTP 内核。

HTTP 内核继承自 `Illuminate\Foundation\Http\Kernel` 类，它定义了一个 `bootstrappers` 数组，数组中的类在请求真正执行前进行前置执行。 这些引导程序配置了错误处理，日志记录，[检测应用程序环境](/docs/{{version}}/configuration#environment-configuration)，以及其他在请求被处理前需要完成的工作。

HTTP 内核同时定义了一个 HTTP [中间件](/docs/{{version}}/middleware) 列表，所有的请求必须在处理前通过这些中间件，这些中间件处理 [HTTP session](/docs/{{version}}/session) 的读写，判断应用是否在维护模式， [验证 CSRF token](/docs/{{version}}/csrf) 等等。

HTTP 内核的标志性 `handle` 方法是相当简单的：接收一个 `Request` 并返回一个 `Response`。你可以把内核想成一个代表你应用的大黑盒子。给它喂 HTTP 请求然后它就会吐给你 HTTP 响应。

#### 服务提供者

在内核引导启动的过程中最重要的动作之一就是载入 [服务提供者](/docs/{{version}}/providers) 到你的应用。所有的服务提供者都配置在 `config/app.php` 文件中的 `providers` 数组中。 首先，所有提供者的 `register` 方法会被调用，接下来，一旦所有提供者注册完成，`boot` 方法将会被调用。

服务提供者负责引导启动框架的全部各种组件，例如数据库、队列、验证器、以及路由组件。因为这些组件引导和配置了框架的各种功能，所以服务提供者是整个 Laravel 启动过程中最为重要的部分。

#### 分发请求

一旦应用完成引导和所有服务提供者都注册完成，`Request` 将会移交给路由进行分发。路由将分发请求给一个路由或控制器，同时运行路由指定的中间件。

<a name="focus-on-service-providers"></a>
## 聚焦服务提供者

服务提供者是 Laravel 应用的真正关键部分，应用实例被创建后，服务提供者就会被注册完成，并将请求传递给应用进行处理，真的就是这么简单！


了解 Laravel 是怎样通过服务提供者构建和引导一个稳定的应用是非常有价值的，当然，应用的默认服务提供者都存放在 `app/Providers` 目录中。

在新创建的应用中，`AppServiceProvider` 文件中方法实现都是空的。这个提供者是你添加应用专属的引导和服务的最佳位置，当然的，对于大型应用你可能希望创建几个服务提供者，每个都具有粒度更精细的引导。


## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@麦索](https://github.com/dongm2ez)  | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/9032795?v=3&s=460?imageView2/1/w/100/h/100">  |  翻译  | Follow me [@dongm2ez](https://github.com/dongm2ez) at Github
