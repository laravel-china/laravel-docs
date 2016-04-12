# 任务调度

- [简介](#introduction)
- [定义调度](#defining-schedules)
    - [调度频率设置](#schedule-frequency-options)
    - [避免任务重复](#preventing-task-overlaps)
- [任务输出](#task-output)
- [任务挂勾](#task-hooks)

<a name="introduction"></a>
## 简介

在过去，开发者必须为每个需要调度的任务生成单独的 Cron 项目。然而令人头疼的是任务调度不受版本控制，并且需要 SSH 到服务器上来增加 Cron 项目。Laravel 命令调度器允许你在 Laravel 中对命令调度进行清晰流畅的定义，并且仅需要在服务器上增加一条 Cron 项目即可。

你的调度已经定义在 `app/Console/Kernel.php` 文件的 `schedule` 方法中。为了方便你开始，在该方法内包含了一个简单的例子。你可以随意增加调度到 `Schedule` 对象中。

### 启动调度器

底下是唯一一个需要加入到服务器的 Cron 项目：

    * * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1

该 Cron 将于每分钟调用一次 Laravel 命令调度器，接着 Laravel 会评判你的计划任务并运行预定任务。

<a name="defining-schedules"></a>
## 定义调度

你可以将所有的计划任务定义在 `App\Console\Kernel` 类的 `schedule` 方法中。在开始之前，先让我们来看看一个任务的调度示例。在该例子中，我们计划了一个会在午夜被调用的闭包。该闭包将运行清除某个数据表的数据库查找：

    <?php

    namespace App\Console;

    use DB;
    use Illuminate\Console\Scheduling\Schedule;
    use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

    class Kernel extends ConsoleKernel
    {
        /**
         * 应用程序提供的 Artisan 命令。
         *
         * @var array
         */
        protected $commands = [
            \App\Console\Commands\Inspire::class,
        ];

        /**
         * 定义应用程序的命令调度。
         *
         * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
         * @return void
         */
        protected function schedule(Schedule $schedule)
        {
            $schedule->call(function () {
                DB::table('recent_users')->delete();
            })->daily();
        }
    }

除了计划 `闭包` 调用，你还能计划 [Artisan 命令](/docs/{{version}}/artisan) 以及系统命令操作。举个例子，你可以使用 `command` 方法计划一个 Artisan 命令：

    $schedule->command('emails:send --force')->daily();

`exec` 命令可发送命令到操作系统上：

    $schedule->exec('node /home/forge/script.js')->daily();

<a name="schedule-frequency-options"></a>
### 调度频率设置

当然，你可以针对你的任务来分配多种调度计划：

方法  | 描述
------------- | -------------
`->cron('* * * * * *');`  |  在自定义的 Cron 调度运行该任务
`->everyMinute();`  |  每分钟运行该任务
`->everyFiveMinutes();`  |  每五分钟运行该任务
`->everyTenMinutes();`  |  每十分钟运行该任务
`->everyThirtyMinutes();`  |  每三十分钟运行该任务
`->hourly();`  |  每小时运行该任务
`->daily();`  |  每天午夜运行该任务
`->dailyAt('13:00');`  |  每天的 13:00 运行该任务
`->twiceDaily(1, 13);`  |  每天的 1:00 及 13:00 运行该任务
`->weekly();`  |  每周运行该任务
`->monthly();`  |  每月运行该任务
`->yearly();`  |  每年运行该任务

这些方法可以合并其它限制条件以生成更精确的调度。例如在某周的某几天运行调度。举个例子，计划一个每周周一的调度：

    $schedule->call(function () {
        // 在每个礼拜一的 13:00 运行一次...
    })->weekly()->mondays()->at('13:00');

下方列出其它额外限制条件：

方法  | 描述
------------- | -------------
`->weekdays();`  |  限制任务在工作日
`->sundays();`  |  限制任务在星期日
`->mondays();`  |  限制任务在星期一
`->tuesdays();`  |  限制任务在星期二
`->wednesdays();`  |  限制任务在星期三
`->thursdays();`  |  限制任务在星期四
`->fridays();`  |  限制任务在星期五
`->saturdays();`  |  限制任务在星期六
`->when(Closure);`  |  限制任务基于一个为真的验证

#### 为真验证限制条件

`when` 方法可以用来判断是否要运行任务，主要基于一个指定的为真验证的运行结果。换句话说，如果指定的 `闭包` 返回 `true`，且没有其它限制条件存在，那么这个任务将会被继续运行。

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });

当链式调用了 `when` 方法时，计划命令只有在所有的 `when` 条件返回 `true` 时才运行。

<a name="preventing-task-overlaps"></a>
### 避免任务重复

默认情况，即便之前相同的任务主体仍未结束，现有计划任务依旧会被运行。为了避免这个问题，你可以使用 `withoutOverlapping` 方法：

    $schedule->command('emails:send')->withoutOverlapping();

在这个例子中，如果没有其它 `emails:send` [Artisan 命令](/docs/{{version}}/artisan) 在运行的话，此任务将于每分钟被运行一次。当你有些任务运行时间过长，且无法预测出具体所需时间时，`withoutOverlapping` 方法将会特别有帮助。

<a name="task-output"></a>
## 任务输出

Laravel 调度器为任务调度输出提供多种便捷方法。首先，通过 `sendOutputTo` 你可以发送输出到单个文件上以便后续检查：

    $schedule->command('emails:send')
             ->daily()
             ->sendOutputTo($filePath);

如果想将输出附加到指定的文件上，则可以使用 `appendOutputTo` 方法：

    $schedule->command('emails:send')
             ->daily()
             ->appendOutputTo($filePath);

通过 `emailOutputTo` 方法，你可以发送输出到你所指定的电子邮件上。注意，你必须先通过 `sendOutputTo` 方法将其输出到一个文件。同时，在邮件发出之前，你需要先设置 Laravel 的[电子邮件服务](/docs/{{version}}/mail)：

    $schedule->command('foo')
             ->daily()
             ->sendOutputTo($filePath)
             ->emailOutputTo('foo@example.com');

> ** 注意：** `emailOutputTo` 与 `sendOutputTo` 方法只适用于 `command` 方法，并且不支持 `call` 方法。

<a name="task-hooks"></a>
## 任务挂勾

通过 `before` 与 `after` 方法，你能让特定的代码在任务完成之前及之后运行：

    $schedule->command('emails:send')
             ->daily()
             ->before(function () {
                 // 任务将要开始...
             })
             ->after(function () {
                 // 任务已完成...
             });

#### Ping 网址

通过 `pingBefore` 与 `thenPing` 方法，调度器能自动的在一个任务完成之前或之后 ping 一个指定的网址。该方法在你计划的任务进行或完成时，可用来有效的通知一个外部服务，例如 [Laravel Envoyer](https://envoyer.io)：

    $schedule->command('emails:send')
             ->daily()
             ->pingBefore($url)
             ->thenPing($url);

使用 `pingBefore($url)` 或 `thenPing($url)` 功能需要 Guzzle HTTP 函数库的支持。可在 `composer.json` 文件中加入以下代码来安装 Guzzle：

    "guzzlehttp/guzzle": "~5.3|~6.0"
