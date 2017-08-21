# 请求的生命周期

- [简介](#introduction)
- [生命周期概要](#lifecycle-overview)
- [专注于服务提供者](#focus-on-service-providers)

<a name="introduction"></a>
## 简介

使用「现实世界」中的任何工具时，若能了解它的运作原理，你将会更有信心来用好它。开发应用程序也一样。当你明白开发工具的运作方式时，使用它们将会感到更舒适、更有信心。

这份文档的主要目的在于为你讲解 Laravel 框架是如何「运作」的。当你对整个框架愈加了解时，有些事情便不会再那么「神奇」，也会使你在创建应用程序时更具信心。

若你目前还无法了解所有的术语，请不要灰心！只要对现在提到的东西有个基本概念，你的知识将在后面随着你对这份文档和其它章节的探索而成长。

<a name="lifecycle-overview"></a>
## 生命周期概要

### 首要之事

`public/index.php` 这个文件是 Laravel 应用程序所有请求的进入点。所有的请求都通过你的网页服务器（Apache / Nginx）的设置导向这个文件。`index.php` 这个文件并没有太多的代码。更确切地说，它只是个起始点，用来加载框架的其它部分。

`index.php` 此文件会加载由 Composer 生成的自动加载器定义，并获取由 `bootstrap/app.php` 文件中所生成的 Laravel 应用程序实例。Laravel 自身的第一个动作就是创建一个应用程序／ [服务容器](/docs/{{version}}/container) 的实例。

### HTTP / 终端核心

接下来，进入应用程序的请求的会被送往 HTTP 核心或终端核心，视该请求的种类而定。这两种核心是所有请求流向的中心位置。现在开始，我们只将焦点放在 HTTP 核心，它位于 `app/Http/Kernel.php`。

HTTP 核心扩展了 `Illuminate\Foundation\Http\Kernel` 类，它定义了一个 `bootstrappers` 数组，在请求被运行前会先行运作。这些启动器设置了错误处理、日志记录、[侦测应用程序环境](/docs/{{version}}/installation#environment-configuration)，并运行其它需要在请求实际处理前就该被完成掉的工作。

HTTP 核心也定义了一份 HTTP [中间件](/docs/{{version}}/middleware) 清单，所有的请求在被应用程序处理之前都必须经过它们。这些中间件处理 [HTTP session](/docs/{{version}}/session) 的读写、[验证 CSRF 令牌](/docs/{{version}}/routing#csrf-protection)、决定应用程序是否处于维护模式，以及其它更多任务作。

HTTP 核心 `handle` 方法的方法签章相当简单：接收一个 `Request` 并返回一个 `Response`。把核心想像成一个大的黑盒子，代表你完整的应用程序。喂给它 HTTP 请求，它就会传回 HTTP 响应。

#### 服务提供者

最重要的核心启动加载行为之一，是加载你的应用程序的 [服务提供者](/docs/{{version}}/providers)。应用程序的所有服务提供者，都在 `config/app.php` 此配置文件的 `providers` 数组中被设置。首先，所有提供者的 `register` 方法会被调用，一旦所有提供者都被注册之后，`boot` 方法就会被调用。

服务提供者负责在启动时加载框架的所有组件，例如数据库、队列、验证、以及路由组件。服务提供者启动加载并设置框架提供的各种功能，是整个 Laravel 启动加载过程最为重要的部分。

#### 请求分派

一旦应用程序被启动且所有的服务提供者都被注册之后，`Request` 将被移转给路由器进行分派。路由器会将请求分派给路由或控制器，并运行所有特定路由的中间件。

<a name="focus-on-service-providers"></a>
## 专注于服务提供者

服务提供者是启动 Laravel 应用程序的真正关键。应用程序的实例被创建、服务提供者被注册、请求被移转至已启动的应用程序。真的就是这么简单！

真正掌握 Laravel 应用程序是如何创建并通过服务提供者启动，将是很有价值的。当然，你的应用程序默认的服务提供者存放在 `app/Providers` 此一目录下。

默认情况下，`AppServiceProvider` 几乎是空的。要加入你应用程序自己的启动及服务容器绑定，此提供者是一个很好的地方。当然，对大型应用程序而言，你可能希望创建若干个服务提供者，每一个都具备更精细的启动类型。


