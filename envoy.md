# Laravel 的远程服务器任务处理器 Envoy

- [简介](#introduction)
    - [安装](#installation)
- [编写任务](#writing-tasks)
    - [任务启动](#setup)
    - [任务变量](#variables)
    - [任务故事](#stories)
    - [多个服务器](#multiple-servers)
- [运行任务](#running-tasks)
    - [任务确认](#confirming-task-execution)
- [通知](#notifications)
    - [Slack](#slack)

<a name="introduction"></a>
## 简介

[Laravel Envoy](https://github.com/laravel/envoy) 为定义在远程服务器上运行的通用任务提供了一种简洁、轻便的语法。它使用了 Blade 风格的语法，让你可以很方便的启动任务来进行项目部署、Artisan 命令运行等操作。目前，Envoy 只支持 Mac 及 Linux 操作系统。

<a name="installation"></a>
### 安装

首先，使用 Composer `global require` 命令来安装 Enovy ：

    composer global require laravel/envoy

因为 Composer 的全局库有时会导致包的版本冲突，所以你可以考虑使用 `cgr` ，它是 `composer global require` 命令的一种替代实现 `cgr` 库的安装指导可以在 [GitHub上找到](https://github.com/consolidation-org/cgr)。

> {note} 一定要确保 `~/.composer/vendor/bin` 目录加入到了你的 PATH 中，这样才能在命令行运行 `envoy` 。

#### 更新 Envoy

你也可以使用 Composer 来更新 Envoy 到最新版本。 要注意 `composer global update` 命令是更新你所有在全局安装的包：

    composer global update

<a name="writing-tasks"></a>
## 编写任务

所有的 Envoy 任务都必须定义在项目根目录的 `Envoy.blade.php` 文件中，这里有个例子：

    @servers(['web' => ['user@192.168.1.1']])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask

如你所见， `@servers` 的数组被定义在文件的起始位置处，让你在声明任务时可以在 `on` 选项里参照使用这些服务器。在你的  `@task` 声明里，你可以放置当任务运行时想要在远程服务器运行的 Bash 命令。

你可以通过指定服务器的 IP 地址为 `127.0.0.1` 来执行本地任务：

    @servers(['localhost' => '127.0.0.1'])

<a name="setup"></a>
### 任务启动

有时，你可能想在任务启动前运行一些 PHP 代码。这时可以使用 ```@setup``` 区块在 Envoy 文件中声明变量以及运行普通的 PHP 程序：

    @setup
        $now = new DateTime();

        $environment = isset($env) ? $env : "testing";
    @endsetup

如果你想在任务执行前引入其他 PHP 文件，可以直接在 `Envoy.blade.php` 文件起始位置使用 `@include` ：

    @include('vendor/autoload.php')

    @task('foo')
        # ...
    @endtask

<a name="variables"></a>
### 任务变量

如果需要的话，你也可以通过命令行选项来传递变量至 Envoy 文件，以便自定义你的任务：

    envoy run deploy --branch=master

你可以通过 Blade 的「echo」语法使用这些选项， 当然也能在任务里用「if」 和循环操作。举例来说，我们在执行 `git pull` 命令前，先检查 `$branch` 变量是否存在：

    @servers(['web' => '192.168.1.1'])

    @task('deploy', ['on' => 'web'])
        cd site

        @if ($branch)
            git pull origin {{ $branch }}
        @endif

        php artisan migrate
    @endtask

<a name="stories"></a>
### 任务故事

任务故事通过一个统一的、便捷的名字来划分一组任务，来让你把小而专的子任务合并到大的任务里。比如说，一个名为  `deploy` 的任务故事可以在它定义范围内列出子任务名字 `git` 和 `composer` 来运行各自对应的任务：

    @servers(['web' => '192.168.1.1'])

    @story('deploy')
        git
        composer
    @endstory

    @task('git')
        git pull origin master
    @endtask

    @task('composer')
        composer install
    @endtask

当 story 写好后，像运行普通任务一样运行它就好了：

    envoy run deploy

<a name="multiple-servers"></a>
### 多个服务器

你可以在多个服务器上运行任务。首先，增加额外的服务器至你的 `@servers` 声明，每个服务器必须分配一个唯一的名称。一旦你定义好其它服务器，就能够在任务声明的 `on` 数组中列出这些服务器：

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2']])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

#### 并行运行

默认情况下，任务会按照顺序在每个服务器上运行。这意味着任务会在第一个服务器运行完后才跳到第二个。如果你想在多个服务器上并行运行任务，只需简单的在任务声明里加上 `parallel` 选项即可：

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="running-tasks"></a>
## 运行任务

要想运行一个在 `Envoy.blade.php` 文件中定义好的任务或者故事，就执行 Envoy 的 `run` 命令，并将这个任务的名字传递给它。Envoy 会去执行这个任务并且把任务执行过程中的输出给打印出来：

    envoy run task

<a name="confirming-task-execution"></a>
### 任务确认

如果你想要在运行任务之前进行提示确认，则可以增加 `confirm` 命令到任务声明。这个选项对于破坏性的操作来说是相当有用的：

    @task('deploy', ['on' => 'web', 'confirm' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="notifications"></a>
<a name="hipchat-notifications"></a>
## 通知

<a name="slack"></a>
### Slack

Envoy 也支持任务执行完毕后发送通知至 [Slack](https://slack.com)。`@slack` 命令接收 Slack hook 网址和频道名称。你可以通在在 Slack 的控制面板上创建 「Incoming WebHooks」 时来检索 webhook 网址。webhook-url 参数必须是 `@slack` 的 Incoming WebHooks 所提供的完整网址：

    @finished
        @slack('webhook-url', '#bots')
    @endfinished

你可以选择下方的任意一个来作为 channel 参数：

<div class="content-list" markdown="1">
- 如果要发送通知至一个频道： `#channel`
- 如果要发送通知给一位用户： `@user`
</div>

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@司维](https://phphub.org/users/11602)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/11602_1487572457.png?imageView2/1/w/380/h/380">  |  翻译  |  [@leon0204](https://github.com/leon0204) at Github  |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org