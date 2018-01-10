# Eloquent: 入门
- [简介](#introduction)
- [定义模型](#defining-models)
    - [Eloquent 模型约定](#eloquent-model-conventions)
- [检索多个模型](#retrieving-models)
    - [集合](#collections)
    - [分块结果](#chunking-results)
- [检索单个模型或集合](#retrieving-single-models)
    - [检索集合](#retrieving-aggregates)
- [插入 & 更新模型](#inserting-and-updating-models)
    - [插入](#inserts)
    - [更新](#updates)
    - [批量赋值](#mass-assignment)
    - [其他创建方法](#other-creation-methods)
- [删除模型](#deleting-models)
    - [软删除](#soft-deleting)
    - [查询被软删除的模型](#querying-soft-deleted-models)
- [查询作用域](#query-scopes)
    - [全局作用域](#global-scopes)
    - [本地作用域](#local-scopes)
- [事件](#events)
    - [观察器](#observers)


<a name="introduction"></a>
## 简介


Laravel 的 Eloquent ORM 提供了漂亮、简洁的 ActiveRecord 实现来和数据库交互。每个数据库表都有一个对应的「模型」用来与该表交互。你可以通过模型查询数据表中的数据，并将新记录添加到数据表中。

在开始之前，请确保在 `config/database.php` 中配置数据库连接。更多关于数据库的配置信息，请查看 [文档](/docs/{{version}}/database#configuration)。

<a name="defining-models"></a>
## 定义模型

首先，创建一个 Eloquent 模型，生成的模型通常放在 `app` 目录中，但你可以通过 `composer.json` 随意地将它们放在可被自动加载的地方。所有的 Eloquent 模型都继承了 `Illuminate\Database\Eloquent\Model` 类。

创建模型实例的最简单方法是使用 [Artisan 命令](/docs/{{version}}/artisan) `make:model`：

    php artisan make:model User

如果要在生成模型时生成 [数据库迁移](/docs/{{version}}/migrations)，可以使用 `--migration` 或 `-m` 选项：

    php artisan make:model User --migration

    php artisan make:model User -m

<a name="eloquent-model-conventions"></a>
### Eloquent 模型约定

现在，我们来看一个 `Flight` 模型类的例子，我们将会用它从 `flights` 数据表中检索和存储信息：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        //
    }


#### 数据表名称

请注意，我们并没有告诉 Eloquent，`Flight` 模型该使用哪一个数据表。除非数据表明确地指定了其它名称，否则将使用类的复数形式「蛇形命名」来作为表名。因此，在这种情况下，Eloquent 会假定 `Flight` 模型存储的是 `flights` 数据表中的记录。你可以通过在模型上定义 `table` 属性，来指定自定义数据表：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 与模型关联的数据表
         *
         * @var string
         */
        protected $table = 'my_flights';
    }

#### 主键

Eloquent 也会假定每个数据表都有一个名为 `id` 的主键字段。你可以定义一个受保护的 `$primaryKey` 属性来覆盖这个约定。


另外，Eloquent 假定主键是一个递增的整数值，这意味着在默认情况下主键会自动转换为 `int`。 如果使用的是非递增或者非数字的主键，则必须在模型上设置 `public $incrementing = false`。如果主键不是一个整数，则应该在模型上设置 `protected $keyType = string`。

#### 时间戳

默认情况下，Eloquent 会默认数据表中存在 `created_at` 和 `updated_at` 这两个字段。如果你不需要这两个字段，则需要在模型内将 `$timestamps` 属性设置为 `false`：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 该模型是否被自动维护时间戳
         *
         * @var bool
         */
        public $timestamps = false;
    }

如果你需要自定义时间戳格式，可在模型内设置 `$dateFormat` 属性。这个属性决定了日期属性应如何存储在数据库中，以及模型被序列化成数组或 JSON 时的格式：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 模型的日期字段的存储格式
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

如果需要自定义用于存储时间戳的字段名，可在模型中通过设置 `CREATED_AT` 和 `UPDATED_AT` 常量来实现：

    <?php

    class Flight extends Model
    {
        const CREATED_AT = 'creation_date';
        const UPDATED_AT = 'last_update';
    }

#### 数据库连接

默认情况下，所有的 Eloquent 模型都会使用应用程序中默认的数据库连接设置。如果你想为模型指定不同的连接，可以使用 `$connection` 属性：


    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 此模型的连接名称。
         *
         * @var string
         */
        protected $connection = 'connection-name';
    }

<a name="retrieving-models"></a>
## 检索多个模型

创建完模型 [及其关联的数据表](/docs/{{version}}/schema) 之后，就可以开始从数据库中检索数据。可把每个 Eloquent 模型想像成强大的 [查询构造器](/docs/{{version}}/queries)，它让你可以流畅地查询与该模型相关联的数据库表。例如：


    <?php

    use App\Flight;

    $flights = App\Flight::all();

    foreach ($flights as $flight) {
        echo $flight->name;
    }

#### 添加其他约束

Eloquent 的 `all` 方法会返回模型表中所有的结果。由于每个 Eloquent 模型都可以当作一个 [查询构造器](/docs/{{version}}/queries)，因此你还可以在查询中添加约束，然后使用 `get` 方法来获取结果：

    $flights = App\Flight::where('active', 1)
                   ->orderBy('name', 'desc')
                   ->take(10)
                   ->get();

> {tip} Eloquent 模型是查询构造器，因此你应当去阅读 [查询构造器](/docs/{{version}}/queries) 提供的所有方法，以便你可以在 Eloquent 查询中使用。

<a name="collections"></a>
### 集合

使用 Eloquent 中的方法比如 `all` 和 `get` 可以检索多个结果，并且会返回一个 `Illuminate\Database\Eloquent\Collection` 实例。`Collection` 类提供了 [很多辅助函数](/docs/{{version}}/eloquent-collections#available-methods) 来处理Eloquent 结果。


    $flights = $flights->reject(function ($flight) {
        return $flight->cancelled;
    });

你也可以像数组一样简单地来遍历集合：

    foreach ($flights as $flight) {
        echo $flight->name;
    }

<a name="chunking-results"></a>
### 分块结果

如果你需要处理数千个 Eloquent 记录，可以使用 `chunk` 命令。`chunk` 方法会检索 Eloquent 模型的「分块」，将它们提供给指定的 `Closure` 进行处理。在处理大型结果集时，使用 `chunk` 方法可节省内存：


    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });

传递到方法的第一个参数是希望每个「分块」接收的数据量。闭包则被作为第二个参数传递，它会在每次执行数据库查询传递每个块时被调用。

#### 使用游标

`cursor` 允许你使用游标来遍历数据库数据，该游标只执行一个查询。处理大量数据时，可以使用 `cursor` 方法可以大幅度减少内存的使用量：


    foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
        //
    }

<a name="retrieving-single-models"></a>
## 检索单个模型／集合

除了从指定的数据表检索所有记录外，你也可以通过 `find` 或 `first` 方法来检索单条记录。这些方法不是返回一组模型，而是返回一个模型实例：


    // 通过主键取回一个模型...
    $flight = App\Flight::find(1);

    // 取回符合查询限制的第一个模型 ...
    $flight = App\Flight::where('active', 1)->first();

你也可以用主键数组为参数调用 `find` 方法，它将返回匹配记录的集合：

    $flights = App\Flight::find([1, 2, 3]);

#### 「找不到」异常

如果你希望在找不到模型时抛出异常，可以使用 `findOrFail` 以及 `firstOrFail` 方法。这些方法会检索查询的第一个结果。如果没有找到相应结果，就会抛出一个 `Illuminate\Database\Eloquent\ModelNotFoundException`：

    $model = App\Flight::findOrFail(1);

    $model = App\Flight::where('legs', '>', 100)->firstOrFail();

如果没有对异常进行捕获，则会自动返回 HTTP `404` 响应给用户。也就是说，在使用这些方法时，不需要另外写个检查来返回 `404` 响应：

    Route::get('/api/flights/{id}', function ($id) {
        return App\Flight::findOrFail($id);
    });

<a name="retrieving-aggregates"></a>
### 检索集合

你还可以使用 [查询构造器](/docs/{{version}}/queries) 提供的 `count`、`sum`、`max` 以及其它 [聚合函数](/docs/{{version}}/queries#aggregates)。这些方法只会返回适当的标量值而不是整个模型实例：

    $count = App\Flight::where('active', 1)->count();

    $max = App\Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## 插入 & 更新模型

<a name="inserts"></a>
### 插入

要在数据库中创建新记录，只需创建一个新的模型实例，并在模型上设置属性，然后调用 `save` 方法：

    <?php

    namespace App\Http\Controllers;

    use App\Flight;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class FlightController extends Controller
    {
        /**
         * 创建一个新的航班实例。
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // 验证请求...

            $flight = new Flight;

            $flight->name = $request->name;

            $flight->save();
        }
    }
在这个例子中，我们把来自 HTTP 请求中的 `name` 参数简单地指定给 `App\Flight` 模型实例的 `name` 属性。当我们调用 `save` 方法时，就会添加一条记录到数据库中。`created_at` 以及 `updated_at` 时间戳将在 `save` 方法被调用时会被自动设置，因此我们不需要去手动设置它们。

<a name="updates"></a>
### 更新

`save` 方法也可以用来更新数据库中已经存在的模型。要更新模型，则须先检索模型，再设置要更新的属性，然后再调用 `save` 方法。同样的，`updated_at` 时间戳将会被自动更新，所以我们不需要手动设置它的值：


    $flight = App\Flight::find(1);

    $flight->name = 'New Flight Name';

    $flight->save();

#### 批量更新

还可以对与给定查询匹配的任意数量的模型执行更新。在这个例子中，所有 `active` 为 1 且 `destination` 为 `San Diego` 的航班的 `delayed` 都会更新为 1：

    App\Flight::where('active', 1)
              ->where('destination', 'San Diego')
              ->update(['delayed' => 1]);

`update` 方法需要传入表示要更新的字段的字段的值的键值对数组。

> {note} 通过 Eloquent 执行批量更新时，`saved` 和 `updated` 的模型事件不会被更新的模型触发。这是因为执行批量更新时，不会有任何模型被检索出来。

<a name="mass-assignment"></a>
### 批量赋值

你也可以使用 `create` 方法来保存新模型，然后被插入数据库的模型实例会从方法返回。不过，在这之前，你需要先在你的模型上指定 `fillable` 或 `guarded` 的属性，因为所有的 Eloquent 模型在默认情况下都不能进行批量赋值。

当用户通过 HTTP 请求传入了一个意料之外的参数，并且该参数更改了数据库中你并不打算要更改的字段时，就会发生批量赋值漏洞。例如，恶意用户可能会通过 HTTP 请求发送 `is_admin` 参数，然后将其传递到模型的 `create` 方法中，此操作能让该用户把自己升级为管理者。

所以，在开始之前，你应该定义好哪些模型属性是可以被批量赋值的。你可以使用模型上的 `$fillable` 属性来实现。例如，让 `Flight` 模型的 `name` 属性可以被批量赋值：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 可以被批量赋值的属性。
         *
         * @var array
         */
        protected $fillable = ['name'];
    }

只有我们设置好可以被批量赋值的属性，才能通过 `create` 方法来为数据库添加新记录到。`create` 方法会返回已保存的模型实例：

```
$flight = App\Flight::create(['name' => 'Flight 10']);
```

如果你已经有一个模型实例，你可以传递数组给 `fill` 方法：

```
$flight->fill(['name' => 'Flight 22']);
```

#### 保护属性

`$fillable` 可以作为设置被批量赋值的属性的「白名单」，同样的 `$guarded` 属性也可以实现这个需求。但不同的是，`$guarded` 属性包含的是不想被批量赋值的属性的数组。即所有不在数组里面的属性都是可以被批量赋值的。也就是说，`$guarded` 从功能上讲更像是一个「黑名单」。而在使用的时候，也要注意只能是 `$fillable` 或 `$guarded` 二选一。 下面这个例子中，**除了 `price`** 所有的属性都可以被批量赋值：


    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 不可被批量赋值的属性。
         *
         * @var array
         */
        protected $guarded = ['price'];
    }
如果想让所有的属性都可以被批量赋值，就把 `$guarded` 定义为空数组。

    /**
     * 不可被批量赋值的属性。
     *
     * @var array
     */
    protected $guarded = [];

<a name="other-creation-methods"></a>
### 其他创建方法

#### `firstOrCreate`/ `firstOrNew`
你还可以使用其他两种方法来创建模型：`firstOrCreate` 和 `firstOrNew`。`firstOrCreate` 方法会使用给定的字段及其值在数据库中查找记录。如果在数据库中找不到模型，则将使用第一个参数中的属性以及可选的第二个参数中的属性插入记录。

`firstOrNew` 方法就类似 `firstOrCreate` 方法，会在数据库中查找匹配给定属性的记录。如果模型未被找到，则会返回一个新的模型实例。请注意，在这里面，`firstOrnew` 返回的模型还尚未保存到数据库，必须要手动调用 `save` 方法才能保存它：

    // 通过 name 属性检索航班，当结果不存在时创建它...
    $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

    // 通过 name 属性检索航班，当结果不存在的时候用 name 属性和 delayed 属性去创建它
    $flight = App\Flight::firstOrCreate(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );

    // 通过 name 属性检索航班，当结果不存在时实例化...
    $flight = App\Flight::firstOrNew(['name' => 'Flight 10']);

    // 通过 name 属性检索航班，当结果不存在的时候用 name 属性和 delayed 属性实例化
    $flight = App\Flight::firstOrNew(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );

#### `updateOrCreate`
你也可能会遇到想要更新现有模型或创建新模型（如果不存在）的情况。Laravel 提供了 `updateOrCreate` 方法来完成该操作，像 `firstOrCreate` 方法一样，`updateOrCreate` 方法会保存模型，所以不需要调用 `save()` :

    // 如果有从奥克兰飞往圣地亚哥的航班，将价格设为 99 美元
    // 如果不存在匹配的模型就创建一个
    $flight = App\Flight::updateOrCreate(
        ['departure' => 'Oakland', 'destination' => 'San Diego'],
        ['price' => 99]
    );


<a name="deleting-models"></a>
## 删除模型

可以在模型实例上调用 `delete` 方法来删除模型：

    $flight = App\Flight::find(1);

    $flight->delete();

#### 通过主键删除模型

上面的例子是在调用 `delete` 方法之前先从数据库中检索模型。不过，如果你已知道了这个模型的主键，则可以直接调用 `destroy` 方法删除它：

    App\Flight::destroy(1);

    App\Flight::destroy([1, 2, 3]);

    App\Flight::destroy(1, 2, 3);

#### 通过查询删除模型

你也可以在模型上运行删除语句。在下面例子中删除了所有被标记为不活跃的航班。 像批量更新那样，批量删除不会为删除的模型启动任何模型事件：

    $deletedRows = App\Flight::where('active', 0)->delete();

>{note} 使用 Eloquent 执行批量删除语句时，`deleting` 和 `deleted` 模型事件不会为已删除的模型触发。因为在执行删除语句时，不会检索模型实例。

<a name="soft-deleting"></a>
### 软删除

除了真的从数据库中删除记录，Eloquent 也可以「软删除」模型。当模型被软删除时，它们并不是真的从数据库中被删除。模型上设置了一个 `deleted_at` 属性并将其添加到数据库，也就是说，如果模型具有非空的 `deleted_at` 值，那就代表模型已经被软删除了。要启动模型上的软删除，则必须在模型上使用 `Illuminate\Database\Eloquent\SoftDeletes` trait 并添加 `deleted_at` 字段到 `$dates` 属性上：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;

    class Flight extends Model
    {
        use SoftDeletes;

        /**
         * 需要被转换成日期的属性。
         *
         * @var array
         */
        protected $dates = ['deleted_at'];
    }

你也应该添加 `deleted_at` 字段到数据表中。Laravel [结构生成器](/docs/{{version}}/migrations) 包含了一个辅助函数用来创建此字段：

    Schema::table('flights', function ($table) {
        $table->softDeletes();
    });

当你调用模型上 `delete` 方法时，`deleted_at` 字段会被设置为当前的日期和时间。而且，当查询使用软删除的模型时，被软删除的模型将自动从所有查询结果中排除。

要给定模型实例是否已被软删除，可以使用 `trashed` 方法：

    if ($flight->trashed()) {
        //
    }


<a name="querying-soft-deleted-models"></a>
### 查询被软删除的模型

#### 包括被软删除的模型

如上所述，被软删除的模型将自动从查询结果中排除。但是，你可以使用查询中的 `withTrashed` 方法强制软删除的模型出现在结果集中：

    $flights = App\Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();

`withTrashed` 方法也可用于 [关联](/docs/{{version}}/eloquent-relationships) 查询：

    $flight->history()->withTrashed()->get();

#### 只检索被软删除的模型

`onlyTrashed` 方法只会取出被软删除的模型：

    $flights = App\Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();

#### 恢复被软删除的模型

如果想「取消删除」被软删除的模型。可在模型实例上使用 `restore` 方法将一个被软删除的模型恢复到有效状态：

    $flight->restore();

你也可以在查询上使用 `restore` 方法来快速地恢复多个模型。像其他「批量赋值」操作一样，这并不会触发任何模型事件：

    App\Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();

与 `withTrashed` 方法类似，`restore` 方法也可以被用在 [关联](/docs/{{version}}/eloquent-relationships) 查询上:

    $flight->history()->restore();

#### 永久删除模型

如果要真正地从数据库中永久删除软删除的模型，可以使用 `forceDelete` 方法：

    // 强制删除单个模型实例...
    $flight->forceDelete();

    // 强制删除所有相关模型...
    $flight->history()->forceDelete();


<a name="query-scopes"></a>
## 查询作用域

<a name="global-scopes"></a>
### 全局作用域

全局范围能为给定模型的所有查询添加约束。Laravel 自带的 [软删除功能](#soft-deleting) 就利用全局作用域从数据库中提取「未删除」的模型。编写自定义的全局作用域可以提供一个方便、简单的方法来确保给定模型的每个查询都受到一定的约束。

#### 编写全局作用域

编写全局作用域很简单。首先定义一个实现 `Illuminate\Database\Eloquent\Scope` 接口的类。这个接口要求你实现一个方法：`apply`。`apply` 方法可以根据需要添加 `where` 条件到查询：

    <?php

    namespace App\Scopes;

    use Illuminate\Database\Eloquent\Scope;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;

    class AgeScope implements Scope
    {
        /**
         * 将范围应用于给定的 Eloquent 查询生成器
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $builder
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @return void
         */
        public function apply(Builder $builder, Model $model)
        {
            return $builder->where('age', '>', 200);
        }
    }

> {tip} 如果全局作用域要将字段添加到查询的 select 语句中，则应该使用 `addSelect` 方法而不是 `select`。这是用来防止可能会无意中替换了查询的现有 select 语句。

####  应用全局作用域

要将全局作用域分配给模型，需要重写给定模型的 `boot` 方法并使用 `addGlobalScope` 方法：

    <?php

    namespace App;

    use App\Scopes\AgeScope;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 模型的「启动」方法
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();

            static::addGlobalScope(new AgeScope);
        }
    }

添加作用域后，如果使用 `User::all()` 查询则会生成如下 SQL 语句：

    select * from `users` where `age` > 200

#### 匿名的全局作用域

Eloquent 还能使用闭包定义全局作用域，如此一来，便就没必要定义一个单独的类了：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;

    class User extends Model
    {
        /**
         * 模型的「启动」方法
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();

            static::addGlobalScope('age', function(Builder $builder) {
                $builder->where('age', '>', 200);
            });
        }
    }


#### 删除全局作用域

如果要删除给定的查询的全局作用域，则可以使用 `withoutGlobalScope` 方法。该方法接受全局作用域的类名作为其唯一参数：

    User::withoutGlobalScope(AgeScope::class)->get();
如果你想要删除几个甚至全部的全局作用域，可以使用 `withoutGlobalScopes` 方法：

```
// 删除所有的全局作用域
User::withoutGlobalScopes()->get();

// 删除一些全局作用域
User::withoutGlobalScopes([
    FirstScope::class, SecondScope::class
])->get();
```


<a name="local-scopes"></a>
###  本地作用域

本地作用域能定义通用的约束集合以便在应用中复用。例如，你可能经常需要检索最受欢迎的用户，为此要定义一个作用域，只需要在 `scope` 前加上一个 Eloquent 模型方法即可。

作用域应始终返回查询生成器实例：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 限制查询只包括受欢迎的用户。
         *
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }

        /**
         * 限制查询只包括活跃的用户。
         *
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeActive($query)
        {
            return $query->where('active', 1);
        }
    }

#### 利用本地作用域

定义了范围之后，可以在查询模型时调用 `scope` 方法。注意，在调用方法时，不应包含 `scope` 前缀。你甚至可以链式调用到不同的 `scope`，例如：

    $users = App\User::popular()->active()->orderBy('created_at')->get();

#### 动态作用域

只需将附加参数添加到作用域。作用域参数应该在 `$query` 参数之后定义：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 限制查询只包括指定类型的用户。
         *
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeOfType($query, $type)
        {
            return $query->where('type', $type);
        }
    }

现在，你可以在调用作用域时传递参数：

    $users = App\User::ofType('admin')->get();

<a name="events"></a>
## 事件

Eloquent 的模型触发了几个事件，可以在模型的生命周期的以下几点进行监控： `retrieved`、`creating`、`created`、`updating`、`updated`、`saving`、`saved`、`deleting`、`deleted`、`restoring`、`restored`。事件能在每次在数据库中保存或更新特定模型类时轻松地执行代码。

从数据库中检索现有模型时会触发 `retrieved` 事件。当新模型第一次被保存时， `creating` 以及 `created` 事件会被触发。如果模型已经存在于数据库中并且调用了 `save` 方法，会触发 `updating` 和 `updated` 事件。在这两种情况下，`saving` / `saved` 事件都会触发。

开始前，在 Eloquent 模型上定义一个 `$dispatchesEvents` 属性，将 Eloquent 模型的生命周期的各个点映射到你的 [事件类](/docs/{{version}}/events) 中。


    <?php

    namespace App;

    use App\Events\UserSaved;
    use App\Events\UserDeleted;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * 模型的事件映射。
         *
         * @var array
         */
        protected $dispatchesEvents = [
            'saved' => UserSaved::class,
            'deleted' => UserDeleted::class,
        ];
    }

<a name="observers"></a>
### 观察器

如果要给某个模型监听很多事件，则可以使用观察器将所有监听器分组到一个类中。观察器类里的方法名应该对应 Eloquent 中你想监听的事件。 每种方法接收 model 作为其唯一的参数。Laravel 没有为观察器设置默认的目录，所以你可以创建任何你喜欢你的目录来存放：

    <?php

    namespace App\Observers;

    use App\User;

    class UserObserver
    {
        /**
         * 监听用户创建的事件。
         *
         * @param  User  $user
         * @return void
         */
        public function created(User $user)
        {
            //
        }

        /**
         * 监听用户删除事件。
         *
         * @param  User  $user
         * @return void
         */
        public function deleting(User $user)
        {
            //
        }
    }
要注册一个观察器，需要在模型上使用 `observe` 方法。你可以在服务提供器中的 `boot` 方法注册观察器。在这个例子中，我们将在 `AppServiceProvider` 注册观察器：

    <?php

    namespace App\Providers;

    use App\User;
    use App\Observers\UserObserver;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 运行所有应用.
         *
         * @return void
         */
        public function boot()
        {
            User::observe(UserObserver::class);
        }

        /**
         * 注册服务提供.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }


## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@sml2h3](https://github.com/sml2h3) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/12218_1488347757.jpeg?imageView2/1/w/200/h/200"> | 翻译 | [欢迎访问个人博客→疯狂极客](https://www.fkgeek.com) |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | Review | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
