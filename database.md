# 数据库：入门

- [简介](#introduction)
- [运行原始 SQL 查找](#running-queries)
    - [监听查找事件](#listening-for-query-events)
- [数据库事务](#database-transactions)
- [使用多数据库连接](#accessing-connections)

<a name="introduction"></a>
## 简介

Laravel 对主流数据库系统连接和查询都提供了很好的支持，尤其是：流畅的 [查询语句构造器](/docs/{{version}}/queries) 和强大的 [Eloquent ORM](/docs/{{version}}/eloquent)。

目前，Laravel 支持以下四种数据库系统：

- MySQL
- Postgres
- SQLite
- SQL Server

> **[Summer](http://github.com/summerblue)：**  Mongo DB 的支持可以使用这个项目 - [laravel-mongodb](https://github.com/jenssegers/laravel-mongodb)

<a name="configuration"></a>
### 配置信息

Laravel 应用程序的数据库配置文件放置在 `config/database.php`。在这个配置文件内你可以定义所有的数据库连接，以及指定默认使用哪个连接。在此文件内提供了所有支持的数据库系统示例。

默认情况下，Laravel 的 [环境配置](/docs/{{version}}/installation#environment-configuration) 示例会使用 [Homestead](/docs/{{version}}/homestead)，对于 Laravel 开发来说这是一个相当便利的本地虚拟机。当然你也可以根据需求来随时修改本机端的数据库设置。

<a name="read-write-connections"></a>
#### 数据库读写分离

有时候你也许会希望使用一个数据库作为只读数据库，而另一个数据库则负责写入、更新以及删除。Laravel 让此类操作变得轻而易举，无论使用原始查找、查询语句构造器或是 Eloquent ORM 都可以适用。

如何设置读取与写入的连接，让我们看下这个例子：

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

注意，有两个键加入了这个配置文件数组内：`read` 及 `write`。这两个键内包含了数组，里面也包含了一个单独的键：`host`。`read` 及 `write` 连接的其它数据库设置选项将会合并在主要的 `mysql` 数组内。

所以，如果需要在主要的数组内重写值，只需在 `read` 及 `write` 数组内放置设置参数即可。在这个例子中，`192.168.1.1`将被使用在「读取」连接上，而 `192.168.1.2` 则被使用在「写入」连接上。数据库的凭证、前缀、编码设置，以及所有其它的选项都被存放在 `mysql` 数组内，这两个连接将会共用这些选项。

<a name="running-queries"></a>
## 运行原始 SQL 查找

一旦你设置好了数据库连接，就可以使用 `DB` facade 来进行查找。`DB` facade 提供每个类型的查找方法：`select`、`update`、`insert`、`delete`、`statement`。

#### 运行一个 Select 查找

在 `DB` facade 中使用 `select` 可以运行一个基本的查找：

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

传递给 `select` 方法的第一个参数是原始的 SQL 查找，而第二个参数是任何查找所需要的参数绑定。通常，这些都是 `where` 语句的限定值。参数绑定主要是为了防止 SQL 注入。

`select` 方法总会返回结果的`数组`数据。数组中的每个结果都是一个 PHP `StdClass` 对象，这使你能够访问到结果的值：

    foreach ($users as $user) {
        echo $user->name;
    }

#### 使用命名绑定

除了使用 `?` 来表示你的参数绑定外，你也可以使用命名绑定运行查找：

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

#### 运行 Insert

若要运行 `insert` 语法，则可以在 `DB` facade 使用 `insert` 方法。如同 `select` 一样，这个方法的第一个参数是原始的 SQL 查找，第二个参数则是绑定：

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

#### 运行 Update

`update` 方法用于更新已经存在于数据库的记录。该方法会返回此声明所影响的行数：

    $affected = DB::update('update users set votes = 100 where name = ?', ['John']);

#### 运行 Delete

`delete` 方法用于删除已经存在于数据库的记录。如同 `update` 一样，删除的行数将会被返回：

    $deleted = DB::delete('delete from users');

#### 运行一般声明

有时候一些数据库操作不应该返回任何参数。对于这种类型的操作，你可以在 `DB` facade 使用 `statement` 方法：

    DB::statement('drop table users');

<a name="listening-for-query-events"></a>
### 监听查找事件

如果你希望能够监控到程序执行的每一条 SQL 语句，则可以使用 `listen` 方法。这个方法对于纪录查找跟调试将非常有用。你可以在 [服务容器](/docs/{{version}}/providers) 中注册你的查找侦听器：

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
## 数据库事务

要想在数据库事务中运行一组操作，则可以在 `DB` facade 中使用 `transaction` 方法。如果在事务的`闭包`内抛出异常，事务将会被自动还原。如果`闭包`运行成功，事务将被自动提交。你不需要担心在使用 `transaction` 方法时还需要亲自去手动还原或提交事务：

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    });

#### 手动操作事务

如果你想手动处理事务并对还原或提交操作进行完全控制，则可以在 `DB` facade 使用 `beginTransaction` 方法：

    DB::beginTransaction();

你也可以通过 `rollBack` 方法来还原事务：

    DB::rollBack();

最后，可以通过 `commit` 方法来提交这个事务：

    DB::commit();

> **注意：** `DB` facade 的事务方法也可以用来控制 [查询语句构造器](/docs/{{version}}/queries) 及 [Eloquent ORM](/docs/{{version}}/eloquent) 的事务。

<a name="accessing-connections"></a>
## 使用多数据库连接

当你使用了多个连接时，则可以通过 `DB` facade 的 `connection` 方法来访问每个连接。传递给 `connection` 方法的 `name` 必须对应至 `config/database.php` 配置文件中的连接列表的其中一个：

    $users = DB::connection('foo')->select(...);

你也可以在连接的实例中使用 `getPdo` 方法访问原始的底层 PDO 实例：

    $pdo = DB::connection()->getPdo();


