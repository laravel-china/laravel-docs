# 队列

- [简介](#introduction)
- [编写任务类](#writing-job-classes)
    - [产生任务类](#generating-job-classes)
    - [任务类结构](#job-class-structure)
- [将任务推送到队列上](#pushing-jobs-onto-the-queue)
    - [延迟性任务](#delayed-jobs)
    - [从请求中派送任务](#dispatching-jobs-from-requests)
    - [任务事件](#job-events)
- [运行队列侦听器](#running-the-queue-listener)
    - [Supervisor 设置](#supervisor-configuration)
    - [将任务侦听器设为后台服务](#daemon-queue-listener)
    - [随着在后台服务的任务侦听器进行布署](#deploying-with-daemon-queue-listeners)
- [处理失败的任务](#dealing-with-failed-jobs)
    - [任务失败事件](#failed-job-events)
    - [重新尝试运行失败任务](#retrying-failed-jobs)

<a name="introduction"></a>
## 简介

Laravel 的队列服务为不同的队列后端系统提供一个统一的 API 。队列允许你将一个耗时的任务延迟处理，例如像寄送 e-mail，这会使得你的应用程序对网页请求有更快的反应。

<a name="configuration"></a>
### 配置

队列的配置文件被保存在 `config/queue.php`。在这个文件里你可以找到包含在 Laravel 框架中，每一种队列驱动的链接设置。它们包含了数据库、[Beanstalkd](http://kr.github.com/beanstalkd)、[IronMQ](http://iron.io)、[Amazon SQS](http://aws.amazon.com/sqs)、[Redis](http://redis.io) 以及提供本机使用的 synchronous 驱动。

另外框架也提供了 `null` 这个队列驱动，用来丢弃队列任务。

### 驱动必要设置

#### 数据库

要使用 `database` 这个队列驱动的话，需要创建一个数据表来记住任务，你可以用 `queue:table` 这个 Artisan 命令来创建这个数据表的迁移类。当迁移类建好后，就可以用 `migrate` 这个命令来将数据表在数据库中创建起来。

    php artisan queue:table

    php artisan migrate

#### 其他队列系统的依赖扩展包

要使用列表里的队列服务前，必须安装以下的依赖扩展包：

- Amazon SQS：`aws/aws-sdk-php ~3.0`
- Beanstalkd：`pda/pheanstalk ~3.0`
- IronMQ：`iron-io/iron_mq ~2.0|~4.0`
- Redis：`predis/predis ~1.0`

<a name="writing-job-classes"></a>
## 编写任务类

<a name="generating-job-classes"></a>
### 产生任务类

在你的应用程序中，所有能放在队列的任务类默认放在 `app/Jobs` 目录下，你可以用以下的 Artisan 命令来产生一个新的队列任务：

    php artisan make:job SendReminderEmail --queued

这个命令将会在 `app/Jobs` 下产生一个新类，而这个类会实现 `Illuminate\Contracts\Queue\ShouldQueue` 接口，让 Laravel 知道这个任务应该是被放到队列里，而不是直接运行。

<a name="job-class-structure"></a>
### 任务类结构

任务类的结构很简单，一般来说只会包含一个让队列用来调用此任务的 `handle` 方法。我们用以下这个类来做示范：

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
         * 创建一个新的任务实例。
         *
         * @param  User  $user
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * 运行任务。
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

注意，在这个例子里我们在任务类的构造器中直接传递了一个 [Eloquent 模型](/docs/{{version}}/eloquent)。因为我们在任务类里引用了 `SerializesModels` 这个 trait，使得 Eloquent 模型在处理任务的时候可以被优雅地序列化和反序列化。如果你的队列任务类在构造器接受一个 Eloquent 模型，那么只有可识别出该模型的属性会被序列化至队列里。当任务实际被运行时，队列系统会自动从数据库中重新取回完整的模型。整个过程对你的应用程序来说是透明的，这样可以避免序列化完整的 Eloquent 的模式实例所带来的问题。

在队列处理任务时，会调用 `handle` 方法，而这里我们也可以通过 `handle` 方法的参数类型提示，让 Laravel 的[服务容器](/docs/{{version}}/container)自动注入依赖对象。

#### 当发生错误的时候

如果在任务处理时抛出了一个异常，它会自动被释放回队列里再次尝试运行。当该任务一直出错时，它会不断被发布再重试，直到超过你的应用程序所允许的最大重试值。最大重试值可以在运行 `queue:listen` 或 `queue:work` 命令时，用 `--tries` 选项来设置；运行队列侦听器的更多信息在稍后会有[详细说明](#running-the-queue-listener)。

#### 手动释放任务

如果你想手动释放任务，那么在产生出来的任务类已经引用了 `InteractsWithQueue` 这个 trait，它提供了 `release` 方法让我们可以释放任务。在 `release` 方法中接受一个数值参数，它表示直到这个任务可以被重新运行之前，你愿意等待的秒数。

    public function handle(Mailer $mailer)
    {
        if (condition) {
            $this->release(10);
        }
    }

#### 检查重试次数

如同前面提到的，当任务被处理时发生了一个异常，它会自动被释放回队列中。这时候你可以用 `attempt` 方法来检查已经重试的次数：

    public function handle(Mailer $mailer)
    {
        if ($this->attempts() > 3) {
            //
        }
    }

<a name="pushing-jobs-onto-the-queue"></a>
## 将任务推送到队列上

在 `app/Http/Controllers/Controller.php` 中 Laravel 定义了一个默认控制器，它引用了 `DispatchesJob` 这个 trait；而这个 trait 提供了数个可以让你方便推送任务到队列的方法，例如 `dispatch` 方法：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Jobs\SendReminderEmail;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 寄送提醒的 e-mail 给指定用户。
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

当然有时你不见得是从应用程序的路由或控制器来派发任务，因此你可以在应用程序中的任何类里引用 `DispatchesJobs` 这个 trait ，以便使用它的各种派发方法。以下就是使用该 trait 的类例子：

    <?php

    namespace App;

    use Illuminate\Foundation\Bus\DispatchesJobs;

    class ExampleClass
    {
        use DispatchesJobs;
    }

#### 指定任务所属的队列

你可以指定任务应该要送到哪一个队列。

要推送任务到不同的队列上，你要将任务先「分类」，甚至可能要排定每个队列能有多少作业器可以运行任务。这并不是指任务会推送到你在配置文件所定义的不同队列链接里，而是推送到某个有单一链接的队列。要指定任务运行的队列，可以用任务实例的 `onQueue` 方法。`onQueue` 是 `Illuminate\Bus\Queueable` trait 所提供的方法，而它已经包含在 `App\Jobs\Job` 基类中：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Jobs\SendReminderEmail;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 寄送提醒的 e-mail 给指定用户。
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
### 延迟性任务

有时你可能会希望队列任务能晚一点再运行，例如在用户注册后 15 分钟后才通过队列任务寄送提醒信件。你可以通过任务类引用的 `Illuminate\Bus\Queueable` 这个 trait 所提供的 `delay` 方法来达成这个目的：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Jobs\SendReminderEmail;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 寄送提醒的 e-mail 给指定用户。
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

在这个例子里，我们指定该任务要在交给作业器运行前先延迟 60 秒。

> **注意：**Amazon 的 SQS 服务最大延迟时间是 15 分钟。

<a name="dispatching-jobs-from-requests"></a>
### 从请求中派送任务

在任务中对应到 HTTP 请求的变量是很常见的，所以与其强制你手动去对每个请求做这件事，Laravel 直接提供了一些辅助方法让你更容易做到它；像是在 `DispatchesJobs` 这个 trait 就提供了 `dispatchFrom` 方法，而 Laravel 基础控制器就默认引入了这个 trait：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class CommerceController extends Controller
    {
        /**
         * 处理指定的订单。
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function processOrder(Request $request, $id)
        {
            // 处理该请求...

            $this->dispatchFrom('App\Jobs\ProcessOrder', $request);
        }
    }

这个方法会检查任务类的构造器，并且从 HTTP 请求 (或任何 `ArrayAccess` 对象) 中取出变量，来填补构造器中需要的参数。所以如果我们的任务类构造器接受一个 `productId` 变量的话，那么队列就会试着从 HTTP 请求中提出 `productId` 这个参数。

你也可以直接将一个数组传入到 `dispatchFrom` 方法的第三个参数里，这个数组就会被用来填补构造器中任何不在请求里的变量：

    $this->dispatchFrom('App\Jobs\ProcessOrder', $request, [
        'taxPercentage' => 20,
    ]);

<a name="job-events"></a>
### 任务事件

#### 任务完成事件

`Queue::after` 方法让你能够注册一个回调，当队列任务运行完成后就会被运行。在此回调进行额外的纪录、队列后续任务、或为仪表板增加统计都是很好的时机。举个例子，我们可以在 Laravel 所包含的 `AppServiceProvider` 附加一个回调到此事件：

    <?php

    namespace App\Providers;

    use Queue;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 启动所有应用程序服务。
         *
         * @return void
         */
        public function boot()
        {
            Queue::after(function ($connection, $job, $data) {
                //
            });
        }

        /**
         * 注册服务提供者。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="running-the-queue-listener"></a>
## 运行队列侦听器

#### 启动队列侦听器

Laravel 引入了一个 Artisan 命令，用来运行被推送到队列里的任务。你可以通过 `queue:listen` 命令来运行侦听器：

    php artisan queue:listen

你也可以指定侦听器应该利用哪一个队列链接：

    php artisan queue:listen connection

要注意的是，一旦这个工作命令启动后，它会持续运作直到它被手动停止。你可以利用像 [Supervisor](http://supervisord.org/) 这样的行程监控软件，来确保队列侦听器不会停止运行。

#### 队列优先序

你可以给 `listen` 命令一个以逗号分隔的队列链接列表，来设置队列的优先序：

    php artisan queue:listen --queue=high,low

在这个例子里，在 `high` 这个队列里的任务永远会先被处理，然后才是 `low` 队列里的任务。

#### 指定任务的逾时参数

你还可以设置每个任务所被允许运行的时间长度，单位是秒数：

    php artisan queue:listen --timeout=60

#### 指定队列的休眠期

此外，你可以指定队列要等几秒再拿取新的任务来运行：

    php artisan queue:listen --sleep=5

注意这里是指队列在没有任务的状态下才会休眠；如果已经有多个任务卡在这个队列上，那么它会持续运作而不会休眠。

<a name="supervisor-configuration"></a>
### Supervisor 设置

Supervisor 是一个在 Linux 操作系统上的行程监控软件，它会在 `queue:listen` 或 `queue:work` 命令发生失败后自动重启它们。要在 Ubuntu 安装 Supervisor，可以用以下命令：

    sudo apt-get install supervisor

Supervisor 的配置文件一般是放在 `/etc/supervisor/conf.d` 目录下，在这个目录中你可以创建任意数量的配置文件，要求 Supervisor 来监控你的行程。例如我们创建一个 `laravel-worker.conf` 来启动与监控一个 `queue:work` 行程：

    [program:laravel-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3 --daemon
    autostart=true
    autorestart=true
    user=forge
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/worker.log

在这个例子里的 `numprocs` 命令会要求 Supervisor 运行并监控 8 个 `queue:work` 行程，并且在它们运行失败时重新启动。当然，你必须更改 `command` 命令的 `queue:work sqs` 部分，以显示你所选择的队列驱动。

当这个配置文件创建后，你需要更新 Supervisor 的设置，并用以下命令来启动该行程：

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start laravel-worker:*

更多有关 Supervisor 的设置与使用，请参考 [Supervisor 官方文档](http://supervisord.org/index.html)。或是你可以使用 [Laravel Forge](https://forge.laravel.com) 所提供的 Web 接口，来自动设置与管理你的 Supervisor 设置。

<a name="daemon-queue-listener"></a>
### 将任务侦听器设为后台服务

在 `queue:work` Artisan 命令里包含了 `--daemon` 选项，强迫队列服务器持续处理任务，而不需要重新启动整个框架。比起 `queue:listen` 命令，这会显著减少 CPU 的用量。

用 `--daemon` 旗标在后台服务模式下启动队列服务器：

    php artisan queue:work connection --daemon

    php artisan queue:work connection --daemon --sleep=3

    php artisan queue:work connection --daemon --sleep=3 --tries=3

如你所见，`queue:work` 命令提供了多数和 `queue:listen` 命令相同的选项；你可以用 `php artisan help queue:work` 命令来查看所有可用的选项。

### 在后台服务的队列侦听器中开发所要考量的事项

在后台运行的队列侦听器在处理完每个任务前，不会重新启动框架；因此你应该在任务运行完成前，谨慎地释放任何占用内存较重的资源。例如你利用 GD 函数库处理影像，就要在结束前用 `imagedestroy` 来释放内存。

相同地，你的数据库链接也要在使用完后关闭连接；你可以用 `DB::reconnect` 方法来确保有新的数据库连接。

<a name="deploying-with-daemon-queue-listeners"></a>
### 随着在后台服务的任务侦听器进行布署

从后台服务的队列服务器是长时间运行的行程来看，除非重新启动，否则它们将不会理会任何代码上的修改。所以要布署一个有用到后台服务的队列服务器的应用程序，最简单的方法就是在布署脚本文件中重新启动作业器。你可以在你的布署命令里加上以下命令，来优雅地重新启动所有作业器：

    php artisan queue:restart

这个命令会优雅地告知所有队列服务器，在它们完成处理目前任务后重新启动，所以就不会任务遗失的问题。

> **注意：**这个命令依靠缓存系统来安排重新启动；默认状况下，APCu 无法在 CLI 的任务上运作；如果你正在使用 APCu 的话，要把 `apc.enable_cli=1` 加到你的 APCu 设置里。

<a name="dealing-with-failed-jobs"></a>
## 处理失败的任务

计划永远跟不上变化，有时候你的队列任务就是会失败。不过别担心，我们有准备好它发生时的应付方法。Laravel 有个便利的方式可以指定任务的最大重试次数。当任务运行超过该重试次数，它就会被写入至 `failed_jobs` 这个数据表。而失败任务的名称可以在 `config/queue.php` 这个配置文件中设置。

要创建 `failed_jobs` 数据表的迁移类，你可以用 `queue:failed-table` 命令：

    php artisan queue:failed-table

当你运行[队列侦听器](#running-the-queue-listener)时，你可以用 `queue:listen` 命令的 `--tries` 参数来指定任务的最大重试次数：

    php artisan queue:listen connection-name --tries=3

<a name="failed-job-events"></a>
### 任务失败事件

如果你想注册一个当队列任务失败时会被调用的事件，你可以用 `Queue::failing` 方法；这样你就有机会通过这个事件，用 e-mail 或 [HipChat](https://www.hipchat.com) 来通知你的团队。例如我们可以在 Laravel 内置的 `AppServiceProvider` 中对这个事件附加一个回调函数：

    <?php

    namespace App\Providers;

    use Queue;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 启动任何应用程序的服务。
         *
         * @return void
         */
        public function boot()
        {
            Queue::failing(function ($connection, $job, $data) {
                // 通知团队失败的任务...
            });
        }

        /**
         * 注册服务容器。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

#### 任务类里处理失败的方法

如果想有更细腻的控制，你可以在直接在任务类里定义一个 `failed` 方法，这个方法允许你指定在错误发生时该怎么动作。

    <?php

    namespace App\Jobs;

    use App\Jobs\Job;
    use Illuminate\Contracts\Mail\Mailer;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Bus\SelfHandling;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendReminderEmail extends Job implements SelfHandling, ShouldQueue
    {
        use InteractsWithQueue, SerializesModels;

        /**
         * 运行任务。
         *
         * @param  Mailer  $mailer
         * @return void
         */
        public function handle(Mailer $mailer)
        {
            //
        }

        /**
         * 处理一个失败的任务
         *
         * @return void
         */
        public function failed()
        {
            // 当任务失败时会被调用...
        }
    }

<a name="retrying-failed-jobs"></a>
### 重新尝试运行失败任务

要查看你在 `failed_jobs` 数据表中所有失败的任务，你可以用 `queue:failed` 这个 Artisan 命令：

    php artisan queue:failed

`queue:failed` 命令会列出所有任务 ID、链接、队列以及失败时间，任务 ID 会用在重试失败的任务。例如要重试一个 ID 为 5 的失败任务，其命令如下：

    php artisan queue:retry 5

要重试所有失败的任务，可以使用 `queue:retry` 并使用 `all` 作为 ID：

    php artisan queue:retry all

如果你想删除掉一个失败任务，可以用 `queue:forget` 命令：

    php artisan queue:forget 5

`queue:flush` 命令可以让你删除所有失败的任务：

    php artisan queue:flush
