# Laravel 的事件系统

- [简介](#introduction)
- [注册事件与监听器](#registering-events-and-listeners)
    - [生成事件与监听器](#generating-events-and-listeners)
    - [手动注册事件](#manually-registering-events)
- [定义事件](#defining-events)
- [定义监听器](#defining-listeners)
- [队列化事件监听器](#queued-event-listeners)
    - [手动访问队列](#manually-accessing-the-queue)
    - [处理失败任务](#handling-failed-jobs)
- [触发事件](#dispatching-events)
- [事件订阅者](#event-subscribers)
    - [编写事件订阅者](#writing-event-subscribers)
    - [注册事件订阅者](#registering-event-subscribers)

<a name="introduction"></a>
## 简介

Laravel 事件机制实现了一个简单的观察者模式，让我们可以订阅和监听应用中出现的各种事件。事件类 (Event) 类通常保存在 `app/Events` 目录下，而它们的监听类 (Listener) 类被保存在 `app/Listeners` 目录下。如果你在应用中看不到这些文件夹也不要担心，因为当你使用 Artisan 命令来生成事件和监听器时他们会被自动创建。

事件机制是一种很好的应用解耦方式，因为一个事件可以拥有多个互不依赖的监听器。例如，每次把用户的订单发完货后都希望给他发个 Slack 通知。这时候你可以发起一个 `OrderShipped` 事件，它会被监听器接收到再传递给 Slack 通知模块，这样你就不用把订单处理的代码跟 Slack 通知的代码耦合在一起了。

<a name="registering-events-and-listeners"></a>
## 注册事件和监听器

Laravel 应用中的 `EventServiceProvider` 提供了一个很方便的地方来注册所有的事件监听器。它的 `listen` 属性是一个数组，包含所有的事件（键）以及事件对应的监听器（值）。你也可以根据应用需求来增加事件到这个数组中。例如，增加一个 `OrderShipped` 事件：

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

当然，手动创建每个事件和监听器是很麻烦的。简单的方式是，在 `EventServiceProvider` 类中添加好事件和监听器，然后使用 `event:generate` 命令。这个命令会自动生成 `EventServiceProvider` 类中列出的所有事件和监听器。当然已经存在的事件和监听器将保持不变：

    php artisan event:generate

<a name="manually-registering-events"></a>
### 手动注册事件

一般来说，事件必须通过 `EventServiceProvider` 类的 `$listen` 数组进行注册；不过，你也可以在 `EventServiceProvider` 类的 `boot` 方法中注册闭包事件。

    /**
     * 注册应用程序中的任何其他事件。
     *
     * @return void
     */
    public function boot()
    {
        parent::boot();

        Event::listen('event.name', function ($foo, $bar) {
            //
        });
    }

#### 通配符事件监听器

你甚至可以在注册监听器时使用 `*` 通配符参数，它让你在一个监听器中可以监听到多个事件。通配符监听器接受的第一个参数是事件名称，第二个参数是整个的事件数据：

    Event::listen('event.*', function ($eventName, array $data) {
        //
    });

<a name="defining-events"></a>
## 定义事件

事件类就是一个包含与事件相关信息数据的容器。例如，假设我们生成的 `OrderShipped` 事件接受一个 [Eloquent ORM](/docs/{{version}}/eloquent) 对象：

    <?php

    namespace App\Events;

    use App\Order;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped
    {
        use SerializesModels;

        public $order;

        /**
         * 创建一个事件实例。
         *
         * @param  Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }
    }

正如你所见，这个事件类中没有包含其它逻辑。它仅只是一个被构建的 `Order` 对象的容器。如果使用 PHP 的 `serialize` 函数对事件进行序列化，使用了 `SerializesModels` trait 的事件将会优雅的序列化任何的 Eloquent 模型。

<a name="defining-listeners"></a>
## 定义监听器

接下来，让我们看一下例子中事件的监听器。事件监听器在 `handle` 方法中接受了事件实例作为参数。 `event:generate` 命令将会在事件的 `handle` 方法中自动加载正确的事件类和类型提示。在 `handle` 方法中，你可以运行任何需要响应该事件的业务逻辑。

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
         * 处理事件
         *
         * @param  OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            // 使用 $event->order 来访问 order ...
        }
    }

> {tip} 你的事件监听器也可以在构造函数中对任何依赖使用类型提示。所有的事件监听器会经由 Laravel 的 [服务容器](/docs/{{version}}/container) 做解析，所以所有的依赖都将会被自动注入：

#### 停止事件传播

有时，你可能希望停止一个事件传播到其他的监听器。这时你可以通过在监听器的 `handle` 方法中返回 `false` 来实现。

<a name="queued-event-listeners"></a>
## 队列化事件监听器

如果你的监听器中需要实现一些耗时的任务，比如发送邮件或者进行 HTTP 请求，那把它放到队列中处理是非常有用的。在使用队列化监听器之前，一定要在服务器或者本地环境中配置 [队列](/docs/{{version}}/queues) 并开启一个队列监听器。

要对监听器进行队列化的话，只需增加 `ShouldQueue` 接口到你的监听器类。由 Artisan 命令 `event:generate` 生成的监听器已经将此接口导入到命名空间了，因此你可以直接使用它：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        //
    }

就这样！当事件被监听器调用时， 事件分发器会使用 Laravel 的 [队列系统](/docs/{{version}}/queues) 自动将它进行队列化。如果监听器通过队列运行且没有抛出任何异常，则已执行完的任务将会自动从队列中删除。

#### 自定义队列的连接和名称

如果你想要自定义队列的连接和名称，你可以在监听器类中定义 `$connection` 和 `$queue` 属性。

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        /**
         * 队列化任务使用的连接名称。
         *
         * @var string|null
         */
        public $connection = 'sqs';

        /**
         * 队列化任务使用的队列名称。
         *
         * @var string|null
         */
        public $queue = 'listeners';
    }

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

<a name="handling-failed-jobs"></a>
### 处理失败任务

有时你队列化的事件监听器可能失败了。如果队列监听器任务执行次数超过在工作队列中定义的最大尝试次数，监听器的 `failed` 方法将会被自动调用。 `failed` 方法接受事件实例和失败的异常作为参数：

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
            //
        }

        public function failed(OrderShipped $event, $exception)
        {
            //
        }
    }

<a name="dispatching-events"></a>
## 触发事件

如果要触发事件，你可以传递一个事件实例给 `event` 辅助函数。这个函数将会把事件分发到它所有已经注册的监听器上。因为 `event` 函数是全局可访问的，所以你可以在应用中的任何地方调用它：

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * 将传递过来的订单发货。
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

> {tip} 测试时，不用真的触发监听器就能断言事件类型是很有用的。 Laravel [内置的测试辅助方法](/docs/{{version}}/mocking#mocking-events) 能让这件事变得很容易。

<a name="event-subscribers"></a>
## 事件订阅者

<a name="writing-event-subscribers"></a>
### 编写事件订阅者

事件订阅者是一个在自身内部可以订阅多个事件的类，允许你在单个类中定义多个事件处理器。订阅者应该定义一个 `subscribe` 方法，这个方法接受一个事件分发器的实例。你可以调用事件分发器的 `listen` 方法来注册事件监听器：

    <?php

    namespace App\Listeners;

    class UserEventSubscriber
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
                'Illuminate\Auth\Events\Login',
                'App\Listeners\UserEventSubscriber@onUserLogin'
            );

            $events->listen(
                'Illuminate\Auth\Events\Logout',
                'App\Listeners\UserEventSubscriber@onUserLogout'
            );
        }

    }

<a name="registering-event-subscribers"></a>
### 注册事件订阅者

一旦订阅者被定义，它就可以被注册到事件分发器中。你可以在 `EventServiceProvider` 类的 `$subscribe` 属性注册订阅者。例如，添加 `UserEventSubscriber` 到列表中：

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
         * 需要注册的订阅者类。
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventSubscriber',
        ];
    }



--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org