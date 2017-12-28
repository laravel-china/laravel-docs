# Laravel 的开发环境 Valet

- [简介](#introduction)
    - [Valet 还是 Homestead ？](#valet-or-homestead)
- [安装](#installation)
    - [升级](#upgrading)
- [服务站点](#serving-sites)
    - [Park 命令](#the-park-command)
    - [Link 命令](#the-link-command)
    - [使用 TLS 构建安全站点](#securing-sites)
- [分享站点](#sharing-sites)
- [自定义 Valet 驱动](#custom-valet-drivers)
    - [本地驱动](#local-drivers)
- [其他 Valet 命令](#other-valet-commands)

<a name="introduction"></a>
## 简介

Valet 是 Mac 极简主义者的 Laravel 开发环境。没有 Vagrant，没有 `/etc/hosts` 文件。甚至可以使用本地隧道公开分享你的站点。 _Yeah, we like it too._

Laravel Valet 为你的 Mac 设置了启动后始终在后台运行 Nginx。然后，Valet 使用 [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq) 将所有指向安装在本地计算机的站点的请求代理到 `*.test` 域上。

换句话说，一个速度极快的 Laravel 开发环境只占用 7MB 内存。Valet 并不是想要完全替换 Vagrant 或 Homestead，只是提供另外一种使用起来更加灵活、方便、以及内存的占用更小的选择。

Valet 支持但不局限于以下内容：

<div class="content-list" markdown="1">
- [Laravel](https://laravel.com)
- [Lumen](https://lumen.laravel.com)
- [Bedrock](https://roots.io/bedrock/)
- [CakePHP 3](https://cakephp.org)
- [Concrete5](http://www.concrete5.org/)
- [Contao](https://contao.org/en/)
- [Craft](https://craftcms.com)
- [Drupal](https://www.drupal.org/)
- [Jigsaw](http://jigsaw.tighten.co)
- [Joomla](https://www.joomla.org/)
- [Katana](https://github.com/themsaid/katana)
- [Kirby](https://getkirby.com/)
- [Magento](https://magento.com/)
- [Sculpin](https://sculpin.io/)
- [Slim](https://www.slimframework.com)
- [Statamic](https://statamic.com)
- Static HTML
- [Symfony](https://symfony.com)
- [WordPress](https://wordpress.org)
- [Zend](https://framework.zend.com)
  </div>

However, you may extend Valet with your own [custom drivers](#custom-valet-drivers).

你可以使用 [自定义驱动](#custom-valet-drivers) 来扩展 Valet。

<a name="valet-or-homestead"></a>
### Valet 还是 Homestead？

Laravel 还提供了另一种本地开发环境 [Homestead](/docs/{{version}}/homestead)。Homestead 和 Valet 的不同之处在于目标受众和对本地开发的方式。Homestead 提供了一个完整的、具有自动化的 Nginx 配置的 Ubuntu 虚拟机。如果你想要完全虚拟化的 Linux 开发环境或 Windows／Linux，Homestead 是一个不错的选择。

Valet 仅支持 Mac，并要求你将 PHP 和数据库服务器直接安装到本地机器上。这可以很容易地通过使用 Homebrew 命令来实现，像 `brew install php71` 和 `brew install mysql`。Valet 提供了一个极快的、资源消耗最少本地开发环境，非常适合只需要 PHP／MySQL 并且不需要虚拟开发环境的开发人员。

Valet 和 Homestead 都是配置 Laravel 开发环境的绝佳选择。选择哪一个仅仅只是取决于个人喜好和团队的需求。

<a name="installation"></a>
## 安装

**Valet 需要 macOS 和 [Homebrew](http://brew.sh/)。在安装之前，要确保没有其他程序（如 Apache 或 Nginx ）占用了本地机器的端口 80。**

<div class="content-list" markdown="1">
- 使用 `brew update` 将 Homebrew 安装或更新到最新版本。

- 通过 Homebrew 使用 `brew install homebrew/php/php71` 命令安装 PHP 7.1 。

- 通过 Composer 使用 `composer global require laravel/valet` 安装 Valet。确保 `~/.composer/vendor/bin` 目录位于系统的「PATH」中。

- 运行 `valet install` 命令来配置和安装 Valet 和 DnsMasq，并注册 Valet 后台随机启动。

  </div>

安装完 Valet，使用 `ping foobar.test` 命令在终端上的 ping 任何一个 `*.test` 的域名。如果 Valet 安装正确，可以在终端上看到来自 `127.0.0.1` 的响应。

每次机器启动时，Valet 会自动启动其进程。所以只要完成了 Valet 的初始化，就无需再次运行 `valet start` 或 `valet install`。

#### 使用其它域名

默认情况下，Valet 使用 `.test`  顶级域名为你的项目提供服务。如果你想使用其他域名，可以使用 `valet domain tld-name` 命令。

例如，如果你要使用 `.app` 而不是 `.test`，就运行 `valet domain app`，Valet 会自动将站点域名改为 `*.app`。

#### 数据库

如果你要用数据库，在终端运行 `brew install mysql` 安装 MySQL。安装完后，你可以使用 `brew services start mysql` 命令启动它。然后，你可以使用 `root` 作为用户名和空字符串作为密码连接到 `127.0.0.1` 的数据库。

<a name="upgrading"></a>
### 升级

你可以在使用终端使用 `composer global update` 命令更新 Valet。升级后，如果需要，最好再运行一次  `valet install`，以便 Valet 可以对配置文件进行升级。

#### 升级到 Valet 2.0

Valet 2.0 将 Valet 底层的 Web 服务器从 Caddy 转移到 Nginx。升级到此版本之前，你应该运行以下命令停止并卸载现有的 Caddy 进程：

    valet stop
    valet uninstall

接下来，就根据你如何采用的安装方式来升级 Valet （通常是通过 Git 或 Composer ）。如果是通过 Composer 安装了 Valet，则应使用以下命令更新到最新的主要版本：

    composer global require laravel/valet
记住，只要下载了 Valet 的源码，就要运行 `install` 命令：

    valet install
    valet restart

因为升级后，可能需要重新设置或重新链接你的站点。

<a name="serving-sites"></a>
## 服务站点

安装了 Valet 之后，你就可以开始设置站点。Valet 提供两个命令来为 Laravel 的站点提供服务：`park` 和 `link`。

<a name="the-park-command"></a>

**`park` 命令**

<div class="content-list" markdown="1">
- 运行 `mkdir ~/Sites` 命令在 Mac 上创建一个新的目录。接下来，运行 `cd ~/Sites` 和 `valet park` 将当前的工作目录作为 Valet 搜索站点的路径。

- 接下来，在这个目录中创建一个新的 Laravel 站点：`laravel new blog`。

- 在浏览器中打开 `http://blog.test`。

  </div>

**就这么多。**现在，你在 「parked」的目录中创建的任何 Laravel 项目将自动使用 `http://folder-name.test` 这种方式访问。

<a name="the-link-command"></a>
**`link` 命令**

如果要在目录中提供单个站点而不是整个目录，就使用 `link` 命令。

<div class="content-list" markdown="1">
- 要使用该命令，先切换到你的某个项目并运行 `valet link app-name`。Valet 会在 `~/.valet/Sites` 中创建一个符号链接指向当前的目录。
- 运行 `link` 命令后，你可以在浏览器通过 `http://app-name.test` 访问站点。
- </div>

运行 `valet links` 命令可以查看所有目录链接的列表。你还可以使用 `valet unlink app-name` 来删除符号链接。

> {tip} 你可以使用 `valet link` 将多个（子）域指向同一个应用。要添加子域名或其它域名到应用，可以在应用目录下运行 `valet link subdomain.app-name`。

<a name="securing-sites"></a>
**使用 TLS 构建安全站点**

默认情况下，Valet 使用 HTTP 协议提供站点。但是，如果你想使用 HTTP/2 提供加密的 TLS 站点，使用 `secure` 命令。例如，如果你的站点域名是 `laravel.test`，可以这样：

    valet secure laravel
要还原为 HTTP，使用 `unsecure` 命令。像 `secure` 命令一样，此命令接受你要取消安全加密的站点名称：

    valet unsecure laravel

<a name="sharing-sites"></a>
## 分享站点

Valet 甚至包括与世界分享你的本地网站的命令。一旦安装了 Valet，就不需要额外的软件安装。

要共享站点，先在终端切换到站点目录，然后运行 `valet share` 命令。将可公开访问的网址插入剪贴板，然后直接粘贴到浏览器中访问。就这么简单～

要停止分享你的网站，按 `Control + C` 取消分享。

> {note} `valet share` 目前不支持使用 `valet secure` 命令保护共享的站点。

<a name="custom-valet-drivers"></a>
## 自定义 Valet 驱动

你可以编写自己的 Valet 「驱动」来提供运行在另一个不是由 Valet 支持的框架或 CMS 上的 PHP 应用程序。安装 Valet 时，会创建一个包含 `SampleValetDriver.php` 文件 `~/.valet/Drivers` 目录。这个文件包含实现驱动程序的例子，演示了如何编写自定义驱动程序。编写驱动程序只需要实现三种方法：`serves`、 `isStaticFile` 和  `frontControllerPath`。

这三种方法都会收到 `$sitePath`、`$siteName` 和 `$uri` 值作为参数。`$sitePath` 是计算机上提供的站点的完整路径，例如 `/Users/Lisa/Sites/my-project`。`$siteName` 是 「主机」 / 「站点名」 的域名部分，如 `my-project`。 `$uri` 则是输入的请求地址，如 `/foo/bar`。

完成自定义 Valet 驱动后，使用 `FrameworkValetDriver.php` 这样的命名将其放在 `~/.valet/Drivers` 目录中。例如，如果你正在为 WordPress 编写自定义 Valet 驱动程序，那你的文件名应该是 `WordPressValetDriver.php`。

接下来看看自定义 Valet 驱动应该实现的每种方法的实现示例。

#### `serves` 方法

如果你的驱动程序应该处理传入的请求，那么 `serves` 方法应该返回 `true`。否则，该方法应返回 `false`。因此，在此方法中，你需要确定给定的 `$sitePath` 是否包含你要提供的类型的项目。

例如，假设我们正在编写一个 `WordPressValetDriver`。`serve` 方法可能如下所示：

    /**
     * 确定驱动程序是否提供请求。
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

`isStaticFile` 应该确定传入的请求是否是一个「静态」的文件，如图像或样式表。如果文件是静态的，该方法应该将完整的路径返回到磁盘上的静态文件。如果传入的请求不是静态文件，则该方法应返回 `false`：

    /**
     * 确定传入请求是否为静态文件。
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

> {note} 只有当传入请求的 `serves` 方法返回 `true` 并且请求的 URI 不是 `/` 时，才会调用 `isStaticFile` 方法。

#### `frontControllerPath` 方法

`frontControllerPath` 方法应该返回完整的路径到你应用的「前端控制器」，这通常是你的「index.php」文件或其他同等文件：

    /**
     * 获取完全解析的路径到应用程序的前端控制器。
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

<a name="local-drivers"></a>
### 本地驱动

如果要为单个应用定义一个自定义 Valet 驱动程序，请在应用程序的根目录中创建一个 `LocalValetDriver.php`。你的自定义驱动程序可以继承基础 `ValetDriver` 类或继承现有应用的特定驱动程序，如 `LaravelValetDriver`：

    class LocalValetDriver extends LaravelValetDriver
    {
        /**
         * 确定驱动程序是否提供请求。
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return bool
         */
        public function serves($sitePath, $siteName, $uri)
        {
            return true;
        }

        /**
         * 获取完全解析的路径到应用程序的前端控制器。
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return string
         */
        public function frontControllerPath($sitePath, $siteName, $uri)
        {
            return $sitePath.'/public_html/index.php';
        }
    }

<a name="other-valet-commands"></a>
## 其它 Valet 命令

| 命令 | 注释 |
| ------------- | -------------
| `valet forget`    | 在被「parked」的目录中运行这个命令，能够将这个命令从「parked」目录列表中删除。 |
| `valet paths`     | 查看所有「parked」过的路径。|
| `valet restart`   | 重启 Valet |
| `valet start`     | 启动 Valet |
| `valet stop`      | 停止 Valet |
| `valet uninstall` | 卸载 Valet |

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
