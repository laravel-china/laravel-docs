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

Laravel 框架的所有配置文件都保存在 `config` 目录中。 每个选项都有说明，你可随时查看这些文件并熟悉都有哪些配置选项可供你使用。

<a name="environment-configuration"></a>
## 环境配置

对于应用程序运行的环境来说，不同的环境有不同的配置通常是很有用的。 例如，你可能希望在本地使用的缓存驱动不同于生产服务器所使用的缓存驱动。

Laravel 利用 Vance Lucas 的 PHP 库 [DotEnv](https://github.com/vlucas/phpdotenv) 使得此项功能的实现变得非常简单。在新安装好的 Laravel 应用程序中，其根目录会包含一个 `.env.example` 文件。如果是通过 Composer 安装的 Laravel，该文件会自动更名为 `.env`。否则，需要你手动更改一下文件名。

你的 `.env` 文件不应该提交到应用程序的源代码控制系统中，因为每个使用你的应用程序的开发人员 / 服务器可能需要有一个不同的环境配置。此外，在入侵者获得你的源代码控制仓库的访问权的情况下，这会成为一个安全隐患，因为任何敏感的凭据都被暴露了。

如果是团队开发，则可能希望应用程序中仍包含 ` .env.example` 文件。因为通过在示例配置文件中放置占位值，团队中的其他开发人员可以清楚地看到哪些环境变量是运行应用程序所必需的。你也可以创建一个 `.env.testing` 文件，当运行 PHPUnit 测试或以 `--env=testing` 为选项执行 Artisan 命令时，该文件将覆盖 `.env` 文件中的值。

> {tip} `.env` 文件中的所有变量都可被外部环境变量（比如服务器级或系统级环境变量）所覆盖。

<a name="retrieving-environment-configuration"></a>
### 检索环境配置

当应用程序收到请求时，`.env` 文件中列出的所有变量将被加载到 PHP 的超级全局变量 `$ _ENV` 中。你可以使用 `env` 函数检索这些变量的值。事实上，如果你查看 Laravel 的配置文件，你就能注意到有数个选项已经使用了这个函数：

    'debug' => env('APP_DEBUG', false),

传递给 `env` 函数的第二个值是「默认值」。如果给定的键不存在环境变量，则会使用该值。

<a name="determining-the-current-environment"></a>
### 确定当前环境

应用程序当前所处环境是通过 `.env` 文件中的 `APP_ENV` 变量确定的。你可以通过 `App` [facade](/docs/{{version}}/facades) 中的 `environment`  方法来访问此值：

    $environment = App::environment();

你还可以传递参数给 `environment` 方法，以检查当前的环境配置是否与给定值匹配。 如果与给定值匹配，该方法将返回 `true`：

    if (App::environment('local')) {
        // 环境为 local
    }

    if (App::environment(['local', 'staging'])) {
        // 环境为 local 或 staging
    }

> {tip} 应用程序当前所处环境检测可以被服务器级的 `APP_ENV` 环境变量覆盖。这为相同的应用程序配置不同的环境时是非常有用的，这样你可以在你的服务器配置中为给定的主机设置与其匹配的给定的环境。

<a name="accessing-configuration-values"></a>
## 访问配置值

你可以轻松地在应用程序的任何位置使用全局 `config` 函数来访问配置值。配置值的访问可以使用「点」语法，这其中包含了要访问的文件和选项的名称。还可以指定默认值，如果配置选项不存在，则返回默认值：

    $value = config('app.timezone');

要在运行时设置配置值，传递一个数组给 `config` 函数：

    config(['app.timezone' => 'America/Chicago']);

<a name="configuration-caching"></a>
## 配置缓存

为了给你的应用程序提升速度，你应该使用 Artisan 命令 `config:cache` 将所有的配置文件缓存到单个文件中。这会把你的应用程序中所有的配置选项合并成一个单一的文件，然后框架会快速加载这个文件。

通常来说，你应该把运行 `php artisan config:cache` 命令作为生产环境部署常规的一部分。这个命令不应在本地开发环境下运行，因为配置选项在应用程序开发过程中是经常需要被更改的。

> {note} 如果在部署过程中执行 `config:cache` 命令，那你应该确保只从配置文件内部调用 `env` 函数。

<a name="maintenance-mode"></a>
## 维护模式

当应用程序处于维护模式时，所有对应用程序的请求都显示为一个自定义视图。这样可以在更新或执行维护时轻松地「关闭」你的应用程序。 维护模式检查包含在应用程序的默认中间件栈中。如果应用程序处于维护模式，则将抛出一个状态码为 503 的 `MaintenanceModeException` 异常。

要启用维护模式，只需执行下面的 Artisan 命令 `down`：

    php artisan down

你还可以向 down 命令提供 `message` 和 `retry` 选项。其中 message 选项的值可用于显示或记录自定义消息，而 retry 值可用于设置 HTTP 请求头中 `Retry-After` 的值：

    php artisan down --message="Upgrading Database" --retry=60

要关闭维护模式，请使用 `up` 命令：

    php artisan up

> {tip} 你可以通过修改 `resources/views/errors/503.blade.php` 模板文件来自定义默认维护模式模板。

#### 维护模式和队列

当应用程序处于维护模式时，不会处理 [队列任务](/docs/{{version}}/queues)。而这些任务会在应用程序退出维护模式后再继续处理。

#### 维护模式的替代方案

维护模式会导致应用程序有数秒的停机（不响应）时间，因此你可以考虑使用像 [Envoyer](https://envoyer.io) 这样的替代方案，以便与 Laravel 完成零停机时间部署。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
| --- | --- | --- | --- |
| [@痛饮狂歌](https://laravel-china.org/users/7636) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/7636_1500076845.png?imageView2/1/w/100/h/100"> | 翻译 | 独立开发者，全栈工程师 |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  Review  | Stay Hungry. Stay Foolish. |



---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
