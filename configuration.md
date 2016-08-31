# 配置

- [基础介绍](#introduction)
- [获取设置值](#accessing-configuration-values)
- [环境配置](#environment-configuration)
    - [获取目前应用程序的环境](#determining-the-current-environment)
- [缓存配置信息](#configuration-caching)
- [维护模式](#maintenance-mode)

<a name="introduction"></a>
## 基础介绍

所有 Laravel 框架的配置文件都放置在 `config` 目录下。每个选项都有说明，请仔细阅读这些说明，并熟悉这些选项配置。

<a name="accessing-configuration-values"></a>
## 获取设置值

可以使用 `config` 辅助函数获取你的设置值，设置值可以通过「点」语法来获取，其中包含了文件与选项的名称。你也可以指定一个默认值，当该设置选项不存在时就会返回默认值：

    $value = config('app.timezone');

若要在运行期间修改设置值，请传递一个数组至 `config` 辅助函数：

    config(['app.timezone' => 'America/Chicago']);

<a name="environment-configuration"></a>
## 环境配置

应用程序常常需要根据不同的运行环境设置不同的值。例如，你会希望在本机开发环境上有与正式环境不同的缓存驱动。类似这种环境变量，只需通过 `.env` 配置文件就可轻松完成。

Laravel 使用 Vance Lucas 的 [DotEnv](https://github.com/vlucas/phpdotenv) PHP 函数库来实现项目内环境变量的控制，在安装好的全新 Laravel 应用程序里，在根目录下会包含一个 `.env.example` 文件。如果你通过 Composer 安装 Laravel，这个文件将自动被更名为 `.env`，否则你只能手动更改文件名。

当你的应用程序收到请求时，这个文件所有的变量都会被加载到 PHP 超级全局变量 `$_ENV` 里。你可以使用辅助函数 `env` 来获取这些变量的值。事实上，如果你阅读过 Laravel 的相关配置文件，你会注意到里面有几个选项已经在使用着这个辅助函数！

    'debug' => env('APP_DEBUG', false),

`env` 函数的第二个参数是默认值，如果未找到对应的环境变量配置的话，此值就会被返回。

根据本机服务器或者正式环境的需求的不同，可自由修改环境变量。但是，`.env` 文件不应该被提交到版本控制系统，因为每个开发人员或服务器在使用应用程序时，可能需要不同的环境配置。

不妨将 `.env.example` 文件放进你的应用程序，通过样本配置文件里的预设值，团队中的其他开发人员就可以清楚地知道，在运行你的应用程序时有哪些环境变量是必须有的。

<a name="determining-the-current-environment"></a>
### 获取目前应用程序的环境

应用程序的当前环境是由 `.env` 文件中的 `APP_ENV` 变量所决定的。你可以通过 `App` [facade](/docs/{{version}}/facades) 的 `environment` 方法来获取该值：

    $environment = App::environment();

你也可以传递参数至 `environment` 方法来确认当前环境是否与参数相符合：

    if (App::environment('local')) {
        // The environment is local
    }

    if (App::environment('local', 'staging')) {
        // The environment is either local OR staging...
    }

也可通过 `app` 辅助函数获取应用程序实例：

    $environment = app()->environment();

<a name="configuration-caching"></a>
## 缓存配置信息

为了让应用程序的速度获得提升，可以使用 Artisan 命令 `config:cache` 将所有的配置文件缓存到单个文件。通过此命令将所有的设置选项合并成一个文件，让框架能够更快速的加载。

你应该将运行 `php artisan config:cache` 命令作为部署工作的一部分。此命令不应该在开发时运行，因为设置选项会在开发时经常变动。

> 译者注：想知道更多 Laravel 程序调优的技巧？请参阅：[Laravel 5 程序优化技巧](https://phphub.org/topics/2020)

<a name="maintenance-mode"></a>
## 维护模式

当你的应用程序处于维护模式时，所有传递至应用程序的请求都会显示出一个自定义视图。在你更新应用或进行性能维护时，这么做可以很轻松的「关闭」整个应用程序。维护模式会检查包含在应用程序的默认的中间件堆栈。如果应用程序处于维护模式，则 `HttpException` 会抛出 503 的状态码。

启用维护模式，只需要运行 Artisan 命令 `down`：

    php artisan down

关闭维护模式，请使用 Artisan 命令 `up`：

    php artisan up

#### 维护模式的响应模板

维护模式的默认模板放在 `resources/views/errors/503.blade.php`。

#### 维护模式与队列

当应用程序处于维护模式中时，将不会处理任何 [队列工作](/docs/{{version}}/queues)。所有的队列工作将会在应用程序离开维护模式后被继续运行。

#### 维护模式的替代方案

维护模式有几秒钟的服务器不可用时间，如果你想做到平滑迁移的话，推荐使用 [Envoyer](https://envoyer.io) 服务。

