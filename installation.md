# 安装

- [安装](#installation)
- [设置](#configuration)
    - [基本设置](#basic-configuration)
    - [环境配置](#environment-configuration)
    - [设置缓存](#configuration-caching)
    - [取得设置值](#accessing-configuration-values)
    - [命名你的应用程序](#naming-your-application)
- [维护模式](#maintenance-mode)

<a name="installation"></a>
## 安装

### 服务器需求

Laravel 框架有一些系统上的需求。当然，[Laravel Homestead](/docs/{{version}}/homestead) 虚拟机都能满足这些需求：

<div class="content-list" markdown="1">
- PHP >= 5.5.9
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension
</div>

<a name="install-laravel"></a>
### 安装 Laravel

Laravel 使用 [Composer](http://getcomposer.org) 来管理相依性。所以，在使用 Laravel 之前，你必须确认电脑上是否安装了 Composer。

#### 透过 Laravel 安装工具

首先，使用 Composer 下载 Laravel 安装包：

    composer global require "laravel/installer"

请确定把 `~/.composer/vendor/bin` 路径放置于你的 PATH 里，这样你的系统才能找到 `laravel` 运行档。

一旦安装完成后，就可以使用 `laravel new` 命令在指定的目录创建一份新安装的 Laravel 项目，例如：`laravel new blog` 将会在当前目录下创建一个叫 `blog` 的目录，此目录里面存放着新安装的 Laravel 和相依代码。这个安装方法比透过 Composer 安装速度快上许多：

    laravel new blog

#### 透过 Composer Create-Project

除此之外，你也可以透过 Composer 在命令行运行 `create-project` 命令来安装 Laravel：

    composer create-project laravel/laravel --prefer-dist blog

<a name="configuration"></a>
## 设置

<a name="basic-configuration"></a>
### 基本设置

所有 Laravel 框架的设置文件都放置在 `config` 目录下。每个选项都有说明，因此你可以轻松地浏览这些文档，并且熟悉这些选项配置。

#### 目录权限

安装 Laravel 之后，你必须设置一些权限。`storage` 和 `bootstrap/cache` 目录必须让服务器有写入权限。如果你使用 [Homestead](/docs/{{version}}/homestead) 虚拟机，那么这些权限应该已经被设置完成。

#### 应用程序密钥

在你安装完 Laravel 后，首先需要做的事情是设置一个随机字符串到应用程序密钥。假设你是透过 Composer 或是 Laravel 安装工具安装 Laravel，那么这个密钥已经透过 `key:generate` 命令帮你设置完成。通常这个密钥应该有 32 字符长。这个密钥可以被设置在 `.env` 环境文件中。如果你还没将 `.env.example` 文件重命名为 `.env`，那么你应该现在开始。**如果应用程序密钥没有被设置的话，你的用户 Sessions 和其他的加密数据都是不安全的！**

#### 其他设置

Laravel 几乎不需设置就可以马上使用，你可以自由的开始开发！当然，你可以浏览 `config/app.php` 文件和对应的文档。它包含数个选项，如`时区`和`语言环境`，你不仿根据你的应用程序来做修改。

你也可以设置 Laravel 的几个附加组件，像是：

- [缓存](/docs/{{version}}/cache#configuration)
- [数据库](/docs/{{version}}/database#configuration)
- [Session](/docs/{{version}}/session#configuration)

一旦 Laravel 安装完成，你应该同时[设置本机环境](/docs/{{version}}/installation#environment-configuration)。

<a name="pretty-urls"></a>
#### 优雅链结

**Apache**

Laravel 框架透过 `public/.htaccess` 文件来让网址中不需要 `index.php`。如果你的服务器是使用 Apache，请确认是否有开启 `mod_rewrite` 模块。

假设 Laravel 附带的 `.htaccess` 档在 Apache 无法作用的话，请尝试下方的做法：

    Options +FollowSymLinks
    RewriteEngine On

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

**Nginx**

若使用 Nginx ，可以在你的网站设置中增加下面的设置，以开启「优雅链接」：

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

当然，如果你使用 [Homestead](/docs/{{version}}/homestead) 的话，优雅链结会自动的帮你设置完成。

<a name="environment-configuration"></a>
### 环境配置

通常应用程序常常需要根据不同的运行环境而有不同的设置值。例如，你会希望在你的本机开发环境上会有与正式环境不同的暂存驱动。只要透过设置文件，就可以轻松完成。

Laravel 透过 Vance Lucas 的 [DotEnv](https://github.com/vlucas/phpdotenv) PHP 函数库来达到这项需求。在全新安装好的 Laravel 里，你的应用程序的根目录下会包含一个 `.env.example` 文件。如果你透过 Composer 安装 Laravel，这个文件将自动被更名为 `.env`，不然你应该手动更改文件名。

当你的应用程序收到请求时，这个文件所有的变量会被加载到 PHP 超级全域变量 `$_ENV` 里。你可以使用辅助方法 `env` 来取得这些变量的值。事实上，如果你检阅过 Laravel 的设置文件，你会注意到有几个选项已经在使用这个辅助方法！

根据你的本机服务器或者正式环境需求，你可以自由的修改你的环境变量。但是，你的 `.env` 文件不应该被提交到应用程序的版本控制系统，因为每个开发人员或服务器在使用你的应用程序时，可能需要不同的环境配置。

如果你是一个团队的开发者，不妨将 `.env.example` 文件放进你的应用程序。透过例子配置文件里的预留值，你的团队中其他开发人员可以清楚地看到，在运行你的应用程序时，有哪些环境变量是必须的。

#### 取得目前应用程序的环境

应用程序的当前环境是由 `.env` 文件中的 `APP_ENV` 变量所决定。你可以透过 `App` [facade](/docs/{{version}}/facades) 的 `environment` 方法取得该值：

    $environment = App::environment();

你也可以传递参数至 `environment` 方法中，来确认目前的环境是否与参数相符合：

    if (App::environment('local')) {
        // 环境是 local
    }

    if (App::environment('local', 'staging')) {
        // 环境是 local 或 staging...
    }

也能透过 `app` 辅助方法取得应用程序实例：

    $environment = app()->environment();

<a name="configuration-caching"></a>
### 设置缓存

为了让你的的应用程序提升一些速度，你可以使用 Artisan 命令 `config:cache` 将所有的配置文件暂存到单一文件。透过命令会将所有的设置选项合并成一个文件，让框架能够快速加载。

你应该将运行 `php artisan config:cache` 命令作为部署工作的一部分。此命令不应该在本机开发的时候运行，因为设置选项需要根据你应用程序的开发而经常变动。

<a name="accessing-configuration-values"></a>
### 取得设置值

你可以很轻松的使用 `config` 辅助方法取得你的设置值。设置值可以透过「点」语法来取得，其中包含了文件与选项的名称。也可以指定默认值，当该设置选项不存在时就会返回默认值：

    $value = config('app.timezone');

若要在运行期间修改设置值，请传递一个数组至 `config` 辅助方法：

    config(['app.timezone' => 'America/Chicago']);

<a name="naming-your-application"></a>
### 命名你的应用程序

在安装完成 Laravel 后，你可以「命名」你的应用程序。默认情况下，`app` 的目录的命名空间是 `App`，然后会透过 Composer 使用 [PSR-4 自动加载标准](http://www.php-fig.org/psr/psr-4/) 来自动加载。不过，你可以轻松地透过 Artisan 命令 `app:name` 来修改命名空间，以配合你的应用程序名称。

举例来说，假设你的应用程序叫做「Horsefly」，你可以在安装完的根目录运行下方的命令：

    php artisan app:name Horsefly

重命名你的应用程序是完全自由的，如果你希望的话也可以保持命名空间为 `App`。

<a name="maintenance-mode"></a>
## 维护模式

当你的应用程序处于维护模式时，所有的传递至应用程序的请求都会显示一个自定的视图。在你要更新或进行维护作业时，这么做可以很轻松的「关闭」整个应用程序。维护模式会检查包含在应用程序的默认的介层堆栈。如果应用程序处于维护模式，`HttpException` 会抛出 503 的状态码。

启用维护模式，只需要运行 Artisan 命令 `down`：

    php artisan down

关闭维护模式，请使用 Artisan 命令 `up`：

    php artisan up

### 维护模式的回应模板

维护模式回应的默认模板放在 `resources/views/errors/503.blade.php`。

### 维护模式与队列

当应用程序处于维护模式中，将不会处理任何[队列工作](/docs/{{version}}/queues)。所有的队列工作将会在应用程序离开维护模式后继续被进行。
