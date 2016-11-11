# Contracts

- [简介](#introduction)
    - [Contracts Vs. Facades](#contracts-vs-facades)
- [何时使用 Contracts](#when-to-use-contracts)
    - [低耦合](#loose-coupling)
    - [简单性](#simplicity)
- [如何使用 Contracts](#how-to-use-contracts)
- [Contract 参考](#contract-reference)

<a name="introduction"></a>
## 简介

Laravel 的 Contracts 是一组定义了框架核心服务的接口。例如，`Illuminate\Contracts\Queue\Queue` contract 定义了队列任务所需要的方法，而 `Illuminate\Contracts\Mail\Mailer` contract 定义了寄送 e-mail 需要的方法。

框架对于每个 contract 都有提供对应的实现，例如，Laravel 提供各种驱动程序的队列实现，以及由 [SwiftMailer](http://swiftmailer.org/) 提供的 mailer 实现。

Laravel 所有的 contracts 放在一个单独的 [GitHub 代码库](https://github.com/illuminate/contracts)。除了提供给所有可用的 contracts 一个快速的参考，也可以单独作为一个低耦合的扩展包来让其他扩展包开发者使用。

<a name="contracts-vs-facades"></a>
### Contracts Vs. Facades

Laravel 的 [facades](/docs/{{version}}/facades) 提供一个简单的方法来使用服务，而不需要使用类型约束和在服务容器之外解析 contracts。大多数情况下，每个 facade 都有一个相应的 contract。

不像 facades 那样，contracts 需要你为你的类显示的定义依赖关系。有些开发者喜欢这种显示的依赖定义，所以他们喜欢使用 contracts，而其他开发者更喜欢方便的 facades。

> {tip} 大多数应用不管你是使用 facades 还是 contracts 都可以很好的工作，但是如果你打算构建扩展包的话，强烈建议使用 contracts，因为它们在扩展包的环境下更容易被测试。

<a name="when-to-use-contracts"></a>
## 何时使用 Contracts

正如我们所说，到底是选择 facade 还是 contracts 取决于你或你的团队的喜好。不管是 facade 还是 contracts，都可以创建出健壮的，易测试的应用。随着你长期关注于类的功能层面，你会发现其实 facades 和 contracts 之间并没有太大的区别。

你可能有很多关于 contracts 的问题。像是为什么要使用接口？使用接口会不会变的更复杂？让我们来提炼一下这样做的原因：低耦合和简单性。

<a name="loose-coupling"></a>
### 低耦合

首先，让我们来查看这一段和缓存功能有高耦合的代码，如下：

    <?php

    namespace App\Orders;

    class Repository
    {
        /**
         * 缓存实例。
         */
        protected $cache;

        /**
         * 创建一个新的仓库实例。
         *
         * @param  \SomePackage\Cache\Memcached  $cache
         * @return void
         */
        public function __construct(\SomePackage\Cache\Memcached $cache)
        {
            $this->cache = $cache;
        }

        /**
         * 借由 ID 获取订单信息。
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

在此类中，程序和缓存实现之间是高耦合。因为它是依赖于扩展包的特定缓存类。一旦这个扩展包的 API 更改了，我们的代码也要跟着改变。

同样的，如果想要将底层的缓存实现（比如 Memcached ）切换成另一种（像 Redis ），又一次的我们必须修改这个 `Repository` 类。我们的 `Repository` 类不应该知道这么多关于谁提供了数据，或是如何提供等细节。

**比起上面的做法，我们可以使用一个简单、和扩展包无关的接口来改进代码:**

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
         * 创建一个新的仓库实例。
         *
         * @param  Cache  $cache
         * @return void
         */
        public function __construct(Cache $cache)
        {
            $this->cache = $cache;
        }
    }

现在上面的代码没有跟任何扩展包耦合，甚至是 Laravel。既然 contracts 扩展包没有包含实现和任何依赖，你就可以很简单的对任何 contract 进行实现，你可以很简单的写一个替换的实现，甚至是替换 contracts，让你可以替换缓存实现而不用修改任何用到缓存的代码。

<a name="simplicity"></a>
### 简单性

当所有的 Laravel 服务都使用简洁的接口定义，就能够很容易决定一个服务需要提供的功能。 **可以将 contracts 视为说明框架特色的简洁文档。**

除此之外，当依赖的接口足够简洁时，代码的可读性和可维护性大大提高。比起搜索一个大型复杂的类里有哪些可用的方法，你有一个简单，干净的接口可以参考。

<a name="how-to-use-contracts"></a>
## 如何使用 Contracts

那么，如何获取一个 contract 的实现呢？其实真的非常简单。

Laravel 里很多类型的类都是通过[服务容器](/docs/{{version}}/container)解析出来的。包括控制器，事件监听器，中间件，任务队列，甚至是路由的闭包。所以说，想要获得一个 contract 的实现，你只需要在类的构造函数里添加相应的类型约束就好了。

举个例子，看一下这个事件监听器：

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

当事件监听器被解析时，服务容器会从构造函数里读取到类型约束，并且注入合适的类。想了解如何注册绑定到容器，请参考[这篇文档](/docs/{{version}}/container)。


<a name="contract-reference"></a>
## Contract 参考

下面的表格提供了 Laravel contracts 及其对应的 facades 的参考:

Contract  |  References Facade
------------- | -------------
[Illuminate\Contracts\Auth\Factory](https://github.com/illuminate/contracts/blob/master/Auth/Factory.php)  |  Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/master/Auth/PasswordBroker.php)  |  Password
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/master/Bus/Dispatcher.php)  |  Bus
[Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/master/Broadcasting/Broadcaster.php)  | &nbsp;
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/master/Cache/Repository.php) | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/master/Cache/Factory.php) | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/master/Config/Repository.php) | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/master/Container/Container.php) | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/master/Cookie/Factory.php) | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/master/Cookie/QueueingFactory.php) | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/master/Encryption/Encrypter.php) | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/master/Events/Dispatcher.php) | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/master/Filesystem/Cloud.php) | &nbsp;
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/master/Filesystem/Factory.php) | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/master/Filesystem/Filesystem.php) | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/master/Foundation/Application.php) | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/master/Hashing/Hasher.php) | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/master/Logging/Log.php) | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/master/Mail/MailQueue.php) | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/master/Mail/Mailer.php) | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/master/Queue/Factory.php) | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/master/Queue/Queue.php) | Queue
[Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/master/Redis/Database.php) | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/master/Routing/Registrar.php) | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/master/Routing/ResponseFactory.php) | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/master/Routing/UrlGenerator.php) | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/master/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/master/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/master/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/master/Validation/Factory.php) | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/master/Validation/Validator.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/master/View/Factory.php) | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/master/View/View.php) | &nbsp;

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@dinghua](https://phphub.org/users/294)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/294_1473147991.png?imageView2/1/w/100/h/100">  |  翻译  | 专注于 PHP 和 Laravel |

