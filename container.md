# Laravel 服务容器解析

- [简介](#introduction)
- [绑定](#binding)
    - [绑定基础](#binding-basics)
    - [绑定接口至实现](#binding-interfaces-to-implementations)
    - [情境绑定](#contextual-binding)
    - [标记](#tagging)
- [解析](#resolving)
    - [Make 方法](#the-make-method)
    - [自动注入](#automatic-injection)
- [容器事件](#container-events)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## 简介

Laravel 服务容器是管理类依赖和运行依赖注入的有力工具。依赖注入是一个花俏的名词，它实质上是指：类的依赖通过构造器或在某些情况下通过「setter」方法进行「注入」。

来看一个简单的例子:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Repositories\UserRepository;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * user repository 的实现。
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * 创建信的控制器实例。
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * 显示指定用户的详细信息。
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

在这个例子中，控制器 `UserController` 需要从数据源中获取 users 。因此，我们要 **注入** 可以获取 users 的服务。在这种情况下， `UserRepository` 可能是通过使用 [Eloquent](/docs/{{version}}/eloquent) 来从数据库中获取 user 信息。因为 `UserRepository` 是通过注入获取，所以我们可以容易地切换为其他实现。当测试应用程序时，我们还可以轻松地 「mock」 ，或创建假的 `UserRepository` 实例。

在构建强大的应用程序，和为 Laravel 核心贡献代码时，必须深入理解 Laravel 的服务容器。


<a name="binding"></a>
## 绑定

<a name="binding-basics"></a>
### 绑定基础

几乎所有服务容器的绑定都是在 [服务提供者](/docs/{{version}}/providers) 中进行的，所以下面的例子将示范在该情景中使用容器。

> {tip} 但是，如果类没有依赖任何接口，那么就没有必要将类绑定到容器中了。容器绑定时，并不需要指定如何构建这些类，因为容器中会通过 PHP 的反射自动解析对象。

#### 简单绑定

在服务提供者中，你经常可以通过 `$this->app` 属性访问容器。我们可以通过 `bind` 方法注册一个绑定，通过传递注册类或接口的名称、及返回该实例的 `Closure` 作为参数：

    $this->app->bind('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

注意，我们将获得的容器本身作为参数传递到解析器中，这样就可以使用容器来解决绑定对象对容器的子依赖。

#### 绑定一个单例

通过 `singleton` 方法可以绑定一个只会被解析一次的类或接口到容器中。且后面的调用都会从容器中返回相同的实例：

    $this->app->singleton('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

#### 绑定实例

你也可以使用 `instance` 方法绑定一个已经存在的对象至容器中。后面的调用都会从容器中返回指定的实例：

    $api = new HelpSpot\API(new HttpClient);

    $this->app->instance('HelpSpot\API', $api);

#### 绑定初始数据

有时，你的类不仅需要注入类，还需要注入一些原始数据，如一个整数。此时，你可以容易地通过情景绑定注入需要的任何值：

    $this->app->when('App\Http\Controllers\UserController')
              ->needs('$variableName')
              ->give($value);

<a name="binding-interfaces-to-implementations"></a>
绑定接口至实现

服务容器有一个强大的功能，就是将一个指定接口的实现绑定到接口上。例如，如果我们有一个 `EventPusher` 接口和一个它的实现类 `RedisEventPusher` 。编写完接口的 `RedisEventPusher` 实现类后，我们就可以在服务容器中像下面例子一样注册它：

    $this->app->bind(
        'App\Contracts\EventPusher',
        'App\Services\RedisEventPusher'
    );

这么做会告诉容器当一个类需要 `EventPusher` 接口的实例时， `RedisEventPusher` 的实例将会被容器注入。现在我们就可以在构造函数中，或者任何其他需要通过容器注入依赖的地方，使用 `EventPusher` 接口的类型提示：

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
### 情境绑定

有时候，你可能有两个类使用到相同的接口，但你希望每个类都能注入不同的实现。例如，两个控制器可能需要依赖不同的 `Illuminate\Contracts\Filesystem\Filesystem` [契约](/docs/{{version}}/contracts) 的实现类。 Laravel 为此定义了一种简单、平滑的接口：

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
### 标记

有时候，你可能需要解析某个「分类」下的所有绑定。例如，你正在构建一个报表的聚合器，它需要接受不同 `Report` 接口的实例。分别注册了 `Report` 实例后，你可以使用 `tag` 方法为他们赋予一个标签：

    $this->app->bind('SpeedReport', function () {
        //
    });

    $this->app->bind('MemoryReport', function () {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

一旦服务被标记后，你可以通过 `tagged` 方法轻松地将它们全部解析：

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="resolving"></a>
## 解析

<a name="the-make-method"></a>
#### `make` 方法

你可以在服务容器外使用 `make` 方法来获得一个实例化的类。它接受你希望解析的类或是接口名称作为参数：

    $api = $this->app->make('HelpSpot\API');

如果你的代码不能直接使用 `$app` 变量，你可以使用全局的 `resolve` 助手：

    $api = resolve('HelpSpot\API');

如果你的某些类依赖的属性不能通过容器去解析, 则可以通过将它们作为关联数组传递到 `makeWith` 方法中来注入它们。

    $api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);

<a name="automatic-injection"></a>
#### 自动注入

另外，并且也是重要的，你可以在类的构造函数中对依赖使用「类型提示」，依赖的类将会被容器自动进行解析，包括在 [控制器](/docs/{{version}}/controllers) ， [事件监听器](/docs/{{version}}/events) ， [队列任务](/docs/{{version}}/queues) ， [中间件](/docs/{{version}}/middleware) 等地方。 事实上，这也是大部分类被容器解析的方式。

例如，你可以在控制器的构造函数中对应用程序定义的 `Repository` 使用类型提示。这样 `Repository` 实例会被自动解析并注入到类中：

    <?php

    namespace App\Http\Controllers;

    use App\Users\Repository as UserRepository;

    class UserController extends Controller
    {
        /**
         * user repository 实例。
         */
        protected $users;

        /**
         * 控制器构造方法。
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * 显示指定 ID 的用户信息。
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

每当服务容器解析一个对象时就会触发一个事件。你可以使用 `resolving` 方法监听这个事件：

    $this->app->resolving(function ($object, $app) {
        // 解析任何类型的对象时都会调用该方法...
    });

    $this->app->resolving(HelpSpot\API::class, function ($api, $app) {
        // 解析「HelpSpot\API」类型的对象时调用...
    });

如你所见，被解析的对象会被传递至回调中，让你在对象被传递到消费者前可以设置任何额外属性到对象上。


<a name="psr-11"></a>
## PSR-11

Laravel 的服务容器实现了[PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md)的接口。因此，你可以使用PSR-11接口的类型提示去实例化一个容器：

    use Psr\Container\ContainerInterface;

    Route::get('/', function (ContainerInterface $container) {
        $service = $container->get('Service');

        //
    });

> {note} 如果标签没有绑定在容器当中，那么调用 `get` 方法将会抛出一个错误。
