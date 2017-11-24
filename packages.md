# Laravel 的扩展插件开发指南

- [简介](#introduction)
    - [Facade 注解](#a-note-on-facades)
- [扩展包发现](#package-discovery)
- [服务提供者](#service-providers)
- [资源文件](#resources)
    - [配置](#configuration)
    - [数据库迁移](#migrations)
    - [路由](#routes)
    - [语言包](#translations)
    - [视图](#views)
- [命令](#commands)
- [公用 Assets](#public-assets)
- [发布群组文件](#publishing-file-groups)

<a name="introduction"></a>
## 简介

扩展包是添加功能到 Laravel 的主要方式。扩展包可以包含许多好用的功能，像 [Carbon](https://github.com/briannesbitt/Carbon) 可用于处理时间，或像 [Behat](https://github.com/Behat/Behat) 这种完整的 BDD 测试框架。

当然，这有非常多不同类型的扩展包。有些扩展包是独立运作的，意思是指他们并不依赖于任何 PHP 框架。刚刚所提到的 Carbon 及 Behat 就是这种扩展包。要在 Laravel 中使用这种扩展包只需要在 `composer.json` 文件里引入它们即可。

另一方面，有些扩展包特别指定只能在 Laravel 里面集成。这些扩展包可能包含路由、控制器、视图以及扩展包的相关设置，目的是增强 Laravel 本身的功能。这份指南里将主要以开发 Laravel 专属的扩展包为目标进行说明。

<a name="a-note-on-facades"></a>
### Facade 注解

当开发 Laravel 应用程序时，通常来讲你使用契约(contracts) 还是 facades 并没有什么区别，因为他们都提供了基本相同的水平的可测试性。但是，在进行扩展包开发的时候，你开发的扩展包并不能访问所有 Laravel 提供的测试辅助函数。如果想要别人在一个 Laravel 应用程序中，能够开发你。如果你想写扩展包的测试，并让这些测试看起来像是在一个典型的 Laravel 应用程序里，您可以使用 [Orchestral Testbench](https://github.com/orchestral/testbench)。

<a name="package-discovery"></a>
## 扩展包发现

在 Laravel 应用程序的 `config/app.php` 配置文件中，`providers` 选项定义了应该被 Laravel 加载的服务提供者的列表。当有人安装你的扩展包时，你需要让你的服务提供者包含在这个列表里。而不是要求用户手动将你的服务提供者添加到这个列表里，你可能需要在你扩展包 `composer.json` 文件的 `extra` 部分的定义这些提供者。除了服务提供者，你也要列出可能想要注册的 [facades](/docs/{{version}}/facades)：

    "extra": {
        "laravel": {
            "providers": [
                "Barryvdh\\Debugbar\\ServiceProvider"
            ],
            "aliases": {
                "Debugbar": "Barryvdh\\Debugbar\\Facade"
            }
        }
    },

当 Laravel 安装的时候，一旦发现你的扩展包被配置，Laravel 将会自动的注册它的服务提供者和 facades，为扩展包的用户提供一个方便的安装体验。

### 选择扩展包发现

如果你是扩展包的使用者，你想要禁用一个包的扩展包发现，你可以在应用程序的 `composer.json` 文件的  `extra` 部分列出这个扩展包：

    "extra": {
        "laravel": {
            "dont-discover": [
                "barryvdh/laravel-debugbar"
            ]
        }
    },

你可以通过在应用程序的 `dont-discover` 指令中使用 `*` 字符，禁用扩展包发现功能：

    "extra": {
        "laravel": {
            "dont-discover": [
                "*"
            ]
        }
    },

<a name="service-providers"></a>
## 服务提供者

[服务提供者](/docs/{{version}}/providers) 是你的扩展包与 Laravel 连接的重点。服务提供者负责绑定一些东西至 Laravel 的 [服务容器](/docs/{{version}}/container) 并告知 Laravel 要从哪加载扩展包的资源，例如视图、配置文件、语言包。

服务提供者继承了 `Illuminate\Support\ServiceProvider` 类并包含了两个方法： `register` 和 `boot`. 。基底的 `ServiceProvider` 类被放置在 Composer 的 `illuminate/support` 扩展包，你必须将它加入至你自己的扩展包依赖中。若要了解更多关于服务提供者的结构与用途，请查阅 [它的文档](/docs/{{version}}/providers)。

<a name="resources"></a>
## 资源文件

<a name="configuration"></a>
### 配置

有时候，你可能想要将扩展包的配置文件发布到应用程序本身的 `config` 目录上。这能够让扩展包的用户轻松的重写这些默认的设置选项。如果要发布扩展包的配置文件，只需要在服务提供者里的 `boot` 方法内使用 `publishes` 方法：

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

现在当扩展包的用户使用 Laravel 的 `vendor:publish` 命令时，扩展包的文件将会被复制到指定的位置上。当然，只要你的配置文件被发布，就可以如其它配置文件一样被访问：

    $value = config('courier.option');

> {note} 您不应在配置文件中定义闭包函数。 当用户执行 `config:cache` Artisan命令时，它们不能正确地序列化。

#### 默认的扩展包配置文件

你也可以选择合并你的扩展包配置文件和应用程序里的副本配置文件。这样能够让你的用户在已经发布的副本配置文件中只包含他们想要重写的设置选项。如果想要合并配置文件，可在服务提供者里的 `register` 方法里使用 `mergeConfigFrom` 方法：

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

> {note} 此方法仅合并配置数组的第一级。如果您的用户部分定义了多维配置数组，则不会合并缺失的选项。

<a name="routes"></a>
### 路由

如果您的扩展包中包含路由，您可以使用 `loadRoutesFrom` 方法加载它们。此方法将自动确定应用程序的路由是否已缓存，如果路由已缓存，将不会加载路由文件：

    /**
     * 执行服务的注册后启动。
     *
     * @return void
     */
    public function boot()
    {
        $this->loadRoutesFrom(__DIR__.'/routes.php');
    }

<a name="migrations"></a>
### 数据库迁移

如果你的扩展包包含 [数据库迁移](/docs/{{version}}/migrations) ，你需要使用 `loadMigrationsFrom` 方法告知 Laravel 如何去加载他们。`loadMigrationsFrom` 方法只需要你的扩展包的迁移文件路径作为唯一参数：

    /**
     * 在注册后进行服务的启动。
     *
     * @return void
     */
    public function boot()
    {
        $this->loadMigrationsFrom(__DIR__.'/path/to/migrations');
    }

完成扩展包迁移文件注册之后，在运行 `php artisan migrate` 命令时，它们就会自动被执行。你并不需要把他们导出到应用程序的 `database/migrations` 目录。

<a name="translations"></a>
### 语言包

如果你的扩展包里面包含了 [本地化](/docs/{{version}}/localization) ，则可以使用 `loadTranslationsFrom` 方法来告知 Laravel 该如何加载它们。举个例子，如果你的扩展包名称为 `courier` ，你可以按照以下方式将其添加至服务提供者的 `boot` 方法：

    /**
     * 在注册后进行服务的启动。
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
    }

扩展包翻译参照使用了双分号 `package::file.line` 语法。所以，你可以按照以下方式来加载 `courier` 扩展包中的 `messages` 文件 `welcome` 语句：

    echo trans('courier::messages.welcome');

#### 发布语言包

如果你想将扩展包的语言包发布至应用程序的 `resources/lang/vendor` 目录，则可以使用服务提供者的 `publishes` 方法。`publishes` 方法接受一个包含扩展包路径及对应发布位置的数组。例如，在 `courier` 扩展包中发布语言包，示例如下：

    /**
     * 在注册后进行服务的启动。
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
    
        $this->publishes([
            __DIR__.'/path/to/translations' => resource_path('lang/vendor/courier'),
        ]);
    }

现在，当使用你扩展包的用户运行 Laravel 的 `vendor:publish` Artisan 命令时，扩展包的语言包将会被复制到指定的位置上。

<a name="views"></a>
### 视图

若要在 Laravel 中注册扩展包 [视图](/docs/{{version}}/views)，则必须告诉 Laravel 你的视图位置。你可以使用服务提供者的 `loadViewsFrom` 方法来实现。`loadViewsFrom` 方法允许两个参数：视图模板路径与扩展包名称。例如，如果你的扩展包名称是 `courier` ，你可以按照以下方式将其添加至服务提供者的 `boot` 方法内：

    /**
     * 在注册后进行服务的启动。
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
    }

扩展包视图参照使用了双分号 `package::view` 语法。所以，你可以通过如下方式从 `courier` 扩展包中加载 `admin` 视图：

    Route::get('admin', function () {
        return view('courier::admin');
    });

#### 重写扩展包视图

当你使用 `loadViewsFrom` 方法时，Laravel 实际上为你的视图注册了 两个位置：一个是应用程序的 `resources/views/vendor` 目录，另一个是你所指定的目录。所以，以 `courier` 为例：当用户请求一个扩展包的视图时，Laravel 会在第一时间检查 `resources/views/vendor/courier` 是否有开发者提供的自定义版本视图存在。接着，如果这个路径没有自定义的视图，Laravel 会搜索你在扩展包 `loadViewsFrom` 方法里所指定的视图路径。这个方法可以让用户很方便的自定义或重写扩展包视图。

#### 发布视图

若要发布扩展包的视图至 `resources/views/vendor` 目录，则必须使用服务提供者的 `publishes` 方法。`publishes` 方法允许一个包含扩展包视图路径及对应发布路径的数组：

    /**
     * 在注册后进行服务的启动。
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
    
        $this->publishes([
            __DIR__.'/path/to/views' => resource_path('views/vendor/courier'),
        ]);
    }

现在，当你的扩展包用户运行 Laravel 的 `vendor:publish` Artisan 命令时，扩展包的视图将会被复制到指定的位置上。

<a name="commands"></a>
## 命令

给你的扩展包注册 Artisan 命令，您可以使用 `commands` 方法。此方法需要一个命令类的数组。一旦命令被注册，您可以使用 [Artisan 命令行](/docs/{{version}}/artisan) 执行它们：

    /**
     * 在注册后进行服务的启动。
     *
     * @return void
     */
    public function boot()
    {
        if ($this->app->runningInConsole()) {
            $this->commands([
                FooCommand::class,
                BarCommand::class,
            ]);
        }
    }

<a name="public-assets"></a>
## 公用 Assets

你的扩展包内可能会包含许多的资源文件，像 JavaScript、CSS 和图片等文件。如果要发布这些资源文件到应用程序的 `public` 目录上，只需使用服务提供者的 `publishes` 方法。在这个例子中，我们也会增加一个 `public` 的资源分类标签，可用于发布与分类关联的资源文件：

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

现在，当您的扩展包的用户执行 `vendor:publish` 命令时，您的 Assets 将被复制到指定的发布位置。由于每次更新包时通常都需要覆盖资源，因此您可以使用 `--force` 标志：

    php artisan vendor:publish --tag=public --force

<a name="publishing-file-groups"></a>
## 发布群组文件

你可能想要分别发布分类的扩展包资源文件或是资源。举例来说，你可能想让用户不用发布扩展包的所有资源文件，只需要单独发布扩展包的配置文件即可。这可以通过在调用 `publishes` 方法时使用「标签」来实现。例如，让我们在扩展包的服务提供者中的 `boot` 方法定义两个发布群组：

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

现在当你的用户使用 `vendor:publish` Artisan 命令时，就可以通过标签名称分别发布不同分类的资源文件：

    php artisan vendor:publish --tag=config



## 译者署名
| 用户名                                      | 头像                                       | 职能   | 签名                                       |
| ---------------------------------------- | ---------------------------------------- | ---- | ---------------------------------------- |
| [@CraryPrimitiveMan](https://github.com/e421083458) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/3020_1449383575.jpeg?imageView2/1/w/100/h/100"> | 翻译   | [构建自己的PHP框架](https://github.com/CraryPrimitiveMan/create-your-own-php-framework)，欢迎 Star。 |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org