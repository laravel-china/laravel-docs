# Laravel 安装指南

- [安装](#installation)
    - [服务器要求](#server-requirements)
    - [安装 Laravel](#installing-laravel)
    - [配置](#configuration)
- [Web 服务器配置](#web-server-configuration)
    - [优雅链接](#pretty-urls)

<a name="installation"></a>
## 安装

> {video} 如果你喜欢看视频学习，Laracasts 为你提供了免费而又全面的 Laravel 教程。

<a name="server-requirements"></a>
### 服务器要求

Laravel 框架对系统有一些要求。所有这些要求 Laravel Homestead 虚拟机都能满足，因此强烈建议你使用 Homestead 作为你本地的 Laravel 开发环境。

但如果你不使用 Homestead，则需要确保你的服务器符合以下要求：

<div class="content-list" markdown="1">
- PHP >= 7.0.0
- PHP OpenSSL 扩展
- PHP PDO 扩展
- PHP Mbstring 扩展
- PHP Tokenizer 扩展
- PHP XML 扩展
  </div>

<a name="installing-laravel"></a>
### 安装 Laravel

Laravel 利用 [Composer](https://getcomposer.org) 来管理依赖。所以，在使用 Laravel 之前，请确保你的机器上安装了 Composer。

#### 通过 Laravel 安装器

首先，使用 Composer 下载 Laravel 安装程序：

    composer global require "laravel/installer"
确保 `$HOME/.composer/vendor/bin` 目录（或你的操作系统的等效目录）已经放在你的环境变量 $PATH 中，以便系统可以找到 `laravel` 的可执行文件。

安装之后， `laravel new` 命令会在你指定的目录中创建一个新的 Laravel 项目。例如，`laravel new blog` 命令会创建一个名为 `blog` 的目录，其中包含所有已经安装好的 Laravel 的依赖项：

    laravel new blog

#### 通过 Composer 创建项目

或者，你还可以通过在终端中运行 `create-project` 命令来安装 Laravel：

    composer create-project --prefer-dist laravel/laravel blog "5.5.*"

#### 本地开发服务器

如果你在本地安装了 PHP，并且想使用 PHP 内置的开发服务器来为你的应用程序提供服务，那就使用 Artisan 命令  `serve`。这个命令会在 `http://localhost:8000` 上启动开发服务器：

    php artisan serve
当然，对于本地开发来说，最好的选择还是 [Homestead](/docs/{{version}}/homestead) 和 [Valet](/docs/{{version}}/valet)。

<a name="configuration"></a>
### 配置

#### Public 目录

安装 Laravel 之后，你要将 Web 服务器的根目录指向 `public` 目录。该目录下的 `index.php` 文件将作为所有进入应用程序的 HTTP 请求的前端控制器。

#### 配置文件

Laravel 框架的所有配置文件都放在 `config` 目录中。每个选项都有注释，方便你随时查看文件并熟悉可用的选项。

#### 目录权限

安装完 Laravel 后，你可能需要给这两个文件配置读写权限：`storage` 目录和 `bootstrap/cache` 目录应该允许 Web 服务器写入，否则 Laravel 将无法运行。如果你使用的是 [Homestead](/docs/{{version}}/homestead) 虚拟机，这些权限已经为你设置好了。

#### 应用密钥

安装 Laravel 之后下一件应该做的事就是将应用程序的密钥设置为随机字符串。如果你是通过 Composer 或 Laravel 安装器安装的 Laravel，那这个密钥已经为你通过 `php artisan key:generate` 命令设置好了。

通常来说，这个字符串长度为 32 个字符。密钥可以在 `.env` 环境文件中设置。前提是你要将 `.env.example` 文件重命名为 `.env`。**如果应用程序密钥没有被设置，就不能确保你的用户会话和其他加密数据的安全！**

#### 更多的配置

除了以上的配置，Laravel 几乎就不需要再配置什么了。你随时就能开发！但是，可能的话，还是希望你查看 `config/app.php` 文件及其注释。它包含几个你可能想要根据你的应用来更改的选项，比如 `timezone` 和 `locale`。

你还可能想要配置 Laravel 的其他几个组件，例如：

<div class="content-list" markdown="1">
- [缓存](/docs/{{version}}/cache#configuration)
- [数据库](/docs/{{version}}/database#configuration)
- [会话](/docs/{{version}}/session#configuration)
  </div>

<a name="web-server-configuration"></a>
## Web 服务器配置

<a name="pretty-urls"></a>
### 优雅链接

#### Apache

Laravel 使用 `public/.htaccess` 文件来为前端控制器提供隐藏了 `index.php` 的优雅链接。如果你的 Laravel 使用了 Apache 作为服务容器，请务必启用 `mod_rewrite `模块，让服务器能够支持 `.htaccess` 文件的解析。

如果 Laravel 附带的 `.htaccess` 文件不起作用，就尝试用下面的方法代替：

    Options +FollowSymLinks
    RewriteEngine On

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

#### Nginx

如果你使用的是 Nginx，在你的站点配置中加入以下内容，它将会将所有请求都引导到 `index.php` 前端控制器：

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

当然，使用 [Homestead](/docs/{{version}}/homestead) 或者 [Valet](/docs/{{version}}/valet) 时，你无需配置这些。

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  翻译  | Stay Hungry. Stay Foolish. |


---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
