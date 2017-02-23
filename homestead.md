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
    - [连接端口](#ports)
- [网络接口](#network-interfaces)
- [更新 Homestead](#updating-homestead)
- [历史版本](#old-versions)

<a name="introduction"></a>
## 简介

Laravel 致力于让 PHP 的开发过程更加轻松愉快，这其中也包括你的本地开发环境。 [Vagrant](https://www.vagrantup.com) 提供了一种简单，优雅的方式来管理和调配虚拟机。

Laravel Homestead 是一个官方预封装的 Vagrant box，它为你提供了一个完美的开发环境，你无需在本地安装 PHP ，web 服务器，或其他服务软件。并且不用担心系统被搞乱！ Vagrant box 是完全一次性的。如果有什么地方出错了，你也可以在几分钟内销毁并重建 box ！

Homestead 可以运行在 Windows 、 Mac 或 Linux 系统上，并且里面包含了 Nginx Web 服务器、 PHP 7.1 、 MySQL 、 Postgres 、 Redis 、 Memcached 、 Node 、 以及所有利于你开发 laravel 应用的其他程序。

> {note} 如果你是 Windows 用户，你可能需要启用硬件虚拟化（VT-x）。这通常需要通过 BIOS 来启用它。如果你在一个 UEFI 系统上使用的是 Hyper-V，你需要关闭 Hyper-V 才能启用 VT-x。

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

<a name="installation-and-setup"></a>
## 安装与设置

<a name="first-steps"></a>
### 第一步

在你启动你的 Homestead 环境之前，你必须安装 [VirtualBox 5.1](https://www.virtualbox.org/wiki/Downloads) 、 [VMWare](https://www.vmware.com) 或 [Parallels](http://www.parallels.com/products/desktop/) 的其中之一，以及 [Vagrant](https://www.vagrantup.com/downloads.html) 。这些软件在各个常用的平台都有提供简单易用的界面安装包。

若要使用 VMware provider，你需要同时购买 VMware Fusion / Workstation 以及 [VMware Vagrant plug-in](https://www.vagrantup.com/vmware). 的软件授权。使用 VMware 可以在共享文件夹上获得较快的性能。

若要使用 Parallels provider，你需要安装 [Parallels Vagrant plug-in](https://github.com/Parallels/vagrant-parallels) 。这是免费使用的。

#### 安装 Homestead Vagrant Box

当 VirtualBox / VMware 以及 Vagrant 安装完成后，你使用以下命令将 laravel/homestead 这个 box 安装进你的 Vagrant 程序中。 box 的下载会花费你一点时间，具体的下载时长由网络速度决定：

	vagrant box add laravel/homestead

如果上面的命令运行失败，代表你使用的可能是旧版的 Vagrant，请升级你的 Vagrant。

#### 安装 Homestead

你可以通过手动克隆代码仓库的方式来安装 Homestead。建议将代码仓库克隆至「home」目录中的 Homestead 文件夹，如此一来 Homestead box 就能将主机服务提供给你所有的 Laravel 项目：

    cd ~

    git clone https://github.com/laravel/homestead.git Homestead

一旦你克隆完 Homestead 的代码仓库，即可在 Homestead 目录中运行 `bash init.sh` 命令
来创建 `Homesstead.yaml` 配置文件。 `Homesstead.yaml` 文件会被放置在你的 Homestead 目录中：

    // Mac / Linux...
    bash init.sh

    // Windows...
    init.bat

<a name="configuring-homestead"></a>
### 配置 Homestead

#### 配置你的提供者

`Homestead.yaml` 中的 `provider` 参数设置取决于你用的是哪一个 Vagrant 提供者 `virtualbox ` 、 `vmware_fusion` 、 `vmware_workstation` ，或者 `parallels` 。你可以根据自己的喜好来设置提供者：

	provider: virtualbox


#### 配置共享文件夹

你可以在 `Homestead.yaml` 文件的 `folders` 属性里列出所有想与 Homestead 环境共享的文件夹。这些文件夹中的文件若有变更，它们将会在你的本机电脑与 Homestead 环境自动更新同步。你可以在这里设置多个共享文件夹：

	folders:
        - map: ~/Code
          to: /home/vagrant/Code

若要启动 [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html) ,只需要在共享文件夹的设置值中加入一个简单的参数：

    folders:
            - map: ~/Code
              to: /home/vagrant/Code
              type: "nfs"

你也可以在配置中传递任何 Vagrant 中的 [共享文件夹](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) 支持的参数，在 `options` 参数下列出它们：

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

务必确认 IP 地址与 `Homestead.yaml` 文件中设置的相同。将域名设置在 `hosts` 文件之后，你就可以通过网页浏览器访问你的网站。

	http://homestead.app

<a name="launching-the-vagrant-box"></a>
### 启动 Vagrant Box

编辑完 `Homestead.yaml` 后，进入你的 Homestead 目录并运行 `vagrant up` 命令。 Vagrant 就会根据 `Homestead.yaml` 里的配置信息，为虚拟机设置共享文件夹和 Nginx 网站。

如果要移除虚拟机，你可以使用 `vagrant destroy --force` 命令

<a name="per-project-installation"></a>
### 为每个项目分开安装

除了全局使用同一个 Homestead 环境， Homestead 还允许你为项目独立配置一个独占的 Homstead 。

通过传递 `Vagrantfile` ，可以实现为每个项目分别安装上 Homestead ，其他项目成员只需要通过简单的 `vagrant up` 即能跟你拥有一样的 Homestead 环境。

使用 Composer 将 Homestead 直接安装至项目中：

	 composer require laravel/homestead --dev

一旦 Homestead 安装完毕，可以使用 `make` 命令生成 `Vagrantfile` 与 `Homestead.yaml` 文件，并存放于项目的根目录。

`make` 命令将会自动在 Homestead.yaml 文件中配置 `sites` 及 `folders` ：

Mac / Linux:

	php vendor/bin/homestead make

Windows:

	vendor\\bin\\homestead make

接下来，在命令行中运行 `vagrant up` 并通过网页浏览器访问 `http://homestead.app` 。再次提醒，你仍然需要在 `/etc/hosts` 里配置 `homestead.app` 或其它想要使用的域名。

<a name="installing-mariadb"></a>
### 安装 MariaDB

如果你希望使用 MariaDB 来替换 MySQL，你可以在 `Homestead.yaml` 文件中增加一个 `mariadb` 的选项，这个选项会移除 MySQL 并安装 MariaDB。因为 MariaDB 可用作 MySQL 的替代品，所以在你的数据库配置信息里，还是选用 `mysql` 配置项。

	box: laravel/homestead
	ip: "192.168.20.20"
	memory: 2048
	cpus: 4
	provider: virtualbox
	mariadb: true

<a name="daily-usage"></a>
## 常见用法

<a name="accessing-homestead-globally"></a>
### 全局使用

如果你希望在文件系统的任何地方都可以 `vagrant up` 开启 Homestead 虚拟机，你可以把以下代码放到你的 Bash profile 里面，这个函数允许你在文件系统的任何位置都可以对 Homestead 运行 Vagrant 命令：

#### Mac / Linux

    function homestead() {
    	( cd ~/Homestead && vagrant $* )
    }

请将 `~/Homestead` 这个路径修改为你的实际 Homestead 的安装路径，一旦这个函数安装成功，就可以在系统的任意位置运行 `homestead up` 或 `homestead ssh` 命令

#### Windows

在系统的任意位置创建一个批处理文件 `homestead.bat` ,并添加如下内容：

	@echo off

    set cwd=%cd%
    set homesteadVagrant=C:\Homestead

    cd /d %homesteadVagrant% && vagrant %*
    cd /d %cwd%

    set cwd=
    set homesteadVagrant=

请将 `C:\Homestead` 这个路径修改为你的实际 Homestead 的安装路径，创建完这个文件后，将这个文件路径添加到 `PATH` 环境变量中，就可以在系统的任意位置运行 `homestead up` 或 `homestead ssh` 命令

<a name="connecting-via-ssh"></a>
### 通过SSH连接

在 Homestead 目录运行 `vagrant ssh` 命令来连接虚拟主机。

你可能会经常需要使用 SSH 来连接 Homestead 主机，你可以考虑将上述「function」添加到你的主机，以便快速的通过 SSH 进入你的 Homestead box

<a name="connecting-to-databases"></a>
### 连接数据库

在 box 中已经为 MySQL 和 Postgres 配置好了一个开箱即用的数据库 `homestead` ，为了更方便的使用它，Laravel 中的 `.env` 文件将这个数据库设置成了框架默认使用的数据库。

如果想要从你主机上的数据库客户端连接 MySQL 或 Postgres，可以通过 `127.0.0.1` 来使用端口 `33060`(MySQL) 或 `54320`(Postgres) 连接。账号密码分别是 `homestead` / `secret`

> {note} 因为虚拟机做了端口转发，所以本机电脑上你应当只使用这些非标准的连接端口，虚拟机里依然使用默认的 3306 及 5432 连接端口。

<a name="adding-additional-sites"></a>
### 增加更多网站

一旦 Homestead 环境配置完毕且成功运行后，你可能会想要为 Laravel 应用程序增加更多的 Nginx 网站。你可以在单个 Homestead 环境中运行多个 Laravel 程序。要添加额外的网站，只需将网站添加到您的 `Homestead.yaml` 文件中：

     sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
        - map: another.app
          to: /home/vagrant/Code/another/public

如果 Vagrant 没有自动管理你的「hosts」文件，你可能需要手动把新增的站点加入到「hosts」文件中：

	192.168.10.10  homestead.app
    192.168.10.10  another.app

当你的网站添加完成，从你的 Homestead 目录运行 `vagrant reload --provision` 命令就可以应用新的更改。

<a name="configuring-cron-schedules"></a>
### 配置 Cron 调度器

Laravel 提供了便利的方式来 [调度 Cron 任务](/docs/{{version}}/scheduling) ，通过 `schedule:run` Artisan 命令，调度便会在每分钟被运行。 `schedule:run` 命令会检查定义在你 `App\Console\Kernel` 类中调度的任务，判断哪个任务该被运行。

如果你想为 Homestead 网站使用 `schedule:run` 命令，你可以在定义网站时将 `schedule` 选项设置为 `true`

	sites:
    	- map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          schedule: true

该网站的 Cron 任务会被定义在虚拟机的 `/etc/cron.d` 文件夹中。

<a name="ports"></a>
### 连接端口

以下本地电脑连接端口将会被转发至 Homestead 环境：

- **SSH:** 2222 &rarr; Forwards To 22
- **HTTP:** 8000 &rarr; Forwards To 80
- **HTTPS:** 44300 &rarr; Forwards To 443
- **MySQL:** 33060 &rarr; Forwards To 3306
- **Postgres:** 54320 &rarr; Forwards To 5432

#### 转发更多的端口

如果你需要的话，也可以借助指定连接端口的通信协议来转发更多额外的连接端口给 Vagrant box ：

 	ports:
    	- send: 93000
          to: 9300
        - send: 7777
          to: 777
          protocol: udp


<a name="network-interfaces"></a>
## 网络接口

## Network Interfaces

`Homestead.yaml` 文件里的 `networks` 配置项允许你为 Homestead 环境配置网络接口。你可以任意配置多个网络接口：

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

接下来。你需要更新 Homestead 的源代码，如果你是通过克隆仓库的方式安装的 Homestead ，你可以简单的运行 `git pull origin master` 命令在你最初克隆仓库的位置。

如果你已经通过你的项目中的 `composer.json` 文件安装了 Homestead ，你应该确认你的 `composer.json` 文件中是否包含 `"laravel/homestead: "^4"` 并且更新你的依赖：

	composer update

<a name="old-versions"></a>
## 历史版本
 
你可以通过添加以下配置到你的 `Homestead.yaml` 文件来方便的覆盖 Homestead 使用的 box 版本

	version: 0.6.0

例如：

	box: laravel/homestead
    version: 0.6.0
    ip: "192.168.20.20"
    memory: 2048
    cpus: 4
    provider: virtualbox

当你使用旧版本的 box 时，你需要确保 Homestead 源代码的版本与之对应，下面的图表展示了支持的 box 版本，以及与之对应的 Homestead 的源代码版本和 box 所提供的 PHP 版本：

|         | Homestead Version | Box Version |
|---------|-------------------|-------------|
| PHP 7.0 |       3.1.0       |    0.6.0    |
| PHP 7.1 |       4.0.0       |    1.0.0    |


## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@lybc](https://laravel-china.org/users/1232)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/1232_1469434413.jpg?imageView2/1/w/100/h/100">  |  翻译  | 有用是有趣的前提  |