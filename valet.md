# Laravel Valet

- [简介](#introduction)
    - [选择 Valet 还是 Homestead](#valet-or-homestead)
- [安装](#installation)
- [发行说明](#release-notes)
- [服务站点](#serving-sites)
    - [「Park」命令](#the-park-command)
    - [「Link」命令](#the-link-command)
    - [使用 TLS 构建完全站点](#securing-sites)
- [分享站点](#sharing-sites)
- [查看日志](#viewing-logs)
- [自定义 Valet 驱动](#custom-valet-drivers)
- [其他 Valet 命令](#other-valet-commands)

<a name="introduction"></a>
## 简介

Valet 是专为 Mac 提供的极简主义开发环境，没有 Vagrant、Apache、Nginx，也没有 `/etc/hosts` 文件，甚至可以通过本地隧道公开共享你的站点。

在 Mac 中，当你启动机器时，Laravel Valet 总是在后台运行 [Caddy](https://caddyserver.com) Web 服务器，然后通过使用 [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq)，Valet 将所有请求代理到 `*.dev` 域名并指向本地机器安装的站点。这样一个极速的 Laravel 开发环境只需要占用 7M 内存。

Valet 并不是想要替代 Vagrant 或者 Homestead，只是提供了另外一种选择，更加灵活、方便、以及占用更小的内存空间。

Valet 为我们提供了以下软件和工具的支持：

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
- 静态 HTML
</div>

当然，你还可以通过 [自定义的驱动](#custom-valet-drivers) 来扩展 Valet。

<a name="valet-or-homestead"></a>
### 选择 Valet 还是 Homestead

正如你所知道的，Laravel 还提供了另外一个开发环境 [Homestead](/docs/{{version}}/homestead)。Homestead 和 Valet 的不同之处在于两者的目标受众和本地开发方式。Homestead 提供了一个完整的包含自动化 Nginx 配置的 Ubuntu 虚拟机，如果你需要一个完整的虚拟化 Linux 开发环境或者使用的是 Windows/Linux 操作系统，那么 Homestead 无疑是最佳选择。

Valet 只支持 Mac，并且要求本地安装了 PHP 和数据库服务器，这可以通过使用 Homebrew 命令轻松实现（`brew install php70` 以及 `brew install mariadb`），Valet 通过最小的资源消耗提供了一个极速的本地开发环境，如果你只需要 PHP / MySQL 并且不需要完整的虚拟化开发环境，那么 Valet 将是最好的选择。

最后，Valet 和 Homestead 都是配置本地 Laravel 开发环境的好帮手，选择使用哪一个取决于你个人的喜好或团队的需求。

<a name="installation"></a>
## 安装

**Valet 要求 Mac 操作系统和 [Homebrew](http://brew.sh/)。安装之前，需要确保没有其他程序如 Apache 或 Nginx 绑定到本地的 80 端口。安装步骤如下：**

<div class="content-list" markdown="1">
- 安装或者使用 `brew update` 更新 [Homebrew](http://brew.sh/) 到最新版本；
- 使用 Homebrew 安装 PHP 7.0 `brew install homebrew/php/php70`;
- 通过 Composer 安装 Valet `composer global require laravel/valet`。 请确保 `~/.composer/vendor/bin` 在系统路径中。
- 运行 `valet install` 命令，这将会配置并安装 Valet 和 DnsMasq，然后注册 Valet 后台随机启动。
</div>

安装完 Valet 后，尝试使用命令如 `ping foobar.dev` 在终端 ping 一下任意 `*.dev` 域名，如果 Valet 安装正确就会看到来自 `127.0.0.1` 的响应。

每次系统启动的时候 Valet 后台会自动启动，而不需要再次手动运行 `valet start` 或 `valet install`。

#### 使用其他顶级域名

默认的，Valet 使用 `.dev` 顶级域名，如果你想使用其他顶级域名的话，可以使用这个命令来修改：

    valet domain tld-name

举例，如果你想使用 `.app` 来作为顶级域名的话，你可以运行：

    valet domain app

配置完成后 Valet 会自动为你的 `.app` 顶级域名提供服务。

#### 数据库

如果你需要数据库，可以使用 `brew install mariadb` 命令行来安装 MariaDB。你可以使用 host 为 `127.0.0.1`，用户名 `root`，密码为空进行连接。

<a name="release-notes"></a>
## 发行说明

### 版本 1.1.5

Valet 1.1.5 版本带来了许多的内部优化。

#### 更新说明

在使用命令 `composer global update` 更新完成后，你应该在命令行运行 `valet install` 来完成更新。

### 版本 1.1.0

Valet 1.1.0 版本带了很多优化，其中最大的优化是使用 [Caddy](https://caddyserver.com/) 替换掉 PHP 内置的 Web 服务器，Caddy 支持多域名之间的请求，这在之前的 PHP 内置服务器中会造成阻塞。

#### 更新说明

在使用命令 `composer global update` 更新完成后，你应该在命令行运行 `valet install` 来完成更新。

<a name="serving-sites"></a>
## 启动服务站点

Valet 安装完成后，就可以启动服务站点，Valet 为此提供了两个命令：`park` 和 `link`。

<a name="the-park-command"></a>
**`park` 命令**

<div class="content-list" markdown="1">
- 在Mac中创建一个新目录，例如 `mkdir ~/Sites`，然后进入这个目录 `cd ~/Sites` 并运行 `valet park`。这个命令会将当前所在目录作为 Web 根目录。
- 接下来，在新建的目录中创建一个新的 Laravel 站点：`laravel new blog`。
- 在浏览器中访问 `http://blog.dev`。
</div>

**这就是我们要做的全部工作**。现在，所有在 Sites 目录中创建的 Laravel 项目都可以通过 `http://folder-name.dev` 这种方式在浏览器中访问，是不是很方便？

<a name="the-link-command"></a>
**`link` 命令**

`link` 命令也可以用于本地 Laravel 站点，这个命令在你想要在目录中提供单个站点时很有用。

<div class="content-list" markdown="1">
- 要使用这个命令，先切换到你的某个项目并运行 `valet link app-name`，这样Valet会在 `~/.valet/Sites` 中创建一个符号链接指向当前工作目录。
- 运行完 `link` 命令后，可以在浏览器中通过 `http://app-name.dev` 访问。
</div>

要查看所有的链接目录，可以运行 `valet links` 命令。你也可以通过 `valet unlink app-name` 来删除符号链接。

<a name="securing-sites"></a>
**安全站点**

默认情况下，Valet 提供简单的 HTTP Web 服务，如果你想利用 HTTP/2 提供加密的 TLS，你可以使用 `secure` 命令，例如，你有一个站点是 `laravel.dev` ，你可以使用以下命令来使其更加安全：

    valet secure laravel

使用 `unsecure` 命令可以去除 `secure` 增加的安全加密：

    valet unsecure laravel

<a name="sharing-sites"></a>
## 分享站点

Valet 还提供了一个命令用于将本地站点共享给其他人，这不需要任何额外工具即可实现。

要共享站点，切换到站点所在目录并运行 `valet share`，这会生成一个可以公开访问的 URL 并复制到剪贴板，以便你直接复制到浏览器地址栏，就是这么简单。

要停止共享站点，使用 `Control + C` 即可。

<a name="viewing-logs"></a>
## 查看日志

如果你想要实时地在终端显示所有站点的日志，可以运行 `valet logs` 命令，这会在终端显示新产生的日志。

<a name="custom-valet-drivers"></a>
## 自定义 Valet 驱动

你还可以编写自定义的 Valet 驱动为运行于非 Valet 原生支持的 PHP 应用提供服务。安装完 Valet 时会创建一个 `~/.valet/Drivers` 目录，该目录中有一个  `SampleValetDriver.php` 文件，这个文件中有一个演示如何编写自定义驱动的示例。编写一个驱动只需要实现三个方法：`serves`、`isStaticFile` 和 `frontControllerPath`。

这三个方法接收 `$sitePath`、`$siteName` 和 `$uri` 值作为参数，其中 `$sitePath` 表示站点目录，如 `/Users/Lisa/Sites/my-project`，`$siteName`表示主域名部分，如 `my-project`，而 `$uri` 则是输入的请求地址，如 `/foo/bar`。

编写好自定义的Valet驱动后，将其放到 `~/.valet/Drivers` 目录并遵循  `FrameworkValetDriver.php` 这种命名方式，举个例子，如果你是在为 Wordpress 编写自定义的 valet 驱动，对应的文件名称为 `WordPressValetDriver.php`。

下面我们来具体讨论并演示自定义 Valet 驱动需要实现的三个方法。

#### `serves` 方法

如果自定义驱动需要继续处理输入请求，`serves` 方法会返回 `true`，否则该方法返回 `false`。因此，在这个方法中应该判断给定的 `$sitePath` 是否包含你服务类型的项目。

例如，假设我们编写的是 `WordPressValetDriver`，那么对应 `serves` 方法如下：

    /**
     * 检查是服务类型的项目
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

`isStaticFile` 方法会判断输入请求是否是静态文件，例如图片或样式文件，如果文件是静态的，该方法会返回磁盘上的完整路径，如果输入请求不是请求静态文件，则返回 `false`：

    /**
     * 检测请求是否是静态内容的请求
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

> **注意：** `isStaticFile` 方法只有在 `serves` 方法返回 `true` 并且请求 URI 不是 `/` 的时候才会被调用。

#### `frontControllerPath` 方法

`frontControllerPath` 方法会返回前端控制器的完整路径，通常是 `index.php`：

    /**
     * 获取前端控制器的完整路径
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

Command  | Description
------------- | -------------
`valet forget` | 从 `parked` 目录运行该命令以便从 `parked` 目录列表中移除该目录
`valet paths` | 查看你的 `parked` 路径
`valet restart` | 重启
`valet start` | 启动
`valet stop` | 停止
`valet uninstall` | 卸载



--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译。
> 
> 文档永久地址： http://d.laravel-china.org