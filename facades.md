# Laravel 的 Facades

- [简介](#introduction)
- [何时使用 Facades](#when-to-use-facades)
    - [Facades Vs. 依赖注入](#facades-vs-dependency-injection)
    - [Facades Vs. 辅助函数](#facades-vs-helper-functions)
- [Facades 工作原理](#how-facades-work)
- [Facade 类参考](#facade-class-reference)

<a name="introduction"></a>
## 简介

Facades（读音：/fəˈsäd/ ）为应用程序的 [服务容器](/docs/{{version}}/container) 中可用的类提供了一个「静态」接口。Laravel 自带了很多 Facades ，可以访问绝大部分 Laravel 的功能。Laravel Facades 实际上是服务容器中底层类的「静态代理」，它提供了简洁而富有表现力的语法，甚至比传统的静态方法更具可测试性和扩展性。

所有的 Laravel Facades 都在 `Illuminate\Support\Facades` 命名空间中定义。所以，我们可以轻松地使用 Facade :

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

在 Laravel 的文档中，很多示例代码都会使用 Facades 来演示框架的各种功能。

<a name="when-to-use-facades"></a>
## 何时使用 Facades

Facades 有很多好处，它为我们使用 Laravel 的功能提供了简单、易记的语法，而无需记住必须手动注入或配置的长长的类名。此外，由于它们对 PHP 动态方法的独特用法，使得测试起来非常容易。

然而，在使用 Facades 时，有些地方还需要特别注意。使用 Facades 最主要的风险就是会引起类作用范围的膨胀。因为 Facades 使用起来非常简单而且不需要注入，就会使得我们在不经意间在单个类中使用许多 Facades，从而导致类变的越来越大。而使用依赖注入的时候，使用的类越多，构造方法就会越长，在视觉上就会引起注意，提醒你这个类有点庞大了。因此在使用 Facades 的时候，要特别注意控制好类的大小，让类的作用范围保持短小。

> {tip} 在开发与 Laravel 进行交互的第三方扩展包时，建议最好选择注入 [Laravel 契约](/docs/{{version}}/contracts) ，而不是使用 Facades 的方式来使用类。因为扩展包是在 Laravel 本身之外构建，所以你无法使用 Laravel Facades 测试辅助函数。

<a name="facades-vs-dependency-injection"></a>
### Facades Vs. 依赖注入

依赖注入的主要优点之一是切换注入类的实现的能力。这在测试的时候很有用，因为你可以注入一个 mock 或者 stub ，并断言在 stub 上调用的各种方法。

通常，真正的静态方法是不可能被 mock 或者 stub。但是，因为 Facades 使用动态方法来代理从服务容器解析的对象的方法调用，我们可以像测试注入的类实例一样来测试 Facades。例如，像下面的路由：

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

我们可以用下面的测试代码来验证使用预期的参数来调用 `Cache::get` 方法：

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

除了 Facades， Laravel 还包含各种「辅助函数」来实现一些常用的功能，比如生成视图、触发事件、调度任务或者发送 HTTP 响应。许多辅助函数的功能都有与之对应的 Facade。例如，下面这个 Facade 的调用和辅助函数的作用是一样的：

    return View::make('profile');

    return view('profile');

这里的 Facades 和辅助函数之间没有实际的区别。当你使用辅助函数时，你可以使用对应的 Facade 进行测试。例如，下面的路由：

    Route::get('/cache', function () {
        return cache('key');
    });

在底层，辅助函数 `cache` 实际是调用 `Cache` facade 中的 `get` 方法。因此，尽管我们使用的是辅助函数，我们依然可以编写以下测试来验证该方法是否使用我们预期的参数来调用：

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

在 Laravel 应用中，Facade 就是一个可以从容器访问对象的类。其中核心的部件就是 `Facade` 类。不管是 Laravel 自带的 Facades，还是用户自定义的 Facades ，都继承自 `Illuminate\Support\Facades\Facade` 类。

`Facade` 基类使用了 `__callStatic()` 魔术方法将你的 Facades 的调用延迟，直到对象从容器中被解析出来。在下面的例子中，调用了 Laravel 的缓存系统。通过浏览这段代码，可以假定在 `Cache` 类中调用了静态方法 `get`：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 显示给定用户的信息。
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

注意在上面这段代码中，我们「导入」`Cache` Facade 。这个 Facade 作为访问 `Illuminate\Contracts\Cache\Factory` 接口底层实现的代理。我们使用 Facade 进行的任何调用都将传递给 Laravel 缓存服务的底层实例。

如果我们看一下 `Illuminate\Support\Facades\Cache` 这个类，你会发现类中根本没有 `get` 这个静态方法：

    class Cache extends Facade
    {
        /**
         * 获取组件的注册名称。
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }

`Cache` Facade 继承了 `Facade` 的类库，并且定义了 `getFacadeAccessor()` 方法。这个方法的作用是返回服务容器绑定的名称。当用户调用 `Cache` Facade 中的任何静态方法时， Laravel 会从 [服务容器](/docs/{{version}}/container) 中解析 `cache` 绑定以及该对象运行所请求的方法（在这个例子中就是 `get` 方法）。

<a name="facade-class-reference"></a>
## Facade 类参考

在下面你可以找到每个 Facade 类及其对应的底层类。这是一个查找给定 Facade 类 API 文档的工具。[服务容器绑定](/docs/{{version}}/container) 的可用键值也包含在内。

| Facade | 类 | 服务容器绑定 |
| ----- | ----- | ----- |
| App                  | [Illuminate\Foundation\Application](https://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html) | `app`            |
| Artisan              | [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html) | `artisan`        |
| Auth                 | [Illuminate\Auth\AuthManager](https://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html) | `auth`           |
| Blade                | [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html) | `blade.compiler` |
| Bus                  | [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html) | &nbsp;           |
| Cache                | [Illuminate\Cache\Repository](https://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html) | `cache`          |
| Config               | [Illuminate\Config\Repository](https://laravel.com/api/{{version}}/Illuminate/Config/Repository.html) | `config`         |
| Cookie               | [Illuminate\Cookie\CookieJar](https://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html) | `cookie`         |
| Crypt                | [Illuminate\Encryption\Encrypter](https://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html) | `encrypter`      |
| DB                   | [Illuminate\Database\DatabaseManager](https://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html) | `db`             |
| DB (Instance)        | [Illuminate\Database\Connection](https://laravel.com/api/{{version}}/Illuminate/Database/Connection.html) | &nbsp;           |
| Event                | [Illuminate\Events\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html) | `events`         |
| File                 | [Illuminate\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html) | `files`          |
| Gate                 | [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html) | &nbsp;           |
| Hash                 | [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html) | `hash`           |
| Lang                 | [Illuminate\Translation\Translator](https://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html) | `translator`     |
| Log                  | [Illuminate\Log\Writer](https://laravel.com/api/{{version}}/Illuminate/Log/Writer.html) | `log`            |
| Mail                 | [Illuminate\Mail\Mailer](https://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html) | `mailer`         |
| Notification         | [Illuminate\Notifications\ChannelManager](https://laravel.com/api/{{version}}/Illuminate/Notifications/ChannelManager.html) | &nbsp;           |
| Password             | [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html) | `auth.password`  |
| Queue                | [Illuminate\Queue\QueueManager](https://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html) | `queue`          |
| Queue (Instance)     | [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html) | `queue`          |
| Queue (Base Class)   | [Illuminate\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html) | &nbsp;           |
| Redirect             | [Illuminate\Routing\Redirector](https://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html) | `redirect`       |
| Redis                | [Illuminate\Redis\Database](https://laravel.com/api/{{version}}/Illuminate/Redis/Database.html) | `redis`          |
| Request              | [Illuminate\Http\Request](https://laravel.com/api/{{version}}/Illuminate/Http/Request.html) | `request`        |
| Response             | [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html) | &nbsp;           |
| Route                | [Illuminate\Routing\Router](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) | `router`         |
| Schema               | [Illuminate\Database\Schema\Blueprint](https://laravel.com/api/{{version}}/Illuminate/Database/Schema/Blueprint.html) | &nbsp;           |
| Session              | [Illuminate\Session\SessionManager](https://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html) | `session`        |
| Session (Instance)   | [Illuminate\Session\Store](https://laravel.com/api/{{version}}/Illuminate/Session/Store.html) | &nbsp;           |
| Storage              | [Illuminate\Contracts\Filesystem\Factory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Factory.html) | `filesystem`     |
| URL                  | [Illuminate\Routing\UrlGenerator](https://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html) | `url`            |
| Validator            | [Illuminate\Validation\Factory](https://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html) | `validator`      |
| Validator (Instance) | [Illuminate\Validation\Validator](https://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html) | &nbsp;           |
| View                 | [Illuminate\View\Factory](https://laravel.com/api/{{version}}/Illuminate/View/Factory.html) | `view`           |
| View (Instance)      | [Illuminate\View\View](https://laravel.com/api/{{version}}/Illuminate/View/View.html) | &nbsp;           |

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@clayidols](http://blog.clayidols.com) | <img class="avatar-66 rm-style" src="https://avatars2.githubusercontent.com/u/21217903?v=4&s=460"> | Review | 痴呆哥哥 |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  Review  | Stay Hungry. Stay Foolish. |


---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
