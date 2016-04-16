# Laravel Homestead

- [简介](#introduction)
- [安装与设置](#installation-and-setup)
    - [第一步](#first-steps)
    - [配置 Homestead](#configuring-homestead)
    - [启动 Vagrant box](#launching-the-vagrant-box)
    - [根据项目分开安装](#per-project-installation)
- [常见用法](#daily-usage)
    - [通过 SSH 连接](#connecting-via-ssh)
    - [连接数据库](#connecting-to-databases)
    - [增加更多网站](#adding-additional-sites)
    - [设置 Cron 调度器](#configuring-cron-schedules)
    - [连接端口](#ports)

<a name="introduction"></a>
## 简介

Laravel 致力于让 PHP 的开发过程变得更加轻松愉快，这其中也包含你的本地开发环境。[Vagrant](http://vagrantup.com) 提供了一个简单、优雅的方式来管理与配置虚拟机。

Laravel Homestead 是一个官方预封装的 Vagrant box，提供给你一个美好的开发环境，你无需在本机电脑上安装 PHP、HHVM、Web 服务器或其它服务器软件。并且不用再担心系统被搞乱！Vagrant box 为你搞定一切。如果有什么地方出错了，你也可以在几分钟内快速的销毁并重建虚拟机！

Homestead 可以在任何 Windows、Mac 或 Linux 系统上面运行，里面包含了 Nginx Web 服务器、PHP 5.6、MySQL、Postgres、Redis、Memcached、Node，以及所有你在使用 Laravel 开发时所需要用到的各种软件。

> **附注：** 如果你是 Windows 用户，你可能需要启用硬件虚拟化（VT-x）。这通常需要通过 BIOS 来启用它。

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

<a name="installation-and-setup"></a>
## 安装与设置

<a name="first-steps"></a>
### 第一步

在你启动 Homestead 环境之前，须先安装 [VirtualBox 5.x](https://www.virtualbox.org/wiki/Downloads) 或 [VMWare](http://www.vmware.com) 以及 [Vagrant](http://www.vagrantup.com/downloads.html)。这些软件在各个常用的平台都有提供简单易用的界面安装包。

若要使用 VMware provider，你需要同时购买 VMware Fusion / Workstation 以及 [VMware Vagrant plug-in](http://www.vagrantup.com/vmware) 的软件授权。使用 VMware 可以在共享文件夹上获得较快的性能。

#### 安装 Homestead Vagrant box

当 VirtualBox / VMware 以及 Vagrant 安装完成后，你使用以下命令将 'laravel/homestead' 这个 box 安装进你的 Vagrant 程序中。box 的下载会花费你一点时间，具体的下载时长由网络速度决定：

    vagrant box add laravel/homestead

如果上面的命令运行失败，代表你使用的可能是旧版的 Vagrant，需要在命令行后补上下载包的完整 URL：

    vagrant box add laravel/homestead https://atlas.hashicorp.com/laravel/boxes/homestead

#### 安装 Homestead

你可以通过手动克隆代码仓库的方式来安装 Homestead。建议将代码仓库克隆至「home」目录中的 `Homestead` 文件夹，如此一来 Homestead box 就能将主机服务提供给你所有的 Laravel 项目：

    git clone https://github.com/laravel/homestead.git Homestead

如果你想尝试 PHP 7.0 版本的 Homestead，可以克隆代码仓库的 `php-7` 分支：

    git clone -b php-7 https://github.com/laravel/homestead.git Homestead

一旦你克隆完 Homestead 的代码仓库，即可在 Homestead 目录中运行 `bash init.sh` 命令来创建 `Homestead.yaml` 配置文件。`Homestead.yaml` 文件将会被放置在你的 `~/.homestead` 目录中：

    bash init.sh

<a name="configuring-homestead"></a>
### 配置 Homestead

#### 配置你的提供者

`Homestead.yaml` 文件中的 `provider` 参数设置取决于你用的是哪一个 Vagrant 提供者：`virtualbox`、`vmware_fusion` 或 `vmware_workstation`。你可以根据自己的喜好来设置提供者：

    provider: virtualbox

#### 配置共享文件夹

你可以在 `Homestead.yaml` 文件的 `folders` 属性里列出所有你想与你的 Homestead 环境共享的文件夹。这些文件夹中的文件若有更动，它们将会在你的本机电脑与 Homestead 环境自动更新同步。你可以在这里设置多个共享文件夹：

    folders:
        - map: ~/Code
          to: /home/vagrant/Code

若要启用 [NFS](http://docs.vagrantup.com/v2/synced-folders/nfs.html)，只需要在共享文件夹的设置值中加入一个简单的参数：

    folders:
        - map: ~/Code
          to: /home/vagrant/Code
          type: "nfs"

#### 配置 Nginx 网站

对 Nginx 不熟悉吗？没关系。`sites` 属性可以帮助你可以轻易指定一个 `域名` 来对应到 homestead 环境中的一个目录上。在 `Homestead.yaml` 文件中已包含了一个网站设置范本。同样的，你也可以增加多个网站到你的 Homestead 环境中。Homestead 可以为你正在开发中的所有 Laravel 项目提供方便的虚拟化环境：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public

你可以将 `hhvm` 属性设置为 `true` 来让 Homestead 里面的任意一个网站改用 [HHVM](http://hhvm.com)：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          hhvm: true

如果你在 Homestead box 配置之后更改了 `sites` 属性，那么你应该重新运行 `vagrant reload --provision` 来更新 Nginx 配置到虚拟机上。

#### 关于 Hosts 文件

你必须将在 Nginx sites 中所添加的「域名」也添加到你本机电脑的 `hosts` 上。`hosts` 文件会将你所发出的请求重定向至你在 Homestead 环境中设置的本地域名。在 Mac 或 Linux 上，该文件通常会存放在 `/etc/hosts`。在 Windows 上，则存放于 `C:\Windows\System32\drivers\etc\hosts`。设置内容如下所示：

    192.168.10.10  homestead.app

务必确认 IP 位置与 `Homestead.yaml` 文件中设置的相同。在将域名设置在 `hosts` 文件之后，你就可以通过网页浏览器访问你的网站啦！

    http://homestead.app

<a name="launching-the-vagrant-box"></a>
### 启动 Vagrant box

在你根据自己的喜好编辑完 `Homestead.yaml` 后，在命令行里进入你的 Homestead 目录并运行 `vagrant up` 命令。Vagrant 就会自将虚拟主机启动并自动设置共享文件夹和 Nginx 网站。

如果要移除虚拟机，可以使用 `vagrant destroy --force` 命令。

<a name="per-project-installation"></a>
### 根据项目分开安装

有别于将 Homestead 安装成全局环境且让所有的项目共用同一个 Homestead box，你可以各别为每一个项目独立配置一个 Homstead。如果你希望直接在项目里传递 `Vagrantfile`，那么你可以考虑替每个项目分别安装上 Homestead，其他人通过简单的 `vagrant up` 即能开始在此项目进行工作。

你可以使用 Composer 将 Homestead 直接安装至你的项目中：

    composer require laravel/homestead

一旦 Homestead 安装完毕，你可以使用 `make` 命令生成 `Vagrantfile` 与 `Homestead.yaml` 文件并存放于项目的根目录。这个 `make` 命令将会自动在 `Homestead.yaml` 文件中配置 `sites` 及 `folders`：

Mac / Linux:

    php vendor/bin/homestead make

Windows:

        vendor\bin\homestead make

接着，在命令行中运行 `vagrant up` 并通过网页浏览器访问 `http://homestead.app`。再次提醒，你仍然需要在 `/etc/hosts` 里设置 `homestead.app` 或其它想要使用的域名。

<a name="daily-usage"></a>
## 常见用法

<a name="connecting-via-ssh"></a>
### 通过 SSH 连接

在 Homestead 目录运行 `vagrant ssh` 命令来连接虚拟主机。

你可能会经常需要使用 SSH 来连接 Homestead 主机，因此你可以在本机电脑上创建一个「别名」以便快速连接 Homestead box。在创建这个别名之后，就可以轻易地通过「vm」这个命令用 SSH 连上你的 Homestead 主机：

    alias vm="ssh vagrant@127.0.0.1 -p 2222"

<a name="connecting-to-databases"></a>
### 连接数据库

在 `Homestead` 中，已经预装了 MySQL 与 Postgres 两种数据库。为了方便使用，Laravel 在 `local` 的数据库设置值中已经默认将其设置好了。

如果想要从本机电脑上通过 Navicat 或者是 Sequel Pro 来连接 MySQL 或 Postgres 数据库，可以通过 `127.0.0.1` 来使用端口 33060 (MySQL) 或 54320 (Postgres) 连接。帐号密码分别是 `homestead` / `secret`。

> **附注：** 在本机电脑上你应当只使用这些非标准的连接端口来连接数据库。因为当 Laravel 运行于虚拟主机时，在 Laravel 的数据库设置值中依然是设置使用默认的 3306 及 5432 连接端口。

<a name="adding-additional-sites"></a>
### 增加更多网站

一旦 Homestead 环境配置完毕且成功运行后，你可能会想要为 Laravel 应用程序增加更多的 Nginx 网站。你可以在单个 Homestead 环境中运行多个 Laravel 安装程序。在 `Homestead.yaml` 文件中增加另一个网站的设置后，进入 Homestead 目录并运行 `vagrant provision` 命令，即可新增一个网站。

<a name="configuring-cron-schedules"></a>
### 配置 Cron 调度器

Laravel 提供了便利的方式来[调度 Cron 任务](/docs/{{version}}/scheduling)，通过 `schedule:run` Artisan 命令，调度便会在每分钟被运行。`schedule:run` 命令会检查定义在你 `App\Console\Kernel` 类中调度的任务，判断哪个任务该被运行。

如果你想为 Homestead 网站使用 `schedule:run` 命令，你可以在定义网站时将 `schedule` 选项设置为 `true`：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          schedule: true

该网站的 Cron 任务会被定义在虚拟机的 `/etc/cron.d` 文件夹中。

<a name="ports"></a>
### 连接端口

以下的连接端口将会被转发至 Homestead 环境：

- **SSH：**2222 &rarr; 转发至 22
- **HTTP：**8000 &rarr; 转发至 80
- **HTTPS：**44300 &rarr; 转发至 443
- **MySQL：**33060 &rarr; 转发至 3306
- **Postgres：**54320 &rarr; 转发至 5432

#### 转发更多连接端口

如果你愿意的话，也可以借助指定连接端口的通信协议来转发更多额外的连接端口给 Vagrant box：

    ports:
        - send: 93000
          to: 9300
        - send: 7777
          to: 777
          protocol: udp
