# Laravel 的请求生命周期

- [简介](#introduction)
- [生命周期概述](#lifecycle-overview)
- [聚焦服务提供器](#focus-on-service-providers)

<a name="introduction"></a>
## 简介

在日常生活中使用任何工具时，如果理解了该工具的工作原理，使用时能更加运用自如。这对于应用开发来说也一样，当你能真正懂得一个功能背后实现原理时，你就离成为大神不远了。

文档存在目的是为了让你更加清晰地了解 Laravel 框架是如何工作。通过更好地了解整个框架，让一切都不再感觉很「神奇」。相信我，这有助于你更加清楚自己在做什么，对自己想做的事情更加胸有成竹。就算你不明白所有的术语，也不用因此失去信心！只要多一点尝试、学着如何运用，随着你浏览文档的其他部分，你的知识一定会因此增长。

<a name="lifecycle-overview"></a>
## 生命周期概述

### 开始

`public/index.php` 文件是所有对 Laravel 应用程序的请求的入口点。而所有的请求都是经由你的 Web 服务器（Apache/Nginx）通过配置引导到这个文件。`index.php` 文件不包含太多的代码，却是加载框架的起点。

`index.php` 文件加载 Composer 生成定义的自动加载器，然后从 `bootstrap/app.php` 脚本中检索 Laravel 应用程序的实例。Laravel 本身采取的第一个动作是创建一个 application/ [service container](/docs/{{version}}/container) 的实例。

### HTTP／控制器内核

接下来，根据进入应用程序的请求类型来将传入的请求发送到 HTTP 内核或控制台内核。而这两个内核是用来作为所有请求都要通过的中心位置。现在，我们先看看位于 `app/Http/Kernel.php` 中的 HTTP 内核。

HTTP 内核继承了 `Illuminate\Foundation\Http\Kernel` 类，它定义了在执行请求之前运行的 `bootstrappers` 数组。这个数组负责在实际处理请求之前完成这些内容：配置错误处理、配置日志记录、[检测应用程序环境](/docs/{{version}}/configuration#environment-configuration) 以及执行其他需要完成的任务。

HTTP 内核还定义了所有请求被应用程序处理之前必须经过的 HTTP 中间件的列表。这些中间件处理 [HTTP 会话](/docs/{{version}}/session) 的读写、确定应用程序是否处于维护模式、[验证 CSRF 令牌](/docs/{{version}}/csrf)等。

HTTP 内核的 `handle` 方法的方法签名非常简单：接收 `Request` 并返回 `Response`。可以把内核当作是代表整个应用程序的大黑盒，给它 HTTP 请求，它就返回 HTTP 响应。

#### 服务提供器

最重要的内核引导操作之一是加载应用程序的 [服务提供器](/docs/{{version}}/providers)。应用程序的所有服务提供器都在 `config/app.php` 配置文件的 `providers` 数组中配置。首先，所有提供器都会调用 `register` 方法，接着，由 `boot` 方法负责调用所有被注册提供器。

服务提供器负责引导所有框架的各种组件，如数据库、队列、验证和路由组件。也就是说，框架提供的每个功能都它们来引导并配置。因此也可以说，服务提供器是整个 Laravel 引导过程中最重要的方面。

#### 分配请求

一旦引导了应用程序且注册所有服务提供器，`Request` 请求就会被转交给路由器来进行调度。路由器将请求发送到路由或控制器或任何运行于路由的特定中间件。

<a name="focus-on-service-providers"></a>
## 聚焦服务提供器

服务提供器是引导 Laravel 应用程序真正的关键。创建应用程序实例、注册服务提供器，并将请求交给被引导的应用程序。就是这么简单～

牢牢掌握 Laravel 应用程序如何通过服务提供器来构建和引导是非常有价值的。应用程序的默认服务提供器存储在 `app/Providers` 目录中。

默认情况下，`AppServiceProvider` 没什么内容。这个提供器是用来添加自定义的应用程序引导和服务容器绑定。对于大型应用程序来说，可以创建几个服务提供器，让每个服务提供器都具有更精细的引导类型。

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  翻译  | Stay Hungry. Stay Foolish. |


---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
