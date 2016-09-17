# 事件

- [简介](#introduction)
- [注册事件与监听器](#registering-events-and-listeners)
    - [生成事件与监听器](#generating-events-and-listeners)
    - [手动注册事件](#manually-registering-events)
- [定义事件](#defining-events)
- [定义监听器](#defining-listeners)
- [队列化的事件监听器](#queued-event-listeners)
    - [手动访问队列](#manually-accessing-the-queue)
- [触发事件](#firing-events)
- [事件订阅者](#event-subscribers)
    - [编写事件订阅者](#writing-event-subscribers)
    - [注册事件订阅者](#registering-event-subscribers)

<a name="introduction"></a>
## 简介

Laravel 事件机制实现了一个简单的观察者模式，来订阅和监听在应用中出现的各种事件，事件类通常被保存在 `app/Events` 目录下，而它们的监听器被保存在 `app/Listeners` 目录下。如果你在应用中看不到这些文件夹也不要担心，当你用 Artisan 命令来生成事件和监听器的时候他们就会被创建了。

事件机制是种很好的对应用各方面进行解耦的一种方式，因为一个事件可以拥有多个互不依赖的监听器。举例来说，每次你把用户的订单发完货后都会希望给他发个 Slack 通知。这时候你就可以发起一个 `OrderShipped` 事件，它会被对应的监听器接收到再传递给 Slack 通知模块，这样你就不用把订单处理的代码跟 Slack 通知的代码耦合在一起了。

<a name="registering-events-and-listeners"></a>
## 注册事件和监听器

包含在你 Laravel 应用中的 `EventServiceProvider` 提供了一个很方便的地方来注册所有的事件监听器。`listen` 属性是一个数组，它包含了所有事件（键）以及事件对应的监听器（值）。你也可以根据应用需求来增加事件到这个数组，例如，我们增加一个 `OrderShipped` 事件：

    /**
     * 应用程序的事件监听器映射。
     *
     * @var array
     */
    protected $listen = [
        'App\Events\OrderShipped' => [
            'App\Listeners\SendShipmentNotification',
        ],
    ];

<a name="generating-events-and-listeners"></a>
### 生成事件和监听器

手动创建事件和监听器是很麻烦的，简单的方式是，在 `EventServiceProvider` 中写上事件和监听器然后使用 `event:generate` 命令。这个命令会自动生成在 `EventServiceProvider` 中列出的所有事件和监听器，当然已经存在的事件和监听器将保持不变：

    php artisan event:generate

<a name="manually-registering-events"></a>
### 手动注册事件

一般来说，事件必须通过 `EventServiceProvider` 的 `$listen` 数组进行注册；不过，你也可以通过 `Event` facade 或是 `Illuminate\Contracts\Events\Dispatcher` contract 实现的事件发送器来手动注册事件：

    /**
     * 注册你应用程序中的任何其它事件。
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

<a name="defining-events"></a>
## 定义事件

一个事件类是包含了相关事件信息的数据容器。例如，假设我们生成的 `OrderShipped` 事件接收一个 [Eloquent ORM](/docs/{{version}}/eloquent) 对象：

    <?php

    namespace App\Events;

    use App\Order;
    use App\Events\Event;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Event
    {
        use SerializesModels;

        public $order;

        /**
         * 创建一个事件实例
         *
         * @param  Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }
    }

正如你所见的，这个事件类没有包含其它逻辑。它只是一个被购买的 `Order` 对象的容器。如果事件对象是使用 PHP 的 `serialized` 函数进行序列化，那么事件所使用的 `SerializesModels` trait 将会优雅的序列化任何的 Eloquent 模型。

<a name="defining-listeners"></a>
## 定义监听器

接下来，让我们看一下例子中事件的监听器。事件监听器的 `handle` 方法接收了事件实例。`event:generate` 命令将会在事件的 `handle` 方法中自动加载正确的事件类和类型提示。在 `handle` 方法内，你可以运行任何需要响应该事件的业务逻辑。

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;

    class SendShipmentNotification
    {
        /**
         * 创建事件监听器。
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
         * @param  OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            // 使用 $event->order 来访问 order ...
        }
    }

> {tip} 你的事件监听器也可以在构造器内对任何依赖使用类型提示。所有事件监听器经由 Laravel [服务容器](/docs/{{version}}/container) 做解析，所以依赖将会被自动注入：

#### 停止事件传播

有时候，你可能希望停止一个事件传播到其它的监听器。你可以通过在侦听器的 `handle` 方法中返回 `false` 来实现。

<a name="queued-event-listeners"></a>
### 队列化的事件监听器

如果你对监听器要实现耗时任务比如发邮件或者进行 HTTP 请求，那把它放到队列中处理是有好处的。在使用队列化监听器之前，一定要在服务器或者本地开发环境中 [配置队列](/docs/{{version}}/queues) 并且开启一个队列监听器。


要指定一个监听器应该队列化的话，增加 `ShouldQueue` 接口到你的监听器类就好了。由 `event:generate` Artisan 命令生成的侦听器已经将此接口导入到命名空间了，因此可以像这样来立即使用它：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        //
    }

就这样！现在，当这个监听器被调用时，事件分发器会使用 Laravel 的 [队列系统](/docs/{{version}}/queues) 自动将它进行队列化。如果监听器通过队列运行而没有抛出任何异常，则已执行完的任务将会自动从队列中被删除。

<a name="manually-accessing-the-queue"></a>
### 手动访问队列

如果你需要手动访问底层队列任务的 `delete` 和 `release` 方法，你可以使用 `Illuminate\Queue\InteractsWithQueue` trait 来实现。这个 trait 在生成的监听器中是默认加载的，它提供了这些方法：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        public function handle(OrderShipped $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="firing-events"></a>
## 触发事件

如果要触发一个事件，你可以发送一个事件的实例到 `event` 辅助函数。这个函数将会把事件分发到它所有已经注册的监听器上。因为 `event` 函数是全局可访问的，所以你可以在应用中任何地方调用：

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * 将传递过来的订单发货
         *
         * @param  int  $orderId
         * @return Response
         */
        public function ship($orderId)
        {
            $order = Order::findOrFail($orderId);

            // 订单的发货逻辑...

            event(new OrderShipped($order));
        }
    }

> {tip} 测试时，不用真的触发监听器就能断言事件类型的话是很有用的。Laravel [内置的测试辅助方法](/docs/{{version}}/mocking#mocking-events) 让这件事变得很容易。

<a name="event-subscribers"></a>
## 事件订阅者

<a name="writing-event-subscribers"></a>
### 编写事件订阅者

事件订阅者是一个在自身内部可以订阅多个事件的类，允许你在单个类内定义多个事件的处理器。订阅者应该定义一个 `subscribe` 方法，这个方法将被传递过来一个事件分发器的实例。你可以调用事件分发器的 `listen` 方法来注册事件监听器：

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
         * 为订阅者注册监听器。
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

<a name="registering-event-subscribers"></a>
### 注册事件订阅者

一旦订阅者被定义，它就可以被注册到事件分发器中。你可以在 `EventServiceProvider` 中使用 `$subscribe` 属性注册订阅者。例如，我们增加 `UserEventListener` 到列表中：

    <?php

    namespace App\Providers;

    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * 应用中事件监听器的映射。
         *
         * @var array
         */
        protected $listen = [
            //
        ];

        /**
         * 要注册的订阅者类。
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventListener',
        ];
    }