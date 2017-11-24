# Laravel 的契约 (Contracts)

- [简介](#introduction)
    - [契约 VS Facades](#contracts-vs-facades)
- [何时使用契约](#when-to-use-contracts)
    - [低耦合](#loose-coupling)
    - [简单性](#simplicity)
- [如何使用契约](#how-to-use-contracts)
- [契约参考](#contract-reference)

<a name="introduction"></a>
## 简介

Laravel 的契约是一组定义框架提供的核心服务的接口。例如，`Illuminate\Contracts\Queue\Queue` 契约定义了队列任务所需的方法，而 `Illuminate\Contracts\Mail\Mailer` 契约定义了发送电子邮件所需的方法。

框架对每个契约都提供了相应的实现。例如，Laravel 提供了具有各种驱动的队列实现和由 [SwiftMailer](http://swiftmailer.org/) 提供支持的邮件驱动实现。

所有的 Laravel 契约都有他们自己的 [GitHub 库](https://github.com/illuminate/contracts)。这为所有可用的契约提供了一个快速参考指南，同时也可单独作为低耦合的扩展包给其他包开发者使用。

<a name="contracts-vs-facades"></a>
### 契约 VS Facades

Laravel [Facades](/docs/{{version}}/facades) 和辅助函数提供了一种使用 Laravel 服务的简单方法，即不需要通过类型提示并从服务容器中解析契约。在大多数情况下，每个 Facades 都有一个等效的契约。

不像 Facades，不需要你在类的构造函数中类型提示，契约则需要你在类中明显地定义依赖项。一些开发者倾向于以契约这种方式明确地定义它们的依赖项，而其它开发者则更喜欢 Facades 带来的便捷。

> {tip} 对于大多数应用程序来说不管是使用门面还是契约都可以。但是，如果你正在构建一个扩展包，为了方便测试，你应该强烈考虑契约。

<a name="when-to-use-contracts"></a>
## 何时使用契约

综上所述，使用契约或是 Facades 很大程度上归结于个人或者开发团队的喜好。不管是契约还是 Facades 都可以创建出健壮的、易测试的 Laravel 应用程序。如果你长期关注类的单一职责，你会注意到使用契约还是 Facades 其实没多少实际意义上的区别。

然而，你可能还是会有几个关于契约的问题。例如，为什么要使用接口？不使用接口会比较复杂吗？下面让我们谈下使用接口的原因：低耦合和简单性。

<a name="loose-coupling"></a>
### 低耦合

首先，让我们来看一些高耦合缓存实现的代码。如下:

    <?php

    namespace App\Orders;

    class Repository
    {
        /**
         * 缓存实例。
         */
        protected $cache;

        /**
         * 创建一个仓库实例。
         *
         * @param  \SomePackage\Cache\Memcached  $cache
         * @return void
         */
        public function __construct(\SomePackage\Cache\Memcached $cache)
        {
            $this->cache = $cache;
        }

        /**
         * 按照 Id 检索订单
         *
         * @param  int  $id
         * @return Order
         */
        public function find($id)
        {
            if ($this->cache->has($id))    {
                //
            }
        }
    }

在这个类中，程序跟给定的缓存实现高耦合。因为我们依赖于一个扩展包的特定缓存类。一旦这个扩展包的 API 被更改了，我们的代码就必须跟着改变。

同样的，如果我们想要将底层的的缓存技术（ Memcached ）替换为另一种缓存技术（ Redis ），那又得再次修改这个 `repository` 类。而 `repository` 类不应该了解太多关于谁提供了这些数据或是如何提供的等等。

**比起上面的做法，我们可以使用一个简单的、与扩展包无关的接口来改进我们的代码：**

    <?php

    namespace App\Orders;

    use Illuminate\Contracts\Cache\Repository as Cache;

    class Repository
    {
        /**
         * 缓存实例。
         */
        protected $cache;

        /**
         * 创建一个仓库实例。
         *
         * @param  Cache  $cache
         * @return void
         */
        public function __construct(Cache $cache)
        {
            $this->cache = $cache;
        }
    }

现在，更改之后的代码没有与任何扩展包甚至是 Laravel 耦合。而契约扩展包不包含任何实现和依赖项，你可以轻松地写任何给定契约的替代实现，来实现不修改任何关于缓存消耗的代码就可以替换缓存实现。

<a name="simplicity"></a>
### 简单性

当所有 Laravel 的服务都使用简洁的接口定义，就很容易判断给定服务提供的功能。 **可以将契约视为说明框架功能的简洁文档。**

除此之外，当依赖的接口足够简洁时，代码的可读性和可维护性会大大提高。比起搜索一个大型复杂的类中有哪些可用的方法，不如检索一个简单、 干净的接口来参考更妥当。

<a name="how-to-use-contracts"></a>
## 如何使用契约

那么，如何获得一个契约的实现呢？这其实很简单～

Laravel 中的许多类型的类都是通过 [服务容器](/docs/{{version}}/container) 解析出来的，包括控制器、事件监听器、中间件、任务队列，甚至路由闭包。所以说，要获得一个契约的实现，你只需要被解析的类的构造函数中添加「类型提示」即可。

例如，看看这个事件监听器：

    <?php

    namespace App\Listeners;

    use App\User;
    use App\Events\OrderWasPlaced;
    use Illuminate\Contracts\Redis\Database;

    class CacheOrderInformation
    {
        /**
         * Redis 数据库实现。
         */
        protected $redis;

        /**
         * 创建事件处理器实例。
         *
         * @param  Database  $redis
         * @return void
         */
        public function __construct(Database $redis)
        {
            $this->redis = $redis;
        }

        /**
         * 处理事件。
         *
         * @param  OrderWasPlaced  $event
         * @return void
         */
        public function handle(OrderWasPlaced $event)
        {
            //
        }
    }

当事件监听器被解析时，服务容器会读取类的构造函数上的类型提示，并注入对应的值。想了解更多关于服务容器的注册，请查看 [其文档](/docs/{{version}}/container)。

<a name="contract-reference"></a>
## 契约参考

下表提供了所有 Laravel 契约及其对应的 Facade：

Contract  |  References Facade
------------- | -------------
[Illuminate\Contracts\Auth\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Factory.php)  |  Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/{{version}}/Auth/PasswordBroker.php)  |  Password
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Bus/Dispatcher.php)  |  Bus
[Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/Broadcaster.php)  | &nbsp;
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Repository.php) | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Factory.php) | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Config/Repository.php) | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/{{version}}/Container/Container.php) | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/Factory.php) | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/QueueingFactory.php) | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/{{version}}/Encryption/Encrypter.php) | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Events/Dispatcher.php) | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Cloud.php) | &nbsp;
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Factory.php) | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Filesystem.php) | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/{{version}}/Foundation/Application.php) | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/{{version}}/Hashing/Hasher.php) | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/{{version}}/Logging/Log.php) | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/{{version}}/Mail/MailQueue.php) | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/{{version}}/Mail/Mailer.php) | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Factory.php) | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Queue.php) | Queue
[Illuminate\Contracts\Redis\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Redis/Factory.php) | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/{{version}}/Routing/Registrar.php) | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/{{version}}/Routing/ResponseFactory.php) | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/{{version}}/Routing/UrlGenerator.php) | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Factory.php) | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Validator.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/{{version}}/View/Factory.php) | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/{{version}}/View/View.php) | &nbsp;

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@iwzh](https://github.com/iwzh) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/3762_1456807721.jpeg?imageView2/1/w/200/h/200"> |  翻译 | 码不能停 [@iwzh](https://github.com/iwzh) at Github  |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  Review  | Stay Hungry. Stay Foolish. |


---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
