# Laravel 数据库之：数据库请求构建器

- [简介](#introduction)
- [获取结果](#retrieving-results)
    - [分块结果](#chunking-results)
    - [聚合](#aggregates)
- [Selects](#selects)
- [原始表达式](#raw-expressions)
- [Joins](#joins)
- [Unions](#unions)
- [Where 子句](#where-clauses)
    - [参数分组](#parameter-grouping)
    - [Where Exists 语法](#where-exists-clauses)
    - [JSON 查询语句](#json-where-clauses)
- [Ordering, Grouping, Limit, & Offset](#ordering-grouping-limit-and-offset)
- [条件语句](#conditional-clauses)
- [Inserts](#inserts)
- [Updates](#updates)
    - [更新 JSON](#updating-json-columns)
    - [自增或自减](#increment-and-decrement)
- [Deletes](#deletes)
- [悲观锁](#pessimistic-locking)

<a name="introduction"></a>
## 简介

Laravel 的数据库查询构造器提供了一个方便、流畅的接口，用来创建及运行数据库查询语句。它能用来执行应用程序中的大部分数据库操作，且能在所有被支持的数据库系统中使用。

Laravel 的查询构造器使用 PDO 参数绑定，来保护你的应用程序免受 SQL 注入的攻击。在绑定传入字符串前不需要清理它们。

<a name="retrieving-results"></a>
## 获取结果

#### 从数据表中获取所有的数据列

你可以使用 `DB` facade 的 `table` 方法开始查询。这个 `table` 方法针对查询表返回一个查询构造器实例，允许你在查询时链式调用更多约束，并使用 `get` 方法获取最终结果：

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
            $users = DB::table('users')->get();

            return view('user.index', ['users' => $users]);
        }
    }

`get` 方法会返回一个 `Illuminate\Support\Collection` 结果，其中每个结果都是一个 PHP `StdClass` 对象的实例。您可以通过访问列中对象的属性访问每个列的值：

    foreach ($users as $user) {
        echo $user->name;
    }

#### 从数据表中获取单个列或行

如果你只需要从数据表中获取一行数据，则可以使用 `first` 方法。这个方法将返回单个 `StdClass` 对象：

    $user = DB::table('users')->where('name', 'John')->first();

    echo $user->name;

如果你不需要一整行数据，则可以使用 `value` 方法来从单条记录中取出单个值。此方法将直接返回字段的值：

    $email = DB::table('users')->where('name', 'John')->value('email');

#### 获取一列的值

如果你想要获取一个包含单个字段值的集合，可以使用 `pluck` 方法。在下面的例子中，我们将取出 roles 表中 title 字段的集合：

    $titles = DB::table('roles')->pluck('title');

    foreach ($titles as $title) {
        echo $title;
    }

你也可以在返回的数组中指定自定义的键值字段：

    $roles = DB::table('roles')->pluck('title', 'name');

    foreach ($roles as $name => $title) {
        echo $title;
    }

<a name="chunking-results"></a>
### 结果分块

如果你需要操作数千条数据库记录，可以考虑使用 `chunk` 方法。这个方法每次只取出一小块结果，并会将每个块传递给一个 `闭包` 处理。这个方法对于编写数千条记录的 [Artisan 命令](/docs/{{version}}/artisan) 是非常有用的。例如，让我们把 `users` 表进行分块，每次操作 100 条数据：

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        foreach ($users as $user) {
            //
        }
    });

你可以从 `闭包` 中返回 `false`，以停止对后续分块的处理：

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        // Process the records...

        return false;
    });

<a name="aggregates"></a>
### 聚合

查询构造器也支持各种聚合方法，如 `count`、 `max`、 `min`、 `avg` 和 `sum`。你可以在创建查询后调用其中的任意一个方法：

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

当然，你也可以将这些方法结合其它子句来进行查询：

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

<a name="selects"></a>
## Selects

#### 指定一个 Select 子句

当然，你并不会总是想从数据表中选出所有的字段。这时可使用 `select` 方法自定义一个 `select` 子句来查询指定的字段：

    $users = DB::table('users')->select('name', 'email as user_email')->get();

`distinct` 方法允许你强制让查询返回不重复的结果：

    $users = DB::table('users')->distinct()->get();

如果你已有一个查询构造器实例，并且希望在现有的 select 子句中加入一个字段，则可以使用 `addSelect` 方法：

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

<a name="raw-expressions"></a>
## 原始表达式

有时候你可能需要在查询中使用原始表达式。这些表达式将会被当作字符串注入到查询中，所以要小心避免造成 SQL 注入攻击！要创建一个原始表达式，可以使用 `DB::raw` 方法：

    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();

<a name="joins"></a>
## Joins

#### Inner Join 语法

查询构造器也可以编写 join 语法。若要执行基本的「inner join」，你可以在查询构造器实例上使用 `join` 方法。传递给 `join` 方法的第一个参数是你要 join 数据表的名称，而其它参数则指定用来连接的字段约束。当然，如你所见，你可以在单个查找中连接多个数据表：

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

#### Left Join 语法

如果你想用「left join」来代替「inner join」，请使用 `leftJoin` 方法。`leftJoin` 方法与 `join` 方法有着相同的用法：

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

#### Cross Join 语法

使用 `crossJoin` 方法和你想要交叉连接的表名来做「交叉连接」。交叉连接通过第一个表和连接表生成一个笛卡尔积：

    $users = DB::table('sizes')
                ->crossJoin('colours')
                ->get();

#### 高级 Join 语法

你还可以指定更高级的 join 子句。让我们传递一个`闭包`作为 `join` 方法的第二个参数来作为开始。此`闭包`将会收到一个 `JoinClause` 对象，让你可以在 `join` 子句中指定约束：

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();

如果你想要在连接中使用「where」风格的子句，则可以在连接中使用 `where` 和 `orWhere` 方法。这些方法将会比较值和对应的字段，而不是比较两个字段：

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

<a name="unions"></a>
## Unions

查询构造器也提供了一个快捷的方法来「合并」 两个查询。例如，你可以先创建一个初始查询，并使用 `union` 方法将它与第二个查询进行合并：

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

> {tip} 也可使用 `unionAll` 方法，它和 `union` 方法有着相同的用法。

<a name="where-clauses"></a>
## Where 子句

#### 简单的 Where 子句

你可以在查询构造器实例中使用 `where` 方法从而把 `where` 子句加入到这个查询中。基本的 `where` 方法需要3个参数。第一个参数是字段的名称。第二个参数是运算符，它可以是数据库所支持的任何运算符。最后，第三个参数是要对字段进行评估的值。

例如，这是一个要验证「votes」字段的值等于 100 的查询：

    $users = DB::table('users')->where('votes', '=', 100)->get();

为方便起见，如果你只是想简单的校验某个字段等于一个指定的值，你可以直接将这个值作为第二个参数传入 `where` 方法：

    $users = DB::table('users')->where('votes', 100)->get();

当然，在编写 `where` 子句时，你也可以使用各种数据库所支持其它的运算符：

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

你也可以通过一个条件数组做 `where` 的查询：

    $users = DB::table('users')->where([
        ['status', '=', '1'],
        ['subscribed', '<>', '1'],
    ])->get();

#### Or 语法

你可以在查询中加入 `or` 子句和 where 链式一起来约束查询。`orWhere` 方法接收和 `where` 方法相同的参数：

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

#### 其它 Where 子句

**whereBetween**

`whereBetween` 方法用来验证字段的值介于两个值之间：

    $users = DB::table('users')
                        ->whereBetween('votes', [1, 100])->get();

**whereNotBetween**

`whereNotBetween` 方法验证字段的值 **不** 在两个值之间：

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();

**whereIn 与 whereNotIn**

`whereIn` 方法验证字段的值包含在指定的数组内：

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

`whereNotIn` 方法验证字段的值 **不** 包含在指定的数组内：

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

**whereNull 与 whereNotNull**

`whereNull` 方法验证字段的值为 `NULL`：

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();

`whereNotNull` 方法验证字段的值 **不** 为 `NULL`：

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();

**whereDate / whereMonth / whereDay / whereYear**

`whereDate` 方法比较某字段的值与指定的日期是否相等：

    $users = DB::table('users')
                    ->whereDate('created_at', '2016-12-31')
                    ->get();

`whereMonth` 方法比较某字段的值是否与一年的某一个月份相等：

    $users = DB::table('users')
                    ->whereMonth('created_at', '12')
                    ->get();

`whereDay` 方法比较某列的值是否与一月中的某一天相等：

    $users = DB::table('users')
                    ->whereDay('created_at', '31')
                    ->get();

`whereYear` 方法比较某列的值是否与指定的年份相等：

    $users = DB::table('users')
                    ->whereYear('created_at', '2016')
                    ->get();

**whereColumn**

 `whereColumn` 方法用来检测两个列的数据是否一致：

    $users = DB::table('users')
                    ->whereColumn('first_name', 'last_name')
                    ->get();

此方法还可以使用运算符：

    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at')
                    ->get();

`whereColumn` 方法可以接收数组参数。条件语句会使用 `and` 连接起来：

    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', '=', 'last_name'],
                        ['updated_at', '>', 'created_at']
                    ])->get();

<a name="parameter-grouping"></a>
### 参数分组

有时你可能需要创建更高级的 where 子句，例如「where exists」或者嵌套的参数分组。Laravel 的查询构造器也能够处理这些。让我们先来看一个在括号中将约束分组的示例：

    DB::table('users')
                ->where('name', '=', 'John')
                ->orWhere(function ($query) {
                    $query->where('votes', '>', 100)
                          ->where('title', '<>', 'Admin');
                })
                ->get();

如你所见，上面例子会传递一个 `闭包` 到 `orWhere` 方法，告诉查询构造器开始一个约束分组。此 `闭包` 接收一个查询构造器实例，你可用它来设置应包含在括号分组内的约束。这个例子会生成以下 SQL：

    select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

<a name="where-exists-clauses"></a>
### Where Exists 语法

`whereExists` 方法允许你编写 `where exists` SQL 子句。此方法会接收一个 `闭包` 参数，此闭包接收一个查询语句构造器实例，让你可以定义应放在「exists」SQL 子句中的查找：

    DB::table('users')
                ->whereExists(function ($query) {
                    $query->select(DB::raw(1))
                          ->from('orders')
                          ->whereRaw('orders.user_id = users.id');
                })
                ->get();

上述查询将生成以下 SQL：

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )

<a name="json-where-clauses"></a>
### JSON 查询语句

Laravel 也支持查询 JSON 类型的字段。目前，本特性仅支持 MySQL 5.7+ 和 Postgres数据库。可以使用 `->` 运算符来查询 JSON 列数据：

    $users = DB::table('users')
                    ->where('options->language', 'en')
                    ->get();

    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();

<a name="ordering-grouping-limit-and-offset"></a>
## Ordering, Grouping, Limit 及 Offset

#### orderBy

`orderBy` 方法允许你根据指定字段对查询结果进行排序。`orderBy` 方法的第一个参数是你想要用来排序的字段，而第二个参数则控制排序的顺序，可以为 `asc` 或 `desc`：

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

#### latest / oldest

`latest` 和 `oldest` 方法允许你更容易的依据日期对查询结果排序。默认查询结果将依据 `created_at` 列。或者,你可以使用字段名称排序：

    $user = DB::table('users')
                    ->latest()
                    ->first();

#### inRandomOrder

`inRandomOrder` 方法可以将查询结果随机排序。例如，你可以使用这个方法获取一个随机用户：

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();

#### groupBy / having / havingRaw

`groupBy` 和 `having` 方法可用来对查询结果进行分组。`having` 方法的用法和 `where` 方法类似：

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

`havingRaw` 方法可以将一个原始的表达式设置为 `having` 子句的值。例如，我们能找出所有销售额超过 2,500 元的部门：

    $users = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > 2500')
                    ->get();

#### skip / take

 你可以使用 `skip` 和 `take` 方法来限制查询结果数量或略过指定数量的查询：

    $users = DB::table('users')->skip(10)->take(5)->get();

或者，你也可以使用 `limit` 和 `offset` 方法：

    $users = DB::table('users')
                    ->offset(10)
                    ->limit(5)
                    ->get();

<a name="conditional-clauses"></a>
## 条件语句

有时候，你希望某个值为 true 时才执行查询。例如，如果在传入请求中存在指定的输入值的时候才执行这个 `where` 语句。你可以使用 `when` 方法实现：

    $role = $request->input('role');

    $users = DB::table('users')
                    ->when($role, function ($query) use ($role) {
                        return $query->where('role_id', $role);
                    })
                    ->get();


只有当 `when` 方法的第一个参数为 `true` 时，闭包里的 `where` 语句才会执行。如果第一个参数是 `false`，这个闭包将不会被执行。

你可能会把另一个闭包当作第三个参数传递给 `when` 方法。如果第一个参数的值为 `false` 时，这个闭包将执行。为了说明如何使用此功能，我们将使用它配置默认排序的查询：

    $sortBy = null;

    $users = DB::table('users')
                    ->when($sortBy, function ($query) use ($sortBy) {
                        return $query->orderBy($sortBy);
                    }, function ($query) {
                        return $query->orderBy('name');
                    })
                    ->get();


<a name="inserts"></a>
## Inserts

查询构造器也提供了 `insert` 方法，用来插入记录到数据表中。`insert` 方法接收一个包含字段名和值的数组作为参数：

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );

你甚至可以在 `insert`  调用中传入一个嵌套数组向表中插入多条记录。每个数组表示要插入表中的行：

    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);

#### 自增 ID

若数据表存在自增 id，则可以使用 `insertGetId` 方法来插入记录并获取其 ID：

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> {note} 当使用 PostgreSQL 时，insertGetId 方法将预测自动递增字段的名称为 `id`。若你要从不同「顺序」来获取 ID，则可以将顺序名称作为第二个参数传递给 `insertGetId` 方法。

<a name="updates"></a>
## Updates

当然，除了在数据库中插入记录外，你也可以使用 `update` 来更新已存在的记录。`update` 方法和 `insert` 方法一样，接收含有字段及值的数组，其中包括要更新的字段。可以使用 `where` 子句来约束 `update` 查找：

    DB::table('users')
                ->where('id', 1)
                ->update(['votes' => 1]);

<a name="updating-json-columns"></a>
### 更新 JSON 列

当更新一个JSON 列时,你应该使用 `->` 语法来访问 JSON 对象的键。仅在数据库支持 JSON 列的时候才可使用这个操作：

    DB::table('users')
                ->where('id', 1)
                ->update(['options->enabled' => true]);

<a name="increment-and-decrement"></a>
### 自增或自减

查询构造器也为指定字段提供了便利的自增和自减方法 。此方法提供了一个比手动编写 `update` 语法更具表达力且更精练的接口。

这两个方法都必须接收至少一个参数（要修改的字段）。也可选择传入第二个参数，用来控制字段应递增／递减的量：

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

您还可以指定要操作中更新其它字段：

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

<a name="deletes"></a>
## Deletes

查询构造器也可使用 `delete` 方法从数据表中删除记录。在 `delete` 前，还可使用 `where` 子句来约束 `delete` 语法：

    DB::table('users')->delete();

    DB::table('users')->where('votes', '>', 100)->delete();

如果你需要清空表，你可以使用 `truncate` 方法，这将删除所有行，并重置自动递增 ID 为零：

    DB::table('users')->truncate();

<a name="pessimistic-locking"></a>
## 悲观锁

查询构造器也包含一些可以帮助你在 `select` 语法上实现「悲观锁定」的函数 。若要在查询中使用「共享锁」，可以使用 `sharedLock` 方法。共享锁可防止选中的数据列被篡改，直到事务被提交为止：

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

另外，你也可以使用 `lockForUpdate` 方法。使用「更新」锁可避免行被其它共享锁修改或选取：

    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@iwzh](https://github.com/iwzh) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/3762_1456807721.jpeg?imageView2/1/w/200/h/200"> |  翻译 | 码不能停 [@iwzh](https://github.com/iwzh) at Github  |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org