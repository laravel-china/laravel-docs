# Facades

- [简介](#introduction)
- [使用 Facades](#using-facades)
- [Facade 类参考](#facade-class-reference)

<a name="introduction"></a>
## 简介

> 译者注：Facade 中文意为 - 门面，外观，包装器。专有名词的属性多一点，故不采用直译，而是直接做专有名词使用。

Facades 为应用程序的 [服务容器](/docs/{{version}}/container) 中可用的类提供了一个「静态」接口。Laravel 本身附带许多的 facades，甚至你可能在不知情的状况下已经在使用他们！Laravel 「facades」作为在服务容器内基类的「静态代理」，拥有简洁、易表达的语法优点，同时维持着比传统静态方法更高的可测试性和灵活性。

<a name="using-facades"></a>
## 使用 Facades

在 Laravel 应用程序环境（Context）中，facade 是个提供从容器访问对象的类。`Facade` 类是这个机制运作的核心部件。Laravel 的 facades，以及任何你创建的自定义 facades，会继承基底 `Illuminate\Support\Facades\Facade` 类。

facade 类只需要去实现一个方法：`getFacadeAccessor`。`getFacadeAccessor` 方法的工作定义是从容器中解析出什么。`Facade` 基类利用 `__callStatic()` 魔术方法从你的 facade 延迟调用来解析对象。

在下面的例子，调用了 Laravel 的缓存系统。看了一下这个代码，或许有人认为静态方法 `get` 是被 `Cache` 类调用的：

    <?php

    namespace App\Http\Controllers;

    use Cache;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 显示指定用户的个人数据。
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

注意在文件的上方，我们「导入」`Cache` facade。这个 facade 做为访问底层实现 `Illuminate\Contracts\Cache\Factory` 接口的代理。我们使用 facade 的任何调用将会发送给 Laravel 缓存服务的底层实例。

如果我们查看 `Illuminate\Support\Facades\Cache` 类，你会发现没有静态方法 `get`：

    class Cache extends Facade
    {
        /**
         * 获取组件的注册名称。
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }

相反的，`Cache` facade 继承了基底 `Facade` 类以及定义了 `getFacadeAccessor()` 方法。记住，这个方法的工作是返回服务容器绑定的名称。当用户在 `Cache` facade 上参考任何的静态方法，Laravel 会从 [服务容器](/docs/{{version}}/container) 解析被绑定的 `cache` 以及针对对象运行请求的方法（在这个例子中是 `get`）。

<a name="facade-class-reference"></a>
## Facade 类参考

在下方你可以找到每个 facade 及其底层的类。这个工具对于通过指定 facade 的来源快速寻找 API 文档相当有用。可应用的 [服务容器绑定](/docs/{{version}}/container) 关键字也包含在里面。

Facade  |  Class  |  Service Container Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](http://laravel-china.org/api/{{version}}/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Contracts\Console\Kernel](http://laravel-china.org/api/{{version}}/Illuminate/Contracts/Console/Kernel.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](http://laravel-china.org/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Auth\Guard](http://laravel-china.org/api/{{version}}/Illuminate/Auth/Guard.html)  |
Blade  |  [Illuminate\View\Compilers\BladeCompiler](http://laravel-china.org/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](http://laravel-china.org/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |
Cache  |  [Illuminate\Cache\Repository](http://laravel-china.org/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](http://laravel-china.org/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](http://laravel-china.org/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](http://laravel-china.org/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](http://laravel-china.org/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](http://laravel-china.org/api/{{version}}/Illuminate/Database/Connection.html)  |
Event  |  [Illuminate\Events\Dispatcher](http://laravel-china.org/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](http://laravel-china.org/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](http://laravel-china.org/api/5.1/Illuminate/Contracts/Auth/Access/Gate.html)  |
Hash  |  [Illuminate\Contracts\Hashing\Hasher](http://laravel-china.org/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Input  |  [Illuminate\Http\Request](http://laravel-china.org/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Lang  |  [Illuminate\Translation\Translator](http://laravel-china.org/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](http://laravel-china.org/api/{{version}}/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](http://laravel-china.org/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Password  |  [Illuminate\Auth\Passwords\PasswordBroker](http://laravel-china.org/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password`
Queue  |  [Illuminate\Queue\QueueManager](http://laravel-china.org/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance) |  [Illuminate\Queue\QueueInterface](http://laravel-china.org/api/{{version}}/Illuminate/Queue/QueueInterface.html)  |
Queue (Base Class) |  [Illuminate\Queue\Queue](http://laravel-china.org/api/{{version}}/Illuminate/Queue/Queue.html)  |
Redirect  |  [Illuminate\Routing\Redirector](http://laravel-china.org/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](http://laravel-china.org/api/{{version}}/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](http://laravel-china.org/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](http://laravel-china.org/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |
Route  |  [Illuminate\Routing\Router](http://laravel-china.org/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](http://laravel-china.org/api/{{version}}/Illuminate/Database/Schema/Blueprint.html)  |
Session  |  [Illuminate\Session\SessionManager](http://laravel-china.org/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](http://laravel-china.org/api/{{version}}/Illuminate/Session/Store.html)  |
Storage  |  [Illuminate\Contracts\Filesystem\Factory](http://laravel-china.org/api/{{version}}/Illuminate/Contracts/Filesystem/Factory.html)  |  `filesystem`
URL  |  [Illuminate\Routing\UrlGenerator](http://laravel-china.org/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](http://laravel-china.org/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](http://laravel-china.org/api/{{version}}/Illuminate/Validation/Validator.html) |
View  |  [Illuminate\View\Factory](http://laravel-china.org/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](http://laravel-china.org/api/{{version}}/Illuminate/View/View.html)  |


