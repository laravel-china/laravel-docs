# Contracts

- [简介](#introduction)
- [为何要用 Contracts?](#why-contracts)
- [Contract 参考](#contract-reference)
- [如何使用 Contracts](#how-to-use-contracts)

<a name="introduction"></a>
## 简介

Laravel 的 Contracts 是一组定义了框架核心服务的接口（ php class interfaces ）。例如，`Illuminate\Contracts\Queue\Queue` contract 定义了队列任务所需要的方法，而 `Illuminate\Contracts\Mail\Mailer` contract 定义了寄送 e-mail 需要的方法。

框架对于每个 contract 都有提供对应的实现，例如，Laravel 提供各种驱动程序的队列实现，以及由  [SwiftMailer](http://swiftmailer.org/) 提供的 mailer 实现。

Laravel 所有的 contracts 都放在 [各自的 GitHub 代码库](https://github.com/illuminate/contracts)。除了提供给所有可用的 contracts 一个快速的参考，也可以单独作为一个低耦合的扩展包来让其他扩展包开发者使用。

### Contracts Vs. Facades

Laravel 的 [facades](/docs/{{version}}/facades) 提供一个简单的方法来使用服务，而不需要使用类型提示和在服务容器之外解析 contracts。然而，使用 contracts 可以明显地定义出类的依赖，对大部分应用进程而言，使用 facade 就足够了，然而，若你实在需要特别的低耦合，使用 contracts 可以做到这一点，来让我们继续往下看！

<a name="why-contracts"></a>
## 为何要用 Contracts?

你可能有很多关于 contracts 的问题。像是为什么要使用接口？使用接口会不会变的更复杂？

让我们用下面的标题来解释为什么要使用接口：低耦合和简单性。

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
            if ($this->cache->has($id)) {
                //
            }
        }
    }

在此类中，程序和缓存实现之间是高耦合。因为它是依赖于扩展包的特定缓存类。一旦这个扩展包的 API 更改了，我们的代码也要跟着改变。

同样的，如果想要将底层的缓存技术（比如 Memcached ）切换成另一种（像 Redis ），又一次的我们必须修改这个 `Repository` 类。我们的 `Repository` 类不应该知道这么多关于谁提供了数据，或是如何提供等细节。

**比起上面的做法，我们可以使用一个简单、和扩展包无关的接口来改进代码：**

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

### 简单性

当所有的 Laravel 服务都使用简洁的接口定义，就能够很容易决定一个服务需要提供的功能。 **可以将 contracts 视为说明框架特色的简洁文档。**

除此之外，当依赖的接口足够简洁时，代码的可读性和可维护性大大提高。比起搜索一个大型复杂的类里有哪些可用的方法，你有一个简单，干净的接口可以参考。

<a name="contract-reference"></a>
## Contract 参考

以下是大部分 Laravel Contracts 的参考，以及相对应的「facade」：

Contract  |  对应的 Facade
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

<a name="how-to-use-contracts"></a>
## 如何使用 Contracts

要如何使用一个 contract 呢？实际上非常的简单。

很多 Laravel 的类都是经由 [服务容器](/docs/{{version}}/container) 来解析，包含控制器，事件监听，中间件，队列任务，甚至是路由闭包。所以，要实现一个 contract，你可以在类的构造器使用「类型提示」解析类。

例如，我们来看看这个事件监听程序：

    <?php

    namespace App\Listeners;

    use App\User;
    use App\Events\NewUserRegistered;
    use Illuminate\Contracts\Redis\Database;

    class CacheUserInformation
    {
        /**
         * 实现 Redis 数据库
         */
        protected $redis;

        /**
         * 创建一个新的事件处理对象
         *
         * @param  Database  $redis
         * @return void
         */
        public function __construct(Database $redis)
        {
            $this->redis = $redis;
        }

        /**
         * 处理事件
         *
         * @param  NewUserRegistered  $event
         * @return void
         */
        public function handle(NewUserRegistered $event)
        {
            //
        }
    }

当事件监听被解析时，服务容器会经由类构造器参数的类型提示，注入适当的值。要知道怎么注册更多服务容器，参考 [这份文档](/docs/{{version}}/container).





--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译。
> 
> 文档永久地址： http://d.laravel-china.org