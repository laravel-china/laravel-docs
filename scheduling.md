# Laravel 的任务调度（计划任务）功能 Task Scheduling

- [简介](#introduction)

- [定义调度](#defining-schedules)

    - [Artisan 命令调度](#scheduling-artisan-commands)
    - [队列任务调度](#scheduling-queued-jobs)
    - [Shell 调度命令](#scheduling-shell-commands)
    - [调度频率设置](#schedule-frequency-options)
    - [避免任务重复](#preventing-task-overlaps)
    - [维护模式](#maintenance-mode)

- [任务输出](#task-output)
- [任务钩子](#task-hooks)

<a name="introduction"></a>
## 简介

在过去，开发者必须在服务器上为每个任务生成单独的 Cron 项目。而令人头疼的是任务调度不受源代码控制，而且必须通过 SSH 连接到服务器上来增加 Cron 项目。

Laravel 的命令调度程序允许你在 Laravel 中对命令调度进行清晰流畅的定义。并且在使用调度程序时，只需要在服务器上增加一条 Cron 项目即可。调度是在 `app/Console/Kernel.php` 文件的 `schedule` 方法中定义的。为了方便你开始，该方法已经包含了一个简单的例子。

### 启动调度器

使用调度器时，只需将以下 Cron 项目添加到服务器。如果你不知道如何将 Cron 项目添加到服务器，可以考虑使用 [Laravel Forge](https://forge.laravel.com) 等服务来管理你的 Cron 项目：

````
* * * * * php /path-to-your-project/artisan schedule:run >> /dev/null 2>&1
````

上面这个 Cron 会每分钟调用一次 Laravel 命令调度器。执行 `schedule:run` 命令时， Laravel 会根据你的调度运行预定任务。

<a name="defining-schedules"></a>
## 定义调度

你可以在 `App\Console\Kernel` 类的 `schedule` 方法中定义所有调度任务。在开始之前，先看看一个调度任务的例子。在该例子中，我们将计划在每天午夜调用一个 `Closure`。在这个 `Closure` 中，将执行一个数据库查询来清除一个表：

````
<?php

namespace App\Console;

use DB;
use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

class Kernel extends ConsoleKernel
{
    /**
     * 应用提供的 Artisan 命令
     *
     * @var array
     */
    protected $commands = [
        \App\Console\Commands\Inspire::class,
    ];

    /**
     * 定义应用的命令调度
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
````

<a name="scheduling-artisan-commands"></a>

### Artisan 命令调度

除了计划 `Closure` 调用，你还能调度 [Artisan 命令](/docs/{{version}}/artisan) 和操作系统命令。举个例子，你可以给 `command` 方法传递命令名称或者类名称来调度一个 `Artisan` 命令：

````
$schedule->command('emails:send --force')->daily();

$schedule->command(EmailsCommand::class, ['--force'])->daily();
````
<a name="scheduling-queued-jobs"></a>

### 队列任务调度

`job` 方法可以用来调度 [队列任务](/docs/{{version}}/queues)。这个方法提供了一种快捷方式来调度任务，无需使用 `call` 方法手动创建闭包来调度任务：

````
$schedule->job(new Heartbeat)->everyFiveMinutes();
````

<a name="scheduling-shell-commands"></a>
### Shell 命令调度

`exec` 方法可用于向操作系统发出命令：

````
$schedule->exec('node /home/forge/script.js')->daily();
````

<a name="schedule-frequency-options"></a>
### 调度频率设置

当然，你可以为你的任务分配多种调度计划：

| 方法                                | 描述                        |
| --------------------------------- | ------------------------- |
| `->cron('* * * * * *');`          | 在自定义的 Cron 时间表上执行该任务      |
| `->everyMinute();`                | 每分钟执行一次任务                 |
| `->everyFiveMinutes();`           | 每五分钟执行一次任务                |
| `->everyTenMinutes();`            | 每十分钟执行一次任务                |
| `->everyFifteenMinutes();`        | 每十五分钟执行一次任务               |
| `->everyThirtyMinutes();`         | 每半小时执行一次任务                |
| `->hourly();`                     | 每小时执行一次任务                 |
| `->hourlyAt(17);`                 | 每小时的第 17 分钟执行一次任务         |
| `->daily();`                      | 每天午夜执行一次任务                |
| `->dailyAt('13:00');`             | 每天的 13:00 执行一次任务          |
| `->twiceDaily(1, 13);`            | 每天的 1:00 和 13:00 分别执行一次任务 |
| `->weekly();`                     | 每周执行一次任务                  |
| `->monthly();`                    | 每月执行一次任务                  |
| `->monthlyOn(4, '15:00');`        | 在每个月的第四天的 15:00 执行一次任务    |
| `->quarterly();`                  | 每季度执行一次任务                 |
| `->yearly();`                     | 每年执行一次任务                  |
| `->timezone('America/New_York');` | 设置时区                      |

这些方法可以合并其它限制条件以生成更精确的调度。例如，计划每周周一的调度：

````
// 每周一的下午一点钟运行
$schedule->call(function () {
    //
})->weekly()->mondays()->at('13:00');

// 周一至周五上午 8 点至下午 5 点每小时运行一次...
$schedule->command('foo')
          ->weekdays()
          ->hourly()
          ->timezone('America/Chicago')
          ->between('8:00', '17:00');
````

以下是额外限制条件的列表：

| 方法                         | 描述                |
| -------------------------- | ----------------- |
| `->weekdays();`            | 将任务限制在工作日         |
| `->sundays();`             | 将任务限制在星期天         |
| `->mondays();`             | 将任务限制在星期一         |
| `->tuesdays();`            | 将任务限制在星期二         |
| `->wednesdays();`          | 将任务限制在星期三         |
| `->thursdays();`           | 将任务限制在星期四         |
| `->fridays();`             | 将任务限制在星期五         |
| `->saturdays();`           | 将任务限制在星期六         |
| `->between($start, $end);` | 限制任务运行在开始到结束时间范围内 |
| `->when(Closure);`         | 根据闭包函数的返回来限制任务    |

#### 时间范围限制

`between` 方法可以用来限制一天中某个时间范围内的任务执行：

````
$schedule->command('reminders:send')
    ->hourly()
    ->between('7:00', '22:00');
````

类似的，`unlessBetween` 方法可以用来在一段时间内排除任务的执行：

````
$schedule->command('reminders:send')
    ->hourly()
    ->unlessBetween('23:00', '4:00');
````

#### 闭包测试限制

`when` 方法可以用来根据给定的测试的结果来限制任务的执行。换句话说，如果给定的 `Closure` 返回 `true`，那么只要没有其他约束条件阻止任务运行，任务就会执行：

````
$schedule->command('emails:send')->daily()->when(function () {
    return true;
});
````

`skip` 方法可以被看作是 `when` 的逆过程。如果 `skip` 方法返回 `true` 的话，那么任务将不会运行：

````
$schedule->command('emails:send')->daily()->skip(function () {
    return true;
});
````

当链式调用多个 `when` 方法时，调度命令只有在所有的 `when` 条件返回 `true` 时才运行。

<a name="preventing-task-overlaps"></a>
### 避免任务重复

默认情况，即便有相同的任务还在运行，调度内的任务依旧会被执行。为了避免这个问题，你可以使用  `withoutOverlapping` 方法：

````
$schedule->command('emails:send')->withoutOverlapping();
````

在上面这个例子中，如果没有其它 [Artisan 命令](/docs/{{version}}/artisan)  `emails:send` 在运行的话，此任务将于每分钟被运行一次。如果你的任务在执行时间上有很大的不同，你无法准确预测给定任务需要多长时间，`withoutOverlapping` 方法将会特别有帮助。

<a name="maintenance-mode"></a>
### 维护模式

当 Laravel 处于 [维护模式](/docs/{{version}}/configuration#maintenance-mode) 时，Laravel 的调度功能将不会生效。这是因为我们不想让任务调度干扰你服务器上可能还未完成的项目。然而，如果你想强制某个任务在维护模式下运行的话，你可以使用 `evenInMaintenanceMode` 方法：

````
$schedule->command('emails:send')->evenInMaintenanceMode();
````

<a name="task-output"></a>
## 任务输出

Laravel 调度器提供了几个方便的方法来处理调度任务生成的输出。首先，使用 `sendOutputTo` 方法可以将输出发送到单个文件上以便后续检查：

````
$schedule->command('emails:send')
     ->daily()
     ->sendOutputTo($filePath);
````

如果想将输出附加到指定的文件上，则可以使用 `appendOutputTo` 方法:

````
$schedule->command('emails:send')
     ->daily()
     ->appendOutputTo($filePath);
````

使用 `emailOutputTo` 方法，你可以通过电子邮件将输出发送到你所指定的邮箱上。在发送任务的输出之前，你应该先配置 Laravel 的 [电子邮件服务](/docs/{{version}}/mail):

````
$schedule->command('foo')
     ->daily()
     ->sendOutputTo($filePath)
     ->emailOutputTo('foo@example.com');
````

> {note} `emailOutputTo`、`sendOutputTo` 和 `appendOutputTo` 方法是 `command` 方法才有的，不支持在 `call` 方法上使用。

<a name="task-hooks"></a>
## 任务钩子

通过 `before` 与 `after` 方法，你可以指定要在调度任务完成之前和之后执行的代码：

````
$schedule->command('emails:send')
     ->daily()
     ->before(function () {
         // 任务就要开始了…
     })
     ->after(function () {
         // 任务完成…
     });
````

#### Ping 网址

使用 `pingBefore` 与 `thenPing` 方法，调度器可以在任务完成之前或之后自动 ping 给定的 URL。这个方法对于通知外部服务（例如 [Laravel Envoyer](https://envoyer.io)）你的调度任务正在开始或已经完成执行很有用：

````
$schedule->command('emails:send')
     ->daily()
     ->pingBefore($url)
     ->thenPing($url);
````

无论是使用 `pingBefore($url)` 还是 `thenPing($url)` 的功能，都需要 Guzzle HTTP 函数库的支持。你可以使用 `Composer` 将 Guzzle 函数库添加到你的项目中：

````
composer require guzzlehttp/guzzle
````

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@沈益飞](https://laravel-china.org/users/13655) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/13655_1490162781.png?imageView2/1/w/100/h/100"> | 翻译 | [@m809745357](https://github.com/m809745357) at Github |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | Review | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
