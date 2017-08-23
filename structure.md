# Laravel 的文件夹结构

- [简介](#introduction)
- [根目录](#the-root-directory)
    - [ `app` 目录](#the-root-app-directory)
    - [ `bootstrap` 目录](#the-bootstrap-directory)
    - [ `config` 目录](#the-config-directory)
    - [ `database` 目录](#the-database-directory)
    - [ `public` 目录](#the-public-directory)
    - [ `resources` 目录](#the-resources-directory)
    - [ `routes` 目录](#the-routes-directory)
    - [ `storage` 目录](#the-storage-directory)
    - [ `tests` 目录](#the-tests-directory)
    - [ `vendor` 目录](#the-vendor-directory)
- [ App 目录](#the-app-directory)
    - [ `Console` 目录](#the-console-directory)
    - [ `Events` 目录](#the-events-directory)
    - [ `Exceptions` 目录](#the-exceptions-directory)
    - [ `Http` 目录](#the-http-directory)
    - [ `Jobs` 目录](#the-jobs-directory)
    - [ `Listeners` 目录](#the-listeners-directory)
    - [ `Mail` 目录](#the-mail-directory)
    - [ `Notifications` 目录](#the-notifications-directory)
    - [ `Policies` 目录](#the-policies-directory)
    - [ `Providers` 目录](#the-providers-directory)
    - [ `Rules` 目录](#the-rules-directory)

<a name="introduction"></a>
## 简介

`Laravel` 默认的目录结构意在为不同大小的应用提供一个好的起点。当然，你可以按照喜好整理自己的应用目录结构。`Laravel` 没有严格地限制何处的类能被加载 —— 只要 `Composer` 可以自动加载它们即可。

#### 为什么没有 `Models` 目录?

许多初学者都因为找不到 `models` 目录而困惑。其实，这是 Laravel 有意为之，因为 `models` 这个词对不同的人有不同的理解。一些开发者把应用的模型当做它的业务逻辑，另一些人把它用来与关系型数据库交互。

因此，我们默认把 Eloquernt 的模型放在 `app` 目录下，并且允许开发者自行选择放置在何处。

<a name="the-root-directory"></a>
## 根目录

<a name="the-root-app-directory"></a>
#### `app` 目录

如你所料，`app` 目录包含应用程序的核心代码。你编写的所有的类都应该放在这里。稍后我们会深入地了解这个目录的更多细节。

<a name="the-bootstrap-directory"></a>
#### `Bootstrap` 目录

`bootstrap` 目录包含了框架的启动文件和配置好的自动加载文件。里面也包含一个 `cache` 目录，存放着框架生成的用来提升性能的文件，比如路由和服务缓存文件。

<a name="the-config-directory"></a>
#### `Config` 目录

`config` 目录，顾名思义，包含应用程序的所有配置文件。通读这些文件能帮你熟悉所有的可用选项。

<a name="the-database-directory"></a>
#### `Database` 目录

`database` 目录包含数据填充和迁移文件。你还可以把它作为 `SQLite` 数据库的数据存储目录。

<a name="the-public-directory"></a>
#### `Public` 目录

`public` 目录包含了入口文件 `index.php`，它是所有访问你的应用程序的入口文件。一些资源文件（如图片、JavaScript 和 CSS）也在这里。

<a name="the-resources-directory"></a>
#### `Resources` 目录

`resource` 目录包含了视图和原始资源文件（例如没有压缩过的 LESS、SASS 和 JavaScript）和语言包。

<a name="the-routes-directory"></a>
#### `Routes` 目录

`routes` 目录包含了应用所有路由定义，Laravel 默认包含了几个路由文件：`web.php`, `api.php`, `console.php` 和 `channels.php`。

`web.php` 里定义的路由都会被 `RouteServiceProvider` 包含在 `web` 中间件组里，具备 session、CSRF 防护和 cookie 加密功能。如果你的应用无需提供无状态的、RESTful 风格的 API，所有的路由定义都应该定义在 `web.php` 中。

`api.php` 里定义的路由都会被 `RouteServiceProvider` 包含在 `api` 中间件组里，具备频率限制功能。这些路由都是无状态的，所以通过这里的请求应该进行令牌认证，并且它们不能访问 session 状态。

`console.php` 定义所有基于闭包的控制台命令。每个闭包都被绑定到一个命令实例并且允许和命令行 IO 方法进行简单的交互。尽管这些文件不能定义 `HTTP` 路由，但它定义了基于命令行的应用入口。

`channels.php` 用来注册你的应用支持的所有的广播事件。

<a name="the-storage-directory"></a>
#### `Storage` 目录

`storage` 目录包含编译后的 `Blade` 模板文件、基于文件的 session 和 缓存、以及框架生成的其他文件。这个目录包含 `app `、 `framework` 和 `logs` 三个子目录。`app` 目录可以用来存储应用的任何文件。`framework` 目录用来存储框架生成的文件和缓存。`logs` 目录包含一些日志文件。

`storage/app/public` 可以用来存储用户生成的文件，比如需要公开访问的用户头像。你可以创建一个 `public/storage` 的软连接指向这个目录。并且，你可以直接通过 `php artisan storage:link` 来创建此连接。

<a name="the-tests-directory"></a>
#### `Tests` 目录

`tests` 目录存放你的自动化测试文件。Laravel 推荐了一个 [PHPUnit](https://phpunit.de/) 的例子可以直接参考。每个测试用例都应该以 `Test` 作为后缀。你可以使用 `phpunit` 或者 `php vendor/bin/phpunit` 来运行测试。

<a name="the-vendor-directory"></a>
#### `Vendor` 目录

`vendor` 目录包含了你的 [Composer](https://getcomposer.org) 的依赖包。

<a name="the-app-directory"></a>
## `App` 目录

应用程序的核心代码位于 `app` 目录内，默认情况下，这个目录被包含在命名空间 `App` 里并且被 Composer 按照 [PSR-4 自动载入标准](http://www.php-fig.org/psr/psr-4/) 自动加载。

`app` 目录包含了多个可选的子目录，比如 `Console`、`Http` 和 `Providers` 等。其中 `Console` 和 `Http` 为应用提供了入口 API 。其实 HTTP 协议和 CLI 都只是应用的交互方式，而不包含应用的逻辑。也就是说，它们只是两种发送命令给应用的方式。`Console` 目录里包含了所有的 Artisan 命令，`Http` 目录包含了应用的控制器、中间件和请求。

`app` 里其它可选的子目录都可以通过 Artisan 提供的 make 命令来生成。例如， `app/Jobs` 只有在你执行 `make:job` 命令来生成任务类时才会出现。

> {tip} `app` 目录里的许多类都可以通过 `Artisan` 命令来生成。要查看所有的可用命令，可以在终端里运行 `php artisan list make` 命令。

<a name="the-console-directory"></a>
#### `Console` 目录

`Console` 目录包含了所有自定义的 Artisan 命令。这些命令可以通过 `make:command` 来生成。同时，这个目录还包含了 Console Kernel，可以用来注册你的自定义命令和你定义的 「[计划任务](/docs/{{version}}/scheduling)」。

<a name="the-events-directory"></a>
#### `Events` 目录

`Events` 目录默认是不存在的，它会在你运行 `event:generate` 或 `event:make` 时生成。如你所见，`Events` 目录存放了 「[事件类](/docs/{{version}}/events)」。事件类用于在一些指定的动作被触发时通知其他部分，它为应用程序提供了一种需要灵活性和解耦的方式。

<a name="the-exceptions-directory"></a>
#### `Exceptions` 目录

`Exceptions` 目录包含了应用的异常处理器，并且是处理应用异常的好地方。如果你想自定义异常的记录和渲染方式，你可以修改此目录下的 Handler 类。

<a name="the-http-directory"></a>
#### `Http` 目录

`Http` 目录包含了控制器、中间件和表单请求。几乎所有的请求处理逻辑都应该放在这里。

<a name="the-jobs-directory"></a>
#### `Jobs` 目录

`Jobs` 目录默认是不存在的，它会在你运行 `make:job` 命令时生成。这个目录存放了应用中的「[队列任务](/docs/{{version}}/queues)」。应用的任务可以被推送到队列，或者在当前请求的生命周期同步的运行。在该请求的生命周期同步运行的任务可以看做是一个命令，因为它们实现了「[命令模式](https://en.wikipedia.org/wiki/Command_pattern)」。

<a name="the-listeners-directory"></a>
#### `Listeners` 目录

`Listeners` 目录默认不存在，它会在你运行 `event:generate` 或 `make:listenr` 命令时生成。`Listeners` 目录包含了用来处理「[事件](/docs/{{version}}/events)」的类。事件监听器接受一个事件实例并实现该事件触发后的响应逻辑。例如，`UserRegistered` 事件可以被监听器 `SendWelcomeEmail` 处理。

<a name="the-mail-directory"></a>
#### `Mail` 目录

`Mail` 目录默认不存在，它会在你运行 `make:mail` 命令时生成。`Mail` 目录包含所有的邮件发送类。邮件对象允许你在一个地方构建邮件所需的所有逻辑，然后使用 `Mail::send` 方法来发送邮件。

<a name="the-notifications-directory"></a>
#### `Notifications` 目录

`Notifications` 目录默认不存在，它会在你运行 `make:notification` 命令后生成。`Notifications` 目录包含应用发送的所有通知，比如事务发送的通知。Laravel 提供了抽象的通知接口，你可以使用不同的驱动来发送通知，例如邮件、Slack、短信或是存储在数据库。

<a name="the-policies-directory"></a>
#### `Policies` 目录

`Policies` 目录可以通过运行 `make:policy` 来创建。`Policies` 目录包含了应用的授权策略。授权策略可以用来决定一个用户是否有权限去访问指定资源。更多详情可以参考「[授权系统](/docs/{{version}}/authorization)」。

<a name="the-providers-directory"></a>
#### `Providers` 目录

`Providers` 目录包含了应用的「[服务提供者](/docs/{{version}}/providers)」。服务提供者在应用启动的时候绑定服务到服务容器、注册事件、以及一些其他的任务来为即将到来的请求做准备。

在新安装的 Laravel 应用里，这个目录已经包含了一些服务提供者。你可以按照需要把自己的服务提供者添加到该目录。

<a name="the-rules-directory"></a>
#### `Rules` 目录

`Rules` 目录默认不存在，它会在运行 `make:rule` 命令时被创建。`Rules` 目录包含应用自定义的验证规则对象。这些规则意在使用简单的对象来包含复杂的验证逻辑。更多详情可以参考「[验证机制详解](/docs/{{version}}/validation)」。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@huiren](https://laravel-china.org/users/5583)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5583_1472366142.png?imageView2/1/w/100/h/100">  |  翻译  | 认领翻译，只是起点。 |