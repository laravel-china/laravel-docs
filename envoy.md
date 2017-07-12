# Envoy 任务运行器

- [简介](#introduction)
- [编写任务](#writing-tasks)
    - [任务变量](#task-variables)
    - [多个服务器](#envoy-multiple-servers)
    - [任务宏](#envoy-task-macros)
- [运行任务](#envoy-running-tasks)
- [通知](#envoy-notifications)
    - [HipChat](#hipchat)
    - [Slack](#slack)

<a name="introduction"></a>
## 简介

[Laravel Envoy](https://github.com/laravel/envoy) 使用了 Blade 风格的语法，让你可以很方便的进行部署任务设置、Artisan 命令运行等操作。目前，Envoy 只支持 Mac 及 Linux 操作系统。

<a name="envoy-installation"></a>
### 安装

首先，先使用 Composer 的 `global` 命令安装 Envoy：

    composer global require "laravel/envoy=~1.0"

记得将 `~/.composer/vendor/bin` 目录加入至你的 PATH，这样才能在命令行运行 `envoy`。

#### 更新 Envoy

使用 Composer 来更新 Envoy 到最新版本：

    composer global update

> 译者注：注意上面的命令是更新所有的 composer 全局安装过的包。    

<a name="writing-tasks"></a>
## 编写任务

所有的 Envoy 任务都必须定义在项目根目录的 `Envoy.blade.php` 文件中，这里有个例子：

    @servers(['web' => 'user@192.168.1.1'])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask

如你所见，`@servers` 的数组被定义在文件的起始位置处，让你在声明任务时可以在 `on` 选项里参照使用这些服务器。在你的 `@task` 声明里，你可以放置当任务运行时想要在远程服务器运行的 Bash 命令。

#### 启动

有时，你可能想在任务启动前运行一些 PHP 代码。这时可以使用 ```@setup``` 区块在 Envoy 文件中声明变量以及运行普通的 PHP 程序：

    @setup
        $now = new DateTime();

        $environment = isset($env) ? $env : "testing";
    @endsetup

你也可以使用 ```@include``` 来引入任何外部 PHP 文件：

    @include('vendor/autoload.php')

#### 任务确认

如果你想要在运行任务之前进行提示确认，则可以增加 `confirm` 命令到任务声明：

    @task('deploy', ['on' => 'web', 'confirm' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="task-variables"></a>
### 任务变量

如果需要的话，你也可以通过命令行选项来传递变量至 Envoy 文件，以便自定义你的任务：

    envoy run deploy --branch=master

你可以通过 Blade 的「echo」语法使用这些选项：

    @servers(['web' => '192.168.1.1'])

    @task('deploy', ['on' => 'web'])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="envoy-multiple-servers"></a>
### 多个服务器

你可以在多个服务器上运行任务。首先，增加额外的服务器至你的 `@servers` 声明，每个服务器必须分配一个唯一的名称。一旦你定义好其它服务器，就能够在任务声明的 `on` 数组中列出这些服务器：

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2']])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

默认情况下，任务会按照顺序在每个服务器上运行。意味着任务会在第一个服务器运行完后才跳到下一个。

#### 平行运行

如果你想在多个服务器上同时运行任务，只需简单的在任务声明里加上 `parallel` 选项即可：

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="envoy-task-macros"></a>
### 任务宏

宏可以让你使用一个命令来定义要顺序运行的一组任务。举例来说，一个 `deploy` 宏可能会运行 `git` 及 `composer` 任务：

    @servers(['web' => '192.168.1.1'])

    @macro('deploy')
        git
        composer
    @endmacro

    @task('git')
        git pull origin master
    @endtask

    @task('composer')
        composer install
    @endtask

一旦该宏被定义之后，就可以通过一行简单的命令来运行它们：

    envoy run deploy

<a name="envoy-running-tasks"></a>
## 运行任务

要从你的 `Envoy.blade.php` 文件运行一个任务，只需运行 Envoy 的 `run` 命令，并传递你想运行的任务或宏命令的名称。Envoy 会运行该任务并显示任务运行时的服务器输出。

    envoy run task

<a name="envoy-notifications"></a>
<a name="envoy-hipchat-notifications"></a>
## 通知

<a name="hipchat"></a>
### HipChat

在任务运行之后，你可以使用 Envoy 的 `@hipchat` 发送通知到团队的 HipChat 聊天室。该命令接受的参数是 API token、聊天室名称、及发送消息时显示的用户名：

    @servers(['web' => '192.168.1.1'])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask

    @after
        @hipchat('token', 'room', 'Envoy')
    @endafter

你也可以自定义发送到 HipChat 聊天室的消息。任何在 Envoy 任务里可用的变量都能被使用在消息里：

    @after
        @hipchat('token', 'room', 'Envoy', "{$task} ran in the {$env} environment.")
    @endafter

<a name="slack"></a>
### Slack

除了 HipChat 之外，Envoy 也支持发送通知至 [Slack](https://slack.com)。`@slack` 命令接收 Slack hook 网址、频道名称、及你想发送至频道的消息：

    @after
        @slack('hook', 'channel', 'message')
    @endafter

当你在 Slack 的网站创建 `Incoming WebHooks` 时会获取一组 webhook 网址。`hook` 参数必须是 Slack 的 Incoming WebHooks 所提供的整串网址。例如：

    https://hooks.slack.com/services/ZZZZZZZZZ/YYYYYYYYY/XXXXXXXXXXXXXXX

你可以选择下方的任意一个来作为 channel 参数：

- 如果要发送通知至一个频道：`#channel`
- 如果要发送通知给一位用户：`@user`





--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译。
> 
> 文档永久地址： http://d.laravel-china.org