# 服务提供者

- [简介](#introduction)
- [编写服务提供者](#writing-service-providers)
    - [注册方法](#the-register-method)
    - [启动方法](#the-boot-method)
- [注册提供者](#registering-providers)
- [延迟提供者](#deferred-providers)

<a name="introduction"></a>
## 简介

服务提供者是所有 Laravel 应用程序启动的中心所在。包括你自己的应用程序，以及所有的 Laravel 核心服务，都是通过服务提供者启动的。

但我们所说的「启动」指的是什么？一般而言，我们指的是 **注册** 事物，包括注册服务容器绑定、事件侦听器、中间件，甚至路由。服务提供者是设置你的应用程序的中心所在。

若你打开 Laravel 的 `config/app.php` 文件，你将会看到 `providers` 数组。这些都是你的应用程序会加载到的所有服务提供者类。当然，它们之中有很多属于「延迟」提供者，意味着除非真正需要它们所提供的服务，否则它们并不会在每一个请求中都被加载。

在这份概述中，你会学到如何编写你自己的服务提供者，并将它们注册于你的 Laravel 应用程序。

<a name="writing-service-providers"></a>
## 编写服务提供者

所有的服务提供者都继承了 `Illuminate\Support\ServiceProvider` 类。大多数服务提供者包含一个 `register` 方法和一个 `boot` 方法，在 `register` 方法中，你应该 **只将事物绑定至 [服务容器](/docs/{{version}}/container) 之中**。永远不要试图在 `register` 方法中注册任何事件侦听器、路由或任何其它功能。

Artisan 命令行接口可以很容易地通过 `make:provider` 命令生成新的提供者：

    php artisan make:provider RiakServiceProvider

<a name="the-register-method"></a>
### 注册方法

如同之前提到的，在 `register` 方法中，你应该只将事物绑定至 [服务容器](/docs/{{version}}/container) 中。永远不要尝试在 `register` 方法中注册任何事件侦听器、路由或任何其它功能。否则的话，你可能会意外地使用到由尚未加载的服务提供者所提供的服务。

现在，让我们来看看基本的服务提供者。在你的任意一个服务提供者方法中，你总是可以通过访问 `$app` 属性它提供了访问服务容器的能力。

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

此服务提供者只定义了一个 `register` 方法，并在服务容器中使用此方法定义了一份 `Riak\Contracts\Connection` 的实现。若你不了解服务容器是如何运作的，可查阅[其文档](/docs/{{version}}/container)。

<a name="the-boot-method"></a>
### 启动方法

因此，若我们需要在我们的服务提供者中注册一个视图 composer 则应该在 `boot` 方法中完成。**此方法会在所有其它的服务提供者被注册后才被调用**，意味着你能访问已经被框架注册的所有其它服务：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * 启动任意应用服务。
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

#### 启动方法依赖注入

我们可以为我们 `boot` 方法中的依赖作类型提式。[服务容器](/docs/{{version}}/container) 会自动注入你所需要的任何依赖：

    use Illuminate\Contracts\Routing\ResponseFactory;

    public function boot(ResponseFactory $response)
    {
        $response->macro('caps', function ($value) {
            //
        });
    }

<a name="registering-providers"></a>
## 注册提供者

所有的服务提供者都在 `config/app.php` 配置文件中被注册。这个文件包含了一个 `providers` 数组，你可以在其中列出你所有服务提供者的名称。此数组默认会列出一组 Laravel 的核心服务提供者。这些提供者启动了 Laravel 的核心组件，例如邮件寄送者、队列、缓存及其它等等。

将你的提供者加入此数组来注册服务提供者：

    'providers' => [
        // 其它的服务提供者

        App\Providers\AppServiceProvider::class,
    ],

<a name="deferred-providers"></a>
## 延迟提供者

若你的提供者 **仅** 在 [服务容器](/docs/{{version}}/container) 中注册绑定，你可以选择延缓其注册，直到真正需要其中已注册的绑定，延迟提供者加载可提高应用程序的性能。

Laravel 编译并保存了一份清单，包括所有由延缓服务提供者所提供的服务，以及其服务提供者类的名称。因此，只有在当你企图解析其中的服务时，Laravel 才会加载该服务提供者。

要延迟提供者加载，可将 `defer` 属性设置为 `true`，并定义一个 `provides` 方法。`provides` 方法会返回提供者所注册的服务容器绑定：

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * 指定提供者加载是否延缓。
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
         * 获取提供者所提供的服务。
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
| [@rang9527](https://github.com/rang9527)  | <img class="avatar-66 rm-style" src="https://avatars2.githubusercontent.com/u/10138783?v=3&s=132">  |  翻译  | 进城务工人员，[@rang9527](https://github.com/rang9527/) at Github  |
