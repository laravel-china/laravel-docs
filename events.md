# Laravel 的事件系统

- [简介](#introduction)
- [注册事件与监听器](#registering-events-and-listeners)
    - [生成事件 & 监听器](#generating-events-and-listeners)
    - [手动注册事件](#manually-registering-events)
- [定义事件](#defining-events)
- [定义监听器](#defining-listeners)
- [事件监听器队列](#queued-event-listeners)
    - [手动访问队列](#manually-accessing-the-queue)
    - [处理失败任务](#handling-failed-jobs)
- [分发事件](#dispatching-events)
- [事件订阅者](#event-subscribers)
    - [编写事件订阅者](#writing-event-subscribers)
    - [注册事件订阅者](#registering-event-subscribers)

<a name="introduction"></a>
## 简介

Laravel 的事件提供了一个简单的观察者实现，能够订阅和监听应用中发生的各种事件。事件类保存在 `app/Events` 目录中，而这些事件的的监听器则被保存在 `app/Listeners` 目录下。这些目录只有当你使用 Artisan 命令来生成事件和监听器时才会被自动创建。

事件机制是一种很好的应用解耦方式，因为一个事件可以拥有多个互不依赖的监听器。例如，如果你希望每次订单发货时向用户发送一个 Slack 通知。你可以简单地发起一个 `OrderShipped` 事件，让监听器接收之后转化成一个 Slack 通知，这样你就可以不用把订单的业务代码跟 Slack 通知的代码耦合在一起了。

<a name="registering-events-and-listeners"></a>
## 注册事件和监听器

Laravel 应用中的 `EventServiceProvider` 有个 `listen` 数组包含所有的事件（键）以及事件对应的监听器（值）来注册所有的事件监听器，可以灵活地根据需求来添加事件。例如，让我们增加一个 `OrderShipped` 事件：

````
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
````

<a name="generating-events-and-listeners"></a>
### 生成事件 & 监听器

为每个事件和监听器手动创建文件是件很麻烦的事情，而在这里，你只需将监听器和事件添加到  `EventServiceProvider` 中，再使用 `event:generate` 命令即可。这个命令会生成在 `EventServiceProvider` 中列出的所有事件和监听器。当然，已经存在的事件和监听器将保持不变：

````
php artisan event:generate
````

<a name="manually-registering-events"></a>
### 手动注册事件

事件通常是在 `EventServiceProvider` 类的 `$listen` 数组中注册，但是，你也可以在 `EventServiceProvider` 类的 `boot` 方法中注册基于事件的闭包。

````
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
````

#### 通配符事件监听器

你可以在注册监听器时使用 `*` 通配符参数，这样能够在同一个监听器上捕获多个事件。通配符监听器接受事件名称作为其第一个参数，并将整个事件数据数组作为其第二个参数：

````
Event::listen('event.*', function ($eventName, array $data) {
    //
});
````

<a name="defining-events"></a>
## 定义事件

事件类其实就只是一个保存与事件相关的信息的数据容器。例如，假设我们生成的 `OrderShipped` 事件接收一个 [Eloquent ORM](/docs/{{version}}/eloquent) 对象：

````
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
````

正如你所见，这个事件类中没有包含其它逻辑。它只是一个被构建的 `Order` 对象的容器。如果使用 PHP 的 `serialize` 函数序列化事件对象，事件使用的 `SerializesModels` trait 将会优雅地序列化任何 Eloquent 模型。

<a name="defining-listeners"></a>
## 定义监听器

接下来，让我们看一下例子中事件的监听器。事件监听器在 `handle` 方法中接收事件实例。 `event:generate` 命令会自动加载正确的事件类和在 `handle` 加入的类型提示。在 `handle` 方法中，你可以执行任何必要的响应事件的操作：

````
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
````

> {tip} 你的事件监听器也可以在构造函数中加入任何依赖关系的类型提示。所有的事件监听器都是通过 Laravel 的 [服务容器](/docs/{{version}}/container) 来解析的，因此所有的依赖都将会被自动注入。

#### 停止事件传播

你可以通过在监听器的 `handle` 方法中返回 `false` 来阻止事件被其他的监听器获取。

<a name="queued-event-listeners"></a>
## 事件监听器队列

如果你的监听器中要执行诸如发送邮件或者进行 HTTP 请求等比较慢的任务，你可以选择将其丢给队列处理。在开始使用监听器队列之前，请确保在你的服务器或本地开发环境中能够配置并启动 [队列](/docs/{{version}}/queues) 监听器。

要指定监听器启动队列，只需将 `ShouldQueue` 接口添加到监听器类。由 Artisan 命令 `event:generate` 生成的监听器已经将此接口导入到当前命名空间中，因此你可以直接使用它：

````
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    //
}
````

当这个监听器被事件调用时，事件调度器会自动使用 Laravel 的 [队列系统](/docs/{{version}}/queues)。如果在队列中执行监听器时没有抛出异常，任务会在执行完成后自动从队列中删除。

#### 自定义队列的连接和名称

如果你想要自定义事件监听器使用的队列的连接和名称，可以在监听器类中定义 `$connection` 和 `$queue` 属性。

````
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    /**
     * 任务应该发送到的队列的连接的名称
     *
     * @var string|null
     */
    public $connection = 'sqs';

    /**
     * 任务应该发送到的队列的名称
     *
     * @var string|null
     */
    public $queue = 'listeners';
}
````

<a name="manually-accessing-the-queue"></a>
### 手动访问队列

如果你需要手动访问监听器下面队列任务的 `delete` 和 `release` 方法，你可以添加 `Illuminate\Queue\InteractsWithQueue` trait 来实现。这个 trait 会默认加载到生成的监听器中，并提供对这些方法的访问：

````
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Handle the event.
     *
     * @param  \App\Events\OrderShipped  $event
     * @return void
     */
    public function handle(OrderShipped $event)
    {
        if (true) {
            $this->release(30);
        }
}
````

<a name="handling-failed-jobs"></a>
### 处理失败任务

事件监听器的队列任务可能会失败，而如果监听器的队列任务超过了队列中定义的最大尝试次数，则会监听器上调用 `failed` 方法。`failed` 方法接受接收事件实例和导致失败的异常作为参数：

````
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * 处理事件
     *
     * @param  \App\Events\OrderShipped  $event
     * @return void
     */
    public function handle(OrderShipped $event)
    {
        //
    }

    /**
     * 处理任务失败
     *
     * @param  \App\Events\OrderShipped  $event
     * @param  \Exception  $exception
     * @return void
     */
    public function failed(OrderShipped $event, $exception)
    {
        //
    }
}
````

<a name="dispatching-events"></a>
## 分发事件

如果要分发事件，你可以将事件实例传递给辅助函数 `event`。这个函数将会把事件分发到所有已经注册的监听器上。因为辅助函数 `event` 是全局可访问的，所以你可以在应用中的任何地方调用它：

````
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
````

> {tip} 在测试时，Laravel [内置的测试辅助函数](/docs/{{version}}/mocking#mocking-events) 能不需要实际触发监听器就能对事件类型断言。

<a name="event-subscribers"></a>
## 事件订阅者

<a name="writing-event-subscribers"></a>
### 编写事件订阅者

事件订阅者是一个可以在自身内部订阅多个事件的类，即能够在单个类中定义多个事件处理器。订阅者应该定义一个 `subscribe` 方法，这个方法接受一个事件分发器的实例。你可以调用给定的事件分发器上的 `listen` 方法来注册事件监听器：

````
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
````

<a name="registering-event-subscribers"></a>
### 注册事件订阅者

订阅者写好后，就将其注册到事件分发器中。你可以在 `EventServiceProvider` 类的 `$subscribe` 属性中注册订阅者。例如，将 `UserEventSubscriber` 添加到数组列表中：

````
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
````

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | 翻译 | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
