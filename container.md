# 服务容器

- [简介](#introduction)
- [绑定](#binding)
    - [基础绑定](#binding-basics)
    - [通过接口绑定实现](#binding-interfaces-to-implementations)
    - [环境绑定](#contextual-binding)
    - [标记](#tagging)
- [解析](#resolving)
    - [Make 方法](#the-make-method)
    - [自动注入](#automatic-injection)
- [容器事件](#container-events)

<a name="introduction"></a>
## 简介

Laravel 服务容器是管理类依赖和运行依赖注入的有力的工具。依赖注入是一个花俏的名词，它实质上是指：类的依赖通过构造器或在某些情况下通过「setter」方法「注入」。

来看一个简单的例子：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Repositories\UserRepository;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            $user = $this->users->find($id);

            return view('user.profile', ['user' => $user]);
        }
    }

在这个例子中，控制器 `UserController` 需要从数据源中获取 users 。因此，我们将 **注入** 可以获取 users 的服务。在这种情况下， `UserRepository` 最有可能通过使用 [Eloquent](/docs/5.3/eloquent) 来从数据库中获取 user 信息。因为 `UserRepository` 是通过注入获取，我们可以容易地切换成其他实现。当测试应用程序时，我们以可以轻松地 「mock」 ，或创建假的 `UserRepository` 实现。

在构建强大的应用程序，以及为 Laravel 核心贡献代码时，必须深入理解 Laravel 的服务容器。

<a name="binding"></a>
## 绑定

<a name="binding-basics"></a>
### 基础绑定

几乎所有服务容器的绑定都是通过 [服务提供者](/docs/{{version}}/providers) 进行的，所以下面的例子将示范在该情景中使用容器。

> {tip} 不过，如果类没有依赖任何接口，那么就没有必要将类绑定到容器中的。在容器绑定时，并不需要指定如何构建这些类，因为容器中会通过 PHP 的反射自动实例化对象。

#### 简单绑定

在服务提供者中，你总是可以通过 `$this->app` 属性访问服务容器。我们可以通过 `bind` 方法注册一个绑定，在方法的参数中，传递需要注册的类或接口名称、返回该实例的 `Closure` ：

    $this->app->bind('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

注意，我们将获得的容器本身作为参数传递到解析器中，这样就可以使用容器来解决绑定对象对容器的子依赖。

#### 绑定一个单例

`singleton` 方法绑定一个只会被解析一次的类或接口到容器中。且后面的调用都会从容器中返回相同的实例：

    $this->app->singleton('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

#### 绑定实例

你也可以使用 `instance` 方法绑定一个已经实例化的对象到容器中。后面的调用都会从容器中返回指定的实例：

    $api = new HelpSpot\API(new HttpClient);

    $this->app->instance('HelpSpot\Api', $api);

#### 绑定初始数据

有时，类初始化时，不仅需要注入类，还需要注入一些原始数据，如一个整数。这时，你可以通过情景绑定容易地注入任何需要的任何值：

    $this->app->when('App\Http\Controllers\UserController')
              ->needs('$variableName')
              ->give($value);

<a name="binding-interfaces-to-implementations"></a>
### 通过接口绑定实现

服务容器有一个强大的功能，就是将一个给定的实例绑定到接口上。例如，如果我们有一个 `EventPusher` 接口和一个 `RedisEventPusher` 类的实例。一旦我们将类 `RedisEventPusher` 编写实现 `EventPusher` 接口，我们就可以像下面方式绑定实例：

    $this->app->bind(
        'App\Contracts\EventPusher',
        'App\Services\RedisEventPusher'
    );

这么做会告诉容器当一个类需要 `EventPusher` 的实例时， `RedisEventPusher` 的实例将会被容器注入。现在我们就可以在构造函数中，使用 `EventPusher` 接口的类型提示，或者任何其他需要通过容器注入依赖的地方：

    use App\Contracts\EventPusher;

    /**
     * Create a new class instance.
     *
     * @param  EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

<a name="contextual-binding"></a>
### 情景绑定

Sometimes you may have two classes that utilize the same interface, but you wish to inject different implementations into each class. For example, two controllers may depend on different implementations of the `Illuminate\Contracts\Filesystem\Filesystem` [contract](/docs/{{version}}/contracts). Laravel provides a simple, fluent interface for defining this behavior:

    use Illuminate\Support\Facades\Storage;
    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\VideoController;
    use Illuminate\Contracts\Filesystem\Filesystem;

    $this->app->when(PhotoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('local');
              });

    $this->app->when(VideoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('s3');
              });

<a name="tagging"></a>
### Tagging

Occasionally, you may need to resolve all of a certain "category" of binding. For example, perhaps you are building a report aggregator that receives an array of many different `Report` interface implementations. After registering the `Report` implementations, you can assign them a tag using the `tag` method:

    $this->app->bind('SpeedReport', function () {
        //
    });

    $this->app->bind('MemoryReport', function () {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

Once the services have been tagged, you may easily resolve them all via the `tagged` method:

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="resolving"></a>
## Resolving

<a name="the-make-method"></a>
#### The `make` Method

You may use the `make` method to resolve a class instance out of the container. The `make` method accepts the name of the class or interface you wish to resolve:

    $api = $this->app->make('HelpSpot\API');

If you are in a location of your code that does not have access to the `$app` variable, you may use the global `resolve` helper:

    $api = resolve('HelpSpot\API');

<a name="automatic-injection"></a>
#### Automatic Injection

Alternatively, and importantly, you may simply "type-hint" the dependency in the constructor of a class that is resolved by the container, including [controllers](/docs/{{version}}/controllers), [event listeners](/docs/{{version}}/events), [queue jobs](/docs/{{version}}/queues), [middleware](/docs/{{version}}/middleware), and more. In practice, this is how most of your objects should be resolved by the container.

For example, you may type-hint a repository defined by your application in a controller's constructor. The repository will automatically be resolved and injected into the class:

    <?php

    namespace App\Http\Controllers;

    use App\Users\Repository as UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Show the user with the given ID.
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
## Container Events

The service container fires an event each time it resolves an object. You may listen to this event using the `resolving` method:

    $this->app->resolving(function ($object, $app) {
        // Called when container resolves object of any type...
    });

    $this->app->resolving(HelpSpot\API::class, function ($api, $app) {
        // Called when container resolves objects of type "HelpSpot\API"...
    });

As you can see, the object being resolved will be passed to the callback, allowing you to set any additional properties on the object before it is given to its consumer.
