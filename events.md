# 事件

- [简介](#introduction)
- [注册事件或侦听器](#registering-events-and-listeners)
- [定义事件](#defining-events)
- [定义侦听器](#defining-listeners)
    - [可队列的事件侦听器](#queued-event-listeners)
- [触发事件](#firing-events)
- [广播事件](#broadcasting-events)
    - [设置](#broadcast-configuration)
    - [将事件标示为广播](#marking-events-for-broadcast)
    - [广播数据](#broadcast-data)
    - [消耗事件广播](#consuming-event-broadcasts)
- [事件订阅器](#event-subscribers)
- [框架事件](#framework-events)

<a name="introduction"></a>
## 简介

Laravel 事件提供了简单的侦听器实现，允许你订阅和监听事件，事件类通常被保存在 `app/Events` 目录下，而它们的侦听器保存在 `app/Listeners` 目录下。

<a name="registering-events-and-listeners"></a>
## 注册事件或侦听器

你可以在 `EventServiceProvider` 注册所有的事件侦听器，`listen` 属性是一个数组，包含所有事件（键）以及事件对应的侦听器（值），你也可以根据需求增加事件到这个数组，例如：让我们增加 `PodcastWasPurchased` 事件：

    /**
     * 应用程序的事件侦听器映射。
     *
     * @var array
     */
    protected $listen = [
        'App\Events\PodcastWasPurchased' => [
            'App\Listeners\EmailPurchaseConfirmation',
        ],
    ];

### 产生事件或侦听器类

你可以使用 `event:generate` 来协作你处理此类操作，这个命令会自动生成所有列出在 `EventServiceProvider` 的事件文件和侦听器文件，已经存在的事件和侦听器将保持不变：

    php artisan event:generate

### 手动注册事件

一般来说，事件必须通过 `EventServiceProvider` 的 `$listen` 数组进行注册；不过，你也可以通过 `Event` facade 或是 `Illuminate\Contracts\Events\Dispatcher` contract 实现的事件发送器来手动注册事件：

    /**
     * 注册你应用程序中的任何其他事件。
     *
     * @param  \Illuminate\Contracts\Events\Dispatcher  $events
     * @return void
     */
    public function boot(DispatcherContract $events)
    {
        parent::boot($events);

        $events->listen('event.name', function ($foo, $bar) {
            //
        });
    }

#### 事件侦听器的通配符

你也可以使用 `*` 通配符注册侦听器，让你可以在同个侦听器拦截多个事件。通配符侦听器会接收完整事件数据的数组作为唯一的参数：

    $events->listen('event.*', function (array $data) {
        //
    });

<a name="defining-events"></a>
## 定义事件

一个事件类只是一个数据容器包含了相关的事件信息。例如，假设我们产生了 `PodcastWasPurchased` 事件来接收一个 [Eloquent ORM](/docs/{{version}}/eloquent) 对象：

    <?php

    namespace App\Events;

    use App\Podcast;
    use App\Events\Event;
    use Illuminate\Queue\SerializesModels;

    class PodcastWasPurchased extends Event
    {
        use SerializesModels;

        public $podcast;

        /**
         * 创建一个新的事件实例。
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }
    }

正如你所见的，这个事件类没有包含特别的逻辑。它只是一个当 `Podcast` 对象被购买时的容器。如果事件对象是使用 PHP 的 `serialized` 函数进行序列化，那么事件所使用的 `SerializesModels` trait 将会优雅的序列化任何的 Eloquent 模型。

<a name="defining-listeners"></a>
## 定义侦听器

接下来，让我们看一下例子事件的侦听器。事件侦听器的 `handle` 方法接收了事件实例。`event:generate` 命令将会在事件的 `handle` 方法自动加载正确的事件类和类型提示。在 `handle` 方法内，你可以运行任何必要回应该事件的逻辑。

    <?php

    namespace App\Listeners;

    use App\Events\PodcastWasPurchased;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class EmailPurchaseConfirmation
    {
        /**
         * 创建事件侦听器。
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * 处理事件。
         *
         * @param  PodcastWasPurchased  $event
         * @return void
         */
        public function handle(PodcastWasPurchased $event)
        {
            // 使用 $event->podcast 访问播客（podcast）...
        }
    }

你的事件侦听器也可以在构造器内对任何依赖使用类型提示。所有事件侦听器经由 Laravel [服务容器](/docs/{{version}}/container)做解析，所以依赖将会自动的被注入：

    use Illuminate\Contracts\Mail\Mailer;

    public function __construct(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

#### 停止一个事件的传播

有时候，你可能希望停止一个事件的传播到其他的侦听器。你可以在侦听器的 `handle` 方法返回 `false` 达到这项目的。

<a name="queued-event-listeners"></a>
### 可队列的事件侦听器

需要一个可[队列](/docs/{{version}}/queues) 的事件侦听器吗？它是再容易不过了。只要增加 `ShouldQueue` 接口到你的侦听器类。由 `event:generate` Artisan 命令生成的侦听器已经将目前存在的接口加载到命名空间，所以你可以立即的使用它：

    <?php

    namespace App\Listeners;

    use App\Events\PodcastWasPurchased;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class EmailPurchaseConfirmation implements ShouldQueue
    {
        //
    }

如此而已！现在，当这个侦听器调用事件时，事件发送器会使用 Laravel 的[队列系统](/docs/{{version}}/queues) 自动的进行队列处理。如果侦听器是通过队列运行而没有抛出任何异常，已处理的队列任务将自动的被删除。

#### 手动访问队列

如果你需要手动访问底层队列任务的 `delete` 和 `release` 方法，在默认产生的侦听器会加载 `Illuminate\Queue\InteractsWithQueue` trait，这样你就可以访问这些方法了：

    <?php

    namespace App\Listeners;

    use App\Events\PodcastWasPurchased;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class EmailPurchaseConfirmation implements ShouldQueue
    {
        use InteractsWithQueue;

        public function handle(PodcastWasPurchased $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="firing-events"></a>
## 触发事件

如果要触发一件事件，你可以使用 `Event` [facade](/docs/{{version}}/facades)，发送一个事件的实例到 `fire` 方法。`fire` 方法将会发送事件到所有已经注册的侦听器：

    <?php

    namespace App\Http\Controllers;

    use Event;
    use App\Podcast;
    use App\Events\PodcastWasPurchased;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 显示给定用户的基本数据
         *
         * @param  int  $userId
         * @param  int  $podcastId
         * @return Response
         */
        public function purchasePodcast($userId, $podcastId)
        {
            $podcast = Podcast::findOrFail($podcastId);

            // 购买播客（podcast）逻辑...

            Event::fire(new PodcastWasPurchased($podcast));
        }
    }

另外，你也可以使用全局 `event` 辅助函数来触发事件：

    event(new PodcastWasPurchased($podcast));

<a name="broadcasting-events"></a>
## 广播事件

在构建实时响应的 Web App 时，经常会使用到 Web Sockets，当在服务器上更新一些数据，Web Socket 连接通常会发送一个消息通知客户端处理。

为了协助你创建这些类型的应用程序，Laravel 让你可以简单的经由 Web Socket 连接来「广播」你的事件。广播 Laravel 事件让你能够在服务器端代码和客户端 JavaScript 框架间分享相同的事件名称。

<a name="broadcast-configuration"></a>
### 配置

所有的事件广播设置选项都保存在 `config/broadcasting.php` 配置文件内。Laravel 内置支持多种广播驱动：[Pusher](https://pusher.com)、[Redis](/docs/{{version}}/redis)，和一个用于本机开发和调试的 `log` 驱动程序。配置文件例子包含了每个驱动程序。

#### 广播先决条件

事件广播需要以下的依赖：

- Pusher: `pusher/pusher-php-server ~2.0`
- Redis: `predis/predis ~1.0`

#### 队列先决条件

在广播事件之前，你还需要设置和运行[队列侦听器](/docs/{{version}}/queues)。所有事件广播经由队列任务完成，因此你的应用程序回应时间不会有严重影响。

<a name="marking-events-for-broadcast"></a>
### 将事件标示为广播

为了通知 Laravel 应该广播一个特定事件，在你的事件类实现 `Illuminate\Contracts\Broadcasting\ShouldBroadcast`。`ShouldBroadcast` 要求你实现单一方法：`broadcastOn`。`broadcastOn` 方法应该返回一个必须被广播的「频道」名称数组：

    <?php

    namespace App\Events;

    use App\User;
    use App\Events\Event;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ServerCreated extends Event implements ShouldBroadcast
    {
        use SerializesModels;

        public $user;

        /**
         * 创建一个新的事件实例。
         *
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * 取得事件应该被广播的频道。
         *
         * @return array
         */
        public function broadcastOn()
        {
            return ['user.'.$this->user->id];
        }
    }

接着，你只需要像往常的[触发事件](#firing-events)。一旦事件被触发之后，[队列任务](/docs/{{version}}/queues)将会自动的广播事件到你指定的广播驱动。

<a name="overriding-broadcast-event-name"></a>
#### 重写广播事件名称

默认情况下，广播事件名称会使用完整的事件类名称。以下方类为例子，该广播事件会是 `App\Events\ServerCreated`。你可以使用 `broadcastAs` 方法来自定你想要的广播事件名称：

    /**
     * 取得广播事件名称。
     *
     * @return string
     */
    public function broadcastAs()
    {
        return 'app.server-created';
    }

<a name="broadcast-data"></a>
### 广播数据

当事件被广播时，所有的 `public` 属性都自动的被序列化和广播作为事件的有效数据，允许你可以从你的 JavaScript 应用程序中访问任何公开的数据。所以，在这个例子中，假设事件有一个单一公开的 `$user` 属性且包含了一个 Eloquent 模型，广播数据将会是：

    {
        "user": {
            "id": 1,
            "name": "Jonathan Banks"
            ...
        }
    }

然而，如果你希望有更多精确的控制在你的广播数据，你可以增加 `broadcastWith` 方法到你的事件。这个方法应该返回一个你希望广播的事件数据数组：

    /**
     * 取得广播数据。
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['user' => $this->user->id];
    }

<a name="consuming-event-broadcasts"></a>
### 消耗事件广播

#### Pusher

通过 [Pusher](https://pusher.com) 驱动，你可以使用 Pusher 的 JavaScript SDK 方便的消耗事件广播。例如，让我们从先前的例子消耗 `App\Events\ServerCreated` 事件：

    this.pusher = new Pusher('pusher-key');

    this.pusherChannel = this.pusher.subscribe('user.' + USER_ID);

    this.pusherChannel.bind('App\\Events\\ServerCreated', function(message) {
        console.log(message.user);
    });

#### Redis

如果你使用 Redis 广播器，你需要编写自己的 Redis pub/sub 消耗器来接收消息和广播，并使用你选择的 WebSocket 技术。例如，你可能选择使用 Node 编写，很受欢迎的 [Socket.io](http://socket.io) 函数库。

使用 `socket.io` 和 `ioredis` Node 函数库，你可以快速的编写一个事件广播器，在你的 Laravel 应用程序发布所有事件的广播：

    var app = require('http').createServer(handler);
    var io = require('socket.io')(app);

    var Redis = require('ioredis');
    var redis = new Redis();

    app.listen(6001, function() {
        console.log('Server is running!');
    });

    function handler(req, res) {
        res.writeHead(200);
        res.end('');
    }

    io.on('connection', function(socket) {
        //
    });

    redis.psubscribe('*', function(err, count) {
        //
    });

    redis.on('pmessage', function(subscribed, channel, message) {
        message = JSON.parse(message);
        io.emit(channel + ':' + message.event, message.data);
    });

<a name="event-subscribers"></a>
## 事件订阅器

事件订阅器是一个类，让你可以在该类内订阅多个事件，允许你从单一类内定义多个事件的操作。订阅器应该定义一个 `subscribe` 方法，可以发送一个事件发送器实例：

    <?php

    namespace App\Listeners;

    class UserEventListener
    {
        /**
         * 处理用户登录事件。
         */
        public function onUserLogin($event) {}

        /**
         * 处理用户注销事件。
         */
        public function onUserLogout($event) {}

        /**
         * 注册侦听器的订阅者。
         *
         * @param  Illuminate\Events\Dispatcher  $events
         */
        public function subscribe($events)
        {
            $events->listen(
                'App\Events\UserLoggedIn',
                'App\Listeners\UserEventListener@onUserLogin'
            );

            $events->listen(
                'App\Events\UserLoggedOut',
                'App\Listeners\UserEventListener@onUserLogout'
            );
        }

    }

#### 注册事件订阅器

一旦订阅器被定义，它可以被注册到事件发送器。你可以在 `EventServiceProvider` 中使用 `$subscribe` 属性注册订阅器。例如，让我们增加 `UserEventListener`。

    <?php

    namespace App\Providers;

    use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * 事件侦听器映射到应用程序。
         *
         * @var array
         */
        protected $listen = [
            //
        ];

        /**
         * 订阅者类进行注册。
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventListener',
        ];
    }

<a name="framework-events"></a>
## 框架事件

Laravel 为框架运行的行为提供了许多「核心」事件。你可以使用与你订阅自定事件同样的方式订阅它们：

事件  |  参数
------------- | -----------
artisan.start | $application
auth.attempt | $credentials, $remember, $login
auth.login | $user, $remember
auth.logout | $user
cache.missed | $key
cache.hit | $key, $value
cache.write | $key, $value, $minutes
cache.delete | $key
connection.{name}.beginTransaction | $connection
connection.{name}.committed | $connection
connection.{name}.rollingBack | $connection
illuminate.query | $query, $bindings, $time, $connectionName
illuminate.queue.after | $connection, $job, $data
illuminate.queue.failed | $connection, $job, $data
illuminate.queue.stopping | null
mailer.sending | $message
router.matched | $route, $request
composing:{view name} | $view
creating:{view name} | $view
