# Laravel 队列监控面板 - Horizon

- [介绍](#introduction)
- [安装](#installation)
    - [配置](#configuration)
    - [仪表盘授权](#dashboard-authentication)
- [运行 Horizon](#running-horizon)
    - [部署 Horizon](#deploying-horizon)
- [标签](#tags)
- [通知](#notifications)
- [Metrics](#metrics)

<a name="introduction"></a>
## 介绍

Horizon 为 Laravel 官方出品的 Redis 队列提供了一个可以通过代码进行配置、并且非常漂亮的仪表盘，并且能够轻松监控队列的任务吞吐量、执行时间以及任务失败情况等关键指标。

队列执行者的所有配置项都存放在一个简单的配置文件中，所以团队可以通过版本控制进行协作维护。

<a name="installation"></a>
## 安装

> {note} 由于 Horizon 中使用了异步处理信号，所以需要 PHP 7.1+

可以使用 Composer 将 Horizon 安装进你的 Laravel 项目：

    composer require laravel/horizon

安装完成后，使用 `vendor:publish` Artisan 命令发布相关文件：

    php artisan vendor:publish --provider="Laravel\Horizon\HorizonServiceProvider"

<a name="configuration"></a>
### 配置

发布相关文件过程中，Horizon 的主要配置文件会被放置到 `config/horizon.php`，我们可以通过此文件配置队列执行者的所有配置项，此文件中的每个配置项都包含一份完整的使用说明，所以推荐认真阅读此文件。

#### 负载均衡配置

Horizon 有三种负载均衡策略：`simple`、`auto`、 和 `false`，默认策略是 `simple`，会将接收到的任务均分给队列进程：

    'balance' => 'simple',

策略 `auto` 会根据每个队列的压力自动调整其执行者进程数目，例如：如果 `notifications` 有 1000 个待执行的任务，但是你的 `render` 队列是空的，Horizon 会分派更多执行者进程给 `notifications` 队列，直到队列任务全部执行完毕（即队列为空）。当配置项 `balance` 设置为 `false` 时，Horizon 的执行策略与 Laravel 默认行为一致，及根据队列在配置文件中配置的顺序处理队列任务。

<a name="dashboard-authentication"></a>
### 仪表盘权限验证

Horizon 仪表盘的路由是 `/horizon` ，默认只能在 `local` 环境中访问仪表盘。我们可以使用 `Horizon::auth` 函数定义更具体的访问策略。`auth` 函数能够接受一个回调函数，此回调函数需要返回 `true` 或 `false` ，从而确认当前用户是否有权限访问 Horizon 仪表盘：

    Horizon::auth(function ($request) {
        // return true / false;
    });

<a name="running-horizon"></a>
## 运行 Horizon

修改 `config/horizon.php` 完成队列执行者的配置之后，可以使用 Artisan 命令 `horizon` 启动 Horizon，下面一条命令可以启动所有已配置的执行者：

    php artisan horizon

使用 Artisan 命令 `horizon:pause` 和 `horizon:continue` 来暂停和恢复队列的执行：

    php artisan horizon:pause

    php artisan horizon:continue

使用 Artisan 命令 `horizon:terminate` 来正常停止系统中的 Horizon 主进程，此命令执行时，Horizon 当前执行中的任务会被正常完成，然后 Horizon 执行结束：

    php artisan horizon:terminate

<a name="deploying-horizon"></a>
### 部署 Horizon

生产环境中，我们需要配置一个进程管理工具来监控 `php artisan horizon` 命令的执行，以便在其意外退出时自动重启。当服务器部署新代码时，需要终止当前 Horizon 主进程，然后通过进程管理工具来重启，从而使用最新的代码。

使用 Artisan 命令 `horizon:terminate` 来正常停止系统中的 Horizon 主进程，此命令执行时，Horizon 当前执行中的任务会被正常完成，然后 Horizon 执行结束：

    php artisan horizon:terminate

#### Supervisor 配置

可以使用进程管理工具 Supervisor 来管理 `horizon` 进程，下面配置文件就已够用：

    [program:horizon]
    process_name=%(program_name)
    command=php /home/forge/app.com/artisan horizon
    autostart=true
    autorestart=true
    user=forge
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/horizon.log

> {tip} 如果你不喜欢自己维护服务器，可以考虑使用 [Laravel Forge](https://forge.laravel.com) ，Forge 提供了运行一个带有 Horizon 的现代、强大的 Laravel 应用所需的 PHP7+ 以及其他所有环境。

<a name="tags"></a>
## 标签

Horizon 允许我们给队列任务打上一系列标签，包括 mailables、事件广播、通知以及队列中的时间侦听器，事实上，Horizon 会智能并且自动根据任务携带的 Eloquent 模型给大多数任务打上标签，如下任务示例：

    <?php

    namespace App\Jobs;

    use App\Video;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;

    class RenderVideo implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * The video instance.
         *
         * @var \App\Video
         */
        public $video;

        /**
         * Create a new job instance.
         *
         * @param  \App\Video  $video
         * @return void
         */
        public function __construct(Video $video)
        {
            $this->video = $video;
        }

        /**
         * Execute the job.
         *
         * @return void
         */
        public function handle()
        {
            //
        }
    }

如果此任务放入队列时携带了一个 `App\Video` 实例，此实例的 `id` 为 `1`，这个任务会接收到一个 `App\Video:1` 标签，这是因为 Horizon 会检查任务的所有属性是否携带 Eloquent 模型，如果发现携带，Horizon 会给该任务标记上模型的类名和主键：

    $video = App\Video::find(1);

    App\Jobs\RenderVideo::dispatch($video);

#### 自定义标签

如果需要自定义一个可被放入队列对象的标签，可以在此类中定义 `tags` 函数：

    class RenderVideo implements ShouldQueue
    {
        /**
         * Get the tags that should be assigned to the job.
         *
         * @return array
         */
        public function tags()
        {
            return ['render', 'video:'.$this->video->id];
        }
    }

<a name="notifications"></a>
## 通知

> **Note:** 使用通知之前，需要将 Composer 包 `guzzlehttp/guzzle` 安装到目标项目，如果配置 Horizon 发送短信通知，也要注意阅读[Nexmo 通知驱动的依赖条件](https://laravel.com/docs/5.4/notifications#sms-notifications)。

如果需要在队列等待时间过长时发起通知，可以在应用的 `AppServiceProvider` 中调用 `Horizon::routeSlackNotificationsTo` 和 `Horizon::routeSmsNotificationsTo` 函数：

    Horizon::routeSlackNotificationsTo('slack-webhook-url');

    Horizon::routeSmsNotificationsTo('15556667777');

#### 配置等待时间过长通知的阈值

可以在 `config/horizon.php` 中配置等待时间过长具体秒数，配置项 `waits` 可以针对每个 链接/队列 配置阈值：

    'waits' => [
        'redis:default' => 60,
    ],

<a name="metrics"></a>
## Metrics

Horizon 包含一个 metrics 仪表盘，它可以提供任务和队列等待时间和吞吐量信息，为了填充此仪表盘，需要使用应用的 [scheduler](/docs/{{version}}/scheduling) 每五分钟运行一次 Horizon 的 Artisan 命令 `snapshot`：

    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('horizon:snapshot')->everyFiveMinutes();
    }


## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@zhwei](https://github.com/zhwei)  | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/1446459?v=3&s=100">  |  翻译  | [欢迎加入蛋壳公寓](http://www.dankegongyu.com/about/join.html?from=laravel-doc-55)  |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org