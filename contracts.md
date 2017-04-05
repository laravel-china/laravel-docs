# 契约 (Contracts)

- [简介](#introduction)
    - [契约 (Contracts) Vs. 门面 (Facades)](#contracts-vs-facades)
- [何时使用契约](#when-to-use-contracts)
    - [低耦合](#loose-coupling)
    - [简单性](#simplicity)
- [如何使用 Contracts](#how-to-use-contracts)
- [Contract 参考](#contract-reference)

<a name="introduction"></a>
## 简介

Laravel 的 契约(Contracts ) 是一系列框架用来定义核心服务的接口。例如，`Illuminate\Contracts\Queue\Queue` 契约中定义了队列任务所需的方法，而 `Illuminate\Contracts\Mail\Mailer` 契约中定义了发送电子邮件所需的方法。

框架对每个契约都提供了相应的实现。例如，Laravel 为队列提供了各种驱动的实现，为邮件提供了由  [SwiftMailer](http://swiftmailer.org/) 驱动的实现。

所有 Laravel 契约都有相应的 [GitHub 库](https://github.com/illuminate/contracts) ，这给所有可用的契约提供了一个快速参考指南，同时也可单独作为低耦合的扩展包给其他包开发者去使用。

<a name="contracts-vs-facades"></a>
### 契约 (contracts) VS 门面 (facades)

[门面 (facades)](/docs/{{version}}/facades) 和一些辅助函数提供了一种使用 Laravel 服务的简单方法，即不再需要通过类型约束和在服务容器之外解析契约。 在大多数情况下，每个门面都有一个等效契约。

不像门面，门面不需要你在类构造函数中约束什么，契约则是需要你明显地定义依赖关系。 一些开发人员喜欢契约这种方式明显地去定义它们的依赖关系，而另一些开发人员则更喜欢门面带来的便捷。

> {tip} 对于大多数应用程序来说不管是使用门面还是契约只要你喜欢都行。但是 ，如果你正在构建一个扩展包，为了方便测试建议考虑使用契约比较好。

<a name="when-to-use-contracts"></a>
## 何时使用 contracts

综上所述，使用契约或者门面很大程度上归结于个人或者开发团队的喜好。不管是契约还是门面都可以创建出强大的、容易测试的 Laravel 应用程序。 如果你长期关注类的单一职责，你会注意到使用契约和门面其实没多少实际意义上的区别。

然而，你可能还是会有几个关于契约的问题。像是，为什么要使用接口？使用接口会不会变得更加复杂？下面让我们简单阐述一下使用接口的原因：低耦合和简单性。


<a name="loose-coupling"></a>
### 低耦合

首先，让我们回顾一些与缓存实现的高耦合代码。如下：


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

在这个类中，程序跟给定缓存实现之间是高耦合的。因为我们依赖于一个扩展包的特定缓存类。一旦这个扩展包的 API 被更改了，那我们的代码也必须得跟着改变。

同样的，如果想要将底层的缓存技术（Memcached ）替换成另一种技术来实现（ Redis ），那又得再一次修改这个 `repository` 类。而 `repository` 类不应该知道这么多信息，比如关于谁提供了这些数据，或是他们又是如何提供的等等。

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

现在，更改之后的代码没有与任何扩展包耦合，甚至是 Laravel 。而契约扩展包不包含实现和依赖，你可以轻松地对任何契约包进行实现，比如不需要修改任何关于缓存的代码就可以替换缓存实现。

<a name="simplicity"></a>
### 简单性

当所有的 Laravel 服务都使用简洁的接口定义，就能够很容易决定一个服务需要提供的功能。 **可以将契约视为说明框架特色的简洁文档。**

除此之外，当依赖的接口足够简洁时，代码的可读性和可维护性会大大提高。比起搜索一个大型复杂的类里有哪些可用的方法，不如检索一个简单、干净的接口来参考更妥当。


<a name="how-to-use-contracts"></a>
## 如何使用 contracts

那么，如何获取一个契约的实现呢？这其实很简单。

Laravel 中的许多类型的类都是通过 [服务容器](/docs/{{version}}/container) 解析出来的。包括控制器、事件监听器、中间件、任务队列，甚至路由的闭包。所以说，要获得一个契约的实现，你只需要解析在类的构造函数中相应的类型约束即可。

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

当事件监听器被解析时，服务容器会从构造函数里读取到类型约束，并注入对应的值。 想了解关于容器的注册绑定，可以查看 [服务容器](/docs/{{version}}/container)。


<a name="contract-reference"></a>
## Contract 参考

下面的表格提供了 Laravel 契约及其对应的门面的参考:

| Contract                                 | References Facade |
| ---------------------------------------- | ----------------- |
| [Illuminate\Contracts\Auth\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Factory.php) | Auth              |
| [Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/{{version}}/Auth/PasswordBroker.php) | Password          |
| [Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Bus/Dispatcher.php) | Bus               |
| [Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/Broadcaster.php) | &nbsp;            |
| [Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Repository.php) | Cache             |
| [Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Factory.php) | Cache::driver()   |
| [Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Config/Repository.php) | Config            |
| [Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/{{version}}/Container/Container.php) | App               |
| [Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/Factory.php) | Cookie            |
| [Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/QueueingFactory.php) | Cookie::queue()   |
| [Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/{{version}}/Encryption/Encrypter.php) | Crypt             |
| [Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Events/Dispatcher.php) | Event             |
| [Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Cloud.php) | &nbsp;            |
| [Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Factory.php) | File              |
| [Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Filesystem.php) | File              |
| [Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/{{version}}/Foundation/Application.php) | App               |
| [Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/{{version}}/Hashing/Hasher.php) | Hash              |
| [Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/{{version}}/Logging/Log.php) | Log               |
| [Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/{{version}}/Mail/MailQueue.php) | Mail::queue()     |
| [Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/{{version}}/Mail/Mailer.php) | Mail              |
| [Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Factory.php) | Queue::driver()   |
| [Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Queue.php) | Queue             |
| [Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/{{version}}/Redis/Database.php) | Redis             |
| [Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/{{version}}/Routing/Registrar.php) | Route             |
| [Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/{{version}}/Routing/ResponseFactory.php) | Response          |
| [Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/{{version}}/Routing/UrlGenerator.php) | URL               |
| [Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Arrayable.php) | &nbsp;            |
| [Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Jsonable.php) | &nbsp;            |
| [Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Renderable.php) | &nbsp;            |
| [Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Factory.php) | Validator::make() |
| [Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Validator.php) | &nbsp;            |
| [Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/{{version}}/View/Factory.php) | View::make()      |
| [Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/{{version}}/View/View.php) | &nbsp;            |

## 译者署名
| 用户名                                      | 头像                                       | 职能   | 签名                                       |
| ---------------------------------------- | ---------------------------------------- | ---- | ---------------------------------------- |
| [@e421083458](https://github.com/e421083458) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/10802_1486368142.jpeg?imageView2/1/w/100/h/100"> | 翻译   | Github求star，[@e421083458](https://github.com/e421083458/) at Github |
