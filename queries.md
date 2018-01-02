# Laravel 数据库之：数据库请求构建器

- [简介](#introduction)
- [获取结果](#retrieving-results)
    - [分块结果](#chunking-results)
    - [聚合](#aggregates)
- [Selects](#selects)
- [原生表达式](#raw-expressions)
- [Joins](#joins)
- [Unions](#unions)
- [Where 语句](#where-clauses)
    - [参数分组](#parameter-grouping)
    - [Where Exists 语法](#where-exists-clauses)
    - [JSON 查询语句](#json-where-clauses)
- [Ordering, Grouping, Limit, & Offset](#ordering-grouping-limit-and-offset)
- [条件语句](#conditional-clauses)
- [Inserts](#inserts)
- [Updates](#updates)
    - [更新 JSON](#updating-json-columns)
    - [自增 & 自减](#increment-and-decrement)
- [Deletes](#deletes)
- [悲观锁](#pessimistic-locking)

<a name="introduction"></a>
## 简介

Laravel 的数据库查询构造器提供了一个方便的接口来创建及运行数据库查询语句。它能用来执行应用程序中的大部分数据库操作，且能在所有被支持的数据库系统中使用。

Laravel 的查询构造器使用 PDO 参数绑定来保护你的应用程序免受 SQL 注入的攻击。因此没有必要清理作为绑定传递的字符串。

<a name="retrieving-results"></a>
## 获取结果

#### 从数据表中获取所有的数据

你可以在 `DB` facade 上使用 `table` 方法开始查询。这个 `table` 方法为给定的表返回一个查询构造器实例，允许你在查询上链式调用更多的约束，最后使用 `get` 方法获取最终结果：

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
            $users = DB::table('users')->get();

            return view('user.index', ['users' => $users]);
        }
    }

`get` 方法会返回一个包含 `Illuminate\Support\Collection` 的结果，其中每个结果都是一个 PHP `StdClass` 对象的一个实例。你可以通过访问字段作为对象的属性来访问每列的值：

    foreach ($users as $user) {
        echo $user->name;
    }

#### 从数据表中获取单个列或行

如果你只需要从数据库表中获取一行数据，就使用 `first` 方法。这个方法将返回一个 `StdClass` 对象：

    $user = DB::table('users')->where('name', 'John')->first();

    echo $user->name;

如果你甚至不需要整行数据，就使用 `value` 方法从记录中取出单个值。该方法将直接返回字段的值：

    $email = DB::table('users')->where('name', 'John')->value('email');

#### 获取一列的值

如果你想要获取包含单个字段值的集合，可以使用 `pluck` 方法。在下面的例子中，我们将取出 roles 表中 title 字段的集合：

    $titles = DB::table('roles')->pluck('title');

    foreach ($titles as $title) {
        echo $title;
    }

你也可以在返回的集合中指定字段的自定义键值：

    $roles = DB::table('roles')->pluck('title', 'name');

    foreach ($roles as $name => $title) {
        echo $title;
    }

<a name="chunking-results"></a>
### 结果分块

如果你需要操作数千条数据库记录，可以考虑使用 `chunk` 方法。这个方法每次只取出一小块结果传递给 `闭包` 处理，这对于编写数千条记录的 [Artisan 命令](/docs/{{version}}/artisan) 而言是非常有用的。例如，一次处理整个 `users` 表中的 100 个记录：

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        foreach ($users as $user) {
            //
        }
    });

你可以从 `闭包` 中返回 `false` 来阻止进一步的分块的处理：

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        // Process the records...

        return false;
    });

<a name="aggregates"></a>
### 聚合

查询构造器还提供了各种聚合方法，如 `count`、 `max`、 `min`、 `avg` 和 `sum`。你可以在创建查询后调用其中的任意一个方法：

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

当然，你也可以将这些方法和其它语句结合起来：

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

<a name="selects"></a>
## Selects

#### 指定一个 Select 语句

你并不会总是想从数据表中选出所有的字段，这时可使用 `select` 方法自定义一个 `select` 语句来指定查询的字段：

    $users = DB::table('users')->select('name', 'email as user_email')->get();

`distinct` 方法允许你强制让查询返回不重复的结果：

    $users = DB::table('users')->distinct()->get();

如果你已有一个查询构造器实例，并且希望在现有的 select 语句中加入一个字段，则可以使用 `addSelect` 方法：

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

<a name="raw-expressions"></a>
## 原生表达式

有时候你可能需要在查询中使用原生表达式，使用 `DB::raw` 方法可以创建原生表达式：

    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();

> {note} 原生表达式将会被当作字符串注入到查询中，所以要小心避免创建 SQL 注入漏洞。

<a name="raw-methods"></a>

### 原生方法

可以使用以下的方法代替 `DB::raw` 将原生表达式插入查询的各个部分。

#### `selectRaw`

`selectRaw` 方法可以用来代替 `select(DB::raw(...))`。这个方法的第二个参数接受一个可选的绑定参数的数组：

    $orders = DB::table('orders')
                    ->selectRaw('price * ? as price_with_tax', [1.0825])
                    ->get();

#### `whereRaw / orWhereRaw`

可以使用 `whereRaw` 和 `orWhereRaw` 方法将原生的 `where` 语句注入到查询中。这些方法接受一个可选的绑定数组作为他们的第二个参数：

    $orders = DB::table('orders')
                    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
                    ->get();

#### `havingRaw / orHavingRaw`

`havingRaw` 和 `orHavingRaw` 方法可用于将原生字符串设置为 `having` 语句的值：

    $orders = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > 2500')
                    ->get();

#### `orderByRaw`

`orderByRaw` 方法可用于将原生字符串设置为 `order by` 语句的值：

    $orders = DB::table('orders')
                    ->orderByRaw('updated_at - created_at DESC')
                    ->get();
<a name="joins"></a>

## Joins

#### Inner Join 语句

查询构造器也可以编写 join 语句。若要执行基本的「内连接」，你可以在查询构造器实例上使用 `join` 方法。传递给 `join` 方法的第一个参数是你要需要连接的表的名称，而其它参数则用来指定连接的字段约束。你还可以在单个查询中连接多个数据表：

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

#### Left Join 语句

如果你想用「左连接」来代替「内连接」，请使用 `leftJoin` 方法。`leftJoin` 方法的用法和 `join` 方法一样：

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

#### Cross Join 语句

使用 `crossJoin` 方法和你想要交叉连接的表名来做「交叉连接」。交叉连接在第一个表和连接之间生成笛卡尔积：

    $users = DB::table('sizes')
                ->crossJoin('colours')
                ->get();

#### 高级 Join 语句

你也可以指定更高级的 join 语句。比如传递一个 `闭包` 作为 `join` 方法的第二个参数。此 `闭包` 接收一个 `JoinClause` 对象，从而在其中指定 `join` 语句中指定约束：

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();

如果你想要在连接上使用「where」风格的语句，可以在连接上使用 `where` 和 `orWhere` 方法。这些方法可以用来比较值和对应的字段：

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

<a name="unions"></a>
## Unions

查询构造器还提供了将两个查询「合并」起来的快捷方式。例如，你可以先创建一个初始查询，并使用 `union` 方法将它与第二个查询进行合并：

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

> {tip} 也可使用 `unionAll` 方法，它和 `union` 方法有着相同的用法。

<a name="where-clauses"></a>
## Where 语句

#### 简单的 Where 语句

你可以在查询构造器实例中使用 `where` 方法从而把 `where` 语句添加到查询中。基本的 `where` 方法需要三个参数。第一个参数是字段的名称，第二个参数是运算符，它可以是数据库所支持的任何运算符。最后，第三个参数是要对字段进行评估的值。

例如，下面是一个要验证「votes」字段的值等于 100 的查询：

    $users = DB::table('users')->where('votes', '=', 100)->get();

如果你只是想简单的校验某个字段等于指定的值，你可以直接将这个值作为第二个参数传递给 `where` 方法：

    $users = DB::table('users')->where('votes', 100)->get();

当然，在编写 `where` 语句时，也可以使用其它各种数据库支持的运算符：

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

你也可以传递条件数组给 `where` 函数：

    $users = DB::table('users')->where([
        ['status', '=', '1'],
        ['subscribed', '<>', '1'],
    ])->get();

#### Or 语句

你可以一起链式调用 where，也可以在查询添加中 `or` 语句。`orWhere` 方法接受与 `where` 方法相同的参数：

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

#### 其它 Where 语句

**whereBetween**

`whereBetween` 方法用来验证字段的值介于两个值之间：

    $users = DB::table('users')
                        ->whereBetween('votes', [1, 100])->get();

**whereNotBetween**

`whereNotBetween` 方法验证字段的值不在两个值之间：

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();

**whereIn / whereNotIn**

`whereIn` 方法验证字段的值在指定的数组内：

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

`whereNotIn` 方法验证字段的值不在指定的数组内：

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

**whereNull / whereNotNull**

`whereNull` 方法验证字段的值为 `NULL`：

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();

`whereNotNull` 方法验证字段的值不为 `NULL`：

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();

**whereDate / whereMonth / whereDay / whereYear / whereTime**

`whereDate` 方法用于比较字段的值和日期：

    $users = DB::table('users')
                    ->whereDate('created_at', '2016-12-31')
                    ->get();

`whereMonth` 方法用于比较字段的值与一年的特定月份：

    $users = DB::table('users')
                    ->whereMonth('created_at', '12')
                    ->get();

`whereDay` 方法用于比较字段的值与特定的一个月的某一天：

    $users = DB::table('users')
                    ->whereDay('created_at', '31')
                    ->get();

`whereYear` 方法用于比较字段的值与特定年份：

    $users = DB::table('users')
                    ->whereYear('created_at', '2016')
                    ->get();

`whereTime` 方法用于比较字段的值与特定的时间：

```
$users = DB::table('users')
                ->whereTime('created_at', '=', '11:20')
                ->get();
```

**whereColumn**

 `whereColumn` 方法用于验证两个字段是否相等：

    $users = DB::table('users')
                    ->whereColumn('first_name', 'last_name')
                    ->get();

还可以将比较运算符传递给该方法：

    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at')
                    ->get();

`whereColumn` 方法也可以传递一个包含多个条件的数组。这些条件将使用 `and` 运算符进行连接：

    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', '=', 'last_name'],
                        ['updated_at', '>', 'created_at']
                    ])->get();

<a name="parameter-grouping"></a>
### 参数分组

有时你可能需要创建更高级的 where 语句，例如「where exists」或者嵌套的参数分组。Laravel 的查询构造器也能够处理这些。下面有一个括号内的分组约束的示例：

    DB::table('users')
                ->where('name', '=', 'John')
                ->orWhere(function ($query) {
                    $query->where('votes', '>', 100)
                          ->where('title', '<>', 'Admin');
                })
                ->get();

在上面例子中将 `闭包` 传递到 `orWhere` 方法中，指示查询构造器开始一个约束分组。此 `闭包` 接收一个查询构造器实例，你可以用它来设置应包含在括号组中的约束。上面的例子会产生下面的 SQL：

    select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

<a name="where-exists-clauses"></a>
### Where Exists 语句

`whereExists` 方法允许你编写 `where exists` SQL 语句。此方法接受一个 `闭包` 参数，此闭包要接收一个查询构造器实例，让你可以定义放在「exists」语句中的查询：

    DB::table('users')
                ->whereExists(function ($query) {
                    $query->select(DB::raw(1))
                          ->from('orders')
                          ->whereRaw('orders.user_id = users.id');
                })
                ->get();

上述的查询将生成以下 SQL：

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )

<a name="json-where-clauses"></a>
### JSON where 语句

Laravel 也支持查询 JSON 类型的字段（仅在对 JSON 类型支持的数据库上）。目前，本特性仅支持 MySQL 5.7+ 和 Postgres数据库。可以使用 `->` 运算符来查询 JSON 列数据：

    $users = DB::table('users')
                    ->where('options->language', 'en')
                    ->get();

    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();

<a name="ordering-grouping-limit-and-offset"></a>
## Ordering, Grouping, Limit, & Offset

#### orderBy

`orderBy` 方法允许你根据指定字段对查询结果进行排序。`orderBy` 方法的第一个参数是你想要用来排序的字段，而第二个参数控制排序的方向，可以是 `asc` 或 `desc`：

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

#### latest / oldest

`latest` 和 `oldest` 方法允许你轻松地按日期对查询结果排序。默认情况下是对 `created_at` 字段进行排序。或者，你可以传递你想要排序的字段名称：

    $user = DB::table('users')
                    ->latest()
                    ->first();

#### inRandomOrder

`inRandomOrder` 方法可以将查询结果随机排序。例如，你可以使用这个方法获取一个随机用户：

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();

#### groupBy / having

`groupBy` 和 `having` 方法可用来对查询结果进行分组。`having` 方法的用法和 `where` 方法类似：

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

可以将多个参数传递给 `groupBy` 方法，按多个字段进行分组：

    $users = DB::table('users')
                    ->groupBy('first_name', 'status')
                    ->having('account_id', '>', 100)
                    ->get();

对于更高级的语句，请参阅 [`havingRaw`](https://laravel.com/docs/5.5/queries#raw-methods) 方法。

#### skip / take

 可以使用 `skip` 和 `take` 方法来限制从查询返回的结果数量或跳过查询中给定数量的结果：

    $users = DB::table('users')->skip(10)->take(5)->get();

或者，你也可以使用 `limit` 和 `offset` 方法：

    $users = DB::table('users')
                    ->offset(10)
                    ->limit(5)
                    ->get();

<a name="conditional-clauses"></a>
## 条件语句

有时你可能想要子句只适用于某个情况为真时才执行查询。例如，如果给定的输入值出现在传入请求中时，你可能想要判断它能达成某个 `where` 语句，你可以使用 `when` 方法来完成此操作：

    $role = $request->input('role');

    $users = DB::table('users')
                    ->when($role, function ($query) use ($role) {
                        return $query->where('role_id', $role);
                    })
                    ->get();


只有当 `when` 方法的第一个参数为 `true` 时，闭包里的 `where` 语句才会执行。如果第一个参数是 `false`，这个闭包将不会被执行。

你可以将另一个闭包当作第三个参数传递给 `when` 方法。如果第一个参数的值为 `false` 时，这个闭包将执行。为了说明如何使用此功能，我们将使用它配置查询的默认排序：

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

查询构造器也提供了将记录插入数据库表的 `insert` 方法。`insert` 方法接受一个字段名和值的数组作为参数：

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );

你还可以在 `insert` 中传入一个嵌套数组向表中插入多条记录。每个数组代表要插入表中的行：

    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);

#### 自增 ID

若数据表存在自增的 ID，则可以使用 `insertGetId` 方法来插入记录然后获取其 ID：

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> {note} 当使用 PostgreSQL 时，insertGetId 方法将默认把 `id` 作为自动递增字段的名称。若你要从不同「顺序」来获取 ID，则可以将字段名称作为第二个参数传递给 `insertGetId` 方法。

<a name="updates"></a>
## Updates

当然，除了在数据库中插入记录外，你也可以使用 `update` 来更新已存在的记录。`update` 方法和 `insert` 方法一样，接受包含要更新的字段及值的数组。你可以使用 `where` 语句来约束 `update` 的查询：

    DB::table('users')
                ->where('id', 1)
                ->update(['votes' => 1]);

<a name="updating-json-columns"></a>
### 更新 JSON 字段

更新 JSON 字段时，应该使用 `->` 语法来访问 JSON 对象中的相应键。此操作只能在支持 JSON 字段的数据库上操作：

    DB::table('users')
                ->where('id', 1)
                ->update(['options->enabled' => true]);

<a name="increment-and-decrement"></a>
### 自增 & 自减

查询构造器还为给定字段的递增或递减提供了方便的方法 。此方法提供了一个比手动编写 `update` 语句更具表达力且更精练的接口。

这两个方法都必须接收至少一个参数——要修改的字段。可以选择传递第二个参数来控制字段递增或递减的量：

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

你也可以在操作过程中指定要更新的字段：

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

<a name="deletes"></a>
## Deletes

查询构造器也可使用 `delete` 方法从数据表中删除记录。在调用 `delete` 方法前，还可以通过添加 `where` 语句来约束 `delete` 语句：

    DB::table('users')->delete();

    DB::table('users')->where('votes', '>', 100)->delete();

如果你需要清空表，你可以使用 `truncate` 方法，这将删除所有行，并重置自动递增 ID 为零：

    DB::table('users')->truncate();

<a name="pessimistic-locking"></a>
## 悲观锁

查询构造器也包含一些可以帮助你在 `select` 语句上实现「悲观锁定」的函数 。若要在查询中使用「共享锁」，可以使用 `sharedLock` 方法。共享锁可以防止选中的行被篡改，直到事务被提交为止：

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

或者，你也可以使用 `lockForUpdate` 方法。使用「更新」锁可避免行被其它共享锁修改或选取：

    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@iwzh](https://github.com/iwzh) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/3762_1456807721.jpeg?imageView2/1/w/200/h/200"> | 翻译   | 码不能停 [@iwzh](https://github.com/iwzh) at Github |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | Review | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
