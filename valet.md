# Laravel Valet

- [简介](#introduction)
    - [选择 Valet 还是 Homestead](#valet-or-homestead)
- [安装](#installation)
    - [升级](#upgrading)
- [服务站点](#serving-sites)
    - [「Park」 命令](#the-park-command)
    - [「Link」 命令](#the-link-command)
    - [使用 TLS 构建站点](#securing-sites)
- [分享站点](#sharing-sites)
- [查看日志](#viewing-logs)
- [自定义 Valet 驱动](#custom-valet-drivers)
- [其他 Valet 命令](#other-valet-commands)

<a name="introduction"></a>
## 简介

Valet 是一个 MAC 上的 Laravel 极简主义开发环境。不需要安装 Vagrant、 Apache、Nginx，也不需要更改 `/etc/hosts` 文件。甚至你能在本地局域网公开的分享你的站点。 _Yeah, we like it too._

Laravel Valet 安装到你的 MAC 中时，当你启动你的机器时在后台总是运行 [Caddy](https://caddyserver.com) web 服务器。 然后，使用 [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq) 轻量级DNS转发服务器，Valet 将所有请求代理到 `*.dev` 域名并指向你本地机器安装的站点。

换句话说，一个极速的 Laravel 开发环境大概只需要 7 MB 左右内存。Valet 不是一个完整的想替换 Vagrant 或者 Homestead 的方案，仅仅是提供一个好的替换选择 —— 在你想要一个更加灵活的基础开发环境，或者是喜欢极致快速，亦或者是工作在一台内存有限的机器上。

开箱即用，Valet 提供以下框架或项目的支持，但不仅限于此:

<div class="content-list" markdown="1">
- [Laravel](https://laravel.com)
- [Lumen](https://lumen.laravel.com)
- [Symfony](https://symfony.com)
- [Zend](http://framework.zend.com)
- [CakePHP 3](http://cakephp.org)
- [WordPress](https://wordpress.org)
- [Bedrock](https://roots.io/bedrock)
- [Craft](https://craftcms.com)
- [Statamic](https://statamic.com)
- [Jigsaw](http://jigsaw.tighten.co)
- Static HTML
</div>

不仅如此, 你还可以扩展 Valet 来使用你自己的 [自定义驱动](#custom-valet-drivers).

<a name="valet-or-homestead"></a>
###　选择 Valet 还是 Homestead

正如你可能知道的，Laravel 提供另一个本地开发环境 [Homestead](/docs/{{version}}/homestead) 。Homestead 和 Valet 的不同在于他们的目标受众和本地开发方式。Homestead 提供一个完整的 Ubuntu 虚拟机和自动化的 Nginx 配置。如果你需要一个完全地虚拟化的 Linux 开发环境或者是使用 Windows / Linux 系统，那么 Homestead 是一个最佳的选择。

Valet 只支持 Mac，并且要求你在本地机器安装 PHP 和一个数据库服务器。使用 [Homebrew](http://brew.sh/) 这是很容易做到的，只需要使用命令 `brew install php70` 和 `brew install mariadb`。Valet 提供一个最小资源消耗的本地极速开发环境，因此它是一个极好的开发环境，如果你只是需要 PHP / MySQL 而不需要完整的虚拟化开发环境。

Valet 和 Homestead 两者都是配置本地 Laravel 开发环境的非常好的选择。选择哪一个取决于你的个人喜好和你的团队需求。

<a name="installation"></a>
## 安装

**Valet 要求 mac 系统和 [Homebrew](http://brew.sh/)。 安装之前，你应该进行确认没有其他程序占用你本地机器的 80 端口，例如 Apache 或是 Nginx。**

<div class="content-list" markdown="1">
- 安装或升级 [Homebrew](http://brew.sh/) 到最新版本，使用命令 `brew update`。
- 安装 PHP 7.0 使用 Homebrew 命令 `brew install homebrew/php/php70`。
- 安装 Valet 通过 Composer 命令 `composer global require laravel/valet`。 并且确保 `~/.composer/vendor/bin` 目录在你的系统环境变量 「PATH」 中。
- 运行 `valet install` 命令。 这将会安装和配置 Valet 和 DnsMasq，并且注册 Valet 的守护进程随你的系统自动启动。
</div>

一旦 Valet 安装完成, 尝试在命令终端 ping 任何 `*.dev` 域名，例如 `ping foobar.dev`。 如果 Valet 是安装正确的你应该看到这个域名返回响应 `127.0.0.1`。

Valet 将会在你的机器每次启动时自动启动守护进程。没必要每次手动使用 `valet start` 或 `valet install` 命令。

#### 使用另一个域名

默认的，Valet 为你的项目提供服务使用 `.dev` 顶级域名。如果你喜欢其他域名，你可以使用 `valet domain tld-name` 命令。

例如，如果你喜欢使用 `.app` 替代 `.dev`，运行 `valet domain app` 然后 Valet 将会自动的将你的项目启动在 `*.app` 顶级域名下。

#### 数据库

如果你需要一个数据库，尝试一下 MariaDB ，使用命令 `brew install mariadb` 安装。使用 `root` 用户名和空密码你可以连接这个运行在 `127.0.0.1` 的本地数据库。

<a name="upgrading"></a>
### 升级

你可以升级你的 Valet ，在你的终端中使用 `composer global update` 命令。 升级之后，最佳实践是运行 `valet install` 命令，这样 Valet 能够获得额外的增强。

<a name="serving-sites"></a>
## 服务站点

一旦 Valet 安装完成，你就可以准备启动服务站点。 Valet 提供两个命令帮助你服务你的 Laravel 站点：`park` 和 `link`。

<a name="the-park-command"></a>
** `park` 命令 **

<div class="content-list" markdown="1">
- 在你的 MAC 中创建一个新目录，运行 `mkdir ~/Sites` 类似的命令。 接下来，运行 `cd ~/Sites` 和 `valet park` 命令。 这些命令将会将你当前工作目录注册为 Valet 搜索站点的路径。
- 然后，创建一个新的 Laravel 站点在这个目录： `laravel new blog`　。
- 在你的的浏览器中打开 `http://blog.dev`。
</div>

** 这就是所有要做的工作。 ** 现在, 任何你创建的 Laravel 项目只要放在这个目录中都可以通过类似 `http://folder-name.dev` 自动的配置工作。　

<a name="the-link-command"></a>
** `link` 命令 **

 `link` 命令也可以用于服务你的 Laravel 站点。 这个命令在你想要在一个目录提供单个站点而不是整个目录时很有用。

<div class="content-list" markdown="1">
- 在你的终端中进入你的某个项目并运行 `valet link app-name` 。 Valet 将会创建一个符号链接在 `~/.valet/Sites` 目录中，且指向当前工作目录。
- 之后运行 `link` 命令，你可以在浏览器中访问 `http://app-name.dev`。
</div>

想要查看包含所有链接目录的列表，运行 `valet links` 命令。你可以使用 `valet unlink app-name` 销毁一个符号链接。

<a name="securing-sites"></a>
** 使用 TLS 构建站点 **

默认的，Valet 服务站点使用普通的 HTTP 提供服务。然而，如果你想要利用 HTTP/2 提供加密的 TLS ，使用 `secure` 命令。例如，你有一个存在的 Valet 站点 `laravel.dev` ，你可以使用一些命让它得到保护:

    valet secure laravel

让一个站点 「不安全」或是恢复普通的 HTTP 站点，使用 `unsecure` 命令:

    valet unsecure laravel

<a name="sharing-sites"></a>
## 分享站点

Valet 甚至包含一个命令将你的本地站点分享给其他人。不需要安装附加软件即可以实现。

要分享一个站点，在终端中切换站点的目录并运行 `valet share` 命令。 一个公开的可访问的 URL 将会插入你的剪切板然后粘贴到你的浏览器地址栏就可以了，就是这么简单。

想要停止分享你的站点，使用 `Control + C` 组合键就可以撤销分享。

<a name="viewing-logs"></a>
## 查看日志

如果你想在终端查看所有的站点日志流，运行 `valet logs` 命令。新的日志将会显示在你的终端。这是一个极好的方法 —— 保持你的日志文件而不必离开你的终端。

<a name="custom-valet-drivers"></a>
## 自定义 Valet 驱动

你可以编辑属于自己的 Valet 「驱动」来运行那些 Valet 暂不支持的其他 PHP 框架或 CMS 系统。 Valet 安装完成时会创建 `~/.valet/Drivers` 目录，该目录中有一个 `SampleValetDriver.php` 文件。这个文件保含一个简单的驱动实现，以演示如何编写一个自定义驱动。 编写驱动只需要实现三个方法: `serves` ， `isStaticFile` ，和 `frontControllerPath` 。

这三个方法都接收 `$sitePath` ，`$siteName` ，和 `$uri` 值作为他们的参数。 `$sitePath` 参数是站点在你机器上的绝对路径，例如 `/Users/Lisa/Sites/my-project`。 `$siteName` 参数为「主机」/ 「站点名称」，是域名的一部分，例如 (`my-project`) 。而 `$uri` 参数则是输入的请求 URI 部分，如 (`/foo/bar`).

一旦完成你的自定义 Valet 驱动，将其放置在 `~/.valet/Drivers` 目录中使用 `FrameworkValetDriver.php` 这样的命名规范去命名它。 例如，如果你编写了一个自定义的 WordPress 驱动，你的文件名应该是 `WordPressValetDriver.php` 。

下面将简单实现每个方法，以帮助你理解和创建你自己的自定义驱动实现。

#### `serves` 方法

如果你的驱动要处理进入的请求， `serves` 方法应该返回 `true` 。否则应该返回 `false`。所以，在这个方法内你应该判断给定的参数 `$sitePath` 是否是一个包含你服务类型项目。

例如，假设我们编写一个 `WordPressValetDriver`。我们的 server 方法看起来会像以下这样:

    /**
     * 判断驱动服务请求是否正确。
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return void
     */
    public function serves($sitePath, $siteName, $uri)
    {
        return is_dir($sitePath.'/wp-admin');
    }

#### `isStaticFile` 方法

 `isStaticFile` 判断一个进入的请求的文件是否是「静态」的，例如一个图片或者样式请求。如果这个文件是静态的，这个方法会返回这个静态文件在磁盘上的绝对路径。 如果进入的请求不是一个静态文件，这个方法返回 `false`：

    /**
     * 判断进入请求是否是静态文件。
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string|false
     */
    public function isStaticFile($sitePath, $siteName, $uri)
    {
        if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
            return $staticFilePath;
        }

        return false;
    }

> {注意} `isStaticFile` 方法只有在 `serves` 方法返回 `true` 并且进入的请求 URI 不是 `/` 才会调用。

#### `frontControllerPath` 方法

 `frontControllerPath` 方法返回你的应用的「前端控制器」的绝对路径，通常是你的框架入口文件 「index.php」：

    /**
     * 获取应用的前端控制器绝对路径。
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string
     */
    public function frontControllerPath($sitePath, $siteName, $uri)
    {
        return $sitePath.'/public/index.php';
    }

<a name="other-valet-commands"></a>
  ## 其他 Valet 命令

命令  | 描述
------------- | -------------
`valet forget` | 在站点根目录运行这个命令，将会移除此目录不再为 Valet 的站点根目录。
`valet paths` | 查看所有的站点根目录。
`valet restart` | 重启 Valet 守护进程。
`valet start` | 启动 Valet 守护进程。
`valet stop` | 停止 Valet 守护进程。
`valet uninstall` | 完全卸载 Valet 守护进程。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@dongm2ez](https://github.com/dongm2ez)  | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/9032795?v=3&s=460">  |  翻译  | 编程界的小学生 |
