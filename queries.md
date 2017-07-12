# 数据库：查询构造器

- [简介](#introduction)
- [获取结果](#retrieving-results)
    - [结果分块](#chunking-results)
    - [聚合](#aggregates)
- [Selects](#selects)
- [原始表达式](#raw-expressions)
- [Joins](#joins)
- [Unions](#unions)
- [Where 子句](#where-clauses)
    - [参数分组](#parameter-grouping)
    - [Where Exists 语法](#where-exists-clauses)
    - [JSON 查询语句](#json-where-clauses)
- [Ordering, Grouping, Limit, 及 Offset](#ordering-grouping-limit-and-offset)
- [条件语句](#conditional-clauses)
- [Inserts](#inserts)
- [Updates](#updates)
    - [更新 JSON](#updating-json-columns)
    - [递增或递减](#increment-and-decrement)
- [Deletes](#deletes)
- [悲观锁定](#pessimistic-locking)

<a name="introduction"></a>
## 简介

Laravel 的数据库查询构造器提供了方便、流畅的接口，以用来创建及运行数据库查询。可用来执行应用程序中的大部分数据库操作，且能在所有被支持的数据库系统中使用。

Laravel 的查询构造器使用 PDO 参数绑定，以保护你的应用程序不受数据库注入攻击。在传入字符串作为绑定前不需要先清理它们。

<a name="retrieving-results"></a>
## 获取结果

#### 从数据表中获取所有的数据列

若要开始进行查找，可在 `DB` facade 上使用 `table` 方法。`table` 方法会针对指定的数据表返回一个查询构造器实例，允许你在查询时链式调用更多约束，并使用 get 方法得到最终结果：

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

`get` 方法会返回一个 `Illuminate\Support\Collection` 结果，其中每一个结果都是 PHP `StdClass` 对象的实例。你可以将列作为对象的属性来访问每个列的值：

    foreach ($users as $user) {
        echo $user->name;
    }

#### 从数据表中获取单个列或行

若你只需从数据表中取出单行数据，则可以使用 `first` 方法。这个方法会返回单个 `StdClass` 对象：

    $user = DB::table('users')->where('name', 'John')->first();

    echo $user->name;

若你不想取出完整的一行，则可以使用 `value` 方法来从单条记录中取出单个值。这个方法会直接返回字段的值：

    $email = DB::table('users')->where('name', 'John')->value('email');

#### 获取一列的值

若你想要获取一个包含单个字段值的数组，你可以使用 `pluck` 方法。在这个例子中，我们将取出 roles 数据表 title 字段的数组：

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

若你需要操作数千条数据库记录，则可考虑使用 `chunk` 方法。这个方法一次只取出一小「块」结果，并会将每个区块传给一个`闭包`进行处理。这个方法对于要编写处理数千条记录的 [Artisan 命令](/docs/{{version}}/artisan) 非常有用。例如，让我们将整个 `users` 数据表进行分块，每次处理 100 条记录：

    DB::table('users')->orderBy('id')->chunk(100, function($users) {
        foreach ($users as $user) {
            //
        }
    });

你可以从`闭包`中返回 `false`，以停止对后续分块的处理：

    DB::table('users')->orderBy('id')->chunk(100, function($users) {
        // Process the records...

        return false;
    });

<a name="aggregates"></a>
### 聚合

查询构造器也提供了各种聚合方法，例如 `count`、`max`、`min`、`avg` 以及 `sum`。你可以在创建查找后调用其中的任意一个方法：

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

当然，你也可以将这些方法合并到其它的子句上来构建查找：

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

<a name="selects"></a>
## Selects

#### 指定一个 Select 子句

当然，你并不会总是想从数据表中选出所有的字段。这时可以使用 `select` 方法为查找指定一个自定义的 `select` 子句：

    $users = DB::table('users')->select('name', 'email as user_email')->get();

`distinct` 方法允许你强制让查找返回不重复的结果：

    $users = DB::table('users')->distinct()->get();

若你已有一个查询构造器实例，且希望在其现存的 select 子句中加入一个字段，则可以使用 `addSelect` 方法：

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

<a name="raw-expressions"></a>
## 原始表达式

有时你可能需要在查询中使用原始表达式。这些表达式会被当作字符串注入到查找中，因此要小心避免造成数据库注入攻击！要创建一个原始表达式，可以使用 `DB::raw` 方法：

    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();

<a name="joins"></a>
## Joins

#### Inner Join 语法

查询构造器也可用来编写 join 语法。要操作基本的 SQL「inner join」，则可以在查询构造器实例上使用 `join` 方法。传入 `join` 方法的第一个参数是你所需要连接的数据表名称，其它参数则指定用来连接的字段约束。如你所见，你可在单个查找中连接多个数据表：

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

#### Left Join 语法

如果你想以操作「left join」来代替「inner join」，请使用 `leftJoin` 方法。`leftJoin` 方法和 `join` 方法有着相同的签名：

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

#### Cross Join 语法

使用 `crossJoin` 方法和你想要交叉连接的表名来做「交叉连接」。交叉连接通过第一个表和连接表生成一个笛卡尔积：

    $users = DB::table('sizes')
                ->crossJoin('colours')
                ->get();

#### 高级的 Join 语法

你也可以指定更高级的 join 子句。让我们以传入一个`闭包`当作 `join` 方法的第二参数来作为开始。此`闭包`会接收 `JoinClause` 对象，让你可以在 `join` 子句上指定约束：

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();

若你想要在连接中使用「where」风格的子句，则可以在连接中使用 `where` 和 `orWhere` 方法。这些方法会比较字段和一个值，来代替两个字段的比较：

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

<a name="unions"></a>
## Unions

查询语句构造器也提供了一个快捷的方法来「合并」两个查找。例如，你可以先创建一个初始查找，然后使用 `union` 方法将它与第二个查找进行合并：

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

> {tip} 也可使用 `unionAll` 方法，它和 `union` 有着相同的方法签名。

<a name="where-clauses"></a>
## Where 子句

#### 简单的 Where 子句

你可以在查询语句构造器实例中使用 `where` 方法从而在查找中加入 `where` 子句。基本的 `where` 方法调用需要三个参数。第一个参数是字段的名称；第二个参数是一个运算符，它可以是数据库所支持的任何运算符；第三个参数是要对字段进行评估的值。

例如，这是一个要验证「votes」字段的值等于 100 的查找：

    $users = DB::table('users')->where('votes', '=', 100)->get();

为了方便起见，若你只想简单地验证某个字段等于一个指定的值，则可以直接将这个值作为第二个参数传入 `where` 方法：

    $users = DB::table('users')->where('votes', 100)->get();

当然，在编写 `where` 子句时，你也可以使用各种其它的运算符：

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

你也可以通过一个条件数组来做 `where` 的查询：

    $users = DB::table('users')->where([
        ['status', '=', '1'],
        ['subscribed', '<>', '1'],
    ])->get();

#### Or 语法

你也可以在查找中加入 `or` 子句来跟 where 约束链式调用在一起。`orWhere` 方法和 `where` 方法接受相同的参数：

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

#### 其它的 Where 子句

**whereBetween**

`whereBetween` 方法验证一个字段的值介于两个值之间：

    $users = DB::table('users')
                        ->whereBetween('votes', [1, 100])->get();

**whereNotBetween**

`whereNotBetween` 方法验证一个字段的值不在两个值之内：

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();

**whereIn 与 whereNotIn**

`whereIn` 方法验证指定字段的值包含在指定的数组之内：

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

`whereNotIn` 方法验证指定字段的值**不**包含在指定的数组之内：

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

**whereNull 与 whereNotNull**

`whereNull` 方法验证指定列的值为 `NULL`：

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();

`whereNotNull` 方法验证一个列的值**不**为 `NULL`：

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();

**whereDate / whereMonth / whereDay / whereYear**

`whereDate` 方法可以用来比较一列的值和日期是否相等：

    $users = DB::table('users')
                    ->whereDate('created_at', '2016-10-10')
                    ->get();

`whereMonth` 方法可以用来比较一列的值和一年当中某一月是否相等：

    $users = DB::table('users')
                    ->whereMonth('created_at', '10')
                    ->get();

`whereDay` 方法可以用来比较一列的值和一月当中的某一天是否相等：

    $users = DB::table('users')
                    ->whereDay('created_at', '10')
                    ->get();

`whereYear` 方法可以用来比较一列的值和某年是否相等：

    $users = DB::table('users')
                    ->whereYear('created_at', '2016')
                    ->get();

**whereColumn**

`whereColumn` 用来检测两个列的数据是否一致：

    $users = DB::table('users')
                    ->whereColumn('first_name', 'last_name')
                    ->get();

你也可以使用运算符来做匹对：

    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at')
                    ->get();

`whereColumn` 可以接受数组传参，条件语句会使用 `and` 连接起来：

    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', '=', 'last_name'],
                        ['updated_at', '>', 'created_at']
                    ])->get();

<a name="parameter-grouping"></a>
### 参数分组

有时你可能会需要创建更高级的 where 子句，例如「where exists」或者嵌套的参数分组。Laravel 的查询语句构造器也能处理这些。让我们先来看下一个在括号中将约束分组的例子：

    DB::table('users')
                ->where('name', '=', 'John')
                ->orWhere(function ($query) {
                    $query->where('votes', '>', 100)
                          ->where('title', '<>', 'Admin');
                })
                ->get();

如你所见，上面例子会将`闭包`传入 `orWhere` 方法，以告诉查询语句构造器开始一个约束分组。此`闭包`接收一个查询语句构造器的实例，你可用它来设置应包含在括号分组内的约束。上面的例子会生成以下 SQL：

    select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

<a name="where-exists-clauses"></a>
### Where Exists 语法

`whereExists` 方法允许你编写 `where exists` SQL 子句。此方法会接受一个`闭包`参数，此闭包接收一个查询语句构造器实例，让你可以定义应放在「exists」SQL 子句中的查找：

    DB::table('users')
                ->whereExists(function ($query) {
                    $query->select(DB::raw(1))
                          ->from('orders')
                          ->whereRaw('orders.user_id = users.id');
                })
                ->get();

上述的查找会生成以下的 SQL：

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )

<a name="json-where-clauses"></a>
### JSON 查询语句

Laravel 支持 MySQL 5.7 以上版本和 Postgres 数据库的 JSON 类型的字段查询。可以使用 `->` 运算符来查询 JSON 列数据：

    $users = DB::table('users')
                    ->where('options->language', 'en')
                    ->get();

    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();

<a name="ordering-grouping-limit-and-offset"></a>
## Ordering, Grouping, Limit 及 Offset

#### orderBy

`orderBy` 方法允许你针对指定字段将查找结果进行排序。`orderBy` 的第一个参数为你要用来排序的字段，第二个参数则控制排序的顺序，可以是 `asc` 或 `desc`：

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

#### inRandomOrder

`inRandomOrder` 会对数据结果进行随机排序，例如以下读取随机用户：

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();

#### groupBy / having / havingRaw

`groupBy` 和 `having` 方法可用来将查找结果进行分组。`having` 方法的签名和 `where` 方法的类似：

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

#### skip / take

要限制查找所返回的结果数量，或略过指定数量的查找结果（`偏移`），则可使用 `skip` 和 `take` 方法：

    $users = DB::table('users')->skip(10)->take(5)->get();

<a name="conditional-clauses"></a>
## 条件查询语句

有时候，你希望某个值为 true 的时候才执行查询，例如，如果一个请求中存在给定的输入值的时候才执行这个 `where` 语句，你可以使用 `when` 方法实现：

    $role = $request->input('role');

    $users = DB::table('users')
                    ->when($role, function ($query) use ($role) {
                        return $query->where('role_id', $role);
                    })
                    ->get();


只有当 `when` 的第一个参数为 `true` 的话，匿名函数里的 `where` 语句才会被执行。如果第一个参数是 `false` 闭包将不会被执行。

<a name="inserts"></a>
## Inserts

查询语句构造器也提供了 `insert` 方法，用来将记录插入数据表。`insert` 方法接收一个数组，包含要插入的字段名称及值：

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );

你甚至可以在 `insert` 调用中传入一个嵌套数组，来一次性插入多条记录到数据表中。每个数组代表要插入数据表中的一列记录：

    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);

#### 自动递增 ID

若数据表有自动递增的 id，则可使用 `insertGetId` 方法来插入记录并获取其 ID：

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> {note} 当使用 PostgreSQL 时，insertGetId 方法将预测自动递增字段的名称为 `id`。若你要从不同「顺序」来获取 ID，则可以将顺序名称作为第二个参数传给 `insertGetId` 方法。

<a name="updates"></a>
## Updates

当然，除了可在数据库中插入记录之外，也可使用 `update` 方法来让查询语句构造器更新已存在的记录。`update` 方法和 `insert` 方法一样，接收含有一对字段及值的数组，其中包含了要被更新的字段。可使用 `where` 子句来约束 `update` 查找：

    DB::table('users')
                ->where('id', 1)
                ->update(['votes' => 1]);

<a name="updating-json-columns"></a>
### Updating JSON Columns

更新 JSON 列的时候，你可以使用 `->` 语法在 JSON 对象中访问想要的键。只有当数据库支持 JSON 列的时候才支持这个操作：

    DB::table('users')
                ->where('id', 1)
                ->update(['options->enabled' => true]);

<a name="increment-and-decrement"></a>
### 递增或递减

查询语句构造器也提供了便利的方法来递增或递减指定字段的值。此方法提供了一个比手动编写 `update` 语法更具表达力且更精练的接口。

这两个方法都必须接收至少一个参数（要修改的字段）。也可选择性地传入第二个参数，用来控制字段应递增／递减的量：

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

你也可以指定要在操作中更新其它字段：

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

<a name="deletes"></a>
## Deletes

查询语句构造器也可通过 `delete` 方法来将记录从数据表中删除。在调用 `delete` 方法之前，也可加上 `where` 子句来约束 `delete` 语法：

    DB::table('users')->delete();

    DB::table('users')->where('votes', '>', 100)->delete();

若你希望截去整个数据表的所有数据列，并将自动递增 ID 重设为零，则可以使用 `truncate` 方法：

    DB::table('users')->truncate();

<a name="pessimistic-locking"></a>
## 悲观锁定

查询语句构造器也包含一些可用以协助你在 `select` 语法上作「悲观锁定」的函数。若要以「共享锁」来运行语句，则可在查找上使用 `sharedLock` 方法。共享锁可避免选择的数据列被更改，直到事务被提交为止：

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

此外，你也可以使用 `lockForUpdate` 方法。「用以更新」锁可避免数据列被其它共享锁修改或选取：

    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@贺钧威](https://phphub.org/users/5711)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5711_1473489317.jpg?imageView2/1/w/100/h/100">  |  翻译  | 感谢[BlueStone](http://bluestoneapp.thexrverge.com/)翻译支持，[@贺钧威](https://github.com/HejunweiCoder/) at Github  |



--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/2752/laravel-53-document-translation-completed)。
> 
> 文档永久地址： http://d.laravel-china.org