# 请求生命周期

- [简介](#introduction)
- [生命周期概述](#lifecycle-overview)
- [聚焦服务提供者](#focus-on-service-providers)

<a name="introduction"></a>
## 简介

当使用「真实世界」中的任何工具时, 如果你能理解它的运作原理，你会更加自信。应用开发也没有什么不同。当你理解了你的开发工具是如何运作的之后，你会用得更舒服、更自信。

本文档的目标是给你一个良好、高层次的、关于Laravel框架如何「运作」的概述。通过对整体框架的更好理解，任何事情都不再会显得那么「神奇」, 你在构建应用时也会更加自信。

如果你现在还无法理解所有的条目，别灰心！ 只要尝试掌握接下来提到的内容，你的知识就会随着你对文档其他章节的探索而增长。

<a name="lifecycle-overview"></a>
## 生命周期概述

### 首要之事

Laravel 应用的所有请求的入口都是 `public/index.php` 文件，所有的请求都被你的 web 服务器 (Apache / Nginx) 配置定向到该文件。 `index.php` 文件没有包含太多代码，它只是一个加载框架其他部分的起点。

 `index.php` 文件加载 Composer 生成的自动加载定义，并且从 `bootstrap/app.php` 脚本中获取一个 Laravel 应用实例。Laravel 自身的第一个动作就是创建一个应用实例 / [服务容器](/docs/{{version}}/container).

### HTTP / 终端核心

接下来，进来的请求会被发送到 HTTP 核心或者终端核心，这取决于进入应用的请求类型。 这两种核心作为所有请求流过的中心。现在开始，让我们专注于 HTTP 核心，它位于 `app/Http/Kernel.php` 。

HTTP 核心继承自 `Illuminate\Foundation\Http\Kernel` 类， 它定义了一个将在请求执行前被运行的 `bootstrappers` 数组。 bootstrappers 配置了错误处理、日志、 [检测应用环境](/docs/{{version}}/installation#environment-configuration)，并执行其他需要在请求被真正处理前需要完成的工作。

HTTP 核心也定义了一系列 HTTP [中间件](/docs/{{version}}/middleware)，所有请求被应用处理前都必须经过它们。这些中间件处理读写 [HTTP session](/docs/{{version}}/session)、检测应用是否处于维护模式、 [验证 CSRF 令牌](/docs/{{version}}/routing#csrf-protection) 以及更多其他任务。

HTTP 核心的 `handle` 方法的语法相当简单： 获取一个 `Request` 并返回一个 `Response`。把核心想象成一个代表你整个应用的大黑盒子，喂给它 HTTP 请求，就会吐出 HTTP 响应。

#### 服务提供者

最重要的核心启动动作之一是为你的应用加载 [服务提供者](/docs/{{version}}/providers)。 应用中所有的服务提供者都配置在 `config/app.php` 配置文件的 `providers` 数组里。首先，所有服务提供者的 `register` 将被调用, 紧接着，一旦所有的服务提供者被注册，`boot` 方法将被调用。

服务提供者负责启动框架的全部各种组件，例如数据库、队列、验证和路由组件等。 由于服务提供者启动并配置了框架提供的各个功能, 它们是整个 Laravel 启动过程中最重要的部分。

#### 分发请求

一旦应用被启动且所有的服务提供者被注册， `Request` 被转交给路由器进行分发。 路由器将把请求分发到一个路由或者控制器，也会运行任何路由特有的中间件。

<a name="focus-on-service-providers"></a>
## 聚焦服务提供者

服务提供者是启动 Laravel 应用的关键。应用实例被创建、服务提供者被注册、请求被转交到已启动的应用。 真的就是那么简单！

对 Laravel 应用是如何通过服务提供者进行创建和启动有一个牢固的理解是非常有价值的。当然，应用的默认服务提供者存储在 `app/Providers` 目录。

默认情况下， `AppServiceProvider` 几乎是空的。 这个提供者是添加你自己的启动和服务容器绑定的好地方。 当然，对于大型应用，你可能希望创建多个服务提供者, 每个都有更精细的启动类型。
