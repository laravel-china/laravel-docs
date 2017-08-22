# 服务提供者

- [简介](#introduction)
- [编写服务提供者](#writing-service-providers)
    - [注册方法](#the-register-method)
    - [引导方法](#the-boot-method)
- [注册提供者](#registering-providers)
- [延迟的提供者](#deferred-providers)

<a name="introduction"></a>
## 简介

服务提供者是所有 Laravel 应用程序引导启动的中心所在。包括你自己的应用程序，以及所有的 Laravel 核心服务，都是通过服务提供者引导启动的。

但我们所谓的「引导启动」指的是什么？一般而言，我们指的是**注册**事务，包括注册服务容器绑定，事件监听器，中间件，甚至路由。服务提供者是配置你的应用程序的中心位置。

如果你打开 Laravel 的 `config/app.php` 文件，会看到一个 `providers` 数组。这些是将要被你的应用程序加载的所有的服务提供者类。当然，它们其中有许多是「延迟」提供者，意味着它们不会在每次请求时都被加载，而是按需加载。

在本概述中，你将学习如何编写自己的服务提供者，并在你的 Laravel 应用程序中注册它们。

<a name="writing-service-providers"></a>
## 编写服务提供者

所有服务提供者都继承自 `Illuminate\Support\ServiceProvider` 类。大多数服务提供者都包含 `register` 和 `boot` 方法。在 `register` 方法中，你应当**只将事务绑定到 [服务容器](/docs/{{version}}/container)**。不应当在 `register` 方法中尝试注册任何事件监听器，路由或者任何其他功能。

使用 Artisan 命令行接口，你可以通过 `make:provider` 命令生成一个新的提供者：

    php artisan make:provider RiakServiceProvider

<a name="the-register-method"></a>
### 注册方法

如上所述，在 `register` 方法中，你应当只将事务绑定到 [服务容器](/docs/{{version}}/container) 。不应该在 `register` 方法中尝试注册任何事件监听器，路由或者任何其他功能。否则，你可能会意外的使用到尚未加载的服务提供者提供的服务。

让我们来看一个基本的服务提供者。在你的任意一个服务提供者中，你总是可以通过一个 `$app` 属性来访问服务容器：

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

这个服务提供者只定义了一个 `register` 方法，并且使用该方法在服务容器中定义了一个 `Riak\Connection` 类的实现。 如果你不知道服务容器的工作原理，请查阅 [服务容器](/docs/{{version}}/container).

<a name="the-boot-method"></a>
### 引导方法

那么，如果我们需要在我们的服务提供者中注册一个视图组件呢？这应该在 `boot` 方法中完成。**此方法在所有其他服务提供者均已注册之后调用**，这意味着你可以访问已经被框架注册的所有服务：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * 引导启动任何应用程序服务。
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

你可以为服务提供者的 `boot` 方法设置类型提示。[服务容器](/docs/{{version}}/container) 会自动注入你需要的任何依赖：

    use Illuminate\Contracts\Routing\ResponseFactory;

    public function boot(ResponseFactory $response)
    {
        $response->macro('caps', function ($value) {
            //
        });
    }

<a name="registering-providers"></a>
## 注册提供者

所有服务提供者都在 `config/app.php` 配置文件中注册。此文件包含一个服务提供者数组 `providers`，用于存放你的服务提供者的类名 。默认情况下，它列出了 Laravel 的核心服务提供者类。这些服务提供者引导启动 Laravel 核心组件，例如邮件程序，队列，缓存和其他。

要注册你的提供者，只需将其添加到数组：

    'providers' => [
        // 其他服务提供者

        App\Providers\ComposerServiceProvider::class,
    ],

<a name="deferred-providers"></a>
## 延迟的提供者

如果你的提供者**仅**在 [服务容器](/docs/{{version}}/container) 中注册绑定，你可以选择推迟其注册，直到真正需要其注册的绑定时再按需加载。延迟加载服务提供者将提高应用程序的性能，因为它不会在每次请求时都从文件系统中加载。

Laravel 编译并保存了一份清单，包括由延迟加载的服务提供者所提供的所有服务，以及这些服务提供者类的类名。因此，只有当你在试图解析其中的服务时，Laravel 才会加载该服务提供者。

若要推迟提供者的加载，请将 `defer` 属性设置为 `true` ，并定义 `provides` 方法。该 `provides` 方法应该返回由提供者注册的服务容器绑定：

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * 是否延迟加载提供者。
         *
         * @var bool
         */
        protected $defer = true;

        /**
         * 注册服务提供者。
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
         * 获取提供者提供的服务。
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
| [@devindwan](https://github.com/devindwan)  | <img class="avatar-66 rm-style" src="https://avatars2.githubusercontent.com/u/10205466?v=4&s=100">  |  翻译  | There are only two kinds of languages: the ones people complain about and the ones nobody uses. |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/3810/laravel-54-document-translation-come-and-join-the-translation)。
> 
> 文档永久地址： http://d.laravel-china.org