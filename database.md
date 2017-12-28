# 数据库：入门

- [简介](#introduction)
    - [配置](#configuration)
    - [读 & 写连接](#read-and-write-connections)
    - [使用多个数据库连接](#using-multiple-database-connections)
- [运行原生 SQL 查询](#running-queries)
    - [查询事件监听](#listening-for-query-events)
- [数据库事务](#database-transactions)

<a name="introduction"></a>

## 简介

Laravel 能使用原生 SQL、[查询构造器](/docs/{{version}}/queries) 和 [Eloquent ORM](/docs/{{version}}/eloquent) 在各种数据库后台与数据库进行非常简单的交互。当前 Laravel 支持四种数据库:

<div class="content-list" markdown="1">

- MySQL
- Postgres
- SQLite
- SQL Server
  </div>

<a name="configuration"></a>

### 配置

数据库的配置文件放置在 `config/database.php` 文件中，你可以在此定义所有的数据库连接，并指定默认使用的连接。此文件内提供了大部分 Laravel 能支持的数据库配置示例。

默认情况下，Laravel  的示例 [环境配置](/docs/{{version}}/configuration#environment-configuration) 使用了 [Laravel Homestead](/docs/{{version}}/homestead)（这是一种小型虚拟机，能让你很方便地在本地进行 Laravel 的开发）。你可以根据本地数据库的需要修改这个配置。

#### SQLite 配置

使用类似 `touch database/database.sqlite` 之类命令创建一个新的 SQLite 数据库之后，可以使用数据库的绝对路径配置环境变量来指向这个新创建的数据库:

    DB_CONNECTION=sqlite
    DB_DATABASE=/absolute/path/to/database.sqlite

<a name="read-and-write-connections"></a>

### 读 & 写连接

如果你想使一个数据库连接只用于 SELECT ，另一个连接用于 INSERT、UPDATE、和 DELETE。那么在 Laravel 中无论你使用的是原生查询、查询构造器，还是 Eloquent ORM，你都能很轻松地实现这个需求。

看看下面这个例子如何配置读/写连接：

    'mysql' => [
        'read' => [
            'host' => '192.168.1.1',
        ],
        'write' => [
            'host' => '196.168.1.2'
        ],
        'sticky'    => true,
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
        'prefix'    => '',
    ],

要注意的是，上面的示例中的配置数组中添加了三个键 : `read` 和 `write` 和 `sticky`。`read` 和 `write` 键都包含了一个键的值为 `host` 的数组。`read` 和 `write` 连接的其余数据库配置都在 `mysql` 这个主数组里面。

如果你想要覆盖主数组中的值，则只需将配置的内容移入 `read` 和 `write` 数组中即可。因此，在这种情况下，`192.168.1.1` 将被用作「读取」连接的主机，而 `192.168.1.2` 将被用于「写入」连接。这两个连接会共享数据库的凭证、前缀、字符集，以及在主 `mysql` 数组中的选项。

**`sticky` 选项**

`sticky` 是一个 *可选的* 选项，它可用于立即读取在当前请求周期内已写入数据库的记录。

如果 `sticky` 选项被启用，并且在当前的请求周期内在数据库执行过「写入」操作，那么任何「读取」的操作都将使用「写入」连接。这可以确保在请求周期内写入的任何数据可以在同一请求期间立即从数据库读回。这个选项的作用取决于应用程序的需求。

<a name="using-multiple-database-connections"></a>

### 使用多个数据库连接

使用多个连接时，可以通过 `DB` facade 上的 `connection` 方法访问每个连接。传递给 `connection` 方法的 `name` 应该对应于 `config/database.php` 配置信息文件中列出的连接之一：

    $users = DB::connection('foo')->select(...);

你也可以在连接实例上使用 `getPdo` 方法访问底层的原生 PDO 实例 ：

    $pdo = DB::connection()->getPdo();

<a name="running-queries"></a>

## 运行原生 SQL 查询

配置好数据库连接后，可以使用 `DB` facade 运行查询。`DB` facade 为每种类型的查询提供了方法：`select`、`update`、`insert`、`delete` 和 `statement`。
#### 运行 Select 查询

运行基础的查询语句，你可以使用 `DB` facade 上使用 `select` 方法:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 显示所有应用程序用户的列表
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::select('select * from users where active = ?', [1]);

            return view('user.index', ['users' => $users]);
        }
    }

传递到 `select` 方法的第一个参数是一个原生的 SQL 查询，而第二个参数则是传递需要绑定到查询中的参数值。通常，这些是 `where` 子句约束的值。参数绑定提供了对防止 SQL 注入的保护。

`select` 方法将始终返回一个数组。数组中的每个结果都是一个PHP `StdClass` 对象，可以像下面这样访问结果的值:

    foreach ($users as $user) {
        echo $user->name;
    }

#### 使用命名绑定

除了使用 `?` 来表示参数绑定外，你也可以使用命名绑定来执行一个查询：

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

#### 运行插入语句

可以在 `DB` facade 上使用 `insert` 方法来执行 `insert` 语句。与 `select` 一样，该方法将原生 SQL 查询作为其第一个参数，并将其绑定的数据作为第二个参数：

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

#### 运行更新语句

 `update` 方法用于更新数据库中的现有记录。该方法会返回受该语句影响的行数：

    $affected = DB::update('update users set votes = 100 where name = ?', ['John']);

#### 运行删除语句

`delete` 方法用于删除数据库中记录。与 `update` 一样，会返回受该语句影响的行数：

    $deleted = DB::delete('delete from users');

#### 运行普通语句

有些数据库语句不会返回任何值。对于这些语句，可以在 `DB` facade 上使用 `statement` 方法来操作：

    DB::statement('drop table users');

<a name="listening-for-query-events"></a>

### 查询事件监听

如果你想监控程序执行的每个 SQL 查询，你可以使用 `listen` 方法。这个方法对于记录查询或调试非常有用。你可以在 [服务提供器](/docs/{{version}}/providers) 中为你的查询注册监听器:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 启动应用服务
         *
         * @return void
         */
        public function boot()
        {
            DB::listen(function ($query) {
                // $query->sql
                // $query->bindings
                // $query->time
            });
        }

        /**
         * 注册服务提供器
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="database-transactions"></a>

## 数据库事务

你可以在 `DB` facade 上使用 `transaction` 方法来运行数据库事务中的一组操作。如果在事务 `Closure` 中发生了异常，事务将自动回滚。而如果 `Closure` 成功执行，事务将自动被提交。也就是说，使用数据库事务，你就不需要在数据库语句执行发生异常时手动回滚或提交。

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    });

#### 处理死锁

传递第二个可选参数给 `transaction` 方法，该参数定义在发生死锁时应该重新尝试事务的次数。一旦尝试次数都用尽了，就会抛出一个异常：

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    }, 5);

#### 手动操作事务

如果你想要手动开始一个事务，并且能够完全控制回滚和提交，那么你可以在 `DB` facade 上使用 `beginTransaction` 方法：

    DB::beginTransaction();

可以通过 `rollBack` 方法回滚事务：

    DB::rollBack();

最后记得要通过 `commit` 方法提交事务：

    DB::commit();

> {tip}  `DB` facade 的事务方法也适用于 [查询语句构造器](/docs/{{version}}/queries) 和 [Eloquent ORM](/docs/{{version}}/eloquent) 的事务。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@孤雪飘寒](https://laravel-china.org/users/15752) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/15752_1493141445.jpeg"> | 翻译   | 全桟工程师，[Github](https://github.com/piaohan)，[CSDN](http://blog.csdn.net/msmile_my) |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | Review | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
