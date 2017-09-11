# 数据库：入门

- [简介](#introduction)
    - [配置信息](#configuration)
    - [数据库读写链接](#read-and-write-connections)
    - [使用多数据库连接](#using-multiple-database-connections)
- [运行原生 SQL 语句](#running-queries)
    - [监听查询事件](#listening-for-query-events)
- [数据库事务](#database-transactions)

<a name="introduction"></a>

## 简介

Laravel 通过使用原始 SQL 与数据库的各种数据库进行交互, 非常简单。尤其流畅的使用 [查询语句构造器](/docs/{{version}}/queries)，和 [Eloquent ORM](/docs/{{version}}/eloquent)。当前，Laravel 支持四种类型的数据库:

<div class="content-list" markdown="1">

- MySQL
- Postgres
- SQLite
- SQL Server
</div>

<a name="configuration"></a>

### 配置信息

Laravel 应用程序的数据库配置文件放置在 `config/database.php` 文件中。在这个文件中，您可以定义所有的数据库连接，并指定默认使用哪个连接. 在此文件内提供了大多数支持的数据库系统示例。 

默认情况下，Laravel 的[环境配置](/docs/{{version}}/configuration#environment-configuration) 示例会使用 [Laravel Homestead](/docs/{{version}}/homestead)，这是一种方便的虚拟机，用于在本地机器上进行 Laravel 的开发。当然，您可以根据本地数据库的需要随意修改这个配置。

#### SQLite 配置

使用 `touch database/database.sqlite` 命令创建一个新的 SQLite 文件, 您可以通过使用数据库的绝对路径，轻松地配置环境变量，并指向这个新创建的数据库:

    DB_CONNECTION=sqlite
    DB_DATABASE=/absolute/path/to/database.sqlite

#### SQL Server 配置

Laravel 支持 SQL Server 数据库; 无论以何种方式, 您都需要将数据库的连接配置添加到您的 `config/database.php` 配置文件中:

    'sqlsrv' => [
        'driver' => 'sqlsrv',
        'host' => env('DB_HOST', 'localhost'),
        'database' => env('DB_DATABASE', 'forge'),
        'username' => env('DB_USERNAME', 'forge'),
        'password' => env('DB_PASSWORD', ''),
        'charset' => 'utf8',
        'prefix' => '',
    ],

<a name="read-and-write-connections"></a>

### 读&写的分离

有时您可能希望使用数据库的一个连接，只用于 SELECT ，另一个用于 INSERT, UPDATE, 和 DELETE 。 在 Laravel 中无论你使用的是原始查询，查询语句构造器，还是 Eloquent ORM 你都能很轻松的实现.

如何配置读/写连接，让我们看一下这个示例:

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
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
        'prefix'    => '',
    ],

注意，在上面的示例中，配置数组中添加了两个键 : `read` 和 `write` 。这两个键都包含了一个数组，键的值为: `host` 。`read` 和 `write` 连接的其余配置都在 `mysql` 这个主数组里面。

你只需要把项目放在 `read` 和 `write` 数组中，除非你想要覆盖主数组的值。所以，在这种情况下，`192.168.1.1` 将用作「读」连接的主机，而 `192.168.1.2` 将用于「写」连接。这两个连接会共享在 `mysql` 主数组中的配置。如：数据库的凭证，前缀，字符集，以及其他的选项。

<a name="using-multiple-database-connections"></a>

### 使用多个数据库连接

当使用多个连接时，您可以使用 `DB` facade 的 `connection` 方法。
通过 `config/database.php` 配置信息文件中定义好的数据库连接，
将 `name` 做为 `connection` 这个方法的参数传递进去 ：

    $users = DB::connection('foo')->select(...);

您还可以使用 `getPdo` 方法访问原始的PDO实例 ：

    $pdo = DB::connection()->getPdo();

<a name="running-queries"></a>

## 运行原生的 SQL 语句

配置好数据库连接后，可以使用 `DB` facade 运行查询。`DB` facade 为每种类型的查询提供了方法：`select`，`update`，`insert`，`delete` 和 `statement` 。
#### 运行 Select

运行一个基础的查询语句，你可以使用 `DB` facade 的 `select` 方法:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 查询应用中被激活的所有用户列表
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::select('select * from users where active = ?', [1]);

            return view('user.index', ['users' => $users]);
        }
    }

传递到 `select` 方法的第一个参数是一个原生的 SQL 查询，而第二个参数则是传递的所有绑定到查询中的参数值。通常，这些是 `where` 子句约束的值。参数绑定提供了对 SQL 注入的保护。

`select` 方法将始终返回一个数组结果集。数组中的每个结果将是一个PHP `StdClass` 对象，可以像下面这样访问结果值:

    foreach ($users as $user) {
        echo $user->name;
    }

#### 使用命名绑定

除了使用 `?` 来表示参数绑定外，你也可以使用命名绑定运行查找：

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

#### 运行 Insert

要执行 `insert` 语句，您可以在 `DB` facade 上使用 `insert` 方法。与select一样，该方法将原始 SQL 查询作为其第一个参数和绑定作为第二个参数：

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

#### 运行 Update

 `update` 方法用于更新数据库中的已存在的记录。该方法会返回此语句执行所影响的行数：

    $affected = DB::update('update users set votes = 100 where name = ?', ['John']);

#### 运行 Delete

`delete`方法用于从数据库中删除记录。与 `update` 一样，受影响的行数将被返回：

    $deleted = DB::delete('delete from users');

#### 运行一般声明

一些数据库语句不返回任何值。对于这些类型的操作，您可以在 `DB` facade 上使用 `statement` 方法：

    DB::statement('drop table users');

<a name="listening-for-query-events"></a>

### 查询事件的监听

如果你希望能够监控到程序执行的每一条 SQL 语句，那么你可以使用 `listen` 方法。这个方法对于记录查询或调试非常有用。您可以将查询侦听器注册到一个 [服务提供者](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 启动应用服务。
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
         * 注册服务提供者。
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

您可以在 `DB` facade 上使用 `transaction` 方法，在数据库事务中运行一组操作。如果在事务 `Closure` 中抛出一个异常，那么事务将自动回滚。如果 `Closure` 成功执行，事务将自动被提交。您不需要担心在使用事务方法时手动回滚或提交。

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    });

#### 处理死锁

`transaction` 方法接受一个可选的第二个参数，该参数定义在发生死锁时，应该重新尝试事务的次数。一旦这些尝试都用尽了，就会抛出一个异常：

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    }, 5);

#### 手动操作事务

如果您想要手工开始一个事务，并且对回滚和提交有完全的控制，那么您可以在 `DB` facade 上使用 `beginTransaction` 方法：

    DB::beginTransaction();

您可以通过 `rollBack` 方法回滚事务：

    DB::rollBack();

最后, 您可以通过 `commit` 方法提交事务：

    DB::commit();

> {tip}  使用 `DB` facade 的事务方法也适用于 [查询语句构造器](/docs/{{version}}/queries) and [Eloquent ORM](/docs/{{version}}/eloquent)。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@孤雪飘寒](https://laravel-china.org/users/15752)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/15752_1493141445.jpeg">  |  翻译  | 全桟工程师，[Github](https://github.com/piaohan)，[CSDN](http://blog.csdn.net/msmile_my)|

--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org