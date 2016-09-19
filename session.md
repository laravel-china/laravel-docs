# HTTP Session

- [简介](#introduction)
    - [配置](#configuration)
    - [session 驱动介绍](#driver-prerequisites)
- [使用 Session](#using-the-session)
    - [检索数据](#retrieving-data)
    - [存储数据](#storing-data)
    - [闪存数据](#flash-data)
    - [删除数据](#deleting-data)
    - [回收 Session ID](#regenerating-the-session-id)
- [添加自定义 Session 驱动](#adding-custom-session-drivers)
    - [实现驱动](#implementing-the-driver)
    - [注册驱动](#registering-the-driver)

<a name="introduction"></a>
## 简介

由于 HTTP 协议是无状态的，所以 session 提供了一种存储用户多个请求数据的方法。Laravel 附带支持了多种 session 后端驱动，并通过统一的 API 进行使用。也内置支持 [Memcached](http://memcached.org)、[Redis](http://redis.io) 和数据库这些热门的后端驱动。

<a name="configuration"></a>
### 配置

Session 的配置文件在 `config/session.php`。请确认一下此配置文件中可用的设置选项。Laravel 默认使用 `file` 的 session 驱动，它在大多数应用中可以良好运作。在上线的应用程序中，你可能会考虑使用更快的 `memcached ` 或 `redis` 来驱动。

Session `driver` 定义每次请求的 session 数据，会使用怎样的方式进行存储。Laravel 附带了几个不错且可开箱即用的驱动：

<div class="content-list" markdown="1">
- `file` - 将 sessions 保存在 `storage/framework/sessions` 中。
- `cookie` - 将 sessions 安全的保存在加密后的 cookies 中。
- `database` - 将 sessions 保存在应用程序使用的数据库中。
- `memcached` / `redis` - 将 sessions 保存在其中一个快速且基于缓存的存储系统中。
- `array` - 将 sessions 保存在简单的 PHP 数组中，并只存在于本次请求。
</div>

> {tip} 数组驱动一般应在 [测试](/docs/{{version}}/testing) 环境下使用，以防止存储在 session 中的数据持续存在。

--

> 译者注：推荐使用 Redis 来做 Session 驱动。Session 和缓存一起使用 Redis 的话，还需要多余的配置，请参考 - [Laravel 下配置 Redis 让缓存、Session 各自使用不同的 Redis 数据库](https://phphub.org/topics/2466)

<a name="driver-prerequisites"></a>
### session 驱动介绍

#### 数据库

当使用 `database` session 驱动时，你必需先构建保存 session 项目的数据表。以下例子使用 `Schema` 语法建表：

    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->integer('user_id')->nullable();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity');
    });

你也可使用 `session:table` Artisan 命令生成迁移文件！

    php artisan session:table

    php artisan migrate

#### Redis

在 Laravel 使用 Redis Session 之前，你需要先通过 Composer 安装 `predis/predis`(~1.0) 扩展包。

你可能还需要在 `database` 配置文件中配置连接参数，其中的 `connection` 选项允许你指定使用哪个 Redis 来连接。

<a name="using-the-session"></a>
## 使用 Session

<a name="retrieving-data"></a>
### 访问 Session 数据

在 Laravel 中有两种方式使用 session 数据，一是使用全局 `session` 辅助函数，二是通过 HTTP 请求实例。

首先我们看一下第二种方式，在控制器方法内，我们是通过 HTTP 请求的类型提示，访问 session 的。请记住，控制器方法的依赖是通过 Laravel 的 [服务容器](/docs/{{version}}/container) 注入的：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 显示指定用户的个人文件。
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function show(Request $request, $id)
        {
            $value = $request->session()->get('key');

            //
        }
    }

当你需要获取 session 是的某个值时，你可以在 `get` 方法中的第二个参数内设置一个默认值，当指定的键名不存在时，将会返回设置的默认值。如果你传入一个 `闭包` 作为 `get` 方法的默认值，该 `闭包` 将被运行并返回它的结果：

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function() {
        return 'default';
    });

#### 全局 Session 辅助函数

你也可使用全局的 `session` PHP 函数来获取 session 中保存的数据。

当调用 `session` 辅助函数的`行参`是单一字符串时，则返回该 session key 的值。

当调用 `session` 辅助函数的`行参`是数组时，则按 'key / value' 这种格式，将数据存储到 session 中。

    Route::get('home', function () {
        // 获取 session 中的一条数据...
        $value = session('key');

        // 指定一个默认值...
        $value = session('key', 'default');

        // 存储一条数据至 session 中...
        session(['key' => 'value']);
    });

> {tip} 使用全局 `session` 辅助函数与通过 HTTP 请求实例来使用 `session` 两者实际效果差异不大。你可以通过 `assertSessionHas` 方法来进行测试。关于测试的更多信息，请阅读文档 [testable](/docs/{{version}}/testing)。

#### Retrieving All Session Data

If you would like to retrieve all the data in the session, you may use the `all` method:

    $data = $request->session()->all();

#### Determining If An Item Exists In The Session

To determine if a value is present in the session, you may use the `has` method. The `has` method returns `true` if the value is present and is not `null`:

    if ($request->session()->has('users')) {
        //
    }

To determine if a value is present in the session, even if its value is `null`, you may use the `exists` method. The `exists` method returns `true` if the value is present:

    if ($request->session()->exists('users')) {
        //
    }

<a name="storing-data"></a>
### Storing Data

To store data in the session, you will typically use the `put` method or the `session` helper:

    // Via a request instance...
    $request->session()->put('key', 'value');

    // Via the global helper...
    session(['key' => 'value']);

#### Pushing To Array Session Values

The `push` method may be used to push a new value onto a session value that is an array. For example, if the `user.teams` key contains an array of team names, you may push a new value onto the array like so:

    $request->session()->push('user.teams', 'developers');

#### Retrieving & Deleting An Item

The `pull` method will retrieve and delete an item from the session in a single statement:

    $value = $request->session()->pull('key', 'default');

<a name="flash-data"></a>
### Flash Data

Sometimes you may wish to store items in the session only for the next request. You may do so using the `flash` method. Data stored in the session using this method will only be available during the subsequent HTTP request, and then will be deleted. Flash data is primarily useful for short-lived status messages:

    $request->session()->flash('status', 'Task was successful!');

If you need to keep your flash data around for several requests, you may use the `reflash` method, which will keep all of the flash data for an additional request. If you only need to keep specific flash data, you may use the `keep` method:

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

<a name="deleting-data"></a>
### Deleting Data

The `forget` method will remove a piece of data from the session. If you would like to remove all data from the session, you may use the `flush` method:

    $request->session()->forget('key');

    $request->session()->flush();

<a name="regenerating-the-session-id"></a>
### Regenerating The Session ID

Regenerating the session ID is often done in order to prevent malicious users from exploiting a [session fixation](https://en.wikipedia.org/wiki/Session_fixation) attack on your application.

Laravel automatically regenerates the session ID during authentication if you are using the built-in `LoginController`; however, if you need to manually regenerate the session ID, you may use the `regenerate` method.

    $request->session()->regenerate();

<a name="adding-custom-session-drivers"></a>
## Adding Custom Session Drivers

<a name="implementing-the-driver"></a>
#### Implementing The Driver

Your custom session driver should implement the `SessionHandlerInterface`. This interface contains just a few simple methods we need to implement. A stubbed MongoDB implementation looks something like this:

    <?php

    namespace App\Extensions;

    class MongoHandler implements SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }

> {tip} Laravel does not ship with a directory to contain your extensions. You are free to place them anywhere you like. In this example, we have created an `Extensions` directory to house the `MongoHandler`.

Since the purpose of these methods is not readily understandable, let's quickly cover what each of the methods do:

<div class="content-list" markdown="1">
- The `open` method would typically be used in file based session store systems. Since Laravel ships with a `file` session driver, you will almost never need to put anything in this method. You can leave it as an empty stub. It is simply a fact of poor interface design (which we'll discuss later) that PHP requires us to implement this method.
- The `close` method, like the `open` method, can also usually be disregarded. For most drivers, it is not needed.
- The `read` method should return the string version of the session data associated with the given `$sessionId`. There is no need to do any serialization or other encoding when retrieving or storing session data in your driver, as Laravel will perform the serialization for you.
- The `write` method should write the given `$data` string associated with the `$sessionId` to some persistent storage system, such as MongoDB, Dynamo, etc.  Again, you should not perform any serialization - Laravel will have already handled that for you.
- The `destroy` method should remove the data associated with the `$sessionId` from persistent storage.
- The `gc` method should destroy all session data that is older than the given `$lifetime`, which is a UNIX timestamp. For self-expiring systems like Memcached and Redis, this method may be left empty.
</div>

<a name="registering-the-driver"></a>
#### Registering The Driver

Once your driver has been implemented, you are ready to register it with the framework. To add additional drivers to Laravel's session backend, you may use the `extend` method on the `Session` [facade](/docs/{{version}}/facades). You should call the `extend` method from the `boot` method of a [service provider](/docs/{{version}}/providers). You may do this from the existing `AppServiceProvider` or create an entirely new provider:

    <?php

    namespace App\Providers;

    use App\Extensions\MongoSessionStore;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Session::extend('mongo', function($app) {
                // Return implementation of SessionHandlerInterface...
                return new MongoSessionStore;
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Once the session driver has been registered, you may use the `mongo` driver in your `config/session.php` configuration file.
