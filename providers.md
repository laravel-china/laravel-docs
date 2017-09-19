# Laravel 的服务提供器

- [简介](#introduction)
- [编写服务提供器](#writing-service-providers)
    - [注册方法](#the-register-method)
    - [引导方法](#the-boot-method)
- [注册提供器](#registering-providers)
- [延迟提供器](#deferred-providers)

<a name="introduction"></a>
## 简介

服务提供器是所有 Laravel 应用程序引导中心。你的应用程序以及 Laravel 的所有核心服务都是通过服务提供器进行引导。

在这里，我们说的「引导」其实是指**注册**，比如注册服务容器绑定、事件监听器、中间件，甚至是路由的注册。服务提供器是配置你的应用程序的中心。

Laravel 的 `config/app.php` 文件中有一个 `providers` 数组。数组中的内容是应用程序要加载的所有服务提供器类。这其中有许多提供器并不会在每次请求时都被加载，只有当它们提供的服务实际需要时才会加载。这种我们称之为「延迟」提供器。

本篇将带你学习如何编写自己的服务提供器，并将其注册到 Laravel 应用程序中。

<a name="writing-service-providers"></a>
## 编写服务提供器

所有服务提供器都会继承 `Illuminate\Support\ServiceProvider` 类。大多数服务提供器都包含 `register` 和 `boot` 方法。在 `register` 方法中，你**只需要绑定类到 [服务容器](/docs/{{version}}/container)**中。而不需要尝试在 `register` 方法中注册任何事件监听器、路由或任何其他功能。

使用 Artisan 命令行界面，通过 `make:provider` 命令生成一个新的提供器：

    php artisan make:provider RiakServiceProvider

<a name="the-register-method"></a>
### 注册方法

在 `register` 方法中，你只需要将类绑定到 [服务容器](/docs/{{version}}/container) 中。而不需要尝试在 `register` 方法中注册任何事件监听器、路由或者任何其他功能。否则，你可能会意外使用到尚未加载的服务提供器提供的服务。

让我们来看一个基本的服务提供器。在你的任何服务提供器方法中，你可以通过 `$app` 属性来访问服务容器：

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * 在容器中注册绑定。
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection(config('riak'));
            });
        }
    }

这个服务提供器只定义了一个 `register` 方法，并使用该方法在服务容器中定义了一个 `Riak\Connection` 实现。 如果你不了解服务容器的工作原理，请查看其 [文档](/docs/{{version}}/container).

<a name="the-boot-method"></a>
### 引导方法

那么，如果我们需要在我们的服务提供器中注册一个视图组件呢？这应该在 `boot` 方法中完成。**此方法在所有其他服务提供器都注册之后才能调用**，这意味着你可以访问已经被框架注册的所有服务：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * 引导任何应用程序服务。
         *
         * @return void
         */
        public function boot()
        {
            view()->composer('view', function () {
                //
            });
        }
    }

#### 引导方法依赖注入

你可以为服务提供器的 `boot` 方法设置类型提示。[服务容器](/docs/{{version}}/container) 会自动注入你需要的任何依赖项：

    use Illuminate\Contracts\Routing\ResponseFactory;

    public function boot(ResponseFactory $response)
    {
        $response->macro('caps', function ($value) {
            //
        });
    }

<a name="registering-providers"></a>
## 注册提供器

所有服务提供器都在 `config/app.php` 配置文件中注册。该文件中有一个 `providers` 数组，用于存放服务提供器的类名 。默认情况下，这个数组列出了一系列 Laravel 核心服务提供器。这些服务提供器引导 Laravel 核心组件，例如邮件、队列、缓存等。

要注册提供器，只需要将其添加到数组：

    'providers' => [
        // 其他服务提供器

        App\Providers\ComposerServiceProvider::class,
    ],

<a name="deferred-providers"></a>
## 延迟提供器

如果你的提供器**仅**在 [服务容器](/docs/{{version}}/container) 中注册绑定，就可以选择推迟其注册，直到当它真正需要注册绑定。推迟加载这种提供器会提高应用程序的性能，因为它不会在每次请求时都从文件系统中加载。

Laravel 编译并保存延迟服务提供器提供的所有服务的列表，以及其服务提供器类的名称。因此，只有当你在尝试解析其中一项服务时，Laravel 才会加载服务提供器。

要延迟提供器的加载，请将 `defer` 属性设置为 `true` ，并定义 `provides` 方法。`provides` 方法应该返回由提供器注册的服务容器绑定：

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * 是否延时加载提供器。
         *
         * @var bool
         */
        protected $defer = true;

        /**
         * 注册服务提供器。
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection($app['config']['riak']);
            });
        }

        /**
         * 获取提供器提供的服务。
         *
         * @return array
         */
        public function provides()
        {
            return [Connection::class];
        }

    }

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@devindwan](https://github.com/devindwan) | <img class="avatar-66 rm-style" src="https://avatars2.githubusercontent.com/u/10205466?v=4&s=100"> | 翻译 | There are only two kinds of languages: the ones people complain about and the ones nobody uses. |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  Review  | Stay Hungry. Stay Foolish. |


---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
