# Session

- [简介](#introduction)
- [基本用法](#basic-usage)
    - [闪存数据](#flash-data)
- [增加 Session 驱动](#adding-custom-session-drivers)

<a name="introduction"></a>
## 简介

由于 HTTP 协定是无状态的，所以 session 提供一种保存用户数据的方法。Laravel 附带支持了多种 session 后端驱动，并通过统一的 API 进行使用。也内置支持像是 [Memcached](http://memcached.org)、[Redis](http://redis.io) 和数据库的后端驱动。

### 配置

Session 的配置文件配置在 `config/session.php`。请务必看一下此配置文件中可用的选项设置及注释。Laravel 默认使用 `file` 的 session 驱动，它在大多的应用中可以良好运作。在上线的应用程序中，你可能会考虑使用更快的 `memcached ` 或 `redis` 等驱动。

Session `driver` 定义了数据将由什么样的方式保存。Laravel 附带了几个不错且可立即使用的驱动：

<div class="content-list" markdown="1">
- `file` - 将 sessions 保存在 `storage/framework/sessions` 中。
- `cookie` - 将 sessions 安全的保存在加密后的 cookies 中。
- `database` - 将 sessions 保存在应用程序使用的数据库中。
- `memcached` / `redis` - 将 sessions 保存在其中一个快速并基于缓存的保存系统中。
- `array` - 将 sessions 保存在简单的 PHP 数组中，并只存活在当次请求。
</div>

> **注意：**数组驱动一般应使用在[测试](/docs/{{version}}/testing)环境下，以防止 session 数据持续存在。

### 驱动介绍

#### 数据库

当使用 `database` session 驱动时，你必需先构建保存 session 项目的数据表。以下例子使用 `Schema` 语法建表：

    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->text('payload');
        $table->integer('last_activity');
    });

你也可使用 `session:table` Artisan 命令生成迁移文件！

    php artisan session:table

    composer dump-autoload

    php artisan migrate

#### Redis

在 Laravel 使用 Redis Session 之前，你需要先通过 Composer 安装 `predis/predis`(~1.0) 扩展包。

### 其它 Session 注意事项

Laravel 框架在内部有使用 `flash` 作为 session 的键，所以应该避免 session 使用此名称。

如果你的 session 数据需要加密，在配置文件中的 `encrypt` 选项设为 `true`。

<a name="basic-usage"></a>
## 基本用法

#### 访问 Session

首先，让我们访问 session。我们可以在控制器方法内，通过对 HTTP 请求使用类型提示访问 session 实例。请记得，控制器方法的依赖是通过 Laravel 的[服务容器](/docs/{{version}}/container)注入：

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
        public function showProfile(Request $request, $id)
        {
            $value = $request->session()->get('key');

            //
        }
    }

当你从 session 取出值，你可以在 `get` 方法中的第二个参数内设置一个默认值。如果指定的键名不存在时，将会返回设置的默认值。如果你传入一个 `闭包` 作为 `get` 方法的默认值，该 `必包` 将被运行并返回它的结果：

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function() {
        return 'default';
    });

如果你想从 session 中获取所有数据，你可以使用 `all` 方法：

    $data = $request->session()->all();

你也可使用全局的 `session` PHP 函数获取 session 中保存的数据：

    Route::get('home', function () {
        // 获取 session 中的一条数据...
        $value = session('key');

        // 写入一条数据至 session 中...
        session(['key' => 'value']);
    });

#### 判断项目在 Session 中是否存在

`has` 方法被用于检查项目是否存在于 session 内。如果存在将会返回 `true`：

    if ($request->session()->has('users')) {
        //
    }

#### 保存数据到 Session 中

只要你可以访问 session 实例，你就可以调用多种函数调整里面的数据。例如，`put` 方法能将一个新的数据加入现有的 session 内。

    $request->session()->put('key', 'value');

#### 保存数据进 Session 数组值中

`push` 方法可以将一个新的值，加入至一个 session 的数组。例如，假设 `user.teams` 这个键是包含团队名称的数组，你可以将一个新的值加入此数组中：

    $request->session()->push('user.teams', 'developers');

#### 从 Session 取回数据，并且删除数据

`pull` 方法将把数据从 session 内取出，并且删除它：

    $value = $request->session()->pull('key', 'default');

#### 从 Session 中移除数据

`forget` 方法可以从 session 内删除一条数据。如果你想删除 session 内所有的数据，可以使用 `flush` 方法：

    $request->session()->forget('key');

    $request->session()->flush();

#### 重新产生 Session ID

如果你想重新产生 session ID，你可以使用 `regenerate` 方法：

    $request->session()->regenerate();

<a name="flash-data"></a>
### 闪存数据

有时候你想存入一条缓存的数据，让它只有在下一次的请求内有效。你可以使用 `flash` 方法达到这个目的。使用这个方法保存，只能将数据保留到下个 HTTP 请求，然后就会自动删除。闪存数据在短期的状态消息中很有用：

    $request->session()->flash('status', 'Task was successful!');

如果需要保留闪存数据给更多的请求，可以使用 `reflash` 方法，这将会保留所有的闪存数据给额外的请求。如果只想留一个特定的快内数据，你可以使用 `keep` 方法：

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

<a name="adding-custom-session-drivers"></a>
## 加入自定的 Session 驱动

要加入额外驱动至 Laravel 的 session 中，你可以使用 `Session` [facade](/docs/{{version}}/session) 的 `extend` 方法。你可以在[服务提供者](/docs/{{version}}/providers) 的 `boot` 方法内调用 `extend` 方法：

    <?php

    namespace App\Providers;

    use Session;
    use App\Extensions\MongoSessionStore;
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
            Session::extend('mongo', function($app) {
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

请注意，你的自定义 session 驱动必需实现 `SessionHandlerInterface`。这个接口包含了一些需要实现的方法。一个基本的 MongoDB 实现看起来像：

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

由于这些方法就像缓存的 `StoreInterface` 一样并不是那么容易理解，让我们快速的了解每个方法的做用：

<div class="content-list" markdown="1">
- `open` 方法通常用在基于文件的 session 保存系统中。像 Larvel 附带了一个 `file` 的驱动，所以你不用把任何东西放到这个方法内。你可以把这方法看做是空的也没关系；很简单的事实只是因为不佳的接口设计（我们将在之后讨论），所以 PHP 要求必需要有这个方法的实现。
- `close` 方法跟 `open` 方法很像，通常也是被忽略，对大多数的驱动而言，这并不是需要的。
- `read` 方法必须根据给予的 `$sessionId` 返回关联的 session 数据的字符串版本。在驱动中这并不需要做任何的编码跟序列化的动作，在 Laravel 内已自动运行。
- `write` 方法必须在大部分保存系统内写入 `$data` 字符串时关联至 `$sessionId`，如 MongoDB、Dynamo 等等。
- `destroy` 方法能删除与 `$sessionId` 关联的数据。
- `gc` 方法能删除给予 `$lifetime` 之前的所有数据，`$lifetime` 是一个 UNIX 的时间戳。但在一些系统如 Memcached 和 Redis 这个方法可能会留下一段空白的保存数据。
</div>

一但 session 驱动被注册后，你必须在 `config/session.php` 的配置文件内使用 `mongo` 驱动。
