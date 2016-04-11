# 扩展包开发

- [简介](#introduction)
- [服务提供者](#service-providers)
- [路由](#routing)
- [资源](#resources)
    - [视图](#views)
    - [语言](#translations)
    - [配置文件](#configuration)
- [公用资源文件](#public-assets)
- [发布分类文件](#publishing-file-groups)

<a name="introduction"></a>
## 简介

扩展包是扩增功能到 Laravel 的主要方式。扩展包可以包含许多好用的功能，像 [Carbon](https://github.com/briannesbitt/Carbon) 用于处理时间，或像 [Behat](https://github.com/Behat/Behat) 这种完整的 BDD 测试框架。

当然，有非常多不同类型的扩展包。有些扩展包是独立运作的，意思是指他们并不依赖于任何框架，包括 Laravel。刚刚所提到的 Carbon 及 Behat 就是这种扩展包。要使用这种扩展包只需要在 `composer.json` 文件里引入它们即可。

另一方面，有些扩展包特别指定要与 Laravel 集成。这些扩展包可能包含路由、控制器、视图以及扩展包的相关设置，目标是增强 Laravel 本身的功能。这份指南里将主要以开发 Laravel 专属的扩展包为目标进行说明。

<a name="service-providers"></a>
## 服务提供者

[服务提供者](/docs/{{version}}/providers)是你的扩展包与 Laravel 连接的重点。服务提供者负责绑定一些东西至 Laravel 的[服务容器](/docs/{{version}}/container)并告知 Laravel 要从哪加载扩展包的资源，像是视图，配置文件，与语言包。

服务提供者继承了 `Illuminate\Support\ServiceProvider` 类并包含了两个方法：`register` 及 `boot`。基底的 `ServiceProvider` 类被放置在 Composer 的 `illuminate/support` 扩展包，你必须将它加入至你自己的扩展包的依赖。

若要了解更多关于服务提供者的结构与用途，请查阅[它的文档](/docs/{{version}}/provider)。

<a name="routing"></a>
## 路由

要为你的扩展包定义路由，只要简单的在你扩展包的服务提供者的 `boot` 方法 `require` 路由文件。在你的路由文件中，你可以如同在一般的 Laravel 应用程序一样使用 `Route` facade 来[注册路由](/docs/{{version}}/routing)：

    /**
     * 在注册后进行服务的启动。
     *
     * @return void
     */
    public function boot()
    {
        if (! $this->app->routesAreCached()) {
            require __DIR__.'/../../routes.php';
        }
    }

<a name="resources"></a>
## 资源

<a name="views"></a>
### 视图

若要在 Laravel 中注册你扩展包的[视图](/docs/{{version}}/views)，你必须告诉 Laravel 你的视图位置。你可以使用服务提供者的 `loadViewsFrom` 方法来达成。`loadViewsFrom` 方法允许两个参数：你的视图模板路径与你的扩展包名称。例如，如果你的扩展包名称是「courier」，你可以按照以下方式添加至你的服务提供者的 `boot` 方法：

    /**
     * 在注册后进行服务的启动。
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
    }

扩展包视图可以使用双分号 `package::view` 语法参照它。所以，你可以如同以下方式从 `courier` 扩展包加载 `admin` 视图：

    Route::get('admin', function () {
        return view('courier::admin');
    });

#### 重写扩展包视图

当你使用 `loadViewsFrom` 方法时，Laravel 实际上为了你的视图注册了**两个**位置：一个是应用程序的 `resources/views/vendor` 目录，另一个是你所指定的目录。所以，使用我们的 `courier` 为例：当请求一个扩展包的视图时，Laravel 会在第一时间检查 `resources/views/vendor/courier` 是否有开发者提供的自定义版本视图存在。接着，如果这个路径没有自定义的视图，Laravel 会搜索你在扩展包 `loadViewsFrom` 方法里所指定的视图路径。这个方法让用户可以方便的自定义或重写你的扩展包里的视图。

#### 发布视图

若要发布扩展包的视图至 `resources/views/vendor` 目录，你必须使用服务提供者的 `publishes` 方法。`publishes` 方法允许一个包含扩展包视图路径及对应发布路径的数组。

    /**
     * 在注册后进行服务的启动。
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

        $this->publishes([
            __DIR__.'/path/to/views' => base_path('resources/views/vendor/courier'),
        ]);
    }

现在，当你扩展包的用户运行 Laravel 的 `vendor:publish` Artisan 命令时，你扩展包的视图将会被复制到指定的位置。

<a name="translations"></a>
### 语言

如果你的扩展包包含[语言文件](/docs/{{version}}/localization)，你可以使用 `loadTranslationsFrom` 方法来告知 Laravel 该如何加载它们。举个例子，如果你的扩展包名称为「courier」，你可以按照以下方式添加至你服务提供者的 `boot` 方法：

    /**
     * 在注册后进行服务的启动。
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
    }

扩展包语言可以使用双分号 `package::file.line` 语法参照它。所以，你可以按照以下方式加载 `courier` 扩展包中 `messages` 文件的 `welcome` 语句：

    echo trans('courier::messages.welcome');

#### 发布语言包

如果你想将扩展包的语言包发布至应用程序的 `resources/lang/vendor` 目录，你可以使用服务提供者的 `publishes` 方法。`publishes` 方法接受一个包含扩展包路径及对应发布位置的数组。例如，若要在我们的例子 `courier` 扩展包发布语言包：

    /**
     * 在注册后进行服务的启动。
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');

        $this->publishes([
            __DIR__.'/path/to/translations' => base_path('resources/lang/vendor/courier'),
        ]);
    }

现在，当你扩展包的用户运行 Laravel 的 `vendor:publish` Artisan 命令时，你扩展包的语言包将会被复制到指定的位置。

<a name="configuration"></a>
### 配置文件

基本上，你可能想要将你扩展包的配置文件发布到应用程序本身的 `config` 目录。这能够让你扩展包的用户轻松的重写这些默认的设置选项。如果要发布扩展包的配置文件，只需要在服务提供者里的 `boot` 方法里使用 `publishes` 方法：

    /**
     * 在注册后进行服务的启动。
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
        ]);
    }

现在当你扩展包的用户使用 Laravel 的 `vendor:publish` 命令时，你的文件将会被复制到指定的位置。当然，只要你的配置文件被发布后，就可以如其他配置文件一样被访问：

    $value = config('courier.option');

#### 默认扩展包配置文件

你也可以选择合并你的扩展包配置文件和应用程序里的副本配置文件。这样能够让你的用户在已经发布的副本配置文件里只包含他们想要重写的设置选项。如果想要合并配置文件，可在服务提供者里的 `register` 方法里使用 `mergeConfigFrom` 方法：

    /**
     * 在容器中注册绑定。
     *
     * @return void
     */
    public function register()
    {
        $this->mergeConfigFrom(
            __DIR__.'/path/to/config/courier.php', 'courier'
        );
    }

<a name="public-assets"></a>
## 公用资源文件

你的扩展包内可能会包含许多的资源文件，像是 JavaScript、CSS 和图片。如果要发布资源文件至应用程序的 `public` 目录，只需要使用服务提供者的 `publishes` 方法。在这个例子中，我们也会增加一个 `public` 的资源分类标签，可以被使用于发布与分类关联的资源文件：

    /**
     * 在注册后进行服务的启动。
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/assets' => public_path('vendor/courier'),
        ], 'public');
    }

现在当你扩展包的用户运行 `vendor:publish` 命令时，你的资源文件将会被复制到指定的位置。当每次扩展包更新需要重写资源文件时，你可以使用 `--force` 标记：

    php artisan vendor:publish --tag=public --force

如果你想要确保你的公用资源文件始终保持在最新的版本，可以将此命令加入你的 `composer.json` 文件中的 `post-update-cmd` 列表。

<a name="publishing-file-groups"></a>
## 发布分类文件

你可能想要分别发布分类的扩展包资源文件或是资源。举例来说，你可能想让用户不需发布扩展包的所有资源文件，只单独发布扩展包的配置文件。你可以在调用 `publishes` 方法时使用「标签」来做到。例如，让我们在扩展包的服务提供者中的 `boot` 方法定义两个发布群组：

    /**
     * 在注册后进行服务的启动。
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/../config/package.php' => config_path('package.php')
        ], 'config');

        $this->publishes([
            __DIR__.'/../database/migrations/' => database_path('migrations')
        ], 'migrations');
    }

现在当你的用户使用 `vendor:publish` Artisan 命令时，可以通过标签名称分别发布不同分类的资源文件：

    php artisan vendor:publish --provider="Vendor\Providers\PackageServiceProvider" --tag="config"
