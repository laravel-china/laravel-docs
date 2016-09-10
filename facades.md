# Facades

- [简介](#introduction)
- [何时使用 Facades](#when-to-use-facades)
    - [Facades 对比依赖注入](#facades-vs-dependency-injection)
    - [Facades 对比辅助函数](#facades-vs-helper-functions)
- [Facades 工作原理](#how-facades-work)
- [Facade 类参考](#facade-class-reference)

<a name="introduction"></a>
## 简介
Facades /fəˈsäd/ 为应用程序的[服务容器](/docs/{{version}}/container)中可用的类提供了一个「静态」接口。Laravel 自带了许多的 facades，可以用来访问其几乎所有的服务。Laravel facades 就是服务容器里那些基类的「静态代理」，相比于传统的静态方法调用，facades 在提供更简洁且丰富的语法的同时，还有更好的可测试性和扩展性。

所有的 Laravel facades 都在 `Illuminate\Support\Facades` 这个命名空间下。所以我们可以用类似下面的代码轻松调用：

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

在 Laravel 的文档中，很多代码示例都使用了 facades 来演示框架的各种特性。

<a name="when-to-use-facades"></a>
## 何时使用 Facades

Facades 有很多的好处，它们提供了简单易记的语法让你使用 Laravel 的各种功能，你再也不需要记住长长的类名来实现注入或者手动去配置。还有，因为它们对于PHP动态方法的独特用法，测试起来就非常容易。

但是，在使用 facades 时，有些地方还是需要特别注意。使用 facades 最主要的风险就是会引起类的体积的膨胀。由于 facades 使用起来非常简单而且不需要注入，我们会不经意间在单个类中大量使用。不像是使用依赖注入，用得越多，构造方法会越长，在视觉上就会引起注意，提醒你这个类有点太庞大了。所以，在使用 facades 时，要特别注意把类的体积控制在一个合理的范围。

> {提示} 在开发需要和 Laravel 交互的扩展包时，最好是注入 [Laravel contracts](/docs/{{version}}/contracts) 而不是使用 facades，因为扩展包不是在 Laravel 内部编译的，无法使用 Laravel 的 facades 的测试辅助函数。

<a name="facades-vs-dependency-injection"></a>
### Facades 对比依赖注入

依赖注入一个主要的好处就是可以切换注入的类的具体实现。这在测试时很有用，因为你可以注入一个 mock 或者 stub 方法并且断言一些方法在 stub 中被调用。

在以前，静态方法是不可能被 mock 或者 stub 的。但是因为 facades 调用的是对象的动态方法，我可以像测试注入类的实例一样测试 facades。举个例子，有这样的一个路由：

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

我们可以用下面的测试代码去验证 `Cache::get` 方法是否像我们预期那样被调用并传入参数。

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
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
### Facades 对比辅助函数

除了 facades，Laravel 还引入了一些「辅助函数」来实现一些常用的功能，比如生成视图，触发事件，调度任务或者发送 HTTP 响应。许多辅助函数的功能其实和对应的 facades 一样。举例来说，这个 facede 和辅助函数是一样的：

    return View::make('profile');

    return view('profile');

Facade 和辅助函数其实没有本质区别，当使用辅助函数时，你依然可以像使用对应的 facade 一样测试它们。比如，下面的路由：

    Route::get('/cache', function () {
        return cache('key');
    });
    
在底层，辅助函数 `cache` 会调用 `Cache` facade 的基类的 `get` 方法。因此，即使我们是在使用辅助函数，我们依然可以用下面的代码来测试方法是不是被正确的调用了：

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
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

在 Laravel 应用中，一个 facade 其实就是一个提供访问容器中对象功能的类。其中最核心的部件就是 `Facade` 类。不管是 Laravel 自带的，还是用户自定义的 Facades，都是继承了 `Illuminate\Support\Facades\Facade` 这个类。

`Facade` 类利用了 `__callStatic()` 这个魔术方法来延迟调用容器中的对象的方法。在下面的例子里，调用了 Laravel 缓存系统。在代码里，我们可能以为 `Cache` 这个类的静态方法 `get` 被调用了：

    <?php

    namespace App\Http\Controllers;

    use Cache;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
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

注意在代码的最上面，我们导入的是 `Cache` facade。这个 facade 其实是我们获取底层 `Illuminate\Contracts\Cache\Factory` 接口实现的一个代理。我们通过这个 facade 调用的任何方法，都会被传递到 Laravel 缓存服务的底层实例中。

如果我们看一下 `Illuminate\Support\Facades\Cache` 这个类，你会发现根本没有 `get` 这个静态方法：

    class Cache extends Facade
    {
        /**
         * Get the registered name of the component.
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }

其实，`Cache` facade 是继承了基类 `Facade` 并且定义了 `getFacadeAccessor()` 方法。这个方法的功能是返回服务容器中绑定的名字。当用户调用 `Cache` facade 的静态方法时，Laravel 会解析到 `cache` 在服务容积里绑定的具体的那个实例对象，并且调用这个对象的相应方法（在这个例子里就是 `get` 方法）。

<a name="facade-class-reference"></a>
## Facade 类参考

在下方你可以找到每个 facade 及其底层的类。这个工具对于通过指定 facade 的来源快速寻找 API 文档相当有用。可应用的[服务容器绑定](/docs/{{version}}/container)关键字也包含在里面。

Facade  |  Class  |  Service Container Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](http://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Contracts\Console\Kernel](http://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](http://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Blade  |  [Illuminate\View\Compilers\BladeCompiler](http://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](http://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |
Cache  |  [Illuminate\Cache\Repository](http://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](http://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](http://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](http://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](http://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](http://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |
Event  |  [Illuminate\Events\Dispatcher](http://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](http://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](http://laravel.com/api/5.1/Illuminate/Contracts/Auth/Access/Gate.html)  |
Hash  |  [Illuminate\Contracts\Hashing\Hasher](http://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Lang  |  [Illuminate\Translation\Translator](http://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](http://laravel.com/api/{{version}}/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](http://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Password  |  [Illuminate\Auth\Passwords\PasswordBroker](http://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password`
Queue  |  [Illuminate\Queue\QueueManager](http://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance)  |  [Illuminate\Contracts\Queue\Queue](http://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html)  |  `queue`
Queue (Base Class) |  [Illuminate\Queue\Queue](http://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |
Redirect  |  [Illuminate\Routing\Redirector](http://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](http://laravel.com/api/{{version}}/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](http://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](http://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |
Route  |  [Illuminate\Routing\Router](http://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](http://laravel.com/api/{{version}}/Illuminate/Database/Schema/Blueprint.html)  |
Session  |  [Illuminate\Session\SessionManager](http://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](http://laravel.com/api/{{version}}/Illuminate/Session/Store.html)  |
Storage  |  [Illuminate\Contracts\Filesystem\Factory](http://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Factory.html)  |  `filesystem`
URL  |  [Illuminate\Routing\UrlGenerator](http://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](http://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](http://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html) |
View  |  [Illuminate\View\Factory](http://laravel.com/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](http://laravel.com/api/{{version}}/Illuminate/View/View.html)  |
