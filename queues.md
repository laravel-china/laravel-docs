# Queues

- [介绍](#introduction)
- [实现任务类](#writing-job-classes)
    - [生产任务类](#generating-job-classes)
    - [任务类的结构](#job-class-structure)
- [推送任务到队列](#pushing-jobs-onto-the-queue)
    - [延时任务](#delayed-jobs)
    - [从请求分发任务](#dispatching-jobs-from-requests)
- [运行队列监听器](#running-the-queue-listener)
    - [监管配置](#supervisor-configuration)
    - [队列监听器守护进程](#daemon-queue-listener)
    - [用队列监听器守护进程部署](#deploying-with-daemon-queue-listeners)
- [处理失败的任务](#dealing-with-failed-jobs)
    - [失败任务事件](#failed-job-events)
    - [重试失败任务](#retrying-failed-jobs)

<a name="introduction"></a>
## 介绍

Laravel 队列服务提供统一的API集成了许多不同的后端队列。队列允许你延后执行一个耗时的任务，例如延迟到指定时间发送邮件，这样大幅地加快了应用程序处理请求的速度。

<a name="configuration"></a>
### 配置

队列的配置文件保存在 `config/queue.php`。在这个文件中你可以找到框架中所有队列驱动的连接设置，包括数据库、[Beanstalkd](http://kr.github.com/beanstalkd)、[IronMQ](http://iron.io)、[Amazon SQS](http://aws.amazon.com/sqs)、[Redis](http://redis.io) 和一个同步驱动（本地使用）。

还包括一个驱动 `null` ，其仅仅是简单地舍弃队列任务。

### 驱动的必备条件

#### 数据库

为了能够使用 `database` 驱动，你需要建立一个数据库来保存任务。可以执行 `queue:table` Artisan指令创建一个迁移来建立这个数据库。一旦创建迁移，你可以用下面的 `migrate` 命令来迁移你的数据库

    php artisan queue:table

    php artisan migrate

#### 其他队列的依赖

下面的依赖是使用对应的队列驱动所需的扩展包：

- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~3.0`
- IronMQ: `iron-io/iron_mq ~2.0`
- Redis: `predis/predis ~1.0`

<a name="writing-job-classes"></a>
## 实现任务类

<a name="generating-job-classes"></a>
### 生成任务类

默认情况下，你的应用中所有可以进入队列的任务都存放在目录 `app/Jobs` 下面。你可以使用 Artisan CLI 生成一个新的排队任务：

    php artisan make:job SendReminderEmail --queued

这个命令会在 `app/Jobs` 目录下面生产一个新的类。这个类会实现接口 `Illuminate\Contracts\Queue\ShouldQueue`，指示 Laravel 该任务应该推送到队列，而不是同步执行。

<a name="job-class-structure"></a>
### 任务类的结构

任务类非常简单，一搬只包含一个 ｀handle｀ 方法。队列处理任务的时候会调用这个方法。让我们看一下一个任务类的例子：

    <?php

    namespace App\Jobs;

    use App\User;
    use App\Jobs\Job;
    use Illuminate\Contracts\Mail\Mailer;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Bus\SelfHandling;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendReminderEmail extends Job implements SelfHandling, ShouldQueue
    {
        use InteractsWithQueue, SerializesModels;

        protected $user;

        /**
         * Create a new job instance.
         *
         * @param  User  $user
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * Execute the job.
         *
         * @param  Mailer  $mailer
         * @return void
         */
        public function handle(Mailer $mailer)
        {
            $mailer->send('emails.reminder', ['user' => $this->user], function ($m) {
                //
            });

            $this->user->reminders()->create(...);
        }
    }

在这个例子中可以看到，我们直接把一个 [Eloquent model](/docs/{{version}}/eloquent) 传入一个任务的构造器中。因为任务使用了 `SerializesModels` trait， Eloquent 会在任务处理过程中被优雅的序列化和反序列化。如果你的排队任务在其构造器中包含了一个 Eloquent 模型，仅仅是模型的标识符被序列化到队列。在真正处理任务的时候，队列系统会自动地从数据库中取回整改模型实例整个过程对于你的应用是透明的，而且处理所有在序列化整个 Eloquent 模型实例过程中可能产生的问题。

`handle` 方法在队列处理任务的时候被调用。我们可以在任务的 `handle` 方法中提示需要的依赖。Laravel ［服务容器］(/docs/{{version}}/container) 会自动注入这些依赖。

#### 处理问题

如果任务在被处理的时候抛出异常，任务会被自动释放回到队列中以便再次尝试执行该任务。任务会一直被释放回队列直到达到应用程序的尝试上限。这个上限值可以通过 Artisan 命令 `queue:listen` or `queue:work`的的  `--tries` 开关设置。运行队列监听器的更多信息 [可以在下文找到](#running-the-queue-listener)。

#### 手动释放任务

如果你想手动 `release` 一个任务，生产的任务类中包含了一个 `InteractsWithQueue` trait，提供入口进入队列任务的 `release` 方法。这个 `release` 方法接收一个参数：秒数，你希望等待的时间直到任务再次获取。

    public function handle(Mailer $mailer)
    {
        if (condition) {
            $this->release(10);
        }
    }

#### 检查任务尝试次数

正如上面提到的，如果在处理过程中出现异常，任务会自动释放回到队列。你可以使用 `attempts` 方法检任务已经被查尝运行的次数：

    public function handle(Mailer $mailer)
    {
        if ($this->attempts() > 3) {
            //
        }
    }

<a name="pushing-jobs-onto-the-queue"></a>
## 推送任务到队列

默认的 Laravel 控制器位于 `app/Http/Controllers/Controller.php`，使用了一个 `DispatchesJob` trait。这个 trait 提供了几个方法允许你方便地把任务推送到队列，下面就是一个
`dispatch` 方法：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Jobs\SendReminderEmail;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Send a reminder e-mail to a given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function sendReminderEmail(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $this->dispatch(new SendReminderEmail($user));
        }
    }

当然，有时候你可能希望从你的应用程序的某个地方分发一个任务，而不仅仅是从一个路由或者一个控制器。在那个情况下，你可以在你的应用程序的任何一个类中包含 `DispatchesJobs` trait来进入其多个分发方法例如，下面是一个使用该 trait的一个类：

    <?php

    namespace App;

    use Illuminate\Foundation\Bus\DispatchesJobs;

    class ExampleClass
    {
        use DispatchesJobs;
    }

#### 指定任务的执行队列

你可以指定一个任务应该被发送到的队列。

通过将任务推送到不同的队列，你可以“分类”你的排队任务，你甚至可以为不同的队列考虑不同数量的工作线程。这里不能将任务推送到队列配置文件中设置的不同队列“连接”
，而仅是同一个连接的不同的队列。使用任务实例的 `onQueue` 方法制定推送队列。该 `onQueue` 方法由 Laravel 中的 `App\Jobs\Job` 基类提供：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Jobs\SendReminderEmail;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Send a reminder e-mail to a given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function sendReminderEmail(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $job = (new SendReminderEmail($user))->onQueue('emails');

            $this->dispatch($job);
        }
    }

<a name="delayed-jobs"></a>
### 延时任务

有时候，你可能希望延时执行一个排队的任务。例如，你可能希望将一个在客户签约15分钟之后向其发送提醒邮件的任务排队。你可以使用任务类的 `delay` 方法完成这个，

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Jobs\SendReminderEmail;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Send a reminder e-mail to a given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function sendReminderEmail(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $job = (new SendReminderEmail($user))->delay(60);

            $this->dispatch($job);
        }
    }

在这个例子中，我们指明任务需要在队列中延时60秒才可以被工作线程再次获取。

> **注意:** Amazon SQS 服务最大的延时时间是15分钟。

<a name="dispatching-jobs-from-requests"></a>
### 从请求中分发任务

将 HTTP 请求映射成任务是非常普遍的。所有，为了免去你手动映射请求，Laravel 提供了一些辅助方法让这个做起更加容易。让我们看一下`DispatchesJobs` trait 中的`dispatchFrom` 方法。默认情况下，这个 trait 囊括在 Laravel 的基类控制器中：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class CommerceController extends Controller
    {
        /**
         * Process the given order.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function processOrder(Request $request, $id)
        {
            // Process the request...

            $this->dispatchFrom('App\Jobs\ProcessOrder', $request);
        }
    }

这个方法会坚持给定任务类的构造器，并且从 HTTP 请求（或者任何其他 `ArrayAccess` 类）中提取变量去填充类的构造参数。所以，如果我们的任务类的构造器接收变量 `productId` ， 类任务线会尝试从 HTTP 请求中抽取参数  `productId`。

你也可以传递一个数组作为 `dispatchFrom` 方法的第三个参数。这个数组会被用来填充任何从请求中获取不到的参数：

    $this->dispatchFrom('App\Jobs\ProcessOrder', $request, [
        'taxPercentage' => 20,
    ]);

<a name="running-the-queue-listener"></a>
## 运行队列监听器

#### 启动队列监听器

Laravel 包含了一个 Artisan 命令用来运行推送到队列中新的任务。你可以用 `queue:listen` 命令运行监听器：

    php artisan queue:listen

你也可以指定监听器使用的队列连接：

    php artisan queue:listen connection

需要注意，一旦这个任务被启动，它会一直运行直到被手动停止。你也可以使用一个监控进程，例如 [Supervisor](http://supervisord.org/)，来保证监听器不停地运行。

#### 队列优先级

你可以给监听的任务传递一个用逗号分割的队列连接列表，来设置队列的优先级：

    php artisan queue:listen --queue=high,low

在上面的例子中，队列 `high` 上的任务会总是优先于队列 `low` 上的任务被处理。

#### 指定任务超时参数

你可以设定每个任务允许执行的时间长度（秒）：

    php artisan queue:listen --timeout=60

#### 指定队列的休眠时长

另外，你可以设定轮询新任务之前等待的秒数：

    php artisan queue:listen --sleep=5

需要注意队列仅仅在其没有任务的时候才会“休眠”。如果有任务，队列会继续工作而不再进入休眠。

<a name="supervisor-configuration"></a>
### Supervisor 配置

supervisor 是 Linux 操作系统上的一个进程监控程序，在遇到失败的时候会自动的重启你的 `queue:listen` 或者 `queue:work` 命令。在 Ubuntu 上安装 Supervisor，你可以运行下面这条命令：

    sudo apt-get install supervisor

Supervisor 配置文件一般存放在目录 `/etc/supervisor/conf.d`。在这个目录中，你可以创建任意数目的配置文件来告诉 supervisor 怎样监管你的进程。例如，我们可以创建一个 `laravel-worker.conf` 文件来启动和监管 `queue:work` 进程。

    [program:laravel-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3 --daemon
    autostart=true
    autorestart=true
    user=forge
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/worker.log

在这个例子中，`numprocs` 指令会让 Supervisor 运行8个 `queue:work` 进程并监管它们，在遇到失败的时候自动重启它们。一旦创建配置文件，你可以使用下面的命令更新 Supervisor 配置并重启进程：

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start laravel-worker:*

关于配置和使用 supervisor 更多的信息，可以参考 [Supervisor 文档](http://supervisord.org/index.html)。另外，你也可以用 [Laravel Forge](https://forge.laravel.com) 在一个更加便捷的网页上自动配置和管理你的 Supervisor 配置项。

<a name="daemon-queue-listener"></a>
### 队列监听器守护进程

Artisan 命令 `queue:work` 包括一个 `--daemon` 选项，强制队列处理器持续处理任务，而不需要每次都重启框架。这样比起命令 `queue:listen` 可以有效的减少 CPU 的使用量：

要在守护进程模式启动队列处理器，使用 `--daemon` 参数：

    php artisan queue:work connection --daemon

    php artisan queue:work connection --daemon --sleep=3

    php artisan queue:work connection --daemon --sleep=3 --tries=3

如你所见，`queue:work` 命令支持 `queue:listen` 命令的大多相同的参数选项。你也可以使用 `php artisan help queue:work` 命令来查看所有可用的选项。

#### 队列监听器守护进程中的编码注意

队列监听器守护进程在处理每个请求之前不会重启框架，因此你需要在任务完成之前小心地释放所有占用资源。例如，如果你正在使用 GD 库处理图像，在完成处理的时候你应该调用 `imagedestroy` 方法来释放占用的内存。

同样的，在守护进程长时间执行期间，数据库连接可能中断。你可以使用 `DB::reconnect` 方法来确保你有一个全新的连接。
Similarly, your database connection may disconnect when being used by long-running daemon. You may use the `DB::reconnect` method to ensure you have a fresh connection.

<a name="deploying-with-daemon-queue-listeners"></a>
### 部署队列监听器守护进程

因为队列处理器守护进程是常驻进程，它们在没有重启的情况下是无法感知你代码的改动的。所以，最简单的使用队列监听器守护进程部署应用的方式是在部署脚本里面重启所有处理器。你在部署脚本中加入下面的命令，可以优雅的重启所有的处理器：

    php artisan queue:restart

这个命令会优雅的指示所有的队列处理器在完成当前任务后重新启动，所有不会丢失任何存在的任务。
This command will gracefully instruct all queue workers to restart after they finish processing their current job so that no existing jobs are lost.

> **注意:** 这个命令依赖缓存系统调度重启。默认情况下，APCu 对 CLI 任务不起作用。如果你在使用 APCu，把 `apc.enable_cli=1` 加到你的 APCu 配置中。

<a name="dealing-with-failed-jobs"></a>
## 处理失败任务

事情往往不会如你预期的一样，有时候你推送到队列的任务会执行失败。别担心，谁都避免不了这样的情况！Laravel 包含一个简单的方法用来指定一个任务最多可以被执行的次数。在执行任务超过最多尝试次数时，它将会添加至 `failed_jobs` 表中。失败任务的数据表名称可以在 `config/queue.php` 里进行设置。

你可以使用 `queue:failed-table` 命令为 `failed_jobs` 表创建一个迁移：

    php artisan queue:failed-table

在运行你的 [队列监听器](#running-the-queue-listener) 的时候，你可以指定一个最大值来限制一个任务应该最多被尝试执行的次数，只需要你执行命令 `queue:listen` 时加上 `--tries` 选项：

    php artisan queue:listen connection-name --tries=3

<a name="failed-job-events"></a>
### 失败任务事件

假如你想注册一个在队列任务失败时调用的事件，你可以使用 `Queue::failing` 方法。这个事件提供一个很好的机会，用e-mail 或者 [HipChat](https://www.hipchat.com) 通知你的团队。例如，你可以通过 Laravel 中的 `AppServiceProvider` 在这个事件上附加一个回调：

    <?php

    namespace App\Providers;

    use Queue;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Queue::failing(function ($connection, $job, $data) {
                // Notify team of failing job...
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

#### 任务类中的 Failed 方法

你可以在任务里中直接定义 `failed` 来做更加细粒度的控制，允许你在失败的情况下实施任务相关的特殊措施：

    <?php

    namespace App\Jobs;

    use App\Jobs\Job;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Bus\SelfHandling;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendReminderEmail extends Job implements SelfHandling, ShouldQueue
    {
        use InteractsWithQueue, SerializesModels;

        /**
         * Execute the job.
         *
         * @param  Mailer  $mailer
         * @return void
         */
        public function handle(Mailer $mailer)
        {
            //
        }

        /**
         * Handle a job failure.
         *
         * @return void
         */
        public function failed()
        {
            // Called when the job is failing...
        }
    }

<a name="retrying-failed-jobs"></a>
### 重试失败任务

你可以使用 Artisan 命令 `queue:failed` 查看数据库表 `failed_jobs` 中所有的失败任务：

    php artisan queue:failed

`queue:failed` 命令会列出任务的 ID，连接，队列，和失败的时间。任务的 ID 可以用来去重试失败的任务。例如，重试 ID 为 5 的任务，可以使用下面的命令：

    php artisan queue:retry 5

如果你想删除一个失败的任务，你可以用下面的 `queue:forget` 命令：

    php artisan queue:forget 5

删除所有的失败任务，你可以用下面的 `queue:flush` 命令：

    php artisan queue:flush
