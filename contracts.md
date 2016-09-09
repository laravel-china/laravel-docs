# Contracts

- [简介](#introduction)
    - [Contracts 对比 Facades](#contracts-vs-facades)
- [何时使用 Contracts](#when-to-use-contracts)
    - [低耦合](#loose-coupling)
    - [简洁性](#simplicity)
- [如何使用 Contracts](#how-to-use-contracts)
- [Contract 参考](#contract-reference)

<a name="introduction"></a>
## 简介

Laravel 的 Contracts 是一组定义了框架核心服务的接口。例如，`Illuminate\Contracts\Queue\Queue` contract 定义了队列任务所需要的方法，而 `Illuminate\Contracts\Mail\Mailer` contract 定义了寄送 e-mail 需要的方法。

框架对于每个 contract 都有提供对应的实现，例如，Laravel 提供各种驱动程序的队列实现，以及由 [SwiftMailer](http://swiftmailer.org/) 提供的 mailer 实现。

Laravel 所有的 contracts 都放在各自的 [GitHub 代码库](https://github.com/illuminate/contracts)。除了提供给所有可用的 contracts 一个快速的参考，也可以单独作为一个低耦合的扩展包来让其他扩展包开发者使用。

<a name="contracts-vs-facades"></a>
### Contracts 对比 Facades

Laravel 的 [facades](/docs/{{version}}/facades) 提供一个简单的方法来使用服务，而不需要使用类型提示和在服务容器之外解析 contracts。大多数情况下，每个 facade 都有一个相应的 contract。

不像 facades 那样，contracts 需要你为你的类显示的定义依赖关系。有些开发者喜欢这种显示的依赖定义，所以他们喜欢使用 contracts，而其他开发者更喜欢方便的 facades。

> {提示} 大多数应用不管你是使用 facades 还是 contracts 都可以很好的工作，但是如果你打算构建扩展包的话，强烈建议使用 contracts，因为它们在扩展包的环境下更容易被测试。

<a name="when-to-use-contracts"></a>
## 何时使用 Contracts

正如我们所说，到底是选择 facade 还是 contracts 取决于你或你的团队的喜好。不管是 facade 还是 contracts，都可以创建出健壮的，易测试的应用。随着你长期关注于类的功能层面，你会发现其实 facades 和 contracts 之间并没有太大的区别。

但是，你一定还是有些疑惑。比如，为什么总是使用接口？使用接口不是更复杂吗？让我们来提炼一下这样做的原因：松耦合和简洁性。

<a name="loose-coupling"></a>
### 松耦合

首先，让我们来看一下关于缓存的一个紧耦合的实现方式，请阅读下面的代码并好好思考：

    <?php

    namespace App\Orders;

    class Repository
    {
        /**
         * The cache instance.
         */
        protected $cache;

        /**
         * Create a new repository instance.
         *
         * @param  \SomePackage\Cache\Memcached  $cache
         * @return void
         */
        public function __construct(\SomePackage\Cache\Memcached $cache)
        {
            $this->cache = $cache;
        }

        /**
         * Retrieve an Order by ID.
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

在上面的类中，我们的代码和一个具体的缓存的实现是紧耦合的。之所以这样说，是因为我们依赖一个来自扩展包里的特定的 Cache 类。如果那个扩展包的 API 改变了，我们的代码就要随之修改。

同样的，如果我们想要把底层的缓存驱动（Memcached）换成另一个（Redis），也需要修改代码。我们的代码其实不需要考虑数据是谁提供的以及是怎样提供的。

**相比于这种方法, 使用简单的，不依赖具体实现的接口就可以改进我们的代码:**

    <?php

    namespace App\Orders;

    use Illuminate\Contracts\Cache\Repository as Cache;

    class Repository
    {
        /**
         * The cache instance.
         */
        protected $cache;

        /**
         * Create a new repository instance.
         *
         * @param  Cache  $cache
         * @return void
         */
        public function __construct(Cache $cache)
        {
            $this->cache = $cache;
        }
    }

现在代码不依赖任何特殊的扩展包，甚至是 Laravel。因为 contracts 只定义了接口，并不包含任何的依赖和实现，所以你可以很简单的去实现任何给定的 contracts，你只需要根据 contracts 编写另外一个缓存的实现就可以轻松的进行替换，而并不用修改之前写过的代码。

<a name="simplicity"></a>
### 简洁性

当所有的 Laravel 的服务都通过简单整洁的接口进行定义，那么很容易就可以判断某个服务具有哪些功能。**因此 contracts 可以被看做是框架功能的简单文档。**

除此之外，当你只依赖于简单的接口，代码就非常易于阅读和维护。相比于在一个又长又复杂的类里寻找哪些方法是可用的，你可以直接参考简单又整洁的接口。

<a name="how-to-use-contracts"></a>
## 如何使用 Contracts

那么，如何获取一个 contract 的实现呢？其实真的非常简单。

Laravel 里很多类型的类都是通过[服务容器](/docs/{{version}}/container)解析出来的。包括控制器，事件监听器，中间件，任务队列，甚至是路由的闭包。所以说，想要获得一个 contract 的实现，你只需要在类的构造函数里添加相应的类型提示就好了。

举个例子，看一下这个事件监听器：

    <?php

    namespace App\Listeners;

    use App\User;
    use App\Events\OrderWasPlaced;
    use Illuminate\Contracts\Redis\Database;

    class CacheOrderInformation
    {
        /**
         * The Redis database implementation.
         */
        protected $redis;

        /**
         * Create a new event handler instance.
         *
         * @param  Database  $redis
         * @return void
         */
        public function __construct(Database $redis)
        {
            $this->redis = $redis;
        }

        /**
         * Handle the event.
         *
         * @param  OrderWasPlaced  $event
         * @return void
         */
        public function handle(OrderWasPlaced $event)
        {
            //
        }
    }

当事件监听器被解析式，服务容器会从构造函数里读取到类型提示，并且注入合适的类。想了解如何注册绑定到容器，请参考[这篇文档](/docs/{{version}}/container)。


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
