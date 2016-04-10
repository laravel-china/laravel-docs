# 数据库：入门

- [简介](#introduction)
- [运行原始 SQL 查找](#running-queries)
    - [监听查找事件](#listening-for-query-events)
- [数据库交易](#database-transactions)
- [使用多数据库连接](#accessing-connections)

<a name="introduction"></a>
## 简介

Laravel 对主流的数据库系统连接和查询都有很好的支持，无论是使用原始的 SQL、[流畅的查询语句构造器](/docs/{{version}}/queries)，或是强大的 [Eloquent ORM](/docs/{{version}}/eloquent)。

目前，Laravel 支持以下四种数据库系统：

- MySQL
- Postgres
- SQLite
- SQL Server

> [Summer](http://github.com/summerblue): Mongo DB 的支持可以使用这个项目 - [laravel-mongodb](https://github.com/jenssegers/laravel-mongodb)

<a name="configuration"></a>
### 配置信息

Laravel 应用程序的数据库配置文件放置在 `config/database.php`。在这个配置文件内你可以定义所有的数据库连接，以及指定默认使用哪个连接。在这个文件内提供了所有支持的数据库系统例子。

默认情况下，Laravel 的[环境配置](/docs/{{version}}/installation#environment-configuration)例子是使用 [Laravel Homestead](/docs/{{version}}/homestead)，在开发 Laravel 时，这是相当便利的本机虚拟机。当然，你可以因应需求随时修改你本机端的数据库设置。

<a name="read-write-connections"></a>
#### 数据库读写分离

有时候也许你会希望使用一个数据库作为只读，而另一个作为写入、更新以及删除。Laravel 使他变得轻而一举，无论你使用原始查找、查询语句构造器或是 Eloquent ORM 都是可以使用的。

如何设置读取与写入的连接，让我们看看这个例子：

    'mysql' => [
        'read' => [
            'host' => '192.168.1.1',
        ],
        'write' => [
            'host' => '196.168.1.2'
        ],
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset'   => 'utf8',
        'collation' => 'utf8_unicode_ci',
        'prefix'    => '',
    ],

注意，有两个键加入了这个配置文件数组内：`read` 及 `write`。这两个键内包含了数组，里面也包含了一个单独的键：`host`。`read` 及 `write` 连接的其他的数据库设置选项将会合并在主要的 `mysql` 数组内。

所以，如果需要在主要数组内重写值，只需要在 `read` 及 `write` 数组内放置设置参数。在这个例子中，`192.168.1.1`将被使用在「读取」连接，而 `192.168.1.2` 将被使用在「写入」连接。数据库的凭证、前缀、编码设置，以及所有其它的选项都存放在 `mysql` 数组内，两个连接将会共用这些选项。

<a name="running-queries"></a>
## 运行原始 SQL 查找

一旦你设置好了数据库连接，你可以使用 `DB` facade 进行查找。`DB` facade 提供每个类型的查找方法：`select`、`update`、`insert`、`delete`、`statement`。

#### 运行一个 Select 查找

运行一个基本查找，我们可以在 `DB` facade 使用 `select`：

    <?php

    namespace App\Http\Controllers;

    use DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 显示应用程序中所有用户的列表。
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::select('select * from users where active = ?', [1]);

            return view('user.index', ['users' => $users]);
        }
    }

传递给 `select` 方法的第一个参数是原始 SQL 查找，而第二个参数是任何查找需要的参数绑定。通常，这些是 `where` 子句的限定值。参数绑定提供了保护，为防止 SQL 注入。

`select` 方法总是返回结果的`数组`。数组中的每个结果将是一个 PHP `StdClass` 对象，让你能够访问结果的值：

    foreach ($users as $user) {
        echo $user->name;
    }

#### 使用命名绑定

除了使用 `?` 表示你的参数绑定外，你也可以使用命名绑定运行查找：

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

#### 运行 Insert

若要运行 `insert` 语法，你可以在 `DB` facade 使用 `insert` 方法。如同 `select`，这个方法第一个参数是使用原始 SQL 查找，第二个参数则是绑定：

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

#### 运行 Update

`update` 方法用于更新已经存在于数据库的记录。这个方法会返回该语法影响的行数：

    $affected = DB::update('update users set votes = 100 where name = ?', ['John']);

#### 运行 Delete

`delete` 方法用于删除已经存在于数据库的记录。如同 `update`，删除的行数将会被返回：

    $deleted = DB::delete('delete from users');

#### 运行一般语法

有时候一些数据库操作不应该返回任何参数。对于这种类型的操作，你可以在 `DB` facade 使用 `statement` 方法：

    DB::statement('drop table users');

<a name="listening-for-query-events"></a>
### 监听查找事件

如果你希望能够收到来自于你的应用程序每一条 SQL 查找，你可以使用 `listen` 方法。这个方法对于纪录查找跟调试非常有用。你可以在[服务容器](/docs/{{version}}/providers)注册你的查找侦听器：

    <?php

    namespace App\Providers;

    use DB;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 启动任何应用程序的服务。
         *
         * @return void
         */
        public function boot()
        {
            DB::listen(function($sql, $bindings, $time) {
                //
            });
        }

        /**
         * 注册一个服务提供者。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="database-transactions"></a>
## 数据库交易

要在数据库交易中运行一组操作，你可以在 `DB` facade 使用 `transaction` 方法。如果在交易的`闭包`内抛出异常，交易会自动的被还原。如果`闭包`运行成功，交易将自动提交。你不需要担心在使用 `transaction` 方法时手动还原或提交：

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    });

#### 手动操作交易

如果你想手动处理交易并且完全控制还原或提交，你可以在 `DB` facade 使用 `beginTransaction`：

    DB::beginTransaction();

你可以还原交易，通过 `rollBack` 方法：

    DB::rollBack();

最后，你可以提交这个交易，通过 `commit` 方法：

    DB::commit();

> **注意：** 使用 `DB` facade 的交易方法也可以控制[查询语句构造器](/docs/{{version}}/queries)及 [Eloquent ORM](/docs/{{version}}/eloquent) 的交易。

<a name="accessing-connections"></a>
## 使用多数据库连接

当你使用多个连接，你可以使用 `DB` facade 的 `connection` 方法访问每个连接。传递给 `connection` 方法的 `name` 必须对应至 `config/database.php` 配置文件里连接列表的其中一个：

    $users = DB::connection('foo')->select(...);

你也可以在连接的实例使用 `getPdo` 方法访问原始的底层 PDO 实例：

    $pdo = DB::connection()->getPdo();
