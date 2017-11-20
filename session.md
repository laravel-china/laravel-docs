# Laravel 的 HTTP 会话机制

- [简介](#introduction)
    - [配置](#configuration)
    - [驱动之前](#driver-prerequisites)
- [使用 Session](#using-the-session)
    - [获取数据](#retrieving-data)
    - [存储数据](#storing-data)
    - [闪存数据](#flash-data)
    - [删除数据](#deleting-data)
    - [重新生成 Session ID](#regenerating-the-session-id)
- [添加自定义 Session 驱动](#adding-custom-session-drivers)
    - [实现驱动](#implementing-the-driver)
    - [注册驱动](#registering-the-driver)

<a name="introduction"></a>
## 简介

由于 HTTP 驱动的应用程序是无状态的，Session 提供了一种在多个请求之间存储有关用户的信息的方法。Laravel 通过同一个可读性强的 API 处理各种自带的 Session 后台驱动程序。支持诸如比较热门的 [Memcached](https://memcached.org)、[Redis](http://redis.io) 和开箱即用的数据库等常见的后台驱动程序。

<a name="configuration"></a>
### 配置

Session 的配置文件存储在 `config/session.php`。请务必查看此文件中对于你可用的选项。默认情况下，Laravel 配置了适用于大多数应用程序的 `file` Session 驱动。在生产环境下，你可以考虑使用 `memcached` 或 `redis` 驱动来实现更出色的 Session 性能。

Session `driver` 的配置选项定义了每个请求存储 Session 数据的位置。Laravel 自带了几个不错且可开箱即用的驱动：

<div class="content-list" markdown="1">
- `file` - 将 Session 保存在 `storage/framework/sessions` 中。

- `cookie` - Session 保存在安全加密的 Cookie 中。

- `database` - Session 保存在关系型数据库中。

- `memcached` / `redis` - Sessions 保存在其中一个快速且基于缓存的存储系统中。

- `array` - Sessions 保存在 PHP 数组中，不会被持久化。

  </div>

> {tip} 数组驱动一般用于 [测试](/docs/{{version}}/testing)，并防止存储在 Session 中的数据被持久化。

<a name="driver-prerequisites"></a>
### 驱动之前

#### 数据库

使用 `database` 作为 Session 驱动时，你需要创建一张包含 Session 各项数据的表。以下例子是使用 `Schema` 建表：

    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->integer('user_id')->nullable();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity');
    });

使用 Artisan 命令 `session:table` 命令为此生成迁移：

    php artisan session:table

    php artisan migrate

#### Redis

Laravel 在使用 Redis 作为 Session 驱动之前，需要通过 Composer 安装 `predis/predis` 扩展包 (~1.0)。然后在 `database` 配置文件中配置 Redis 连接信息。在 `session` 配置文件中，`connection` 选项可用于指定 Session 使用哪个 Redis 连接。

<a name="using-the-session"></a>
## 使用 Session

<a name="retrieving-data"></a>
### 获取数据

Laravel 中处理 Session 数据有两种主要方法：全局辅助函数 `session` 和通过一个 `Request` 实例。首先，我们来看看通过控制器方法类型提示一个 `Request` 实例来访问 session。控制器方法依赖项会通过 Laravel [服务容器](/docs/{{version}}/container) 自动注入：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 展示给定用户的配置文件
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

当你从 Session 获取值时，你还可以传递一个默认值作为 `get` 方法的第二个参数。如果 Session 中不存在指定的键，便会返回这个默认值。若传递一个闭包作为 `get` 方法的默认值，并且所请求的键并不存在时，`get` 方法将执行闭包并返回其结果：

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function () {
        return 'default';
    });

#### 全局辅助函数 Session

你也可以使用全局的 PHP 函数 `session` 来获取和存储 Session 数据。 使用单个字符串类型的值作为参数调用辅助函数 `session` 时，它会返回字该符串对应的 Session 键的值。当使用一个键值对数组作为参数调用辅助函数 `session` 时，传入的键值将会存储在 Session 中：

    Route::get('home', function () {
        // 获取 Session 中的一条数据...
        $value = session('key');

        // 指定一个默认值...
        $value = session('key', 'default');

        // 在 Session 中存储一条数据...
        session(['key' => 'value']);
    });

> {tip} 通过 HTTP 请求实例操作 Session 与使用全局辅助函数 `session` 两者之间并没有实质上的区别。这两种方法都可以通过所有测试用例中可用的 `assertSessionHas` 方法进行 [测试](/docs/{{version}}/testing)。

#### 获取所有 Session 数据

如果你想要获取所有的 Session 数据，可以使用 `all` 方法：

    $data = $request->session()->all();

#### 判断 Session 中是否存在某个值

要确定 Session 中是否存在某个值，可以使用 `has` 方法。如果该值存在且不为 `null`，那么 `has` 方法会返回 `true`：

    if ($request->session()->has('users')) {
        //
    }

要确定 Session 中是否存在某个值，即使其值为 `null`，也可以使用 `exists` 方法。如果值存在，则 `exists` 方法返回 `true`：

    if ($request->session()->exists('users')) {
        //
    }

<a name="storing-data"></a>
### 存储数据

要存储数据到 Session，你可以使用 `put` 方法，或者辅助函数 `session`。

    // 通过 HTTP 请求实例...
    $request->session()->put('key', 'value');

    // 通过全局辅助函数
    session(['key' => 'value']);

#### 在 Session 数组中保存数据

`push` 方法可以将一个新的值添加到 Session 数组内。例如，假设 `user.teams` 这个键是包含团队名称的数组，你可以像这样将一个新的值加入到此数组中：

    $request->session()->push('user.teams', 'developers');

#### 检索 & 删除

`pull` 方法可以只用一条语句就从 Session 检索并且删除一个项目：

    $value = $request->session()->pull('key', 'default');

<a name="flash-data"></a>
### 闪存数据

有时候你仅想在下一个请求之前在 Session 中存入数据，你可以使用 `flash` 方法。使用这个方法保存在 session 中的数据，只会保留到下个 HTTP 请求到来之前，然后就会被删除。闪存数据主要用于短期的状态消息：

    $request->session()->flash('status', 'Task was successful!');

如果需要保留闪存数据给更多请求，可以使用 `reflash` 方法，这将会将所有的闪存数据保留给其他请求。如果只想保留特定的闪存数据，则可以使用 `keep` 方法：

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

<a name="deleting-data"></a>
### 删除数据

`forget` 方法可以从 Session 内删除一条数据。如果你想删除 Session 内所有数据，可以使用 `flush` 方法：

    $request->session()->forget('key');

    $request->session()->flush();

<a name="regenerating-the-session-id"></a>
### 重新生成 Session ID

重新生成 Session ID，通常是为了防止恶意用户利用 [session fixation](https://en.wikipedia.org/wiki/Session_fixation) 对应用进行攻击。

如果使用了内置函数 `LoginController`，Laravel 会自动重新生成身份验证中 Session ID。否则，你需要手动使用 `regenerate` 方法重新生成 Session ID。

    $request->session()->regenerate();

<a name="adding-custom-session-drivers"></a>
## 添加自定义 Session 驱动

<a name="implementing-the-driver"></a>
#### 实现驱动

你自定义的 Session 驱动必须实现 `SessionHandlerInterface` 接口。这个接口包含了一些我们需要实现的简单方法。下面是一个大概的 MongoDB 实现流程示例：

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

> {tip} Laravel 默认没有附带扩展用的目录，你可以把它放在你喜欢的目录内。在下面这个例子中，我们创建了一个 `Extensions` 目录放置自定义的 `MongoHandler` 扩展。

接口中的这些方法不太容易理解。让我们来快速了解每个方法的作用：

<div class="content-list" markdown="1">
- `open` 方法通常用于基于文件的 Session 存储系统。因为 Laravel 已经附带了一个 `file` 的驱动，所以你不需要在该方法中放置任何代码。PHP 要求必需要有这个方法的实现（这只是一个糟糕的接口设计），你只需要把这个方法置空。
- `close` 方法跟 `open` 方法很相似，通常也可以被忽略。对大多数的驱动而言，此方法不是必须的。
- `read` 方法应当返回与给定的 `$sessionId` 相匹配的 Session 数据的字符串格式。在你的自定义的驱动中获取或存储 Session 数据时，不需要进行任何序列化或其它编码，因为 Laravel 会执行序列化。
- `write` 将与 `$sessionId` 关联的给定的 `$data` 字符串写入到一些持久化存储系统，如 MongoDB、Dynamo 等。再次重申，你不需要进行任何序列化或其它编码，因为 Laravel 会自动处理这些事情。
- `destroy` 方法会从持久化存储中移除与 `$sessionId` 相关联的数据。
- `gc` 方法能销毁给定的 `$lifetime` （UNIX 的时间戳）之前的所有数据。对本身拥有过期机制的系统如 Memcached 和 Redis 而言，该方法可以置空。
  </div>

<a name="registering-the-driver"></a>
#### 注册驱动

你的 Session 驱动实现之后，你还需要在框架中注册该驱动，即将该扩展驱动添加到 Laravel Session 后台。然后在 [服务提供者](/docs/{{version}}/providers) 的 `boot` 方法内调用 `Session` Facade 的 `extend` 方法。之后你就可以从现有的 `AppServiceProvider` 或者新创建的提供器中执行此操作。

    <?php

    namespace App\Providers;

    use App\Extensions\MongoSessionStore;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * 执行注册后引导服务。
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
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | Review | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
