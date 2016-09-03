# 安装

- [安装](#installation)
    - [运行环境要求](#server-requirements)
    - [安装 Laravel](#installing-laravel)
    - [配置信息](#configuration)

<a name="installation"></a>
## 安装

<a name="server-requirements"></a>
### 运行环境要求

Laravel 框架会有一些系统上的要求。当然，这些要求在 [Laravel Homestead](/docs/{{version}}/homestead) 虚拟机上都已经完全配置好了，强烈建议使用 Homestead 作为本地开发环境。

系统要求为以下：

<div class="content-list" markdown="1">
- PHP >= 5.5.9
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension
</div>

> 译者注：强烈推荐使用 Homestead 作为开发环境，尤其是新手，可以避免很多不必要的麻烦。线上环境可以参考 [Homestead 的环境部署脚本](https://github.com/laravel/settler/blob/master/scripts/provision.sh) 进行部署。

<a name="installing-laravel"></a>
### 安装 Laravel

Laravel 使用 [Composer](http://getcomposer.org) 来管理代码依赖。所以，在使用 Laravel 之前，请先确认你的电脑上安装了 Composer。

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


<a name="configuration"></a>
### Configuration

#### Public 目录

安装完成后，你应该指定 Web 服务器的网站根目录到 `public` 文件夹上。`index.php` 文件是 Laravel 的主要入口文件。

#### Configuration Files

所有 Laravel 框架的配置文件都放置在 `config` 目录下。每个选项都有说明，请仔细阅读这些说明，并熟悉这些选项配置。

#### 目录权限

安装 Laravel 之后，你必须设置一些文件目录权限。`storage` 和 `bootstrap/cache` 目录必须让服务器有写入权限。如果你使用 [Homestead](/docs/{{version}}/homestead) 虚拟机，那么这些权限已经被设置好了。

#### 应用程序密钥

在你安装完 Laravel 后，首先需要做的事情是设置一个随机字符串的密钥。假设你是通过 Composer 或是 Laravel 安装工具安装的 Laravel，那么这个密钥已经通过 `key:generate` 命令帮你设置完成。通常这个密钥会有 32 字符长。这个密钥可以被设置在 `.env` 环境文件中。如果你还没将 `.env.example` 文件重命名为 `.env`，那么你现在应该去设置下。

**如果应用程序密钥没有被设置的话，你的用户 Session 和其它的加密数据都是不安全的！**

#### 其它设置

Laravel 几乎不需做任何其它设置就可以马上使用，但是建议你先浏览 `config/app.php` 文件和对应的文档，这里面包含着一些选项，如 `时区` 和 `语言环境`，你可以根据应用程序的情况来修改。

你也可以设置 Laravel 的几个附加组件，像是：

- [缓存](/docs/{{version}}/cache#configuration)
- [数据库](/docs/{{version}}/database#configuration)
- [Session](/docs/{{version}}/session#configuration)

一旦 Laravel 安装完成，你应该立即 [设置本机环境](/docs/{{version}}/installation#environment-configuration)。

