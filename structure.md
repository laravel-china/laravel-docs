# 应用程序结构

- [简介](#introduction)
- [根目录](#the-root-directory)
- [App 目录](#the-app-directory)
- [为应用程序设置命名空间](#namespacing-your-application)

<a name="introduction"></a>
## 简介

默认的 Laravel 应用程序结构意在给不同大小的应用程序提供一个好的起始点。当然，你也可以依照个人喜好来随意组织应用程序。Laravel 几乎没有对任何类的放置位置进行强加限制 - 只要 Composer 可以自动加载这些类即可。

<a name="the-root-directory"></a>
## 根目录

一个新安装的 Laravel 根目录包含许多个文件夹：

`app` 目录，如你所料，这里面包含应用程序的核心代码。我们之后将很快对这个目录的细节进行深入探讨。

`bootstrap` 目录包含了几个框架启动跟自动加载设置的文件。以及在 `cache` 文件夹中包含着一些框架在启动性能优化时所生成的文件。

`config` 目录，顾名思义，包含所有应用程序的配置文件。

`database` 目录包含数据库迁移与数据填充文件。如果你愿意的话，你也可以在此文件夹存放 SQLite 数据库。

`public` 目录包含了前端控制器和资源文件（图片、JavaScript、CSS，等等）。

`resources` 目录包含了视图、原始的资源文件 (LESS、SASS、CoffeeScript) ，以及语言包。

`storage` 目录包含编译后的 Blade 模板、基于文件的 session、文件缓存和其它框架生成的文件。此文件夹分格成 `app`、`framework`，及 `logs` 目录。`app` 目录可用于存储应用程序使用的任何文件。`framework` 目录被用于保存框架生成的文件及缓存。最后，`logs` 目录包含了应用程序的日志文件。

`tests` 目录包含自动化测试。这有一个现成的 [PHPUnit](https://phpunit.de/) 例子。

`vendor` 目录包含你的 [Composer](https://getcomposer.org) 依赖模块。

<a name="the-app-directory"></a>
## App 目录

应用程序的「内容」存在于 `app` 目录中。默认情况下，这个目录在 `App` 命名空间下借助 Composer 使用 [PSR-4 自动加载标准](http://www.php-fig.org/psr/psr-4/)自动加载。**你可以使用 `app:name` Artisan 命令修改这个命名空间**。

`app` 目录附带许多个额外的目录，例如：`Console`、`Http` 和 `Providers`。可以将 `Console` 和 `Http` 目录试想为提供 API 进入应用程序的「核心」。HTTP 协定和 CLI 都是跟应用程序进行交互的机制，但实际上并不包含应用程序逻辑。换句话说，它们是两种简单地发布命令给应用程序的方法。`Console` 目录包含你全部的 Artisan 命令，而 `Http` 目录包含你的控制器、中间件和请求。

`Jobs` 目录用于放置应用程序 [可队列的任务](/docs/{{version}}/queues)。任务可以被应用程序放到队列中，也可以在当前请求生命周期内同步运行。

`Events` 目录，如你所料，是用来放置 [事件类](/docs/{{version}}/events) 的。事件可以被用于当指定动作发生时，通知你应用程序的其它部分，提供了很棒的灵活性及解耦。

`Listeners` 目录了包含事件的处理类。处理进程接收一个事件，并针对该事件运行逻辑。例如，`UserRegistered` 事件可能由 `SendWelcomeEmail` 侦听器处理。

`Exceptions` 目录包含应用程序的异常处理进程，同时也是个处置应用程序抛出异常的好位置。

> **注意：**在 `app` 目录中的许多类可以通过 Artisan 命令生成。若要查看可以使用的命令，只要在命令行运行 `php artisan list make` 命令即可。

<a name="namespacing-your-application"></a>
## 为应用程序设置命名空间

如前面所提到的，默认的应用程序命名空间为 `App`；然而，你可以简单地借由 `app:name` Artisan 命令，将此命名空间修改成应用程序的名称。例如：如果你的应用程序叫做「SocialNet」，你可以运行下方的命令来进行更改：

    php artisan app:name SocialNet

当然，你也可以接着使用 `App` 命名空间。


