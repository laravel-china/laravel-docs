# Laravel 的 Facades 介绍

- [简介](#introduction)
- [何时使用 Facades](#when-to-use-facades)
    - [Facades Vs. 依赖注入](#facades-vs-dependency-injection)
    - [Facades Vs. 辅助函数](#facades-vs-helper-functions)
- [Facades 工作原理](#how-facades-work)
- [Facade 类参考](#facade-class-reference)

<a name="introduction"></a>
## 简介

Facades /fəˈsäd/ 为应用程序的 [服务容器](/docs/{{version}}/container) 中可用的类提供了一个「静态」接口。Laravel 自带了很多 facades ，几乎可以用来访问到 Laravel 中所有的服务。Laravel facades 实际上是服务容器中那些底层类的「静态代理」，相比于传统的静态方法， facades 在提供了简洁且丰富的语法同时，还带来了更好的可测试性和扩展性。

所有的 Laravel facades 都需要定义在命名空间 `Illuminate\Support\Facades` 下。所以，我们可以容易地向下面这样调用 facade :

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

在 Laravel 的文档中，很多示例代码都是使用 facades 来演示框架的各种特性的。

<a name="when-to-use-facades"></a>
## 何时使用 Facades

Facades 有很多好处，它为我们使用 Laravel 的各种功能提供了简单，易记的语法，让你不需要记住长长的类名来实现依赖注入和手动配置。还有，因为它们对于PHP动态方法的独特用法，测试起来非常容易。

然而，在使用 facades 时，有些地方还需要特别注意。使用 facades 最主要的风险就是会引起类作用范围的膨胀。因为 facades 使用起来非常简单而且不需要注入，我们会不经意的在单个类中大量使用。它不会像使用依赖注入那样，使用的类越多，构造方法会越长，在视觉上就会引起注意，提醒你这个类有点庞大了。所以在使用 facades 的时候，要特别注意控制好类的大小，让类的作用范围保持短小。

> {tip} 在开发与 Laravel 交互的第三发扩展包时，最好是在包中通过注入 [Laravel contracts](/docs/{{version}}/contracts) ，而不是在包中通过 facades 来使用 Laravel 的类。因为扩展包不是在 Laravel 内部使用的，无法使用 Laravel's facade 的测试辅助函数。

<a name="facades-vs-dependency-injection"></a>
### Facades Vs. 依赖注入

依赖注入的一个主要的好处是可以切换注入类的具体实现。这在测试的时候很有用，因为你可以注入一个 mock 或者 stub ，并且对在 stub 中被调用的各种方法进行断言。

通常，静态方法是不可以被 mock 或者 stub 。但是，因为 facades 调用的是对象的动态方法，我们可以像测试注入类的实例一样测试 facades ，例如，像下面的路由：

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

我们可以用下面的测试代码去验证 `Cache::get` 方法是否被调用，当传入预期的参数时。

    use Illuminate\Support\Facades\Cache;

    /**
     * 一个基础功能的测试用例。
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $this->visit('/cache')
             ->see('value');
    }

<a name="facades-vs-helper-functions"></a>
### Facades Vs. 辅助函数

除了 facades ， Laravel 包含一些「辅助函数」来实现一些常用的功能，比如生成视图，触发事件，调度任务或者发送 HTTP 响应。许多辅助函数的功能和对应的 facades 一样。例如，下面这个 facade 和辅助函数的作用是一样的：

    return View::make('profile');

    return view('profile');

这里的 facades 和辅助函数是没有任何区别的。当你使用辅助函数时，你依然可以向使用对应的 facade 一样测试他们。例如，下面的路由：

    Route::get('/cache', function () {
        return cache('key');
    });

在底层，辅助函数 `cache` 实际是调用 `Cache` facade 中的 `get` 方法。因此，尽管我们是在使用辅助函数，我们依然可以用下面的测试代码来验证是否方法被正确调用，在传入预期的参数时：

    use Illuminate\Support\Facades\Cache;

    /**
     * 一个基础功能的测试用例。
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $this->visit('/cache')
             ->see('value');
    }

<a name="how-facades-work"></a>
## Facades 工作原理

在 Laravel 应用中，一个 facade 就是一个提供访问容器中对象的类。其中核心的部件就是 `Facade` 类。不管是 Laravel 自带的 Facades ，还是用户自定义的 Facades ，都继承自 `Illuminate\Support\Facades\Facade` 类。

`Facade` 基类使用 `__callStatic()` 魔术方法在你的 facades 中延迟调用容器中对应对象的方法，在下面的例子中，调用了 Laravel 的缓存系统。在代码里，我们可能认为是 `Cache` 类中的静态方法 `get` 被调用了：

    <?php

    namespace App\Http\Controllers;

    use Cache;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 显示给定用户的大体信息。
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Cache::get('user:'.$id);

            return view('profile', ['user' => $user]);
        }
    }

注意在代码的最上面，我们导入的是 `Cache` facade 。这个 facade 其实是我们获取底层 `Illuminate\Contracts\Cache\Factory` 接口实现的一个代理。我们通过这个 facade 调用的任何方法，都会被传递到 Laravel 缓存服务的底层实例中。

如果我们看一下 `Illuminate\Support\Facades\Cache` 这个类，你会发现类中根本没有 `get` 这个静态方法：

    class Cache extends Facade
    {
        /**
         * 获取组件在容器中注册的名称。
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }

其实， `Cache` facade 是继承了 `Facade` 基类，并且定义了 `getFacadeAccessor()` 方法。这个方法的作用是返回服务容器中对应名字的绑定内容。当用户调用 `Cache` facade 中的任何静态方法时， Laravel 会解析到服务容器中绑定的键值为 `cache` 实例对象，并调用这个对象对应的方法（在这个例子中就是 `get` 方法）。

<a name="facade-class-reference"></a>
## Facade 类参考

在下面你可以找到每个 facade 类及其对应的底层类。这是一个查找给定 facade 类 API 文档的有用工具。 也列出了绑定在 [服务容器](/docs/{{version}}/container) 中 facade 类对应的可用键值。

Facade  |  Class  |  Service Container Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](https://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](https://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Blade  |  [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |  &nbsp;
Cache  |  [Illuminate\Cache\Repository](https://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](https://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](https://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](https://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](https://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](https://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |  &nbsp;
Event  |  [Illuminate\Events\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html)  |  &nbsp;
Hash  |  [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Lang  |  [Illuminate\Translation\Translator](https://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](https://laravel.com/api/{{version}}/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](https://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Notification  |  [Illuminate\Notifications\ChannelManager](https://laravel.com/api/{{version}}/Illuminate/Notifications/ChannelManager.html)  |  &nbsp;
Password  |  [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html)  |  `auth.password`
Queue  |  [Illuminate\Queue\QueueManager](https://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance)  |  [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html)  |  `queue`
Queue (Base Class) |  [Illuminate\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |  &nbsp;
Redirect  |  [Illuminate\Routing\Redirector](https://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](https://laravel.com/api/{{version}}/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](https://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |  &nbsp;
Route  |  [Illuminate\Routing\Router](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](https://laravel.com/api/{{version}}/Illuminate/Database/Schema/Blueprint.html)  |  &nbsp;
Session  |  [Illuminate\Session\SessionManager](https://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](https://laravel.com/api/{{version}}/Illuminate/Session/Store.html)  |  &nbsp;
Storage  |  [Illuminate\Contracts\Filesystem\Factory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Factory.html)  |  `filesystem`
URL  |  [Illuminate\Routing\UrlGenerator](https://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](https://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](https://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html) |  &nbsp;
View  |  [Illuminate\View\Factory](https://laravel.com/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](https://laravel.com/api/{{version}}/Illuminate/View/View.html)  |  &nbsp;
