# 模拟器

- [介绍](#introduction)
- [事件](#mocking-events)
    - [使用 Mocks](#using-event-mocks)
    - [使用 Fakes](#using-event-fakes)
- [任务](#mocking-jobs)
    - [使用 Mocks](#using-job-mocks)
    - [使用 Fakes](#using-job-fakes)
- [邮件 Fakes](#mail-fakes)
- [通知 Fakes](#notification-fakes)
- [Facades](#mocking-facades)

<a name="introduction"></a>
## 介绍

测试 Laravel 应用时，有时候你可能想要「模拟」实现应用的部分功能的行为，从而避免该部分在测试过程中真正执行。例如，控制器执行过程中会触发一个事件（ Events ），你想要模拟这个事件的监听器，从而避免该事件在测试这个控制器时真正执行。如上可以让你仅测试控制器的 HTTP 响应情况，而不用去担心触发事件。当然，你可以在单独的测试中测试该事件的逻辑。

Laravel 针对事件、任务和 facades 的模拟提供了开箱即用的辅助函数。这些辅助函数基于 Mockery 封装而成，使用非常简单，无需你手动调用复杂的 Mockery 函数。当然，你也可以使用 [Mockery](http://docs.mockery.io/en/latest/) 或者 PHPUnit 创建自己的模拟器。

<a name="mocking-events"></a>
## 事件

<a name="using-event-mocks"></a>
### 使用 Mocks

如果在你的项目中大量使用了 Laravel 的事件系统，那你可能经常会希望在测试过程中禁用特定的事件。比如，测试用户注册时并不想触发 `UserRegistered` 事件，此类事件的常见行为包括发送「欢迎」邮件等。

Laravel 提供了一个非常方便的方法 `expectsEvents` ，可以用来阻止触发特定事件，而不影响其他事件：

    <?php

    use App\Events\UserRegistered;

    class ExampleTest extends TestCase
    {
        /**
         * 测试用户注册
         */
        public function testUserRegistration()
        {
            $this->expectsEvents(UserRegistered::class);

            // Test user registration...
        }
    }

你也可以使用 `doesntExpectEvents` 函数来验证事件确定没有被触发：

    <?php

    use App\Events\OrderShipped;
    use App\Events\OrderFailedToShip;

    class ExampleTest extends TestCase
    {
        /**
         * 测试订单发货
         */
        public function testOrderShipping()
        {
            $this->expectsEvents(OrderShipped::class);
            $this->doesntExpectEvents(OrderFailedToShip::class);

            // 测试订单发货...
        }
    }

如果你想阻止所有事件的触发，可以使用 `withoutEvents` 方法。该方法调用后，所有事件触发事件都将被模拟器捕获：

    <?php

    class ExampleTest extends TestCase
    {
        public function testUserRegistration()
        {
            $this->withoutEvents();

            // 测试用户注册的代码...
        }
    }

<a name="using-event-fakes"></a>
### 使用 Fakes

另外一个模拟的方式是使用 `Event` facade 的 `fake` 方法，测试的时候不会触发事件监听器运行。然后你就可以断言事件运行了，甚至可以检查它们收到的数据。使用 fakes 的时候，断言一般出现在测试代码的后面。

    <?php

    use App\Events\OrderShipped;
    use App\Events\OrderFailedToShip;
    use Illuminate\Support\Facades\Event;

    class ExampleTest extends TestCase
    {
        /**
         * Test order shipping.
         */
        public function testOrderShipping()
        {
            Event::fake();

            // 处理订单的运送。。。

            Event::assertFired(OrderShipped::class, function ($e) use ($order) {
                return $e->order->id === $order->id;
            });

            Event::assertNotFired(OrderFailedToShip::class);
        }
    }
    
    
<a name="mocking-jobs"></a>
## 任务

<a name="using-job-mocks"></a>
### 使用 Mocks

有些时候，在测试应用程序的一些请求时，可能会希望测试指定任务是否已经发送。上述做法可以对你的控制器进行隔离测试，无需在意任务处理器的逻辑。当然，你可以在单独的测试中测试这个任务处理器。

Laravel 提供了一个非常方便的方法 `expectsJobs` ，可以用来验证指定任务是否已被发送。当然，这个任务不会被执行：

    <?php

    class App\Jobs\ShipOrder;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            $this->expectsJobs(ShipOrder::class);

            // 订单发货的测试...
        }
    }

> {note} 这个方法只能检测到通过 `DispatchesJobs` trait 的辅助函数 `dispatch` 发送的任务，检测不到使用 `Queue::push` 直接发送到队列的任务。

与事件的模拟器辅助函数类似，你可以使用 `doesntExpectJobs` 方法来确定指定任务是否没有被发送：

    <?php

    class App\Jobs\ShipOrder;

    class ExampleTest extends TestCase
    {
        /**
         * 测试订单取消操作
         */
        public function testOrderCancellation()
        {
            $this->doesntExpectJobs(ShipOrder::class);

            // 测试订单取消操作...
        }
    }

或者，你可以使用 `withoutJobs` 方法阻止发送所有任务。在测试中调用此方法后，当前测试中所有任务都会被丢弃：

    <?php

    class App\Jobs\ShipOrder;

    class ExampleTest extends TestCase
    {
        /**
         * 测试订单取消操作
         */
        public function testOrderCancellation()
        {
            $this->withoutJobs();

            // 测试订单取消操作...
        }
    }
    
<a name="using-job-fakes"></a>
### 使用 Fakes

另外一种模拟的方法是使用 `Queue` facade 的 `fake` 方法，测试的时候并不会任务放入队列。你可以断言任务被放进了队列，甚至可以检查它们收到的数据。使用 fakes 的时候，断言一般出现在测试代码的后面。

    <?php

    use App\Jobs\ShipOrder;
    use Illuminate\Support\Facades\Queue;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Queue::fake();

            // 处理订单的运送。。。

            Queue::assertPushed(ShipOrder::class, function ($job) use ($order) {
                return $job->order->id === $order->id;
            });

            // 断言任务进入的队列
            Queue::assertPushedOn('queue-name', ShipOrder::class);

            // 断言任务没有进入队列
            Queue::assertNotPushed(AnotherJob::class);
        }
    }

<a name="mail-fakes"></a>
## 邮件 Fakes

可以使用 `Mail` facade 的 `fake` 方法，测试时不会真的发送邮件。然后你可以断言 `mailables` 发送给了用户, 甚至可以检查他们收到的数据. 使用 fakes 时, 断言一般在测试代码的后面.

    <?php

    use App\Mail\OrderShipped;
    use Illuminate\Support\Facades\Mail;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Mail::fake();

            // 处理订单的运送

            Mail::assertSent(OrderShipped::class, function ($mail) use ($order) {
                return $mail->order->id === $order->id;
            });

            // 断言邮件发送给了指定用户
            Mail::assertSentTo([$user], OrderShipped::class);

            // 断言mailable没有发送
            Mail::assertNotSent(AnotherMailable::class);
        }
    }

<a name="notification-fakes"></a>
## 通知 Fakes

可以使用 `Notification` facade 的 `fake` 方法, 测试的时候并不会真的发送通知. 然后可以断言通知发送给你用户, 甚至可以检查他们收到的数据. 使用 fakes 时, 断言一般出现在测试代码的后面.

    <?php

    use App\Notifications\OrderShipped;
    use Illuminate\Support\Facades\Notification;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Notification::fake();

            // 处理订单运送

            Notification::assertSentTo(
                $user,
                OrderShipped::class,
                function ($notification, $channels) use ($order) {
                    return $notification->order->id === $order->id;
                }
            );

            // 断言通知发送给了用户
            Notification::assertSentTo(
                [$user], OrderShipped::class
            );

            // 断言通知没有发送
            Notification::assertNotSentTo(
                [$user], AnotherNotification::class
            );
        }
    }



<a name="mocking-facades"></a>
## Facades

不像传统的静态函数调用， [facades](/docs/{{version}}/facades) 是可以被模拟的，相对静态函数来说这是个巨大的优势，即使你在使用依赖注入，测试时依然会非常方便。在很多测试中，你可能经常想在控制器中模拟对 Laravel facade 的调用。比如下面控制器中的行为：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * 显示网站的所有用户
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

我们可以通过 `shouldReceive` 方法模拟 `Cache` facade ，此函数会返回一个模拟器 （ [Mockery](https://github.com/padraic/mockery) ） 实例，由于对 facade 的调用实际上都是由 Laravel 的 [service container](/docs/{{version}}/container) 管理的，所以 facade 能比传统的静态类表现出更好的测试便利性。如下，我们来模拟一下 `Cache` facade `get` 方法的行为：

    <?php

    class FooTest extends TestCase
    {
        public function testGetIndex()
        {
            Cache::shouldReceive('get')
                        ->once()
                        ->with('key')
                        ->andReturn('value');

            $this->visit('/users')->see('value');
        }
    }

> {note} 不可以模拟 `Request` facade ，测试时，如果需要定制传递的数据请使用 HTTP 辅助函数，例如 `call` 和 `post`。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@zhwei](https://github.com/zhwei)  | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/1446459?v=3&s=100">  |  翻译  | ^_^ |
| [@JobsLong](https://phphub.org/users/56)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/56_1427370654.jpeg?imageView2/1/w/100/h/100">  |  Review  | 我的个人主页：[http://jobslong.com](http://jobslong.com)  |
| [@summerblue](https://github.com/summerblue)  | <img class="avatar-66 rm-style" src="https://avatars2.githubusercontent.com/u/324764?v=3&s=100">  |  Review  | A man seeking for Wisdom. |
