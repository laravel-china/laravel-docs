# 服务容器

- [简介](#introduction)
- [绑定](#binding)
    - [绑定接口至实现](#binding-interfaces-to-implementations)
    - [情境绑定](#contextual-binding)
    - [标记](#tagging)
- [解析](#resolving)
- [容器事件](#container-events)

<a name="introduction"></a>
## 简介

Laravel 服务容器是管理类依赖与运行依赖注入的强力工具。依赖注入是个花俏的名词，事实上是指：类的依赖通过构造器或在某些情况下通过「setter」方法「注入」。

让我们来看个简单例子：

    <?php

    namespace App\Jobs;

    use App\User;
    use Illuminate\Contracts\Mail\Mailer;
    use Illuminate\Contracts\Bus\SelfHandling;

    class PurchasePodcast implements SelfHandling
    {
        /**
         * 邮件寄送器的实现。
         */
        protected $mailer;

        /**
         * 创建一个新实例。
         *
         * @param  Mailer  $mailer
         * @return void
         */
        public function __construct(Mailer $mailer)
        {
            $this->mailer = $mailer;
        }

        /**
         * 购买一个播客
         *
         * @return void
         */
        public function handle()
        {
            //
        }
    }

在这例子中，当播客被购买时，`PurchasePodcast` 任务会需要寄送 e-mails。因此，我们将 **注入** 能寄送 e-mails 的服务。由于服务被注入，我们能容易地切换成其它实现。当测试应用程序时，我们一样能轻易地「mock」，或创建假的邮件寄送器实现。

在建置强大的应用程序，以及为 Laravel 核心贡献时，须深入理解 Laravel 服务容器。

<a name="binding"></a>
## 绑定

几乎所有的服务容器绑定都会注册至[服务提供者](/docs/{{version}}/providers)中，所以下方所有的例子将示范在该情境中使用容器。不过，如果它们没有依赖任何的接口，那么就没有将类绑定至容器中的必要。并不需要为容器指定如何建构这些对象，因为它会通过 PHP 的反射服务自动解析「实际」的对象。

在服务提供者中，你总是可以通过 `$this->app` 实例变量访问容器。我们可以使用 `bind` 方法注册一个绑定，传递我们希望注册的类或接口名称，并连同返回该类实例的`闭包`：

    $this->app->bind('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app['HttpClient']);
    });

注意，我们获取到的容器本身作为参数传递给解析器。我们可以使用容器来解析我们绑定对象的次要依赖。

#### 绑定一个单例

`singletion` 方法绑定一个只会被解析一次的类或接口至容器中，且尔后的调用都会从容器中返回相同的实例：

    $this->app->singleton('FooBar', function ($app) {
        return new FooBar($app['SomethingElse']);
    });

#### 绑定实例

你也可以使用 `instance` 方法，绑定一个已经存在的对象实例至容器中。尔后的调用都会从容器中返回指定的实例：

    $fooBar = new FooBar(new SomethingElse);

    $this->app->instance('FooBar', $fooBar);

<a name="binding-interfaces-to-implementations"></a>
### 绑定接口至实现

服务容器有个非常强大的特色，就是能够将指定的实现绑定至接口的功能。举个例子，让我们假设我们有个 `EventPusher` 接口及一个 `RedisEventPusher` 实现。一旦我们编写完该接口的 `RedisEventPusher` 实现，就可以如下将它注册至服务容器：

    $this->app->bind('App\Contracts\EventPusher', 'App\Services\RedisEventPusher');

这么做会告知容器当有个类需要 `EventPusher` 的实现时，必须注入 `RedisEventPusher`。现在我们可以在构造器中对 `EventPusher` 接口使用类型提示，或任何需要通过服务容器注入依赖的其他位置：

    use App\Contracts\EventPusher;

    /**
     * 创建一个新的类实例。
     *
     * @param  EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

<a name="contextual-binding"></a>
### 情境绑定

有时候，你可能有两个类使用到相同接口，但你希望每个类能注入不同实现。例如，当系统收到新订单时，我们可能想通过 [PubNub](http://www.pubnub.com/) 来发送事件，而不是 Pusher。Laravel 提供一个简单又流利接口来定义此行为：

    $this->app->when('App\Handlers\Commands\CreateOrderHandler')
              ->needs('App\Contracts\EventPusher')
              ->give('App\Services\PubNubEventPusher');

你甚至可以传递一个闭包至 `give` 方法：

    $this->app->when('App\Handlers\Commands\CreateOrderHandler')
              ->needs('App\Contracts\EventPusher')
              ->give(function () {
                      // Resolve dependency...
                  });

<a name="tagging"></a>
### 标记

有些时候，可能需要解析某个「分类」下的所有绑定。例如，你正在建置一个能接收多个不同 `Report` 接口实现数组的报表汇整器。注册完 Report 实现后，可以使用 `tag` 方法为它们赋予一个标签：

    $this->app->bind('SpeedReport', function () {
        //
    });

    $this->app->bind('MemoryReport', function () {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

一旦服务被标记之后，你可以通过 `tagged` 方法很简单的解析它们全部：

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="resolving"></a>
## 解析

有几种方式可以从容器中解析一些东西。首先，你可以使用 `make` 方法，它接收你希望解析的类或是接口的名称：

    $fooBar = $this->app->make('FooBar');

或者，你可以像数组一样从容器中进行访问，因为他实现了 PHP 的 `ArrayAccess` 接口：

    $fooBar = $this->app['FooBar'];

最后，但是最重要的，你可以简单地在类的构造器对依赖使用「类型提示」，类将会从容器中进行解析，包含[控制器](/docs/{{version}}/controllers)、[事件侦听器](/docs/{{version}}/events)、[对列任务](/docs/{{version}}/queues)、[中间件](/docs/{{version}}/middleware)及其他等等。在实际情形中，这就是为何大部分的对象都是由容器中解析。

容器会自动为类注入解析出的依赖。举个例子，你可以在控制器的构造器中对应用程序中定义的保存库进行类型提示。保存库会自动被解析及注入至类中：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;
    use App\Users\Repository as UserRepository;

    class UserController extends Controller
    {
        /**
         * 用户保存库的实例。
         */
        protected $users;

        /**
         * 创建一个新的控制器实例。
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * 显示指定 ID 的用户。
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            //
        }
    }

<a name="container-events"></a>
## 容器事件

每当服务容器解析一个对象时就会触发事件。你可以使用 `resolving` 方法监听这个事件：

    $this->app->resolving(function ($object, $app) {
        // 当容器解析任何类型的对象时会被调用...
    });

    $this->app->resolving(FooBar::class, function (FooBar $fooBar, $app) {
        // 当容器解析「FooBar」类型的对象时会被调用...
    });

如你所见，被解析的对象会被传递至回调中，让你可以在传递前设置任何额外的属性至对象。
