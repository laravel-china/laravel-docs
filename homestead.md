# Laravel 虚拟开发环境 Homestead

- [简介](#introduction)
- [安装与设置](#installation-and-setup)
    - [第一步](#first-steps)
    - [配置 Homestead](#configuring-homestead)
    - [启动 Vagrant Box](#launching-the-vagrant-box)
    - [根据项目分开安装](#per-project-installation)
    - [安装 MariaDB](#installing-mariadb)
- [常见用法](#daily-usage)
    - [全局可用的 Homestead](#accessing-homestead-globally)
    - [通过 SSH 连接](#connecting-via-ssh)
    - [连接数据库](#connecting-to-databases)
    - [增加更多网站](#adding-additional-sites)
    - [配置 Cron 调度器](#configuring-cron-schedules)
    - [端口](#ports)
    - [共享你的环境](#sharing-your-environment)
    - [多个 PHP 版本](#multiple-php-versions)
- [网络接口](#network-interfaces)
- [更新 Homestead](#updating-homestead)
- [历史版本](#old-versions)
- [Provider 的特殊设置](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## 简介

Laravel 致力于让你在 PHP 开发过程中更加轻松愉快，这其中也包括本地开发环境的搭建。 [Vagrant](https://www.vagrantup.com) 提供了一种简单、优雅的方式来管理和配置虚拟机。

Laravel Homestead 是一个官方预封装的 Vagrant box，它为你提供了一个完美的开发环境，你无需在本地安装 PHP ，web 服务器，或其他服务软件。 Vagrant box 是完全一次性的，你不用担心系统被搞乱！如果有什么地方出错了，你可以在几分钟内销毁并重建 box ！

Homestead 可以运行在 Windows 、 Mac 或 Linux 系统上，它里面包含了 Nginx Web 服务器、 PHP 7.1 、 MySQL 、 Postgres 、 Redis 、 Memcached 、 Node ， 以及一些有利于你开发 laravel 应用的其他程序。

> {note} 如果你使用的是 Windows 系统，你可能需要启用硬件虚拟化（VT-x）。这通常需要通过 BIOS 来启用它。如果你在一个 UEFI 系统上使用 Hyper-V，您可能还需要禁用 Hyper-V 才能启用 VT-x 。

<a name="included-software"></a>
### 内置软件

- Ubuntu 16.04
- Git
- PHP 7.1
- Nginx
- MySQL
- MariaDB
- Sqlite3
- Postgres
- Composer
- Node (With Yarn, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd
- Mailhog
- ngrok

<a name="installation-and-setup"></a>
## 安装与设置

<a name="first-steps"></a>
### 第一步 

在你使用 Homestead 环境之前，你必须先安装 [VirtualBox 5.1](https://www.virtualbox.org/wiki/Downloads) 、 [VMWare](https://www.vmware.com) 或者 [Parallels](http://www.parallels.com/products/desktop/) 中的一个，然后再安装 [Vagrant](https://www.vagrantup.com/downloads.html) 。上述软件均提供了针对不同操作系统的可视化安装包。

若要使用 VMware provider，你需要同时购买 VMware Fusion / Workstation 以及 [VMware Vagrant 插件](https://www.vagrantup.com/vmware) 的软件授权，因为它们不是免费的。使用 VMware 的优势是：可以获得开箱即用的共享文件夹特性。

若要使用 Parallels provider，你需要安装 [Parallels Vagrant 插件](https://github.com/Parallels/vagrant-parallels) ，这是免费的。

#### 安装 Homestead Vagrant Box

当 VirtualBox / VMware 以及 Vagrant 安装完成后，你可以使用以下命令将 laravel/homestead 这个 box 添加进你的 Vagrant 当中。 homestead box 的下载会花费你一点时间，具体的下载时长由网络速度决定：

    vagrant box add laravel/homestead

如果上面的命令运行失败，请先确保你已经安装了最新版本的 Vagrant 。

> {note} 如果使用国内网络，可以复制终端上显示的 homestead box 下载地址手动下载并重命名。例如重命名为 virtualbox-3.0.0.box 

然后，新建一个 metadata.json 文件，并写入以下示例内容：

    {
        "name": "laravel/homestead",
        "versions": 
        [
            {
                "version": "3.0.0",
                "providers": [
                    {
                      "name": "virtualbox",
                      "url": "virtualbox-3.0.0.box"
                    }
                ]
            }
        ]
    }

最后，使用以下命令手动添加 box

    vagrant box add metadata.json # 添加 box
    vagrant box list # 列出所有 box

#### 安装 Homestead

你可以简单使用 Git 克隆代码仓库的方式来安装 Homestead。建议将克隆的代码仓库重命名为 `Homestead` ，并放置到你的「home」目录中，如此一来 Homestead box 就能作为主机，为你的所有 Laravel 项目提供服务：

    cd ~

    git clone https://github.com/laravel/homestead.git Homestead

由于 Homestead 的 `master` 分支并不是稳定分支，你应该检出已经标签过的稳定版本。你可以在 [Github Release Page](https://github.com/laravel/homestead/releases) 找到最新的稳定版本。

    cd Homestead

    // 检出所需要的版本...
    git checkout v5.4.0

一旦你克隆完 Homestead 的代码仓库，就可以在 Homestead 目录中运行 `bash init.sh` 命令来创建 `Homesstead.yaml` 配置文件。 `Homesstead.yaml` 文件会被放置在你的 Homestead 目录中：

    // Mac / Linux...
    bash init.sh

    // Windows...
    init.bat

<a name="configuring-homestead"></a>
### 配置 Homestead

#### 配置你的提供者

`Homestead.yaml` 中的 `provider` 参数设置取决于你用的是哪一个 Vagrant 提供者 `virtualbox ` 、 `vmware_fusion` 、 `vmware_workstation` ，或者 `parallels` 。你可以根据自己的实际情况来设置提供者：

    provider: virtualbox

#### 配置共享文件夹

你可以在 `Homestead.yaml` 文件的 `folders` 属性里列出所有想与 Homestead 环境共享的文件夹。这些文件夹中的文件若有变更，它们将会在你的本机电脑与 Homestead 环境自动更新同步。你可以在这里设置多个共享文件夹：

    folders:
        - map: ~/Code
          to: /home/vagrant/Code

若要启动 [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html) ，只需要在共享文件夹的设置值中加入一个简单的参数：

    folders:
        - map: ~/Code
          to: /home/vagrant/Code
          type: "nfs"

> {note} 如果使用 NFS ， 建议你安装 [vagrant-bindfs](https://github.com/gael-ian/vagrant-bindfs) 插件。 这个插件会替你处理 box 中的文件或目录权限问题。

你也可以在配置中传递任何 Vagrant [共享文件夹](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) 支持的参数，在 `options` 配置项下列出它们：

    folders:
        - map: ~/Code
          to: /home/vagrant/Code
          type: "rsync"
          options:
              rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
              rsync__exclude: ["node_modules"]


#### 配置 Nginx 站点

对 Nginx 不熟悉吗？没关系。`sites` 属性可以帮助你可以轻易指定一个 `域名` 来对应到 homestead 环境中的一个目录上。在 `Homestead.yaml` 文件中已包含了一个网站设置范本。同样的，你也可以增加多个网站到你的 Homestead 环境中。 Homestead 可以同时为多个 Laravel 应用提供虚拟化环境：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public

如果你在 Homestead box 配置之后更改了 `sites` 属性，那么应该重新运行 `vagrant reload --provision` 来更新 Nginx 配置到虚拟机上。

#### 关于 Hosts 文件

你必须将在 Nginx sites 中所添加的「域名」也添加到你本机电脑的 `hosts` 上。 `hosts` 文件会将请求重定向至 Homestead 环境中设置的本地域名。在 Mac 或 Linux 上，该文件通常会存放在 `/etc/hosts` 。在 Windows 上，则存放于 `C:\Windows\System32\drivers\etc\hosts` 。设置内容如下所示：

    192.168.10.10  homestead.app

请务必确认 IP 地址与 `Homestead.yaml` 文件中设置的相同。将域名设置在 `hosts` 文件之后，你就可以通过网页浏览器访问你的网站。

    http://homestead.app

<a name="launching-the-vagrant-box"></a>
### 启动 Vagrant Box

根据你的喜好完成 `Homestead.yaml` 编辑后，进入你的 Homestead 目录并运行 `vagrant up` 命令。 Vagrant 就会根据 `Homestead.yaml` 里的配置信息启动，并为虚拟机设置共享文件夹和 Nginx 网站。

如果要移除虚拟机，你可以使用 `vagrant destroy --force` 命令

<a name="per-project-installation"></a>
### 为每个项目分开安装

除了在全局范围内安装 Homestead 环境，所有项目共享相同的 Homestead box 外，你还可以为每一个项目配置一个独立的 Homestead 实例。通过传递 `Vagrantfile` ，可以实现为每个项目分别安装上 Homestead ，其他项目成员只需要通过简单的 `vagrant up` 即能跟你拥有一样的 Homestead 环境。

要将 Homestead 直接安装到项目中，需要使用 Composer:

    composer require laravel/homestead --dev

一旦 Homestead 安装完毕，可以使用 `make` 命令生成 `Vagrantfile` 与 `Homestead.yaml` 文件，并存放于项目的根目录。`make` 命令将会自动在 Homestead.yaml 文件中配置 `sites` 及 `folders`

Mac / Linux:

    php vendor/bin/homestead make

Windows:

    vendor\\bin\\homestead make

接下来，在命令行中运行 `vagrant up` 并通过网页浏览器访问 `http://homestead.app` 。再次提醒：你仍然需要在 `/etc/hosts` 里配置 `homestead.app` 或其它想要使用的域名。

<a name="installing-mariadb"></a>
### 安装 MariaDB

如果你希望使用 MariaDB 来替换 MySQL，你可以在 `Homestead.yaml` 文件中增加一个 `mariadb` 的选项，这个选项会移除 MySQL 并安装 MariaDB。因为 MariaDB 可用作 MySQL 的替代品，所以在你的数据库配置信息里，可继续使用 `mysql` 数据库驱动。

    box: laravel/homestead
    ip: "192.168.20.20"
    memory: 2048
    cpus: 4
    provider: virtualbox
    mariadb: true

> {note} 安装 MariaDB 需要连接境外网络，请确保网络畅通！

<a name="daily-usage"></a>
## 常见用法

<a name="accessing-homestead-globally"></a>
### 全局使用

有时候你希望在文件系统的任何地方都可以使用 `vagrant up` 命令启动虚拟机，那么你需要添加以下代码到你的 Mac / Linux 系统的 Bash profile 文件里面。对于 Windows 系统，您可以通过在  `PATH` 环境变量中添加 「批处理」 文件的方式来实现此目的。下面这些脚本让你可以在文件系统的任何位置都能运行 Vagrant 命令，它相当于切换到 Homestead 目录运行 Vagrant 命令：

#### Mac / Linux

    function homestead() {
        ( cd ~/Homestead && vagrant $* )
    }

请将 `~/Homestead` 这个路径修改为你的实际 Homestead 的安装路径，一旦这个函数安装成功，就可以在系统的任意位置运行 `homestead up` 或 `homestead ssh` 命令。

#### Windows

在系统的任意位置创建一个批处理文件 `homestead.bat` ，并添加如下内容：

    @echo off

    set cwd=%cd%
    set homesteadVagrant=C:\Homestead

    cd /d %homesteadVagrant% && vagrant %*
    cd /d %cwd%

    set cwd=
    set homesteadVagrant=

请确保将 `C:\Homestead` 这个路径修改为你的实际 Homestead 的安装路径，创建完这个文件后，将这个文件路径添加到 `PATH` 环境变量中，就可以在系统的任意位置运行 `homestead up` 或 `homestead ssh` 命令。

<a name="connecting-via-ssh"></a>
### 通过 SSH 连接

你可以在 Homestead 目录运行 `vagrant ssh` 命令来连接虚拟主机。

但是，由于您可能需要频繁地使用 SSH 来连接 Homestead 主机，请考虑将上述「function」添加到你的主机，以便可以快速的通过 SSH 进入你的 Homestead box

<a name="connecting-to-databases"></a>
### 连接数据库

在 box 中已经为 MySQL 和 Postgres 配置好了一个开箱即用的数据库 `homestead` ，为了更方便的使用它，Laravel 中的 `.env` 文件将这个数据库设置成了框架默认使用的数据库。

如果想要从你主机上的数据库客户端连接 MySQL 或 Postgres，可以通过 `127.0.0.1` 来使用端口 `33060`(MySQL) 或 `54320`(Postgres) 连接。账号密码分别是 `homestead` / `secret`

> {note} 因为虚拟机做了端口转发，所以在本机电脑上你应当只使用这些非标准的连接端口。但在 Laravel 数据库配置文件中，你依然要使用默认的 3306 及 5432 连接端口。

<a name="adding-additional-sites"></a>
### 增加更多网站

一旦 Homestead 环境配置完毕且成功运行后，你可能会想要为 Laravel 应用程序增加更多的 Nginx 网站。你可以在单个 Homestead 环境中运行多个 Laravel 程序。要添加额外的网站，只需将网站配置信息添加到您的 `Homestead.yaml` 文件中：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
        - map: another.app
          to: /home/vagrant/Code/another/public

如果 Vagrant 没有自动管理你的「hosts」文件，你可能需要手动把新增的站点加入到「hosts」文件中：

    192.168.10.10  homestead.app
    192.168.10.10  another.app

当你的网站添加完成后，切换到 Homestead 目录运行 `vagrant reload --provision` 命令就可以应用新的更改。

<a name="site-types"></a>
#### 网站类型

Homestead 支持多种类型的网站，允许您轻松地运行那些不基于 Laravel 的项目。 例如，我们可以使用 「symfony2」 配置项，轻松地在 Homestead 中添加 Symfony 应用程序：

    sites:
        - map: symfony2.app
          to: /home/vagrant/Code/Symfony/web
          type: symfony2

支持的站点类型有： `apache`， `laravel` （默认）， `proxy`， `silverstripe`，`statamic`， `symfony2`， 和 `symfony4`。

<a name="site-parameters"></a>
#### 网站参数

你还可以使用 「params」 配置项，添加额外的 Nginx `fastcgi_param` 值到你的网站。例如添加一个名称为 「FOO」 值为 「BAR」 的额外配置。

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          params:
              - key: FOO
                value: BAR

<a name="configuring-cron-schedules"></a>
### 配置 Cron 调度器

Laravel 提供了便利的方式来 [调度 Cron 任务](/docs/{{version}}/scheduling) ，通过 `schedule:run` Artisan 命令，调度便会在每分钟被运行。 `schedule:run` 命令会检查定义在你 `App\Console\Kernel` 类中调度的任务，判断哪个任务该被运行。

如果你想为 Homestead 网站使用 `schedule:run` 命令，你需要在定义网站时将 `schedule` 选项设置为 `true`

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          schedule: true

该网站的 Cron 任务会被定义在虚拟机的 `/etc/cron.d` 文件夹中。

<a name="ports"></a>
### 端口

默认情况下，以下本地电脑端口将会被转发至 Homestead 环境：

- **SSH:** 2222 &rarr; Forwards To 22
- **HTTP:** 8000 &rarr; Forwards To 80
- **HTTPS:** 44300 &rarr; Forwards To 443
- **MySQL:** 33060 &rarr; Forwards To 3306
- **Postgres:** 54320 &rarr; Forwards To 5432
- **Mailhog:** 8025 &rarr; Forwards To 8025

#### 转发更多端口

如果需要的话，你可以转发更多端给 Vagrant box ，甚至可以指定它们的协议类型。

    ports:
        - send: 50000
          to: 5000
        - send: 7777
          to: 777
          protocol: udp

<a name="sharing-your-environment"></a>
### 共享你的环境

有时候你想跟你的同事或者是客户共享你目前的工作进度。Vagrant 为此提供了一个内置方法 `vagrant share`；不过，如果你在 `Homestead.yaml` 文件中配置了多个站点，那么这条命令将会变得没多大用处。

为了解决这个问题，Homestead 提供了自己的 `share` 命令。开始之前，通过 `vagrant ssh` 命令 SSH 进你的 Homestead 机器中，然后运行 `share homestead.app`。这会从你的 `Homestead.yaml` 配置文件中共享 `homestead.app` 站点。当然了，你也可以用其他已经配置的站点来代替 `homestead.app`。

    share homestead.app

运行完命令之后，你可以看到一个包含活动日志和共享站点外网访问路径的 Ngrok 界面。如果你想要自定义地区或者其他 Ngrok 选项，你可以添加到 `share` 命令后面：

    share homestead.app -region=eu -subdomain=laravel

> {note} 谨记，Vagrant 本质上是不安全的，当你运行 `share` 命令的时候，你会把你的虚拟机暴露在互联网中。

<a name="multiple-php-versions"></a>
### 多个 PHP 版本

> {note} 这个特性仅与 Nginx 兼容。

Homestead 6 支持在同一个虚拟机上引入多个不同版本的 PHP 。您需要在 Homestead.yaml 配置文件中为某个站点指明需要使用的 PHP 版本即可。 可用的 PHP 版本有：「5.6」，「7.0」，「7.1」

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          php: "5.6"

此外，您还可以通过 CLI 使用任何支持的 PHP 版本：

    php5.6 artisan list
    php7.0 artisan list
    php7.1 artisan list

<a name="network-interfaces"></a>
## 网络接口

`Homestead.yaml` 文件里的 `networks` 配置项允许你为 Homestead 环境配置网络接口。您可以根据需要配置任意数量的接口：

    networks:
        - type: "private_network"
          ip: "192.168.10.20"

想要配置一个 [桥接](https://www.vagrantup.com/docs/networking/public_network.html) 接口的话，增加 `bridge` 配置项，然后 `type` 填写为 `public_network` ：

    networks:
        - type: "public_network"
          ip: "192.168.10.20"
          bridge: "en1: Wi-Fi (AirPort)"

想要配置一个 [DHCP](https://www.vagrantup.com/docs/networking/public_network.html) 接口的话，请从配置中移除 `ip` 选项：

    networks:
        - type: "public_network"
          bridge: "en1: Wi-Fi (AirPort)"

<a name="updating-homestead"></a>
## 更新 Homestead

你可以简单的用两个步骤来更新 Homestead ，第一步，使用 `vagrant box update` 命令更新 Vgrant box :

    vagrant box update

接下来。你需要更新 Homestead 的源代码，如果你是通过克隆仓库的方式来安装的 Homestead ，你可以在你最初克隆仓库的位置简单的运行 `git pull origin master` 命令。

如果你已经通过你的项目中的 `composer.json` 文件安装了 Homestead ，你应该确认你的 `composer.json` 文件中是否包含 `"laravel/homestead: "^4"` 并且还要更新依赖：

    composer update

<a name="old-versions"></a>
## 历史版本

> {tip} 如果你需要一个旧版本的 PHP ，请在尝试使用旧版本 Homestead 之前，先阅读文档 <a href="#multiple-php-versions">多个 PHP 版本</a>。

你可以通过添加以下配置到你的 `Homestead.yaml` 文件来方便的覆盖 Homestead 使用的 box 版本:

    version: 0.6.0

例如：

    box: laravel/homestead
    version: 0.6.0
    ip: "192.168.20.20"
    memory: 2048
    cpus: 4
    provider: virtualbox

当你使用旧版本的 box 时，你需要确保 Homestead 源代码的版本与之对应，下面的图表展示了支持的 box 版本，以及与之对应的 Homestead 的源代码版本和 box 所提供的 PHP 版本：

|   | Homestead Version | Box Version |
|---|---|---|
| PHP 7.0 | 3.1.0 | 0.6.0 |
| PHP 7.1 | 4.0.0 | 1.0.0 |

<a name="provider-specific-settings"></a>
## Provider 的特殊设置

<a name="provider-specific-virtualbox"></a>
### VirtualBox

Homestead 默认将 `natdnshostresolver` 设置为 `on`。这允许 Homestead 使用你的主机系统中的 DNS 设置。如果你想重写这行为，你可以在你的 `Homestead.yaml` 文件中添加下面这几行：

    provider: virtualbox
    natdnshostresolver: off

## 译者署名

| 用户名 | 头像 | 贡献 | 签名 |
|---|---|---|---|
| [WangYan](http://blog.wangyan.org)  | <img class="avatar-66 rm-style" src="http://imgcdn.wangyan.org/a/120x120.jpg">  |  翻译  | [About Me](http://blog.wangyan.org/about) |
