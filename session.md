# Laravel 的 HTTP 会话机制

- [简介](#introduction)
    - [配置](#configuration)
    - [驱动条件](#driver-prerequisites)
- [使用 Session](#using-the-session)
    - [获取 Session 数据](#retrieving-data)
    - [存储 Session 数据](#storing-data)
    - [闪存数据到 Session](#flash-data)
    - [删除 Session 数据](#deleting-data)
    - [重新生成 Session ID](#regenerating-the-session-id)
- [添加自定义 Session 驱动](#adding-custom-session-drivers)
    - [实现驱动](#implementing-the-driver)
    - [注册驱动](#registering-the-driver)

<a name="introduction"></a>
## 简介

由于 HTTP 是无状态的，Session 提供了一种在多个请求之间存储有关用户信息的方法。Laravel 附带支持了多种 Session 后端驱动，它们都可以通过语义化统一的 API 访问。Laravel 本身支持比较热门的 Session 后端驱动，如 [Memcached](https://memcached.org)、[Redis](http://redis.io) 和数据库。

<a name="configuration"></a>
### 配置

Session 相关的配置文件存储在 `config/session.php`。请务必查看此文件中对于你可用的选项。默认设置下，Laravel 的配置是使用文件作为 Session 驱动，大多数情况下能够运行良好。在生产环境下，你可以考虑使用 `memcached` 或 `redis` 驱动来达到更出色的性能表现。

Session 配置的 `driver` 的选项定义了每次请求的 Session 数据的存储位置。Laravel 附带了几个不错且可开箱即用的驱动：

<div class="content-list" markdown="1">
- `file` - 将 Session 保存在 `storage/framework/sessions`。
- `cookie` - Session 保存在安全加密的 Cookie 中。
- `database` - Session 保存在关系型数据库。
- `memcached` / `redis` - 将 Sessions 保存在其中一个快速且基于缓存的存储系统中。
- `array` - 将 Sessions 保存在简单的 PHP 数组中，并只存在于本次请求.
</div>

> {tip} 数组驱动一般用于 [测试](/docs/{{version}}/testing) 防止存储在 Session 的数据被持久化。

<a name="driver-prerequisites"></a>
### 驱动条件

#### 数据库

使用 数据库 作为 Session 驱动时，你需要创建一张包含 Session 各项数据的表。以下例子是使用 `Schema` 建表：

    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->integer('user_id')->nullable();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity');
    });

也可以使用 `Artisan` 的 `session:table` 命令生成一个迁移文件：

    php artisan session:table

    php artisan migrate

#### Redis

在使用 Redis 作为 Session 驱动之前，你需要通过 Composer 安装 `predis/predis` 扩展包(~1.0)。你还需要在 `database` 配置文件中指定 Redis 连接参数信息。在 Session 配置文件中的 `connection` 选项中指定 Session 使用的 Redis 连接。

<a name="using-the-session"></a>
## 使用 Session

<a name="retrieving-data"></a>
### 获取 Session 数据

Laravel 中有两种主要的方式使用 Session 数据的方式：一种是全局的辅助函数 `session`，另一种是通过 HTTP 请求实例。首先，我们先看一下第二种方法，就是通过具有控制器方法类型提示的 HTTP 请求实例来访问 Session。请记住，控制器方法的依赖关系会通过 Laravel 的 [服务容器](/docs/{{version}}/container)自动注入：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 展示用户个人信息
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

当从 Session 获取值时，你也可以传递一个默认值作为 `get` 方法的第二个参数。如果 Session 中并不存在指定的键值便会返回传入的默认值。若传递一个闭包作为 `get` 方法的默认值且请求的键值并不存在时，此时 `get` 方法会返回这个闭包函数运行后的返回值：

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function () {
        return 'default';
    });

#### 全局 Session 辅助函数

你也可以使用全局的 PHP 函数 `session` 来获取和存储 Session 数据。 使用单个字符串类型的值作为参数调用 `session` 函数时，它将返回字该符串参数对应的 Session 键值。当使用一个 key / value 键值对数组作为参数调用 `session` 函数时，传入的键值将会存入 Session：

    Route::get('home', function () {
        // 获取 Session 中的一条数据...
        $value = session('key');

        // 指定一个默认值...
        $value = session('key', 'default');

        // 存储一条数据至 Session 中...
        session(['key' => 'value']);
    });

> {tip} HTTP 请求实例与 `Session` 全局辅助函数使用 Session 并没有实质上的区别。两种方法都是可以通过 `assertSessionHas` 方法 [测试](/docs/{{version}}/testing) ，`assertSessionHas` 方法在所有的测试用例都是可用的。关于测试的更多信息，请阅读文档 [测试](/docs/{{version}}/testing)

#### 获取所有 Session 数据

如果你想要获取所有的 Session 数据，可以使用 `all` 方法：

    $data = $request->session()->all();

#### 判断某个 Session 值是否存在

使用 `has` 方法检查某个值是否存在于 Session 内，如果该值存在并且不为 null，那么则返回 true：

    if ($request->session()->has('users')) {
        //
    }

在判断值是否在 Session 中是否存时，如果该值可能为 null，你需要使用 exists 方法，如果该值存在，那么则返回 true：

    if ($request->session()->exists('users')) {
        //
    }

<a name="storing-data"></a>
### 存储 Session 数据

存储数据到 Session，你可用使用 `put` 方法，或者 `session` 辅助函数。

    // 通过 HTTP 请求实例...
    $request->session()->put('key', 'value');

    // 通过全局辅助函数
    session(['key' => 'value']);

#### 保存数据进 Session 数组值中

push 方法可以将一个新的值加入至一个 Session 数组内。例如，假设 user.teams 这个键是包含团队名称的数组，你可以将一个新的值加入此数组中。比如这样：

    $request->session()->push('user.teams', 'developers');

#### 从 Session 中取出并删除数据

`pull` 方法将把数据从 Session 内取出，并且删除：

    $value = $request->session()->pull('key', 'default');

<a name="flash-data"></a>
### 闪存数据到 Session

有时候你想存入一条缓存的数据，让它只在下一次的请求内有效，则可以使用 `flash` 方法。使用这个方法保存 session，只能将数据保留到下个 HTTP 请求，然后就会被自动删除。闪存数据在短期的状态消息中很有用：

    $request->session()->flash('status', 'Task was successful!');

如果需要保留闪存数据给更多请求，可以使用 `reflash` 方法，这将会将所有的闪存数据保留给额外的请求。如果想保留特定的闪存数据，则可以使用 `keep` 方法：

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

<a name="deleting-data"></a>
### 删除 Session 数据

`forget` 方法可以从 Session 内删除一条数据。如果你想删除 Session 内所有数据，则可以使用 `flush` 方法：

    $request->session()->forget('key');

    $request->session()->flush();

<a name="regenerating-the-session-id"></a>
### 重新生成 Session ID

重新生成 Session ID，通常时为了防止恶意用户利用 [session fixation](https://en.wikipedia.org/wiki/Session_fixation) 对应用进行攻击。

如果你使用了内置函数 `LoginController`，那么 Laravel 会自动重新生成 Session ID，否则，你需要手动使用 `regenerate` 方法重新生成 Session ID

    $request->session()->regenerate();

<a name="adding-custom-session-drivers"></a>
## 添加自定义 Session 驱动

<a name="implementing-the-driver"></a>
#### 实现驱动

你自定义的 Session 驱动必须实现 `SessionHandlerInterface` 接口。这个接口包含了一些基本需要实现的方法。一个基本的 MongoDB 实现应该看起来像这样：

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

> {tip} Laravel 默认没有附带扩展目录，你可以把它放在你喜欢的目录内。在下面这个例子中，我们创建了一个 `Extensions` 目录放置自定义的 `MongoHandler` 扩展。

接口中的这些方法不太容易容易理解。让我们来快速了解每个方法的作用：

<div class="content-list" markdown="1">
- `open` 方法通常用于基于文件的 Session 存储系统。因为 Larvel 已经附带了一个 `file` 的驱动，所以在该方法中不需要放置任何代码。PHP 要求必需要有这个方法的实现，但你可以把这方法置空也没关系。
- `close` 方法跟 `open` 方法很相似，通常也可以被忽略。对大多数的驱动而言，此方法并不是需要的。
- `read` 方法应当返回与给定的 `$sessionId` 相匹配的 Session 数据的字符串版本。从这个自定义的驱动中获取或存储 Session 数据不需要做任何序列化或其它编码，因为 Laravel 已经为我们做了序列化。
- `write` 将与 `$sessionId` 关联的特定 `$data` 字符串，写入到持久化存储系统，如 MongoDB、Dynamo 等等。再次重申，你不需要做任何序列化或其它编码，因为 Laravel 会自动处理这些事情。
- `destroy` 方法从持久化存储中移除 `$sessionId` 对应的数据。
- `gc` 方法能销毁 `$lifetime` 之前的所有数据，`$lifetime` 是一个 UNIX 的时间戳。对本身拥有过期机制的系统如 Memcached 和 Redis 而言，该方法可以留空。
</div>

<a name="registering-the-driver"></a>
#### 注册驱动

在 Session 驱动实现了 `SessionHandlerInterface` 接口后，你还需要在框架中注册该驱动，将该扩展驱动添加到 Laravel Session 后端。你可以使用 `Session` Facade 的 `extend` 方法。在 [服务提供者](/docs/{{version}}/providers) 的 `boot` 方法内调用 `extend` 方法。你可用使用已经存在的 `AppServiceProvider` 或者创建一个新的提供者。

    <?php

    namespace App\Providers;

    use App\Extensions\MongoSessionStore;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * 提供注册后运行的服务。
         *
         * @return void
         */
        public function boot()
        {
            Session::extend('mongo', function ($app) {
                // Return implementation of SessionHandlerInterface...
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

一旦 Session 驱动被注册，则必须在 `config/session.php` 的配置文件内使用 `Mongo` 驱动。


## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@wqer1019](https://laravel-china.org/users/5435)  | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/9254545?v=4&s=100">  |  翻译  | laravel是世界上最优雅的框架，[@wqer1019](https://github.com/wqer1019) at Github  |

--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org