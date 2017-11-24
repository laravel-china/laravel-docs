# Laravel 测试之：测试模拟器

- [介绍](#introduction)
- [任务模拟](#bus-fake)
- [事件模拟](#event-fake)
- [邮件模拟](#mail-fake)
- [通知模拟](#notification-fake)
- [队列模拟](#queue-fake)
- [Storage 模拟](#storage-fake)
- [Facades 模拟](#mocking-facades)

<a name="introduction"></a>
## 介绍

测试 Laravel 应用时，有时候你可能想要「模拟」实现应用的部分功能的行为，从而避免该部分在测试过程中真正执行。例如，控制器执行过程中会触发一个事件（ Events ），你想要模拟这个事件的监听器，从而避免该事件在测试这个控制器时真正执行。如上可以让你仅测试控制器的 HTTP 响应情况，而不用去担心触发事件。当然，你可以在单独的测试中测试该事件的逻辑。

Laravel 针对事件、任务和 facades 的模拟提供了开箱即用的辅助函数。这些辅助函数基于 Mockery 封装而成，使用非常简单，无需你手动调用复杂的 Mockery 函数。当然，你也可以使用 [Mockery](http://docs.mockery.io/en/latest/) 或者 PHPUnit 创建自己的模拟器。

<a name="bus-fake"></a>
## 任务模拟

你可以使用 `Bus` facade 的 `fake` 方法来模拟任务执行，测试的时候任务不会被真实执行。使用 fakes 的时候，断言一般出现在测试代码的后面：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Jobs\ShipOrder;
    use Illuminate\Support\Facades\Bus;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Bus::fake();

            // 处理订单发货...

            Bus::assertDispatched(ShipOrder::class, function ($job) use ($order) {
                return $job->order->id === $order->id;
            });

            // 断言任务并没有被执行...
            Bus::assertNotDispatched(AnotherJob::class);
        }
    }

<a name="event-fake"></a>
## 事件模拟

你可以使用 `Event` facade 的 `fake` 方法来模拟事件监听，测试的时候不会触发事件监听器运行。然后你就可以断言事件运行了，甚至可以检查它们收到的数据。使用 fakes 的时候，断言一般出现在测试代码的后面:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Events\OrderShipped;
    use App\Events\OrderFailedToShip;
    use Illuminate\Support\Facades\Event;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        /**
         * 测试订单发货.
         */
        public function testOrderShipping()
        {
            Event::fake();

            // 处理订单发货...

            Event::assertDispatched(OrderShipped::class, function ($e) use ($order) {
                return $e->order->id === $order->id;
            });

            Event::assertNotDispatched(OrderFailedToShip::class);
        }
    }

<a name="mail-fake"></a>
## 邮件模拟

你可以使用 `Mail` facade 的 `fake` 方法来模拟邮件发送，测试时不会真的发送邮件。然后你可以断言 [mailables](/docs/{{version}}/mail) 发送给了用户，甚至可以检查他们收到的数据. 使用 fakes 时，断言一般在测试代码的后面：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Mail\OrderShipped;
    use Illuminate\Support\Facades\Mail;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Mail::fake();

            // 处理订单发货...

            Mail::assertSent(OrderShipped::class, function ($mail) use ($order) {
                return $mail->order->id === $order->id;
            });

            // 断言一封邮件已经发送给了指定用户...
            Mail::assertSent(OrderShipped::class, function ($mail) use ($user) {
                return $mail->hasTo($user->email) &&
                       $mail->hasCc('...') &&
                       $mail->hasBcc('...');
            });
				
            // 断言 mailable 发送了2次...
            Mail::assertSent(OrderShipped::class, 2);

            // 断言 mailable 没有发送...
            Mail::assertNotSent(AnotherMailable::class);
        }
    }

如果你是用后台任务队执行 mailables 的发送，你应该用 `assertQueued` 方法来代替 `assertSent`：

    Mail::assertQueued(...);
    Mail::assertNotQueued(...);
    
<a name="notification-fake"></a>
## 通知模拟

你可以使用 `Notification` facade 的 `fake` 方法来模拟通知发送，测试的时候并不会真的发送通知。然后你可以断言 [通知](/docs/{{version}}/notifications) 已经发送给你的用户，甚至可以检查他们收到的数据。使用 fakes 时, 断言一般出现在测试代码的后面.

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Notifications\OrderShipped;
    use Illuminate\Support\Facades\Notification;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Notification::fake();

            // 处理订单发货...

            Notification::assertSentTo(
                $user,
                OrderShipped::class,
                function ($notification, $channels) use ($order) {
                    return $notification->order->id === $order->id;
                }
            );

            // 断言通知已经发送给了指定用户...
            Notification::assertSentTo(
                [$user], OrderShipped::class
            );

            // 断言通知没有发送...
            Notification::assertNotSentTo(
                [$user], AnotherNotification::class
            );
        }
    }

<a name="queue-fake"></a>
## 队列模拟

你可以使用 `Queue` facade 的 `fake` 方法来模拟任务队列，测试的时候并不会真的把任务放入队列。然后你可以断言任务被放进了队列，甚至可以检查它们收到的数据。使用 fakes 的时候，断言一般出现在测试代码的后面。

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Jobs\ShipOrder;
    use Illuminate\Support\Facades\Queue;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Queue::fake();

            // 处理订单发货...

            Queue::assertPushed(ShipOrder::class, function ($job) use ($order) {
                return $job->order->id === $order->id;
            });

            // 断言任务进入了指定队列...
            Queue::assertPushedOn('queue-name', ShipOrder::class);
				
            // 断言任务进入了2次...
            Queue::assertPushed(ShipOrder::class, 2);
            
            // 断言任务没有进入队列...
            Queue::assertNotPushed(AnotherJob::class);
        }
    }

<a name="storage-fake"></a>
## Storage 模拟

利用 `Storage` facade 的 `fake` 方法，你可以轻松地生成一个模拟的磁盘，结合 `UploadedFile` 类的文件生成工具，极大地简化了文件上传测试。例如：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        public function testAvatarUpload()
        {
            Storage::fake('avatars');

            $response = $this->json('POST', '/avatar', [
                'avatar' => UploadedFile::fake()->image('avatar.jpg')
            ]);

            // 断言文件已存储
            Storage::disk('avatars')->assertExists('avatar.jpg');

            // 断言文件不存在
            Storage::disk('avatars')->assertMissing('missing.jpg');
        }
    }

<a name="mocking-facades"></a>
## Facades 模拟

不同于传统的静态函数的调用， [facades](/docs/{{version}}/facades) 也是可以被模拟的，相对静态函数来说这是个巨大的优势，即使你在使用依赖注入，测试时依然会非常方便。在很多测试中，你可能经常想在控制器中模拟对 Laravel facade 的调用。比如下面控制器中的行为：

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

我们可以通过 `shouldReceive` 方法来模拟 `Cache` facade ，此函数会返回一个 [Mockery](https://github.com/padraic/mockery) 实例，由于对 facade 的调用实际上都是由 Laravel 的 [服务容器](/docs/{{version}}/container) 管理的，所以 facade 能比传统的静态类表现出更好的测试便利性。接下来，让我们来模拟一下 `Cache` facade 的 `get` 方法的调用：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class UserControllerTest extends TestCase
    {
        public function testGetIndex()
        {
            Cache::shouldReceive('get')
                        ->once()
                        ->with('key')
                        ->andReturn('value');

            $response = $this->get('/users');

            // ...
        }
    }

> {note} 不可以模拟 `Request` facade，测试时，如果需要传递指定的数据请使用 HTTP 辅助函数，例如 `get` 和 `post`。类似的，请在你的测试中通过调用 `Config::set` 来模拟 `Config` facade。


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org