# 数据库：入门

- [简介](#introduction)
    - [配置信息](#configuration)
    - [数据库读写分离](#read-and-write-connections)
    - [使用多数据库连接](#using-multiple-database-connections)
- [运行原生 SQL 语句](#running-queries)
    - [监听查询事件](#listening-for-query-events)
- [数据库事务](#database-transactions)

<a name="introduction"></a>
## 简介

Laravel 对主流数据库系统连接和查询都提供了很好的支持，尤其是：流畅的 [查询语句构造器](/docs/{{version}}/queries) ， Laravel 支持四种类型的数据库。

- MySQL
- Postgres
- SQLite
- SQL Server

<a name="configuration"></a>
### 配置信息

Laravel 应用程序的数据库配置文件放置在 `config/database.php` 文件中。在这个配置文件内你可以定义所有的数据库连接，以及指定默认使用哪个连接。在此文件内提供了所有支持的数据库系统示例。

默认情况下，Laravel 的 [环境配置](/docs/{{version}}/installation#environment-configuration) 示例会使用 [Laravel Homestead](/docs/{{version}}/homestead)，对于 Laravel 开发来说这是一个相当便利的本地虚拟机。当然你也可以根据需求来随时修改本机端的数据库设置

#### SQLite 配置

请使用 `touch database/database.sqlite` 命令创建一个 SQLite 文件， 您可以通过使用数据库的绝对路径来轻松地配置您的环境变量来指向这个新创建的数据库：

    DB_CONNECTION=sqlite
    DB_DATABASE=/absolute/path/to/database.sqlite

#### SQL Server 配置

Laravel 支持 SQL Server 数据库，你需要在 `config/database.php` 中为连接 SQL Server 数据库做配置：

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
### 数据库读写分离

有时候你需要把数据库的读取和数据库的写入、更新或者删除分开使用，Laravel 提供了简单的方法，适用于原始查找、查询语句构造器或是 Eloquent ORM。 

通过下面这个例子，我们来学习如何配置数据库读写连接的分离：

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

注意，有两个键加入了这个配置文件数组内： `read` 和 `write` 。 它们两个都是一个数组且只包含了一个键： `host` 。 而 `read` 和 `write` 连接的其他配置都包含在 `mysql` 数组中。

所以，如果需要在主要的数组内重写值，只需在 `read` and `write` 数组内放置设置参数即可。在这个例子中， `192.168.1.1` 将会只提供数据读取数据的功能，而 `192.168.1.2` 提供数据库写入。数据库的凭证、前缀、编码设置，以及所有其它的选项都被存放在 `mysql` 数组内，这两个连接将会共用这些选项。

<a name="using-multiple-database-connections"></a>
### 使用多数据库连接

当使用多个数据连接时，你可以使用 `DB` facade 的 `connection` 方法。在 `config/database.php` 中定义好的数据库连接 `name` 作为 `connection` 的参数进行传递。

    $users = DB::connection('foo')->select(...);

你也可以在连接的实例中使用 `getPdo` 方法访问原始的底层 PDO 实例：

    $pdo = DB::connection()->getPdo();

<a name="running-queries"></a>
## 运行原生sql语句 

配置好数据库连接以后，你可以使用 `DB` facade 来执行查询。 `DB` facade 提供了 `select` 、 `update` 、 `insert` 、 `delete` 和 `statement` 的查询方法。

#### 运行 Select

运行一个基础的查询语句, 你可以使用 `DB` facade 的 `select` 方法：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::select('select * from users where active = ?', [1]);

            return view('user.index', ['users' => $users]);
        }
    }

传递到 `select` 方法的第一个参数是一个原生的 SQL 查询，,而第二个参数则是传递的所有绑定到查询中的参数值。通常，这些都是 `where` 字句约束中的值。参数绑定可以避免 SQL 注入攻击。

 `select` 方法以数组的形式返回结果集，数组中的每一个结果都是一个PHP `StdClass` 对象,你可以像下面这样访问结果值：

    foreach ($users as $user) {
        echo $user->name;
    }

#### 使用命名绑定

除了使用 `?` 来表示参数绑定外，你也可以使用命名绑定运行查找：

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

#### 运行 insert

运行 `Insert` 语句, 你可以是使用 `DB` facade 的 `insert` 方法 。 像 `select` 一样， 该方法将原生SQL语句作为第一个参数，将参数绑定作为第二个参数：:

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

#### 运行 Update

 `update` 方法用于更新已经存在于数据库的记录。该方法会返回此语句执行所影响的行数：

    $affected = DB::update('update users set votes = 100 where name = ?', ['John']);

#### 运行 Delete

 `delete` 方法用于删除已经存在于数据库的记录。如同 `update` 一样，删除的行数将会被返回。

    $deleted = DB::delete('delete from users');

#### 运行一般声明

有些数据库没有返回值， 对于这种类型的操作，可以使用 `DB` facade 的 `statement` 方法。

    DB::statement('drop table users');

<a name="listening-for-query-events"></a>
### 监听查询事件

如果你希望能够监控到程序执行的每一条 SQL 语句，那么你可以使用 `listen` 方法。 该方法对查询日志和调试非常有用， 你可以在 [服务容器](/docs/{{version}}/providers) 中注册该方法:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
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
         * Register the service provider.
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

想要在一个数据库事务中运行一连串操作，可以使用 `DB` facade 的 `transaction` 方法。如果在事务的 `Closure` 中抛出了异常，那么事务会自动的执行回滚操作。如果 `Closure` 成功的执行，那么事务就会自动的进行提交操作。你不需要在使用 `transaction` 方法时考虑手动执行回滚或者提交操作：

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    });

#### 手动操作事务

如果你想要手动开始一个事务的回滚和提交操作，你可以使用 `DB` facade 的 `beginTransaction` 方法。

    DB::beginTransaction();

你也可以通过 `rollBack` 方法来回滚事务：

    DB::rollBack();

最后，可以通过 `commit` 方法来提交这个事务：

    DB::commit();

> {tip} 使用 `DB` facade 的事务方法也适用于 [查询语句构造器](/docs/{{version}}/queries) 和 [Eloquent ORM](/docs/{{version}}/eloquent) 。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@kzh4435](https://phphub.org/users/5698)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5698_1473126483.jpeg?imageView2/1/w/100/h/100">  |  翻译  | 努力学习PHP  |
