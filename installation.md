# Laravel 安装指南

- [安装](#installation)
    - [服务器要求](#server-requirements)
    - [安装 Laravel](#installing-laravel)
    - [配置](#configuration)
- [Web 服务器配置](#web-server-configuration)
    - [优雅链接](#pretty-urls)


<a name="installation"></a>
## 安装

> {video} 你是一个喜欢看视频的学习者么？ Laracasts 为刚刚使用这个框架的新手们提供了一个 [免费、深入的 Laravel视频](https://laracasts.com/series/laravel-from-scratch-2017) 。这是一个开始你学习之途的好地方。

<a name="server-requirements"></a>
### 服务器要求

Laravel 框架会有一些系统上的要求。当然，这些要求在 [Laravel Homestead](/docs/{{version}}/homestead) 虚拟机上都已经完全配置好了。所以，非常推荐你使用 Homestead 作为你的本地 Laravel 开发环境。

然而，如果你没有使用 Homestead ，你需要确保你的服务器上安装了下面的几个拓展：

<div class="content-list" markdown="1">
- PHP >= 5.6.4
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension
</div>

> 译者注：强烈推荐使用 Homestead 作为开发环境，尤其是新手，可以避免很多不必要的麻烦。线上环境可以参考 [Homestead 的环境部署脚本](https://github.com/laravel/settler/blob/master/scripts/provision.sh) 进行部署。

<a name="installing-laravel"></a>

### 安装 Laravel 

Laravel 使用 [Composer](https://getcomposer.org) 来管理代码依赖。所以，在使用 Laravel 之前，请先确认你的电脑上安装了 Composer。

#### 通过 Laravel 安装工具

首先，使用 Composer 下载 Laravel 安装包：

    composer global require "laravel/installer"

请确定你已将 `~/.composer/vendor/bin` 路径加到 PATH，只有这样系统才能找到 `laravel` 的执行文件。

一旦安装完成，就可以使用 `laravel new` 命令在指定目录创建一个新的 Laravel 项目，例如：`laravel new blog` 将会在当前目录下创建一个叫 `blog` 的目录，此目录里面存放着新安装的 Laravel 和代码依赖。这个方法的安装速度比通过 Composer 安装要快上许多：

    laravel new blog

因为代码依赖是直接一起打包安装的。

#### 通过 Composer Create-Project

除此之外，你也可以通过 Composer 在命令行运行 `create-project` 命令来安装 Laravel：

    composer create-project --prefer-dist laravel/laravel blog

#### 本地开发服务器

如果你在本地安装了 PHP，你可能希望像运行 PHP 内置的开发服务器一样来访问自己的应用程序，你可以使用 `serve` Artisan 命令来启动一个本地开发服务器，这样你就可以在 `http://localhost:8000` 来访问它。

	php artisan serve

不过有更健壮的本地开发选项可用，比如 [Homestead](/docs/{{version}}/homestead) 和 [Valet](/docs/{{version}}/valet)。

<a name="configuration"></a>
### 配置

#### 入口目录

在安装 Laravel 之后，你需要配置你的 Web 服务器的根目录为 `public` 目录。 这个目录的 `index.php` 文件作为所有 HTTP 请求进入应用的前端处理器。

#### 配置文件

Laravel 框架所有的配置文件都存放在 `config` 目录下。每个选项都被加入文档，所以你可以自由的浏览文件，轻松的熟悉你的选项。

#### 目录权限

安装 Laravel 之后， 你需要配置一些权限 。 `storage` 和 `bootstrap/cache` 目录应该允许你的 Web 服务器写入，否则 Laravel 将无法写入。如果你使用 [Homestead](/docs/{{version}}/homestead) 虚拟机，这些权限应该已经被设置好了。

#### 应用程序密钥

在你安装完 Laravel 后，首先需要做的事情是设置一个随机字符串的密钥。假设你是通过 Composer 或是 Laravel 安装工具安装的 Laravel，那么这个密钥已经通过 `key:generate` 命令帮你设置完成。

通常这个密钥会有 32 字符长。这个密钥可以被设置在 .env 环境文件中。如果你还没将 .env.example 文件重命名为 .env，那么你现在应该去设置下。**如果你没有设置应用程序密钥，你的用户 Session  和 其他加密数据将不安全！**


#### 额外配置

Laravel 几乎不需做任何其它设置就可以马上使用，但是建议你先浏览 `config/app.php` 文件和对应的文档，这里面包含着一些选项，如 `时区` 和 `语言环境`，你可以根据应用程序的情况来修改。

你也可以设置 Laravel 的几个附加组件，像是：

- [缓存](/docs/{{version}}/cache#configuration)
- [数据库](/docs/{{version}}/database#configuration)
- [Session](/docs/{{version}}/session#configuration)

一旦 Laravel 安装完成，你应该立即 [设置本机环境](/docs/{{version}}/installation#environment-configuration)。

<a name="web-server-configuration"></a>
## Web 服务器配置
<a name="pretty-urls"></a>
### 优雅链接

#### Apache
Laravel 框架通过 `public/.htaccess` 文件来让 URL 不需要 `index.php` 即可访问。在 Apache 启用 Laravel 之前，请确认是否有开启 mod_rewrite 模块，以便 `.htaccess` 文件发挥作用。

如果 Laravel 附带的 .htaccess 文件在 Apache 中无法使用的话，请尝试下方的做法：

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

#### Nginx

如果你使用 Nginx ，在你的网站配置中加入下述代码将会转发所有的请求到 `index.php` 前端控制器。

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

  	
当然如果你使用了 [Homestead](/docs/{{version}}/homestead) 或者 [Valet](/docs/{{version}}/valet) 的话， 它会自动的帮你设置好优雅链接。  

