# Laravel 的配置信息

- [介绍](#introduction)
- [环境配置](#environment-configuration)
    - [检索环境配置](#retrieving-environment-configuration)
    - [确定当前环境](#determining-the-current-environment)
- [访问配置值](#accessing-configuration-values)
- [配置缓存](#configuration-caching)
- [维护模式](#maintenance-mode)

<a name="introduction"></a>
## 介绍

Laravel 框架的所有配置文件都保存于 `config` 目录中。 每个选项都有说明，你可随时查看这些文件并熟悉都有哪些配置选项可供你使用。

<a name="environment-configuration"></a>
## 环境配置

基于应用程序的运行环境，通常不同的环境有不同的配置值是有益的。 例如，你可能希望在本地使用的缓存驱动不同于生产服务器所使用的缓存驱动。

Laravel 利用 Vance Lucas 的 PHP 库 [DotEnv](https://github.com/vlucas/phpdotenv) 使得此项功能的实现变得非常简单。在新安装好的 Laravel 应用程序中，其根目录会包含一个 `.env.example` 文件。如果是通过 Composer 安装的 Laravel，该文件会自动更名为 `.env`。否则，需要你手动更改一下文件名。

你的 `.env` 文件不应该提交到应用程序的源代码控制系统中，因为每个使用你的应用程序的开发人员 / 服务器可能都需要各自不同的环境配置。此外，如果入侵者获得对你的源代码控制系统仓库的访问权，这将会导致安全风险，因为所有敏感的安全凭据都被暴露了。

如果是团队开发，则可能希望在应用程序中仍包含 ` .env.example` 文件。因为通过在示例配置文件中放置占位值，团队中的其他开发人员可以清楚地看到哪些环境变量是运行应用程序所必需的。你也可以创建一个 `.env.testing` 文件。当运行 PHPUnit 测试或以 `--env=testing` 选项执行 Artisan 命令时，该文件将覆盖 `.env` 文件中的值。

> {tip} 你的 `.env` 文件中的所有变量都可被环境变量（比如服务器级或系统级环境变量）所覆盖。

<a name="retrieving-environment-configuration"></a>
### 检索环境配置

当应用程序收到请求时，`.env` 文件包含的所有变量将被加载到 PHP 的超级全局变量 `$ _ENV` 中。你可以使用 `env` 辅助函数检索这些变量的值。实际上，如果你查看 Laravel 的配置文件，你将注意到有数个选项已经使用了此辅助函数：

    'debug' => env('APP_DEBUG', false),

传递给 `env` 函数的第二个值是「默认值」。如果给定的键不存在对应的环境变量，则返回默认值。

<a name="determining-the-current-environment"></a>
### 确定当前环境

应用程序当前所处环境是通过 `.env` 文件中的 `APP_ENV` 变量确定的。你可以通过 `App` [facade](/docs/{{version}}/facades) 中的 `environment`  方法来访问此值：

    $environment = App::environment();

你还可以传递参数给 `environment` 方法，以检查环境是否与给定值匹配。 如果环境与给定值匹配，该方法将返回 `true`：

    if (App::environment('local')) {
        // 环境为 local
    }

    if (App::environment(['local', 'staging'])) {
        // 环境为 local 或 staging
    }

> {tip} 应用程序当前所处环境检测可以被服务器级的 `APP_ENV` 环境变量覆盖。这对于环境配置不同但共享相同应用程序的情况下是有用的，这样你可以在服务器配置中设置给定的主机匹配给定的环境。

<a name="accessing-configuration-values"></a>
## 访问配置值

你可以轻松地在应用程序的任何位置使用全局 `config` 函数来访问配置值。配置值的访问使用「点」语法，包括要访问的文件和选项的名称。还可以指定默认值，如果配置选项不存在，则返回默认值：

    $value = config('app.timezone');

要在运行时设置配置值，传递一个数组给 `config` 辅助函数：

    config(['app.timezone' => 'America/Chicago']);

<a name="configuration-caching"></a>
## 配置缓存

为提升应用程序速度，你应该使用 Artisan 命令 `config:cache` 将所有的配置文件缓存至单个文件。这将把应用程序的所有配置选项合并至一个文件，其将被框架快速加载。

你通常应把运行 `php artisan config:cache` 命令作为生产部署例程的一部分。该命令不应在本地开发期间运行，因为配置选项在应用程序开发过程中会经常需要更改。

> {note} 如果在部署过程中执行 `config:cache` 命令，应该确保只从配置文件内部调用 `env` 函数。

<a name="maintenance-mode"></a>
## 维护模式

当应用程序处于维护模式时，所有对应用程序的请求都显示为一个自定义视图。这样可以在更新或执行维护时轻松地「关闭」你的应用程序。 维护模式检查包含在应用程序的默认中间件栈中。如果应用程序处于维护模式，则将抛出一个状态码为 503 的 `MaintenanceModeException` 异常。

要启用维护模式，只需执行下面的 Artisan 命令 `down`：

    php artisan down

还可以向 down 命令提供 `message` 和 `retry` 选项。其中 message 选项的值可用于显示或记录定制消息，而 retry 值将被用于设置 HTTP 标头 `Retry-After`：

    php artisan down --message="Upgrading Database" --retry=60

要关闭维护模式，使用 `up` 命令：

    php artisan up

> {tip} 你可以通过修改 `resources/views/errors/503.blade.php` 模板文件来定制自己的维护模式模板。

#### 维护模式和队列

当应用程序处于维护模式时，不会处理 [队列任务](/docs/{{version}}/queues)。 任务会在应用程序退出维护模式后得到继续处理。

#### 维护模式的替代方案

由于维护模式会导致应用程序有数秒的停机（不响应）时间，可以考虑使用 [Envoyer](https://envoyer.io) 之类的替代方案，能实现  Laravel 零停机时间部署。

## 译者署名
| 用户名                                      | 头像                                       | 职能   | 签名          |
| ---------------------------------------- | ---------------------------------------- | ---- | ----------- |
| [@痛饮狂歌](https://laravel-china.org/users/7636) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/7636_1500076845.png?imageView2/1/w/100/h/100"> | 翻译   | 独立开发者，全栈工程师 |
