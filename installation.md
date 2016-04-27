# 安装

- [安装](#installation)
- [配置信息](#configuration)
    - [基本配置](#basic-configuration)
    - [环境配置](#environment-configuration)
    - [设置缓存](#configuration-caching)
    - [获取设置值](#accessing-configuration-values)
    - [命名你的应用程序](#naming-your-application)
- [维护模式](#maintenance-mode)

<a name="installation"></a>
## 安装

### 运行环境要求

Laravel 框架会有一些系统上的要求。当然，这些要求在 [Laravel Homestead](/docs/{{version}}/homestead) 虚拟机上都已经完全配置好了：

<div class="content-list" markdown="1">
- PHP >= 5.5.9
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension
</div>

> **[Summer](http://github.com/summerblue)：** 是的，Laravel 的开发中，使用 Homestead 是必须的，不论你是一个人开发项目，还是团队开发，不管你是新手，还是老手，请使用 Homestead。可参考 [Homestead 的环境部署脚本](https://github.com/laravel/settler/blob/master/scripts/provision.sh) 来实现开发环境和生产环境的统一。

<a name="install-laravel"></a>
### 安装 Laravel

Laravel 使用 [Composer](http://getcomposer.org) 来管理代码依赖。所以，在使用 Laravel 之前，请先确认你的电脑上安装了 Composer。

#### 通过 Laravel 安装工具

首先，使用 Composer 下载 Laravel 安装包：

    composer global require "laravel/installer"

请确定你已将 `~/.composer/vendor/bin` 路径加到 PATH，只有这样系统才能找到 `laravel` 的执行文件。

一旦安装完成，就可以使用 `laravel new` 命令在指定的目录创建一个新的 Laravel 项目，例如：`laravel new blog` 将会在当前目录下创建一个叫 `blog` 的目录，此目录里面存放着新安装的 Laravel 和代码依赖。这个方法的安装速度比通过 Composer 安装要快上许多：

    laravel new blog

#### 通过 Composer Create-Project

除此之外，你也可以通过 Composer 在命令行运行 `create-project` 命令来安装 Laravel：

    composer create-project laravel/laravel --prefer-dist blog

> **[Summer](http://github.com/summerblue)：**  安装 Laravel 5.1 LTS，请使用以下命令：

    composer create-project laravel/laravel your-project-name --prefer-dist "5.1.*"

<a name="configuration"></a>
## 配置信息

<a name="basic-configuration"></a>
### 基本配置

所有 Laravel 框架的配置文件都放置在 `config` 目录下。每个选项都有说明，因此你可以轻松地浏览这些文档，并且熟悉这些选项配置。

#### 目录权限

安装 Laravel 之后，你必须设置一些权限。`storage` 和 `bootstrap/cache` 目录必须让服务器有写入权限。如果你使用 [Homestead](/docs/{{version}}/homestead) 虚拟机，那么这些权限应该已经被设置完成。

#### 应用程序密钥

在你安装完 Laravel 后，首先需要做的事情是设置一个随机字符串到应用程序密钥。假设你是通过 Composer 或是 Laravel 安装工具安装的 Laravel，那么这个密钥已经通过 `key:generate` 命令帮你设置完成。通常这个密钥会有 32 字符长。这个密钥可以被设置在 `.env` 环境文件中。如果你还没将 `.env.example` 文件重命名为 `.env`，那么你现在应该去设置下。**如果应用程序密钥没有被设置的话，你的用户 Session 和其它的加密数据都是不安全的！**

#### 其它设置

Laravel 几乎不需做任何其它设置就可以马上使用，但是建议你先浏览 `config/app.php` 文件和对应的文档，这里面包含着一些选项，如`时区`和`语言环境`，你可以根据自己的应用程序来做修改。

你也可以设置 Laravel 的几个附加组件，像是：

- [缓存](/docs/{{version}}/cache#configuration)
- [数据库](/docs/{{version}}/database#configuration)
- [Session](/docs/{{version}}/session#configuration)

一旦 Laravel 安装完成，你应该立即[设置本机环境](/docs/{{version}}/installation#environment-configuration)。

<a name="pretty-urls"></a>
#### 优雅链接

**Apache**

Laravel 框架通过 `public/.htaccess` 文件来让网址不需要 `index.php`。如果你的服务器是使用 Apache，请确认是否有开启 `mod_rewrite` 模块。

如果 Laravel 附带的 `.htaccess` 文件在 Apache 中无法使用的话，请尝试下方的做法：

    Options +FollowSymLinks
    RewriteEngine On

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

**Nginx**

若你使用了 Nginx，则可以在网站设置中增加以下设置，以开启「优雅链接」：

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

如果你使用了 [Homestead](/docs/{{version}}/homestead) 的话，它将会自动的帮你设置好优雅链接。

<a name="environment-configuration"></a>
### 环境配置

应用程序常常需要根据不同的运行环境设置不同的值。例如，你会希望在本机开发环境上有与正式环境不同的缓存驱动。只需通过配置文件就可轻松完成。

Laravel 使用 Vance Lucas 的 [DotEnv](https://github.com/vlucas/phpdotenv) PHP 函数库来实现项目内环境变量的控制，在安装好的全新 Laravel 应用程序里，在根目录下会包含一个 `.env.example` 文件。如果你通过 Composer 安装 Laravel，这个文件将自动被更名为 `.env`，否则你只能手动更改文件名。

当你的应用程序收到请求时，这个文件所有的变量都会被加载到 PHP 超级全局变量 `$_ENV` 里。你可以使用辅助函数 `env` 来获取这些变量的值。事实上，如果你阅读过 Laravel 的相关配置文件，你会注意到里面有几个选项已经在使用着这个辅助函数！

根据本机服务器或者正式环境的需求的不同，可自由修改环境变量。但是，`.env` 文件不应该被提交到应用程序的版本控制系统，因为每个开发人员或服务器在使用应用程序时，可能需要不同的环境配置。

如果你是某个团队的开发者，不妨将 `.env.example` 文件放进你的应用程序。通过样本配置文件里的预设值，你团队中的其他开发人员就可以清楚地知道，在运行你的应用程序时有哪些环境变量是必须有的。

#### 获取目前应用程序的环境

应用程序的当前环境是由 `.env` 文件中的 `APP_ENV` 变量所决定的。你可以通过 `App` [facade](/docs/{{version}}/facades) 的 `environment` 方法来获取该值：

    $environment = App::environment();

你也可以传递参数至 `environment` 方法来确认当前环境是否与参数相符合：

    if (App::environment('local')) {
        // 环境是 local
    }

    if (App::environment('local', 'staging')) {
        // 环境是 local 或 staging...
    }

也可通过 `app` 辅助函数获取应用程序实例：

    $environment = app()->environment();

<a name="configuration-caching"></a>
### 缓存配置信息

为了让应用程序的速度获得提升，可以使用 Artisan 命令 `config:cache` 将所有的配置文件缓存到单个文件。通过此命令将所有的设置选项合并成一个文件，让框架能够更快速的加载。

你应该将运行 `php artisan config:cache` 命令作为部署工作的一部分。此命令不应该在本机开发时运行，因为设置选项会在应用程序的开发时经常变动。

> **[Summer](http://github.com/summerblue)：** 想知道更多 Laravel 程序调优的技巧？请参阅：[Laravel 5 程序优化技巧](https://phphub.org/topics/2020)

<a name="accessing-configuration-values"></a>
### 获取设置值

你可以使用 `config` 辅助函数获取你的设置值，设置值可以通过「点」语法来获取，其中包含了文件与选项的名称。你也可以指定一个默认值，当该设置选项不存在时就会返回默认值：

    $value = config('app.timezone');

若要在运行期间修改设置值，请传递一个数组至 `config` 辅助函数：

    config(['app.timezone' => 'America/Chicago']);

<a name="naming-your-application"></a>
### 命名你的应用程序

在安装完 Laravel 后，你可以来「命名」你的应用程序。默认情况下，`app` 的目录的命名空间是 `App`，然后会通过 Composer 使用 [PSR-4 自动加载标准](http://www.php-fig.org/psr/psr-4/) 来自动加载。不过，你可以轻松地通过 Artisan 命令 `app:name` 来修改命名空间，以配合你的应用程序名称。

举例来说，假设你的应用程序叫做「Horsefly」，则可以在安装完的根目录运行下方的命令：

    php artisan app:name Horsefly

你可以随意重命名你的应用程序，如果你愿意的话也可以继续保持命名空间为 `App`。

<a name="maintenance-mode"></a>
## 维护模式

当你的应用程序处于维护模式时，所有传递至应用程序的请求都会显示出一个自定义视图。在你更新应用或进行性能维护时，这么做可以很轻松的「关闭」整个应用程序。维护模式会检查包含在应用程序的默认的中间件堆栈。如果应用程序处于维护模式，则 `HttpException` 会抛出 503 的状态码。

启用维护模式，只需要运行 Artisan 命令 `down`：

    php artisan down

关闭维护模式，请使用 Artisan 命令 `up`：

    php artisan up

### 维护模式的响应模板

维护模式的默认模板放在 `resources/views/errors/503.blade.php`。

### 维护模式与队列

当应用程序处于维护模式中时，将不会处理任何 [队列工作](/docs/{{version}}/queues)。所有的队列工作将会在应用程序离开维护模式后被继续运行。
