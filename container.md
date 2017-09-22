# Laravel 服务容器解析

- [简介](#introduction)
- [绑定](#binding)
    - [绑定基础](#binding-basics)
    - [绑定接口实现](#binding-interfaces-to-implementations)
    - [上下文绑定](#contextual-binding)
    - [标记](#tagging)
- [解析](#resolving)
    - [Make 方法](#the-make-method)
    - [自动注入](#automatic-injection)
- [容器事件](#container-events)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## 简介

Laravel 服务容器是用于管理类的依赖和执行依赖注入的工具。依赖注入这个花俏名词实质上是指：类的依赖项通过构造函数，或者某些情况下通过「setter」方法「注入」到类中。

来看一个简单的例子:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Repositories\UserRepository;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 用户存储库的实现。
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * 创建新的控制器实例。
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * 显示指定用户的 profile。
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

在这个例子中，控制器 `UserController` 需要从数据源中获取 users 。因此，我们要**注入**可以获取 users 的服务。在这种情况下，`UserRepository` 可能是使用 [Eloquent](/docs/{{version}}/eloquent) 从数据库中获取 user 信息。因为 Repository 是通过 `UserRepository` 注入的，所以我们可以轻易地将其切换为另一个实现。这种注入方式的便利之处还体现在当我们为应用编写测试时，我们还可以轻松地「模拟」或创建 `UserRepository` 的虚拟实现。

想要构建强大的大型应用，至关重要的一件事是：要深刻的理解 Laravel 服务容器。当然，为 Laravel 的核心代码做出贡献也一样。


<a name="binding"></a>
## 绑定

<a name="binding-basics"></a>
### 绑定基础

因为几乎所有服务容器都是在 [服务提供器](/docs/{{version}}/providers) 中注册绑定的，所以文档中大多数例子都是使用了在服务提供器中绑定的容器。

> {tip} 如果类没有依赖任何接口，就没有必要将类绑定到容器中。容器不需要指定如何构建这些对象，因为它可以使用反射自动解析这些对象。

#### 简单绑定

在服务提供器中，你可以通过 `$this->app` 属性访问容器。我们可以通过 `bind` 方法注册绑定，传递我们想要注册的类或接口名称再返回类的实例的 `Closure` ：

    $this->app->bind('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

注意，我们接受容器本身作为解析器的参数。然后，我们可以使用容器来解析正在构建的对象的子依赖。

#### 绑定一个单例

`singleton` 方法将类或接口绑定到只能解析一次的容器中。绑定的单例被解析后，相同的对象实例会在随后的调用中返回到容器中：

    $this->app->singleton('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

#### 绑定实例

你也可以使用 `instance` 方法将现有对象实例绑定到容器中。给定的实例会始终在随后的调用中返回到容器中：

    $api = new HelpSpot\API(new HttpClient);

    $this->app->instance('HelpSpot\API', $api);

#### 绑定初始数据

当你有一个类不仅需要接受一个注入类，还需要注入一个基本值（比如整数）。你可以使用上下文绑定来轻松注入你的类需要的任何值：

    $this->app->when('App\Http\Controllers\UserController')
              ->needs('$variableName')
              ->give($value);

<a name="binding-interfaces-to-implementations"></a>

### 绑定接口到实现

服务容器有一个强大的功能，就是将接口绑定到给定实现。例如，如果我们有一个 `EventPusher` 接口和一个 `RedisEventPusher` 实现。编写完接口的 `RedisEventPusher` 实现后，我们就可以在服务容器中注册它，像这样：

    $this->app->bind(
        'App\Contracts\EventPusher',
        'App\Services\RedisEventPusher'
    );

这么做相当于告诉容器：当一个类需要实现 `EventPusher` 时，应该注入 `RedisEventPusher`。现在我们就可以在构造函数或者任何其他通过服务容器注入依赖项的地方使用类型提示注入 `EventPusher` 接口：

    use App\Contracts\EventPusher;

    /**
     * 创建一个新的类实例
     *
     * @param  EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

<a name="contextual-binding"></a>
### 上下文绑定

有时候，你可能有两个类使用了相同的接口，但你希望每个类都能注入不同的实现。例如，两个控制器可能需要依赖不同的 `Illuminate\Contracts\Filesystem\Filesystem` [契约](/docs/{{version}}/contracts) 实现。  Laravel 提供了一个简单、优雅的接口来定义这个行为：

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

有时候，你可能需要解析某个「分类」下的所有绑定。例如，你正在构建一个报表的聚合器，它接收一个包含不同 `Report` 接口实现的数组。注册了 `Report` 实现后，你可以使用 `tag` 方法为其分配标签：

    $this->app->bind('SpeedReport', function () {
        //
    });

    $this->app->bind('MemoryReport', function () {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

服务被标记后，你可以通过 `tagged` 方法轻松地将它们全部解析：

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="resolving"></a>
## 解析

<a name="the-make-method"></a>
#### `make` 方法

你可以使用 `make` 方法将容器中的类实例解析出来。`make` 方法接受要解析的类或接口的名称：

    $api = $this->app->make('HelpSpot\API');

如果你的代码处于不能访问 `$app` 变量的位置，你可以使用全局的辅助函数 `resolve`：

    $api = resolve('HelpSpot\API');

如果你的某些类的依赖项不能通过容器去解析，那你可以通过将它们作为关联数组传递到 `makeWith` 方法来注入它们。

    $api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);

<a name="automatic-injection"></a>
#### 自动注入

你可以简单地使用「类型提示」的方式在由容器解析的类的构造函数中添加依赖项，包括 [控制器](/docs/{{version}}/controllers)、[事件监听器](/docs/{{version}}/events)、[队列任务](/docs/{{version}}/queues)、[中间件](/docs/{{version}}/middleware) 等。 事实上，这是你的大多数对象也应该由容器解析。

例如，你可以在控制器的构造函数中对应用程序定义的 Repository 使用类型提示。Repository 会被自动解析并注入到类中：

    <?php

    namespace App\Http\Controllers;

    use App\Users\Repository as UserRepository;

    class UserController extends Controller
    {
        /**
         * 用户存储库实例。
         */
        protected $users;

        /**
         * 创建一个新的控制器实例。
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

每当服务容器解析一个对象时触发一个事件。你可以使用 `resolving` 方法监听这个事件：

    $this->app->resolving(function ($object, $app) {
        // 当容器解析任何类型的对象时调用...
    });

    $this->app->resolving(HelpSpot\API::class, function ($api, $app) {
        // 当容器解析类型为「HelpSpot\API」的对象时调用...
    });

如你所见，被解析的对象会被传递给回调中，让你在对象被传递出去之前可以在对象上设置任何属性。

<a name="psr-11"></a>

## PSR-11

Laravel 的服务容器实现了 [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md) 接口。因此，你可以对 PSR-11容器接口类型提示来获取 Laravel 容器的实例：

    use Psr\Container\ContainerInterface;

    Route::get('/', function (ContainerInterface $container) {
        $service = $container->get('Service');

        //
    });

> {note} 如果标签没有明确绑定到容器中，那么调用 `get` 方法时会抛出异常。

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@kair](http://www.jianshu.com/u/7fdb641c0d01) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/18390_1502954686.jpeg?imageView2/1/w/100/h/100"> | 翻译 | [@kair](https://github.com/JKair) |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  Review  | Stay Hungry. Stay Foolish. |


---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
