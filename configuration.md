# 配置

- [基础介绍](#introduction)
- [获取设置值](#accessing-configuration-values)
- [环境配置](#environment-configuration)
    - [获取目前应用程序的环境](#determining-the-current-environment)
- [缓存配置信息](#configuration-caching)
- [维护模式](#maintenance-mode)

<a name="introduction"></a>
## Introduction

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
### Determining The Current Environment

The current application environment is determined via the `APP_ENV` variable from your `.env` file. You may access this value via the `environment` method on the `App` [facade](/docs/{{version}}/facades):

    $environment = App::environment();

You may also pass arguments to the `environment` method to check if the environment matches a given value. If necessary, you may even pass multiple values to the `environment` method. If the environment matches any of the given values, the method will return `true`:

    if (App::environment('local')) {
        // The environment is local
    }

    if (App::environment('local', 'staging')) {
        // The environment is either local OR staging...
    }

An application instance may also be accessed via the `app` helper method:

    $environment = app()->environment();

<a name="configuration-caching"></a>
## Configuration Caching

To give your application a speed boost, you should cache all of your configuration files into a single file using the `config:cache` Artisan command. This will combine all of the configuration options for your application into a single file which will be loaded quickly by the framework.

You should typically run the `php artisan config:cache` command as part of your production deployment routine. The command should not be run during local development as configuration options will frequently need to be changed during the course of your application's development.

<a name="maintenance-mode"></a>
## Maintenance Mode

When your application is in maintenance mode, a custom view will be displayed for all requests into your application. This makes it easy to "disable" your application while it is updating or when you are performing maintenance. A maintenance mode check is included in the default middleware stack for your application. If the application is in maintenance mode, an `HttpException` will be thrown with a status code of 503.

To enable maintenance mode, simply execute the `down` Artisan command:

    php artisan down

To disable maintenance mode, use the `up` command:

    php artisan up

#### Maintenance Mode Response Template

The default template for maintenance mode responses is located in `resources/views/errors/503.blade.php`. You are free to modify this view as needed for your application.

#### Maintenance Mode & Queues

While your application is in maintenance mode, no [queued jobs](/docs/{{version}}/queues) will be handled. The jobs will continue to be handled as normal once the application is out of maintenance mode.

#### Alternatives To Maintenance Mode

Since maintenance mode requires your application to have several seconds of downtime, you may consider alternatives like [Envoyer](https://envoyer.io) to accomplish zero-downtime deployment with Laravel.
