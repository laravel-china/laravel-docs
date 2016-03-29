# Laravel Homestead

- [简介](#introduction)
- [安装与设置](#installation-and-setup)
    - [前置动作](#first-steps)
    - [配置 Homestead](#configuring-homestead)
    - [启动 Vagrant box](#launching-the-vagrant-box)
    - [根据项目分别安装](#per-project-installation)
- [常见用法](#daily-usage)
    - [透过 SSH 连接](#connecting-via-ssh)
    - [连接数据库](#connecting-to-databases)
    - [增加更多网站](#adding-additional-sites)
    - [设置 Cron 调度器](#configuring-cron-schedules)
    - [连接端口](#ports)
    - [Bash 别名](#bash-aliases)
- [Blackfire 分析器](#blackfire-profiler)

<a name="introduction"></a>
## 简介

Laravel 致力于让 PHP 开发体验更愉快，也包含你的本地开发环境。[Vagrant](http://vagrantup.com) 提供了一个简单、优雅的方式来管理与供应虚拟机。

Laravel Homestead 是一个官方预载的 Vagrant「box」，提供你一个美好的开发环境，你不需要在你的本机电脑安装 PHP、HHVM、网页服务器或任何服务器软件。不用担心搞乱你的系统！Vagrant box 可以搞定一切。如果有什么地方烂掉了，你可以在几分钟内快速的砍掉并重建虚拟机！

Homestead 可以在任何 Windows、Mac 或 Linux 上面运行，里面包含了 Nginx 网页服务器、PHP 5.6、MySQL、Postgres、Redis、Memcached、Node，以及所有你在使用 Laravel 开发各种精彩的应用程序时所需要的软件。

> **附注：** 如果您是 Windows 的用户，您可能需要启用硬件虚拟化（VT-x）。这通常需要透过 BIOS 来启用它。

Homestead 目前是建置且测试于 Vagrant 1.7。

<a name="included-software"></a>
### 内置软件

- Ubuntu 14.04
- Git
- PHP 5.6 / 7.0
- Xdebug
- HHVM
- Nginx
- MySQL
- Sqlite3
- Postgres
- Composer
- Node（附带了 PM2、Bower、Grunt 与 Gulp）
- Redis
- Memcached (仅限 PHP 5.x)
- Beanstalkd
- [Laravel Envoy](/docs/{{version}}/envoy)
- [Blackfire 分析器](#blackfire-profiler)

<a name="installation-and-setup"></a>
## 安装与设置

<a name="first-steps"></a>
### 前置动作

在启动你的 Homestead 环境之前，你必须先安装 [VirtualBox 5.x](https://www.virtualbox.org/wiki/Downloads) 或 [VMWare](http://www.vmware.com) 以及 [Vagrant](http://www.vagrantup.com/downloads.html)。这些软件在各个常用的平台都有提供易用的可视化安装程序。

若要使用 VMware provider，你需要同时购买 VMware Fusion / Workstation 及 [VMware Vagrant plug-in](http://www.vagrantup.com/vmware) 的软件授权。使用 VMware 可以在共享文件夹上获得较快的性能。

#### 安装 Homestead Vagrant box

当 VirtualBox / VMware 以及 Vagrant 安装完成后，你可以在终端机以下列命令将 'laravel/homestead' 这个 box 安装进你的 Vagrant 程序中。下载 box 会花你一点时间，时间长短将依据你的网络速度决定：

    vagrant box add laravel/homestead

如果运行上面的命令失败，代表你使用的可能是旧版的 Vagrant，它需要补上完整的 URL：

    vagrant box add laravel/homestead https://atlas.hashicorp.com/laravel/boxes/homestead

#### 手动克隆 Homestead 资源库

你可以简单地透过手动克隆资源库的方式来安装 Homestead。建议可将资源库克隆至你的「home」目录中的 `Homestead` 文件夹，如此一来 Homestead box 将能提供主机服务给你所有的 Laravel 项目：

    git clone https://github.com/laravel/homestead.git Homestead

如果你想尝试 PHP 7.0 版本的 Homestead，可以克隆资源库的 `php-7` 分支：

    git clone -b php-7 https://github.com/laravel/homestead.git Homestead

一旦你克隆完 Homestead 资源库，即可在 Homestead 目录中运行 `bash init.sh` 命令来创建 `Homestead.yaml` 配置文件。`Homestead.yaml` 文件将会被放置在你的 `~/.homestead` 目录中：

    bash init.sh

<a name="upgrading-to-php-7"></a>
#### 升级至 PHP 7.0

如果你已经使用 PHP 5.x 的 Homestead box，你可以很简单的升级你的安装档至 PHP 7.0。首先，克隆 `laravel/homestead` 资源库的 `php-7` 分支至新的文件夹：

    git clone -b php-7 https://github.com/laravel/homestead.git Homestead7

如果你的 `~/.homestead` 目录已经拥有 `Homestead.yaml` 文件，那么你就不需要运行 `init.sh` 脚本。不过，如果这是你机器第一次且唯一 Homestead 安装，你需要在你新的 Homestead 目录运行 `bash init.sh`。

接着，在你的 `~/.homestead/Homestead.yaml` 文件最上方增加 `box` 命令（在 `---` 标记后新的一行）：

    box: laravel/homestead-7

最后，你可以在包含你新克隆 `laravel/homestead` 资源库的资目录中运行 `vagrant up` 命令。

<a name="configuring-homestead"></a>
### 配置 Homestead

#### 设置你的提供者

在 `Homestead.yaml` 文件中的 `provider` 参数是用来设置你想要使用哪一个 Vagrant 提供者：`virtualbox`、`vmware_fusion` 或 `vmware_workstation`。你可以根据你的喜好来决定提供者：

    provider: virtualbox

#### 设置你的 SSH 密钥

你还需要将你的公有 SSH 密钥的路径配置在 `Homestead.yaml` 文件中。你没有 SSH 密钥？在 Mac 和 Linux 下，你可以利用下面的命令来创建一组 SSH 密钥：

    ssh-keygen -t rsa -C "you@homestead"

在 Windows 下，你需要安装 [Git](http://git-scm.com/) 并且使用包含在 Git 里的 `Git Bash` 来运行上述的命令。另外你也可以使用 [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) 和 [PuTTYgen](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)。

一旦你创建了一组 SSH 密钥，记得在你的 `Homestead.yaml` 文件中的 `authorize` 属性去设置公有密钥的路径。

#### 设置共享文件夹

你可以在 `Homestead.yaml` 文件的 `folders` 属性里列出所有你想与你的 Homestead 环境共享的文件夹。这些文件夹中的文件若有更动，它们将会同步更动在你的本机电脑与 Homestead 环境。你可以将多个你所需要的共享文件夹都设置于此：

    folders:
        - map: ~/Code
          to: /home/vagrant/Code

若要启用 [NFS](http://docs.vagrantup.com/v2/synced-folders/nfs.html)，你只需要在共享文件夹的设置值中加入一个简单的参数：

    folders:
        - map: ~/Code
          to: /home/vagrant/Code
          type: "nfs"

#### 设置 Nginx 网站

对 Nginx 不熟悉吗？没关系。`sites` 属性帮助你可以轻易的指定一个 `网域` 对应至 homestead 环境中的一个目录。在 `Homestead.yaml` 文件中已内含一个网站设置的范本。同样的，你可以增加数个你所需要的网站到你的 Homestead 环境中。Homestead 可以为每个你正在开发中的 Laravel 项目提供方便的虚拟化环境：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public

你可以将 `hhvm` 属性设置为 `true` 让 Homestead 里面的任一个网站改用 [HHVM](http://hhvm.com)：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          hhvm: true

若要造访网站，每一个网站默认的连接端口分别是 HTTP 连接端口 8000 及 HTTPS 连接端口 44300。

#### 关于 Hosts 文件

不要忘了在将你在 Nginx sites 中所添加的「网域」也添加至你本机电脑的 `hosts` 里！ `hosts` 文件会将你所发出的请求重定向至你在 Homestead 环境中设置的本地网域。在 Mac 或 Linux 上，该文件通常会存放在 `/etc/hosts`。在 Windows 上，则存放于 `C:\Windows\System32\drivers\etc\hosts`。你要设置于文件中的内容类似如下：

    192.168.10.10  homestead.app

务必确认 IP 位置与 `Homestead.yaml` 文件中设置的相同。一旦你将网域设置在 `hosts` 文件之后，你就可以透过网页浏览器造访你的网站！

    http://homestead.app

<a name="launching-the-vagrant-box"></a>
### 启动 Vagrant box

当你根据你的喜好编辑完 `Homestead.yaml` 后，在终端机里进入你的 Homestead 目录并运行 `vagrant up` 命令。Vagrant 就会自将虚拟主机启动并自动设置你的共享文件夹和 Nginx 网站。

如果要移除虚拟机，可以使用 `vagrant destroy --force` 命令。

<a name="per-project-installation"></a>
### 根据项目分别安装

有别于将 Homestead 安装成全域环境且让所有的项目共用同一个 Homestead box，你可以各别为每一个项目独立配置一个 Homstead。如果你希望直接在项目里传递 `Vagrantfile`，那么替每个项目安装 Homestead 即是你可以考虑的方式，这将会允许其他人可以简单地运行 `vagrant up` 即能开始工作于此项目。

你可以使用 Composer 将 Homestead 直接安装至你的项目中：

    composer require laravel/homestead

一旦 Homestead 安装完毕，你可以使用 `make` 命令生成 `Vagrantfile` 与 `Homestead.yaml` 存放于项目的根目录。这个 `make` 命令将会自动配置 `sites` 及 `folders` 于 `Homestead.yaml` ：

Mac / Linux:

    php vendor/bin/homestead make

Windows:

        vendor\bin\homestead make

接着，运行在终端机中运行 `vagrant up` 并透过网页浏览器造访 `http://homestead.app`。再次提醒，你仍然需要在 `/etc/hosts` 里设置 `homestead.app` 或其他想要使用的网域。

<a name="daily-usage"></a>
## 常见用法

<a name="connecting-via-ssh"></a>
### 透过 SSH 连接

你可以在终端机里进入你的 Homestead 目录并运行 `vagrant ssh` 命令借此以 SSH 连上你的虚拟主机。

但是，你可能会经常需要透过 SSH 连上你的 Homestead 主机，因此你可以考虑在你的本机电脑上创建一个「别名」来快速连上 Homestead box。一旦你创建这个别名，你可以轻易地透过「vm」这个命令从你的电脑以 SSH 连上你的 Homestead 主机：

    alias vm="ssh vagrant@127.0.0.1 -p 2222"

<a name="connecting-to-databases"></a>
### 连接数据库

在 `Homestead` 中，已经预装了 MySQL 与 Postgres 两种数据库。为了方便使用，Laravel 在 `local` 的数据库设置值中已经默认将其设置完成。

如果想要从本机电脑上透过 Navicat 或者是 Sequel Pro 连接 MySQL 或 Postgres 数据库，你可以连接 `127.0.0.1` 的连接端口 33060 (MySQL) 或 54320 (Postgres)。而帐号密码分别是 `homestead` / `secret`。

> **附注：** 从本机电脑你应该只能使用这些非标准的连接端口来连接数据库。因为当 Laravel 运行于虚拟主机时，在 Laravel 的数据库设置值中依然是设置使用默认的 3306 及 5432 连接端口。

<a name="adding-additional-sites"></a>
### 增加更多网站

一旦 Homestead 环境配置完毕且运行后，你可能会想要为 Laravel 应用程序增加更多的 Nginx 网站。你可以在单一个 Homestead 环境中运行非常多 Laravel 安装程序。只要在 `Homestead.yaml` 档中增加另一个网站设置后，接着在终端机中进入到你的 Homestead 目录并运行 `vagrant provision` 命令，即可增加另一个网站。

<a name="configuring-cron-schedules"></a>
### 设置 Cron 调度器

Laravel 提供了便利的方式来[调度 Cron 任务](/docs/{{version}}/scheduling)，透过 `schedule:run` Artisan 命令，调度便会在每分钟被运行。`schedule:run` 命令会检查定义在你 `App\Console\Kernel` 类中调度的任务，判断哪个任务该被运行。

如果你想为 Homestead 网站使用 `schedule:run` 命令，你可以在定义网站时设置 `schedule` 选项为 `true`：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          schedule: true

该网站的 Cron 任务会被定义于虚拟机的 `/etc/cron.d` 文件夹中。

<a name="ports"></a>
### 连接端口

以下的连接端口将会被转发至 Homestead 环境：

- **SSH：**2222 &rarr; 转发至 22
- **HTTP：**8000 &rarr; 转发至 80
- **HTTPS：**44300 &rarr; 转发至 443
- **MySQL：**33060 &rarr; 转发至 3306
- **Postgres：**54320 &rarr; 转发至 5432

#### 转发更多连接端口

如果你希望，你也可以借着指定连接端口的通信协定来转发更多额外的连接端口给 Vagrant box：

    ports:
        - send: 93000
          to: 9300
        - send: 7777
          to: 777
          protocol: udp

<a name="bash-aliases"></a>
### Bash 别名

若要增加 Bash 别名至你的 Homestead box，只需要编辑 Homestead 目录中的 `aliases` 文件。这些别名会在 Homestead box 启动时自动定义。

<a name="blackfire-profiler"></a>
## Blackfire 分析器

由 SensioLabs 推出的 [Blackfire 分析器](https://blackfire.io) 能协助你自动收集程序运行时的相关数据，像是 RAM、CPU time 及 disk I/O。若想在 Homestead 中替你的应用程序使用这个分析器是非常容易的。

在 Homestead box 中同样已经预装好所有需要的扩展包，很简单地你只需要在 `Homestead.yaml` 文件中设置你的 Blackfire **Server ID 及 token 即可：

    blackfire:
        - id: your-server-id
          token: your-server-token
          client-id: your-client-id
          client-token: your-client-token

一旦你设置完 Blackfire 的认证，在你的 Homestead 目录下运行 `vagrant provision` 来重新配置 box。当然，请务必阅读 [Blackfire 使用文档](https://blackfire.io/getting-started) 来学习如何在你的网页浏览器上安装 Blackfire 扩展。
