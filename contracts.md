# Contracts

- [简介](#introduction)
    - [Contracts Vs. Facades](#contracts-vs-facades)
- [何时使用 contracts](#when-to-use-contracts)
    - [低耦合](#loose-coupling)
    - [简单性](#simplicity)
- [如何使用 Contracts](#how-to-use-contracts)
- [Contract 参考](#contract-reference)

<a name="introduction"></a>
## 简介

Laravel 的 Contracts 是一组定义了框架核心服务的接口。例如，`Illuminate\Contracts\Queue\Queue` contract 定义了队列任务所需的方法，而 `Illuminate\Contracts\Mail\Mailer` contract 定义了发送电子邮件所需的方法。

框架对每个 contract 都提供了相应的实现。例如，Laravel 提供了一个对各种驱动程序的队列实现，以及一个由  [SwiftMailer](http://swiftmailer.org/) 驱动的邮件实现。

所有的 Laravel contracts 存放在他们 [单独的GitHub库](https://github.com/illuminate/contracts) 。除了提供给所有可用的 contracts 一个快速的参考，还可以单独作为一个低耦合的扩展包来让其他扩展包开发者使用。

<a name="contracts-vs-facades"></a>
### Contracts Vs. Facades

Laravel 的 [facades](/docs/{{version}}/facades) 和辅助函数提供了一种使用 Laravel 的服务的简单方法，而不需要类型提示和解析服务容器之外的 contracts 。 在大多数情况下，每个 facade 都有一个等效 contract。

不像 facades 那样，contracts 需要显式定义依赖关系。 一些开发人员更喜欢以这种方式显式定义它们的依赖关系，因此更喜欢使用 contracts，而其他开发人员喜欢 facades 的方便性。

> {tip} 大多数应用程序不论使用 facades 还是 contracts 都无所谓。 然而，如果你正在构建一个扩展包，强烈地建议使用 contracts，因为它们将更容易在包上下文中测试。

<a name="when-to-use-contracts"></a>
## 何时使用 contracts

正如其他地方所讨论的，使用 contracts 或 facades 很大程度上归结于个人品味和开发团队的品味。 contracts 和 facades 都可用于创建稳健、经过良好测试的 Laravel 应用程序。 只要你一直能关注类的功能点，你会发现使用 contracts 和 facades 之间很少有实际差异。

但是，您可能仍有几个关于 contracts 的问题。例如，为什么使用接口？是不是使用接口更复杂？使用接口的原因如下：低耦合和简单性。


<a name="loose-coupling"></a>
### 低耦合

首先，让我们回顾一些与缓存实现紧密耦合的代码。如下：

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
         * 按照Id检索订单。
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

在这个类中，代码跟给定的缓存实现紧密耦合。因为我们依赖于一个具体的 Cache 类从一个软件包供应商。如果该包的 API 更改，我们的代码也必须更改。

同样，如果我们要用另一种缓存技术（ Redis ）替换我们的底层缓存技术（ Memcached ），我们再次必须修改我们的 Repository 。我们的 Repository 类不应该知道这么多关于谁提供了数据，或是如何提供等细节。

**比起上面的做法，我们可以使用一个简单，和扩展包无关的接口来改进代码：**

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

现在代码没有与任何软件包耦合，甚至 Laravel 。由于 contracts 不包含实现和依赖，因此您可以轻松地编写任何 contracts 实现，从而替换缓存实现，而无需修改任何缓存代码。

<a name="simplicity"></a>
### 简单性

当所有的 Laravel 服务都使用简洁的接口定义，就能够很容易决定一个服务需要提供的功能。 **可以将 contracts 视为说明框架特色的简洁文档。**

除此之外，当依赖的接口足够简洁时，代码的可读性和可维护性大大提高。比起搜索一个大型复杂的类里有哪些可用的方法，你有一个简单，干净的接口可以参考。


<a name="how-to-use-contracts"></a>
## 如何使用 Contracts

那么，如何获取一个 contract 实现呢？这其实很简单。

Laravel 中的许多类型的类都是通过 [服务容器](/docs/{{version}}/container) 解析，包括控制器，事件监听器，中间件，任务队列，甚至路由闭包。所以，要获得一个 contract 的实现，你只需要在正在解析的类的构造函数中的声明「类型约束」即可。

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

当事件监听器被解析时，服务容器将读取类的构造函数上的类型提示，并注入对应的类实例。 想了解如何注册绑定到容器，请查看[服务容器](/docs/{{version}}/container)。


<a name="contract-reference"></a>
## Contract 参考

下面的表格提供了 Laravel contracts 及其对应的 facades 的参考:

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
[Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/{{version}}/Redis/Database.php) | Redis
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
| [@e421083458](https://github.com/e421083458)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/10802_1486368142.jpeg?imageView2/1/w/100/h/100">  |  翻译  | Github求star，[@e421083458](https://github.com/e421083458/) at Github  |