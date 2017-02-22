# Laravel 的开发环境 Valet

- [简介](#introduction)
    - [选择 Valet 还是 Homestead](#valet-or-homestead)
- [安装](#installation)
    - [升级](#upgrading)
- [服务站点](#serving-sites)
    - [「Park」命令](#the-park-command)
    - [「Link」命令](#the-link-command)
    - [通过TLS让站点更安全](#securing-sites)
- [分享站点](#sharing-sites)
- [自定义 Valet 驱动](#custom-valet-drivers)
- [其他 Valet 命令](#other-valet-commands)

<a name="introduction"></a>
## 简介
Valet 是为 Mac 提供的极简主义开发环境，没有 Vagrant ，也无需 `/etc/hosts` 文件，甚至可以使用本地隧道公开共享你的站点。_Yeah, we like it too._

Laravel Valet 会在你的 Mac 上将 [Nginx](https://www.nginx.com/) 设置为随系统启动后台运行，然后使用 [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq) ， Valet 将所有的请求代理到 `*.dev` 域名并指向本地安装的站点目录。

换句话说，一个速度极快的 Laravel 开发环境仅仅需要占用 7MB 内存。 Valet 并不是想要替代 Vagrant 或者 Homestead，只是提供另外一种选择，更加灵活、方便、以及占用更小的内存。

开箱即用， Valet 为我们提供以下软件和工具支持，然而不仅限于此：

<div class="content-list" markdown="1">
- [Laravel](https://laravel.com)
- [Lumen](https://lumen.laravel.com)
- [Symfony](https://symfony.com)
- [Zend](https://framework.zend.com)
- [CakePHP 3](https://cakephp.org)
- [WordPress](https://wordpress.org)
- [Bedrock](https://roots.io/bedrock/)
- [Craft](https://craftcms.com)
- [Statamic](https://statamic.com)
- [Jigsaw](http://jigsaw.tighten.co)
- Static HTML
</div>

当然，你还可以通过 [自定义驱动](#custom-valet-drivers) 来扩展 Valet.

<a name="valet-or-homestead"></a>

### 选择 Valet 还是 Homestead

正如你所知道的， Laravel 提供另外一个开发环境 [Homestead](/docs/{{version}}/homestead) ， Homestead 和 Valet 不同之处在于两者的目标受众和本地开发方式。 Homestead 提供一个完整的包含自动化配置 Nginx 的 Ubuntu 虚拟机。如果你需要一个完整的虚拟化 Linux 开发环境或者是使用 Windows / Linux 操作系统，那么 Homestead 无疑是最佳选择。

Valet 只支持 Mac ，并且要求本地安装 PHP 和数据库服务器，这可以通过使用 [Homebrew](http://brew.sh/) 命令 `brew install php71` 和 `brew install mysql` 轻松实现。 Valet 通过最小的资源消耗提供一个本地极速开发环境，如果你只需要 PHP / MySQL 而不是完整的虚拟化开发环境，那么 Valet 将是最好的选择。

Valet 和 Homestead 都是配置你本地 Laravel 开发环境的好帮手。选择使用哪一个取决于你的个人喜好和团队需求。

<a name="installation"></a>

## 安装

**Valet 要求 macOS 和 [Homebrew](http://brew.sh/) 安装之前，你需要确保没有其他程序如   Apache 或者 Nginx 占用你本地机器的  80 端口。 安装步骤如下：**

<div class="content-list" markdown="1">
- 安装或更新 [Homebrew](http://brew.sh/) 到最新版本，使用命令 `brew update`
- 使用命令 `brew install homebrew/php/php71` 安装 PHP 7.1 。
- 通过 `composer global require laravel/valet` 命令安装 Valet 。请确定 `~/.composer/vendor/bin` 存在于你的系统环境变量 「PATH」 中。
- 运行 `valet install` 命令。它将会配置并安装 Valet 和 DnsMasq ，并且将 Valet 的进程注册为随系统启动
</div>

一旦你完成 Valet 安装，尝试使用像 `ping foobar.dev` 这样的命令在终端 ping 任意的 `*.dev` 域名。如果 Valet 正常安装你会看到来自 `127.0.0.1` 的响应

Valet 将会在每次系统启动时自动启动，而不需要你每次运行 `valet start` 或 `valet install`。

#### 使用其他的顶级域名

默认情况下， Valet 使用 `.dev` 顶级域名。如果你喜欢其他域名，可以使用 `valet domain tld-name` 命令。

例如，如果你想要使用 `.app` 来代替 `.dev` ，运行 `valet domain app` 然后 Valet 会自动使用 `*.app` 来为你的项目命名。

#### 数据库

如果你需要一个数据库，可以使用 `brew install mariadb` 命令试一试 MySQL 。你可以使用 host 为 `127.0.0.1` ，用户名 `root` ，密码为空进行数据库连接。

<a name="upgrading"></a>

### 升级

你可以使用 `composer global update` 命令升级你的 Valet 程序，升级之后，最好使用 `valet install` 命令更新 Valet 的配置文件。

#### 升级到 Valet 2.0
Valet 2.0 将 Valet 的底层 Web 服务从 Caddy 切换到了 Nginx 。在你升级到这个版本之前你应该运行下面的命令来停止并卸载已经启动的 Caddy 进程：

	valet stop
    valet uninstall

接下来，你应该升级到 Valet 的最新版本。取决于你安装 Valet 的方式，这通常通过 Git 或 Composer 来实现。如果你是通过 Composer 安装的 Valet 你应该使用下面的命令来更新到最新的主版本：

	composer global require laravel/valet

当新的 Valet 源代码下载好了之后，你应该运行 `install` 命令：

	valet install
    valet restart

在升级之后，它需要 re-park 或 re-link 你的站点。

<a name="serving-sites"></a>

## 服务站点

当 Valet 安装完成，你就可以启动服务站点。 Valet 为此提供了两个命令：`park` 和 `link`

<a name="the-park-command"></a>

**`park`命令 **

<div class="content-list" markdown="1">
- 在你的 Mac 中创建一个新的目录，例如 `mkdir ~/Sites` 。然后 `cd ~/Sites` 并且运行 `valet park` 。这个命令将在当前所在目录作为 Web 根目录， Valet 将会在这个目录中搜索站点。
- 接下来，在这个目录中创建一个新的 Laravel 站点： `laravel new blog` 。
- 在浏览器中访问 `http://blog.dev` 。
</div>

**这就是我们所要做的全部工作**。现在，所有在 Site 目录中的 Laravel 项目都可以通过 `http://folder-name.dev` 这种方式访问，是不是很方便。

<a name="the-link-command"></a>

**`link`命令 **

link 命令也被用来服务你的 Laravel 站点。这个命令在你想要在目录中提供单个站点是很有用。

<div class="content-list" markdown="1">
- 要使用这个命令，在你的终端中切换到你的某个项目并运行 `valet link app-name` 。 Valet 将会在 `~/.valet/Sites` 中创建一个符号链接并指向当前工作目录。
- 运行完 `link` 命令, 你可以在浏览器中通过 `http://app-name.dev` 来访问站点。
</div>

要查看所有的链接目录，运行 `valet links` 命令。你也可以通过 `valet unlink app-name` 来删除符号链接。

>  {tip} 你可以通过使用 `valet link` 将多个（子）域名指向同一个应用，要添加子域名或其它域名到应用，可以在应用目录下运行 `valet link subdomain.app-name` 。

<a name="securing-sites"></a>

**通过TLS让站点更安全**

默认的情况下， Valet 通过纯 HTTP 协议服务网站。然而，如果你想利用 HTTP/2 提供加密的 TLS ，你可以使用 `secure` 命令。例如，你有一个站点 `laravel.dev` ，可以使用以下命令让其更安全：

	valet secure laravel

想恢复一个站点到普通的 HTTP 使用 `unsecure 命令`，这个命令可以去除 secure 增加的安全加密：

	valet unsecure laravel

<a name="sharing-sites"></a>
## 分享站点
Valet 还提供一个命令将本地站点分享给其他人，这不需要任何额外安装软件即可实现。

要分享站点，在你的终端中切换到站点目录使用 `valet share` 命令。这会生成一个可以公开访问的 URL 并插入你的剪切板，以便你直接粘贴到浏览器，就是这么简单。

要停止分享站点，使用 `Control + C` 快捷组合键即可。

> {note} `valet share` 目前尚不支持分享使用 `valet secure` 命令进行安全处理的站点。

<a name="custom-valet-drivers"></a>

## 自定义 Valet 驱动

你可以编写自定义的 Valet 「驱动」运行非原生支持的其他 PHP 框架或 CMS 。安装完 Valet 时会创建一个 `~/.valet/Drivers` 目录，该目录中有一个 `SampleValetDriver.php` 文件。这个文件中简单演示如何编写自定义驱动。 编写驱动只需要实现三个方法： `serves` ，`isStaticFile` 和 `frontControllerPath` 。

这三个方法都接收 `$sitePath` ，`$siteName` 和 `$uri` 作为参数。 `$sitePath` 表示站点的绝对路径，例如 `/Users/Lisa/Sites/my-project` 。 `$siteName` 表示站点的 「host」/ 「站点名称」 部分，如 (`my-project`) 。 `$uri` 则是输入的请求 URI，如 (`/foo/bar`) 。

编写好你的自定义 Valet 驱动，将其放到 `~/.valet/Drivers` 目录并遵循 `FrameworkValetDriver.php` 这种命名规范。例如，如果编写一个自定义的 WordPress 驱动，对应的文件名称应是 `WordPressValetDriver.php` 。

下面我们来具体讨论并演示自定义 Valet 驱动需要实现的三个方法。

#### `serves` 方法

如果自定义驱动要继续处理输入请求， `serves` 方法应该返回 `true` ， 否则该方法返回 `false` 。 因此，这个方法应该判断给定的 `$sitePath` 是否是包含你服务项目的类型。

例如，假设我们编写的是 `WordPressValetDriver` 。那么对应的 serves 方法如下：

	/**
     * 判断驱动服务请求
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return bool
     */
    public function serves($sitePath, $siteName, $uri)
    {
        return is_dir($sitePath.'/wp-admin');
    }

#### `isStaticFile` 方法

`isStaticFile` 应该判断进入的请求是否是静态文件，例如图片或者样式文件，如果文件是静态的，该方法会返回磁盘上的绝对路径，否则返回 `false`:

    /**
     * 判断请求内容是否是静态文件。
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

> {note} `isStaticFile` 方法只有在 `serves` 方法返回 `true` 并且请求 URI 不是 `/` 才会被调用。

#### `frontControllerPath` 方法
`frontControllerPath` 方法应该返回「前端控制器」的绝对路径，通常是你的 「index.php」文件或其他同等文件:

    /**
     * 获取应用前端控制器绝对路径。
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
`valet forget` | 在某个站点根路径运行该命令可在根目录列表中移除该目录
`valet paths` | 查看所有站点根路径
`valet restart` | 重启
`valet start` | 启动
`valet stop` | 停止
`valet uninstall` | 卸载

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@lybc](https://laravel-china.org/users/1232)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/1232_1469434413.jpg?imageView2/1/w/100/h/100">  |  翻译  | 有用是有趣的前提  |