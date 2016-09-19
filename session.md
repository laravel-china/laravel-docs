# HTTP Session

- [简介](#introduction)
    - [配置](#configuration)
    - [Session 驱动介绍](#driver-prerequisites)
- [使用 Session](#using-the-session)
    - [获取 Session 数据](#retrieving-data)
    - [存储数据到 Session](#storing-data)
    - [闪存数据到 Session](#flash-data)
    - [从 Session 中移除数据](#deleting-data)
    - [重新生成 Session ID](#regenerating-the-session-id)
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

你也可使用 `session:table` Artisan 命令生成迁移文件：

    php artisan session:table

    php artisan migrate

#### Redis

在 Laravel 使用 Redis Session 之前，你需要先通过 Composer 安装 `predis/predis`(~1.0) 扩展包。

你可能还需要在 `database` 配置文件中配置连接参数，其中的 `connection` 选项允许你指定使用哪个 Redis 来连接。

<a name="using-the-session"></a>
## 使用 Session

<a name="retrieving-data"></a>
### 获取 Session 数据

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

#### 获取所有 Session 数据

如果你想从 session 中获取所有数据，则可以使用 `all` 方法：

    $data = $request->session()->all();

#### 判断项目在 Session 中是否存在

使用 `has` 方法检查某个值是否存在于 session 内，如果该值存在，并且不为`null`，那么则返回 `true`：

    if ($request->session()->has('users')) {
        //
    }

在判断值是否在 Session 中是否存时，如果该值可能为 `null`，你需要使用 `exists` 方法，如果该值存在，那么则返回 `true`：

    if ($request->session()->exists('users')) {
        //
    }

<a name="storing-data"></a>
### 存储数据到 Session

存储数据到 Session，你可用使用 `put` 方法，或者 `session` 辅助函数。

    // 通过 HTTP 请求存储 Session ...
    $request->session()->put('key', 'value');

    // 通过全局辅助函数存储 Session ...
    session(['key' => 'value']);

#### 保存数据进 Session 数组值中

`push` 方法可以将一个新的值加入至一个 session 数组内。例如，假设 `user.teams` 这个键是包含团队名称的数组，你可以将一个新的值加入此数组中。比如这样：

    $request->session()->push('user.teams', 'developers');

#### 从 Session 中取出并删除数据

`pull` 方法将把数据从 session 内取出，并且删除它：

    $value = $request->session()->pull('key', 'default');

<a name="flash-data"></a>
### 闪存数据到 Session

有时候你想存入一条缓存的数据，让它只在下一次的请求内有效，则可以使用 `flash` 方法。使用这个方法保存 session，只能将数据保留到下个 HTTP 请求，然后就会被自动删除。闪存数据在短期的状态消息中很有用：

    $request->session()->flash('status', 'Task was successful!');

如果需要保留闪存数据给更多请求，可以使用 `reflash` 方法，这将会将所有的闪存数据保留给额外的请求。如果想保留特定的闪存数据，则可以使用 `keep` 方法：

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

<a name="deleting-data"></a>
### 从 Session 中移除数据

`forget` 方法可以从 session 内删除一条数据。如果你想删除 session 内所有数据，则可以使用 `flush` 方法：

    $request->session()->forget('key');

    $request->session()->flush();

<a name="regenerating-the-session-id"></a>
### 重新生成 Session ID

重新生成 Session ID，通常时为了防止恶意用户利用 session 对应用进行攻击。 via [session fixation](https://en.wikipedia.org/wiki/Session_fixation)

如果你使用了 `LoginController` 方法，那么 Laravel 会自动重新生成 Session ID，否则，你需要手动使用 `regenerate` 方法重新生成 session ID

    $request->session()->regenerate();

<a name="adding-custom-session-drivers"></a>
## 添加自定义的 Session 驱动

<a name="implementing-the-driver"></a>
#### 实现驱动

你自定义的 session 驱动必须实现 `SessionHandlerInterface` 接口。这个接口包含了一些基本需要实现的方法。一个基本的 MongoDB 实现应该看起来像这样：

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

> {tip} 并不要求你将扩展放在指定目录，你可以把它放在你喜欢的任何地方。在下面这个例子中，我们为这个 `扩展` 创建了一个 ` mongohandler `目录。 

接口中的这些方法不太容易容易理解。让我们来快速了解每个方法的作用：

<div class="content-list" markdown="1">

- `open` 方法通常用在基于文件的 session 存储系统中。像 Larvel 就附带了一个 `file` 的驱动，所以你不用把任何东西放到这个方法内。你可以把这方法看做是空的也没关系。主要是因为其接口设计不佳（我们将在后面讨论），所以 PHP 要求必需要有这个方法的实现。
- `close` 方法跟 `open` 方法很相似，通常也被忽略了。对大多数的驱动而言，此方法并不是需要的。
- `read` 方法必须根据给予的 `$sessionId` 返回关联的 session 数据的字符串版本。这在驱动中并不需要做任何的编码跟序列化动作，在 Laravel 内已经会自动运行。
- `write` 方法必须在大部分存储系统内写入 `$data` 字符串时关联至 `$sessionId`，如 MongoDB、Dynamo 等等。
- `destroy` 方法能删除与 `$sessionId` 相关联的数据。
- `gc` 方法能删除 `$lifetime` 之前的所有数据，`$lifetime` 是一个 UNIX 的时间戳。但在一些如 Memcached 和 Redis 这样的系统中，使用这个方法可能会留下一段空白的存储数据。

</div>

<a name="registering-the-driver"></a>
#### 注册驱动

在 session 驱动实现了 `SessionHandlerInterface` 接口后，你还需要在框架中注册该驱动，将该扩展驱动添加到 Laravel s session 后端。

你可以使用 `Session` [facade](/docs/{{version}}/facades)的 `extend` 方法。在 [服务提供者](/docs/{{version}}/providers) 的 `boot` 方法内调用 `extend` 方法。

你可用使用已经存在的 `AppServiceProvider` 或者创建一个新的提供者。

    <?php

    namespace App\Providers;

    use App\Extensions\MongoSessionStore;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         *  提供注册后运行的服务。
         *
         * @return void
         */
        public function boot()
        {
            Session::extend('mongo', function($app) {
                // 返回 SessionHandlerInterface 的实现...
                return new MongoSessionStore;
            });
        }

        /**
         * 在容器中注册绑定。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

一旦 session 驱动被注册，则必须在 `config/session.php` 的配置文件内使用 `mongo` 驱动。

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [WangYan](http://blog.wangyan.org)  | <img class="avatar-66 rm-style" src="http://imgcdn.wangyan.org/a/120x120.jpg">  |  翻译  | [About Me](http://blog.wangyan.org/about) |