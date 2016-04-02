# 数据库：查询建构器

- [简介](#introduction)
- [取得结果](#retrieving-results)
    - [聚合](#aggregates)
- [Selects](#selects)
- [Joins](#joins)
- [Unions](#unions)
- [Where 子句](#where-clauses)
    - [高端 Where 子句](#advanced-where-clauses)
- [Ordering、Grouping、Limit 及 Offset](#ordering-grouping-limit-and-offset)
- [Inserts](#inserts)
- [Updates](#updates)
- [Deletes](#deletes)
- [悲观锁定](#pessimistic-locking)

<a name="introduction"></a>
## 简介

数据库查询建构器提供方便、流畅的接口，用来创建及运行数据库查找。它可用来运行你的应用程序中大部分的数据库操作，且在所有支持的数据库系统中都能作用。

> **注意：**Laravel 的查询建构器使用 PDO 参数绑定，以保护你的应用程序不受数据库隐码攻击。传入字符串作为绑定前不需先清理它们。

<a name="retrieving-results"></a>
## 取得结果

#### 从数据表中取得所有的数据列

要开始进行流畅查找，在 `DB` facade 上使用 `table` 方法。`table` 方法会针对给定的数据表传回一个流畅查询建构器实例，允许你在查找上链式调用更多的约束，并于最后得到结果。在这个例子，让我们从一个数据表中`取得`所有的记录：

    <?php

    namespace App\Http\Controllers;

    use DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 显示应用程序的所有用户列表。
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->get();

            return view('user.index', ['users' => $users]);
        }
    }

就像[原始查找](/docs/{{version}}/database)，`get` 方法会返回一个结果`数组`，其中每一个结果都是 PHP `StdClass` 对象的实例。你可以将字段作为对象的性质，来访问每个字段的值：

    foreach ($users as $user) {
        echo $user->name;
    }

#### 从数据表中取得单一列或栏

若你只需从数据表中取出单一列，你可以使用 `first` 方法。这个方法会返回单一的 `StdClass` 对象：

    $user = DB::table('users')->where('name', 'John')->first();

    echo $user->name;

若你甚至不需完整的一列，你可以使用 `value` 方法来从一条记录中取出单一值。这个方法会直接传回字段的值：

    $email = DB::table('users')->where('name', 'John')->value('email');

#### 从数据表中将结果切块

若你需要操作数千笔数据库记录，可考虑使用 `chunk` 方法。这个方法一次取出一小「块」结果，并将每个区块喂给一个`闭包`处理。这个方法对编写要处理数千笔记录的 [Artisan 命令](/docs/{{version}}/artisan)非常有用。例如，让我们将整个 `user` 数据表切块，一次处理 100 笔记录：

    DB::table('users')->chunk(100, function($users) {
        foreach ($users as $user) {
            //
        }
    });

你可以从`闭包`中返回 `false`，以停止对后续切块的处理：

    DB::table('users')->chunk(100, function($users) {
        // 处理记录…

        return false;
    });

#### 取得字段值列表

若你想要取得一个包含单一字段值的数组，你可以使用 `lists` 方法。在这个例子中，我们将取出 role 数据表 title 字段的数组：

    $titles = DB::table('roles')->lists('title');

    foreach ($titles as $title) {
        echo $title;
    }

你也可以在传回的数组中指定自定的键值字段：

    $roles = DB::table('roles')->lists('title', 'name');

    foreach ($roles as $name => $title) {
        echo $title;
    }

<a name="aggregates"></a>
### 聚合

查询建构器也提供了各种聚合方法，例如 `count`、`max`、`min`、`avg`、以及 `sum`。你可以在创建你的查找后调用其中任何一个方法：

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

当然，你也可以将这些方法合并其他的子句来打造你的查找：

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

<a name="selects"></a>
## Selects

#### 指定一个 Select 子句

当然，你不会总是想要从数据表中选出所有的字段。你可以使用 `select` 方法为查找指定一个自定义的 `select` 子句：

    $users = DB::table('users')->select('name', 'email as user_email')->get();

`distinct` 方法允许你强制让查找传回不重复的结果：

    $users = DB::table('users')->distinct()->get();

若你已有一个查询建构器的实例，而你希望在其既存的 select 子句中加入一个字段，你可使用 `addSelect` 方法：

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

#### 原始表达式

有时你可能需要在查找中使用原始表达式。这些表达式会被当作字符串注入查找中，因此小心不要造成数据隐码攻击！要创建一个原始表达式，你可以使用 `DB::raw` 方法：

    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();

<a name="joins"></a>
## Joins

#### Inner Join 语法

查询建构器也可用来编写 join 语法。要操作基本的 SQL「inner join」，你可以在查询建构器实例上使用 `join` 方法。传入 `join` 方法的第一个参数是你需要连接的数据表名称，其他参数则指定用以连接的字段约束。当然，如你所见，你可以在单一查找连接多个数据表：

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

#### Left Join 语法

如果你想以操作「left join」来代替「inner join」，使用 `leftJoin` 方法。`leftJoin` 方法和 `join` 方法有相同的署名：

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

#### 高端 Join 语法

你也可以指定更高端的 join 子句。以传入一个`闭包`当作 `join` 方法的第二参数作为开始。此`闭包`会接收 `JoinClause` 对象，让你可以在 `join` 子句上指定约束：

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();

若你想要在连接中使用「where」风格的子句，你可以在连接中使用 `where` 和 `orWhere` 方法。这些方法会比较字段及一个值，来代替两个字段的比较：

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

<a name="unions"></a>
## Unions

查询语句构造器也提供一个快速的方法来「联合」两个查找。例如，你可以创建一个初始查找，然后使用 `union` 方法将它与第二个查找联合：

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

也可使用 `unionAll` 方法，它和 `union` 有相同的方法署名。

<a name="where-clauses"></a>
## Where 子句

#### 简易 Where 子句

要在查找中加入 `where` 子句，在查找建造者实例中使用 `where` 方法。最基本的 `where` 调用需要三个参数。第一个参数是字段的名称。第二个参数是一个运算符，它可以是数据库所支持的任何运算符。第三个参数是要对字段评估的值。

例如，这是一个要验证「votes」字段的值等于 100 的查找：

    $users = DB::table('users')->where('votes', '=', 100)->get();

为了方便起见，若你单纯只想验证某字段等于一个给定值，你可以直接将这个值作为第二个参数传入 `where` 方法：

    $users = DB::table('users')->where('votes', 100)->get();

当然，在编写 `where` 子句时，你可以使用各式其他的运算符：

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

#### Or 语法

你也可以在查找中加入 `or` 子句，将 where 约束链式调用在一起。`orWhere` 方法和 `where` 方法接受相同的参数：

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

#### 其他的 Where 子句

**whereBetween**

`whereBetween` 方法验证一个字段的值介于两个值之间：

    $users = DB::table('users')
                        ->whereBetween('votes', [1, 100])->get();

**whereNotBetween**

`whereNotBetween` 方法验证一个字段的值落在两个值之外：

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();

**whereIn 与 whereNotIn**

`whereIn` 方法验证给定字段的值包含在给定的数组之内：

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

`whereNotIn` 方法验证给定字段的值**不**包含在给定的数组之内：

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

**whereNull 与 whereNotNull**

`whereNull` 方法验证给定㯗位的值为 `NULL`：

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();

`whereNotNull` 方法验证一个字段的值**不**为 `NULL`：

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();

<a name="advanced-where-clauses"></a>
## 高端 Where 子句

#### 参数分组

有时你可能会需要创建更高端的 where 子句，例如「where exists」或者嵌套的参数分组。Laravel 的查询语句构造器也可处理这些。让我们看一个在括号中将约束分组的例子：

    DB::table('users')
                ->where('name', '=', 'John')
                ->orWhere(function ($query) {
                    $query->where('votes', '>', 100)
                          ->where('title', '<>', 'Admin');
                })
                ->get();

如同你看到的，将`闭包`传入 `orWhere` 方法，告诉查询语句构造器开始一个约束分组。此`闭包`接收查询语句构造器的实例，你可以用它来设置应包含在括号分组中的约束。上面的例子会产生以下的 SQL：

    select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

#### Exists 语法

`whereExists` 方法允许你编写 `where exists` SQL 子句。`whereExists` 方法接受一个`闭包`参数，它会接收查询语句构造器实例，让你可以定义应放在「exists」SQL 子句中的查找：

    DB::table('users')
                ->whereExists(function ($query) {
                    $query->select(DB::raw(1))
                          ->from('orders')
                          ->whereRaw('orders.user_id = users.id');
                })
                ->get();

上述的查找会产生以下的 SQL：

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )

<a name="ordering-grouping-limit-and-offset"></a>
## Ordering、Grouping、Limit 及 Offset

#### orderBy

`orderBy` 方法允许你针对给定的字段，将查找结果排序。`orderBy` 的第一个参数应为你要用来排序的字段，第二个参数则控制排序的方向，可以是 `asc` 或 `desc`：

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

#### groupBy、having 与 havingRaw

`groupBy` 和 `having` 方法可以用来将查找结果分组。`having` 方法的署名和 `where` 方法的类似：

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

`havingRaw` 方法可用来将原始字符串设置为 `having` 子句的值。例如，我们可以找出所有销售额大于 2,500 元的部门：

    $users = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > 2500')
                    ->get();

#### skip 与 take

要限制查找所返回的结果数量，或略过给定数量的查找结果（`偏移`），你可使用 `skip` 和 `take` 方法：

    $users = DB::table('users')->skip(10)->take(5)->get();

<a name="inserts"></a>
## Inserts

查询语句构造器也提供了 `insert` 方法，用来将记录插入数据表。`insert` 方法接受一个数组，包含要插入的字段名称及值：

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );

你甚至可以在一次的 `insert` 调用中，传入一个包含数组的数组，来插入数笔记录到数据表里。每个数组代表要插入数据表中的一列记录：

    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);

#### 自动递增 ID

若数据表有一自动递增的 id，使用 `insertGetId` 方法来插入记录并取得其 ID：

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

>**注意：**当使用 PostgreSQL 时，insertGetId 方法预期自动递增字段的名称为 `id`。若你要从不同的「次序」取得 ID，你可以将次序名称作为第二个参数传入 `insertGetId` 方法。

<a name="updates"></a>
## Updates

当然，除了在数据库中插入记录，也可使用 `update` 方法让查询语句构造器更新已存在的记录。`update` 方法和 `insert` 方法一样，接受含一对字段及值的数组，其中包含要被更新的字段。你可以使用 `where` 子句来约束 `update` 查找：

    DB::table('users')
                ->where('id', 1)
                ->update(['votes' => 1]);

#### 递增或递减

查询语句构造器也提供便利的方法来递增或递减给定字段的值。这只是个捷径，提供了一个较手动编写 `update` 语法更具表达力且精练的接口。

这两个方法都接受至少一个参数：要修改的字段。可选择性地传入第二个参数，用来控制字段应递增／递减的量。

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

你也可以指定其余要在操作中更新的字段：

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

<a name="deletes"></a>
## Deletes

当然，通过 `delete` 方法，查询语句构造器也可用来将记录从数据表中删除：

    DB::table('users')->delete();

在调用 `delete` 方法之前，你可加上 `where` 子句用来约束 `delete` 语法：

    DB::table('users')->where('votes', '<', 100)->delete();

若你希望截去整个数据表来移除所有数据列，并将自动递增 ID 重设为零，你可以使用 `truncate` 方法：

    DB::table('users')->truncate();

<a name="pessimistic-locking"></a>
## 悲观锁定

产询产生器也包含一些函数，用以协助你在 `select` 语法上作「悲观锁定」。要以「共享锁」来运行述句，你可以在查找上使用 `sharedLock` 方法。共享锁可避免选择的数据列被更改，直到你的交易提交为止：

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

此外，你可以使用 `lockForUpdate` 方法。「用以更新」锁可避免数据列被其他共享锁修改或选取：

    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
