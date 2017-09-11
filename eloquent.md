# Eloquent: 入门
- [简介](#introduction)
- [定义模型](#defining-models)
    - [Eloquent 模型约定](#eloquent-model-conventions)
- [取回多个模型](#retrieving-models)
    - [集合](#collections)
    - [分块结果](#chunking-results)
- [取回单个模型或集合](#retrieving-single-models)
    - [取回集合](#retrieving-aggregates)
- [添加和更新模型](#inserting-and-updating-models)
    - [基本添加](#inserts)
    - [基本更新](#updates)
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


Laravel 的 Eloquent ORM 提供了漂亮、简洁的 ActiveRecord 实现来和数据库进行交互。每个数据库表都有一个对应的「模型」可用来跟数据表进行交互。你可以通过模型查询数据表内的数据，以及将记录添加到数据表中。

在开始之前，请确认你已在 `config/database.php` 文件中设置好了数据库连接。更多数据库的设置信息请查看 [数据库设置](/docs/{{version}}/database#configuration) 文档。

<a name="defining-models"></a>
## 定义模型

开始之前，让我们先来创建一个 Eloquent 模型。模型通常放在 `app` 目录中，不过你可以将他们随意放在任何可通过 `composer.json` 自动加载的地方。所有的 Eloquent 模型都继承自 `Illuminate\Database\Eloquent\Model` 类。

创建模型实例的最简单方法是使用 `make:model` [Artisan 命令](/docs/{{version}}/artisan)：

    php artisan make:model User

当你生成一个模型时想要顺便生成一个 [数据库迁移](/docs/{{version}}/migrations)，可以使用 `--migration` 或 `-m` 选项：

    php artisan make:model User --migration

    php artisan make:model User -m

<a name="eloquent-model-conventions"></a>
### Eloquent 模型约定

现在，让我们来看一个 `Flight` 模型类的例子，我们将会用它从 `flights` 数据表中取回与保存信息：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        //
    }


#### 数据表名称

请注意，我们并没有告诉 Eloquent `Flight` 模型该使用哪一个数据表。除非数据表明确地指定了其它名称，否则将使用类的「蛇形名称」、复数形式名称来作为数据表的名称。因此在此例子中，Eloquent 将会假设 `Flight` 模型被存储记录在 `flights` 数据表中。你可以在模型上定义一个 `table` 属性，用来指定自定义的数据表名称：

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

Eloquent 也会假设每个数据表都有一个叫做 `id` 的主键字段。你也可以定义一个 `$primaryKey` 属性来重写这个约定。


此外，Eloquent 假定主键是一个递增的整数值，这意味着在默认情况下主键将自动的被强制转换为 `int`。 如果你想使用非递增或者非数字的主键，你必须在你的模型 public `$incrementing` 属性设置为`false`。

#### 时间戳

默认情况下，Eloquent 会认为在你的数据库表有 `created_at` 和 `updated_at` 字段。如果你不希望让 Eloquent 来自动维护这两个字段，可在模型内将 `$timestamps` 属性设置为 `false`：

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

如果你需要自定义自己的时间戳格式，可在模型内设置 `$dateFormat` 属性。这个属性决定了日期应如何在数据库中存储，以及当模型被序列化成数组或 JSON 格式：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 模型的日期字段保存格式。
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

如果你需要自定义用于存储时间戳的字段名，可在模型中设置 `CREATED_AT` 和 `UPDATED_AT` 常量：

    <?php

    class Flight extends Model
    {
        const CREATED_AT = 'creation_date';
        const UPDATED_AT = 'last_update';
    }

#### 数据库连接

默认情况下，所有的 Eloquent 模型会使用应用程序中默认的数据库连接设置。如果你想为模型指定不同的连接，可以使用 `$connection` 属性：


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
## 取回多个模型

一旦你创建并 [关联了一个模型到数据表](/docs/{{version}}/schema) 上，那么你就可以从数据库中获取数据。可把每个 Eloquent 模型想像成强大的 [查询构造器](/docs/{{version}}/queries)，它让你可以流畅地查询与模型关联的数据表。例如：


    <?php

    use App\Flight;

    $flights = App\Flight::all();

    foreach ($flights as $flight) {
        echo $flight->name;
    }

#### 增加额外的限制

Eloquent 的 `all` 方法会返回在模型数据表中的所有结果。由于每个 Eloquent 模型都可以当作一个 [查询构造器](/docs/{{version}}/queries)，所以你可以在查询中增加规则，然后使用 `get` 方法来获取结果：

    $flights = App\Flight::where('active', 1)
                   ->orderBy('name', 'desc')
                   ->take(10)
                   ->get();

> {tip} 由于 Eloquent 模型是查询构造器，因此你应当去阅读 [查询构造器](/docs/{{version}}/queries) 中所有可用的方法。你可在 Eloquent 查询中使用这其中的任何方法。

<a name="collections"></a>
### 集合

类似 `all` 以及 `get` 之类的可以取回多个结果的 Eloquent 方法，将会返回一个 `Illuminate\Database\Eloquent\Collection` 实例。`Collection` 类提供 [多种辅助函数](/docs/{{version}}/eloquent-collections#available-methods) 来处理你的 Eloquent 结果。


    $flights = $flights->reject(function ($flight) {
        return $flight->cancelled;
    });

当然，你也可以简单地像数组一样来遍历集合：

    foreach ($flights as $flight) {
        echo $flight->name;
    }

<a name="chunking-results"></a>
### 分块结果

如果你需要处理数以千计的 Eloquent 查找结果，则可以使用 `chunk` 命令。`chunk` 方法将会获取一个 Eloquent 模型的「分块」，并将它们送到指定的 `闭包 (Closure)` 中进行处理。当你在处理大量结果时，使用 `chunk` 方法可节省内存：


    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });

传递到方法的第一个参数表示每次「分块」时你希望接收的数据数量。闭包则作为第二个参数传递，它将会在每次从数据取出分块时被调用。

#### 使用游标

`cursor` 允许你使用游标来遍历数据库数据，一次只执行单个查询。在处理大数据量请求时 `cursor` 方法可以大幅度减少内存的使用：


    foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
        //
    }

<a name="retrieving-single-models"></a>
## 取回单个模型／集合

当然，除了从指定的数据表取回所有记录，你也可以通过 `find` 和 `first` 方法来取回单条记录。但这些方法返回的是单个模型的实例，而不是返回模型的集合：


    // 通过主键取回一个模型...
    $flight = App\Flight::find(1);

    // 取回符合查询限制的第一个模型 ...
    $flight = App\Flight::where('active', 1)->first();

你也可以用主键的集合为参数调用`find`方法，它将返回符合条件的集合：

    $flights = App\Flight::find([1, 2, 3]);

#### 「未找到」异常

有时候你可能希望在找不到模型时抛出一个异常，这在路由或是控制器内特别有用。`findOrFail` 以及 `firstOrFail` 方法会取回查询的第一个结果。如果没有找到相应结果，则会抛出一个 `Illuminate\Database\Eloquent\ModelNotFoundException`：

    $model = App\Flight::findOrFail(1);

    $model = App\Flight::where('legs', '>', 100)->firstOrFail();

如果该异常没有被捕获，则会自动返回 HTTP `404` 响应给用户，因此当使用这些方法时，你没有必要明确的编写检查来返回 `404` 响应：

    Route::get('/api/flights/{id}', function ($id) {
        return App\Flight::findOrFail($id);
    });

<a name="retrieving-aggregates"></a>
### 取回集合

当然，你也可以使用 `count`、`sum`、`max`，和其它 [查询构造器](/docs/{{version}}/queries) 提供的 [聚合函数](/docs/{{version}}/queries#aggregates)。这些方法会返回适当的标量值，而不是一个完整的模型实例：

    $count = App\Flight::where('active', 1)->count();

    $max = App\Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## 添加和更新模型

<a name="inserts"></a>
### 基本添加

要在数据库中创建一条新记录，只需创建一个新模型实例，并在模型上设置属性和调用 `save` 方法即可：

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
在这个例子中，我们把来自 HTTP 请求中的 `name` 参数简单地指定给 `App\Flight` 模型实例的 `name` 属性。当我们调用 `save` 方法，就会添加一条记录到数据库中。当 `save` 方法被调用时，`created_at` 以及 `updated_at` 时间戳将会被自动设置，因此我们不需要去手动设置它们。

<a name="updates"></a>
### 基本更新

`save` 方法也可以用于更新数据库中已经存在的模型。要更新模型，则须先取回模型，再设置任何你希望更新的属性，接着调用 `save` 方法。同样的，`updated_at` 时间戳将会被自动更新，所以我们不需要手动设置它的值：


    $flight = App\Flight::find(1);

    $flight->name = 'New Flight Name';

    $flight->save();

#### 批量更新

也可以针对符合指定查询的任意数量模型进行更新。在这个例子中，所有 `active` 并且 `destination` 为 `San Diego` 的航班，都将会被标识为延迟：

    App\Flight::where('active', 1)
              ->where('destination', 'San Diego')
              ->update(['delayed' => 1]);

`update` 方法会期望收到一个含有字段与值对应的数组，而这些字段的内容将会被更新。

> {note} 当通过“Eloquent”批量更新时，`saved`和`updated`模型事件将不会被更新后的模型代替。这是因为批量更新时，模型从来没有被取回。

<a name="mass-assignment"></a>
### 批量赋值

你也可以使用 `create` 方法通过一行代码来保存一个新模型。被插入数据库的模型实例将会返回给你。不过，在这样做之前，你需要先在你的模型上定义一个 `fillable` 或 `guarded` 属性，因为所有的 Eloquent 模型都针对批量赋值（Mass-Assignment）做了保护。

当用户通过 HTTP 请求传入了非预期的参数，并借助这些参数更改了数据库中你并不打算要更改的字段，这时就会出现批量赋值（Mass-Assignment）漏洞。例如，恶意用户可能会通过 HTTP 请求发送 `is_admin` 参数，然后对应到你模型的 `create` 方法，此操作能让该用户把自己升级为一个管理者。

所以，在开始之前，你应该定义好哪些模型属性是可以被批量赋值的。你可以在模型上使用 `$fillable` 属性来实现。例如，让我们让 `Flight` 模型的 `name` 属性可以被批量赋值：

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

一旦我们已经设置好可以被批量赋值的属性，便能通过 `create` 方法来添加一条新记录到数据库。`create` 方法将返回已经被保存的模型实例：

    $flight = App\Flight::create(['name' => 'Flight 10']);
    
如果你已经有一个 `model` 实例，你可以使用一个数组传递给 `fill` 方法：
 
    $flight->fill(['name' => 'Flight 22']);

#### Guarding Attributes

`$fillable` 作为一个可以被批量赋值的属性「白名单」。另外你也可以选择使用 `$guarded`。`$guarded` 属性应该包含一个你不想要被批量赋值的属性数组。所有不在数组里面的其它属性都可以被批量赋值。因此，`$guarded` 的功能更类似一个「黑名单」。使用的时候应该只选择 `$fillable` 或 `$guarded` 中的其中一个。 下面这个例子中，**除了 `price`** 所有的属性都可以被批量赋值：


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
如果你想让所有的属性都可以被批量赋值，你应该定义 `$guarded`为空数组。

    /**
     * 不可被批量赋值的属性。
     *
     * @var array
     */
    protected $guarded = [];

<a name="other-creation-methods"></a>
### 其它创建的方法
#### `firstOrCreate`/ `firstOrNew`
还有两种其它方法，你可以用来通过属性批量赋值创建你的模型：`firstOrCreate` 和 `firstOrNew`。`firstOrCreate` 方法将会使用指定的字段／值对，来尝试寻找数据库中的记录。如果在数据库中找不到模型，则会使用指定的属性来添加一条记录。

`firstOrNew` 方法类似 `firstOrCreate` 方法，它会尝试使用指定的属性在数据库中寻找符合的纪录。如果模型未被找到，将会返回一个新的模型实例。请注意 `firstOrnew` 返回的模型还尚未保存到数据库。你需要通过手动调用 `save` 方法来保存它：

    // 通过name属性检索航班，当结果不存在时创建它...
    $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);
    
    // 通过name属性检索航班，当结果不存在的时候用name属性和delayed属性去创建它
    $flight = App\Flight::firstOrCreate(
    	['name' => 'Flight 10'], ['delayed' => 1]
	);

    // 通过name属性检索航班，当结果不存在时实例化一个新实例...
    $flight = App\Flight::firstOrNew(['name' => 'Flight 10']);
    
    // 通过name属性检索航班，当结果不存在的时候用name属性和delayed属性去实例化一个新实例
    $flight = App\Flight::firstOrNew(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );

#### `updateOrCreate`
其次，你可能会碰到模型已经存在则更新，否则创建新模型的情形，Laravel 提供了一个 `updateOrCreate` 方法来一步完成该操作，类似 `firstOrCreate` 方法， `updateOrCreate` 方法会持久化模型，所以无需调用 `save()` :

    // If there's a flight from Oakland to San Diego, set the price to $99.
    // If no matching model exists, create one.
    $flight = App\Flight::updateOrCreate(
        ['departure' => 'Oakland', 'destination' => 'San Diego'],
        ['price' => 99]
    );


<a name="deleting-models"></a>
## 删除模型

要删除模型，必须在模型实例上调用 `delete` 方法：

    $flight = App\Flight::find(1);

    $flight->delete();

#### 通过主键来删除现有的模型

在上面的例子中，我们在调用 `delete` 方法之前会先从数据库中取回模型。不过，如果你已知道了模型中的主键，则可以不用取回模型就能直接删除它。若要直接删除，请调用 `destroy` 方法：

    App\Flight::destroy(1);

    App\Flight::destroy([1, 2, 3]);

    App\Flight::destroy(1, 2, 3);

#### 通过查询来删除模型

当然，你也可以运行在一组模型删除查询。在这个例子中，我们会删除被标记为不活跃的所有航班。 像批量更新那样，批量删除不会删除的任何被删除的模型的事件：

    $deletedRows = App\Flight::where('active', 0)->delete();

>{note} 当使用 Eloquent 批量删除语句时，`deleting` 和 `deleted` 模型事件不会在被删除模型实例上触发。因为删除语句执行时，不会检索回模型实例。

<a name="soft-deleting"></a>
### 软删除

除了从数据库中移除实际记录，Eloquent 也可以「软删除」模型。当模型被软删除时，它们并不会真的从数据库中被移除。而是会在模型上设置一个 `deleted_at` 属性并将其添加到数据库。如果模型有一个非空值 `deleted_at`，代表模型已经被软删除了。要在模型上启动软删除，则必须在模型上使用 `Illuminate\Database\Eloquent\SoftDeletes` trait 并添加 `deleted_at` 字段到你的 `$dates` 属性上：

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

当然，你也应该添加 `deleted_at` 字段到数据表中。Laravel [结构生成器](/docs/{{version}}/migrations) 包含了一个用来创建此字段的辅助函数：

    Schema::table('flights', function ($table) {
        $table->softDeletes();
    });

现在，当你在模型上调用 `delete` 方法时，`deleted_at` 字段将会被设置成目前的日期和时间。而且，当查询有启用软删除的模型时，被软删除的模型将会自动从所有查询结果中排除。

要确认指定的模型实例是否已经被软删除，可以使用 `trashed` 方法：

    if ($flight->trashed()) {
        //
    }


<a name="querying-soft-deleted-models"></a>
### 查询被软删除的模型

#### 包含被软删除的模型

如上所述，被软删除的模型将会自动从所有的查询结果中排除。不过，你可以通过在查询中调用 `withTrashed` 方法来强制查询已被软删除的模型：

    $flights = App\Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();

`withTrashed` 方法也可以被用在 [关联](/docs/{{version}}/eloquent-relationships) 查询：

    $flight->history()->withTrashed()->get();

#### 只取出软删除数据

`onlyTrashed` 会只取出软删除数据：

    $flights = App\Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();

#### 恢复被软删除的模型

有时候你可能希望「取消删除」一个已被软删除的模型。要恢复一个已被软删除的模型到有效状态，则可在模型实例上使用 `restore` 方法：

    $flight->restore();

你也可以在查询上使用 `restore` 方法来快速地恢复多个模型：

    App\Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();

与 `withTrashed` 方法类似，`restore` 方法也可以被用在 [关联](/docs/{{version}}/eloquent-relationships) 查询上:

    $flight->history()->restore();

#### 永久地删除模型

有时候你可能需要真正地从数据库移除模型。要永久地从数据库移除一个已被软删除的模型，则可使用 `forceDelete` 方法：

    // 强制删除单个模型实例...
    $flight->forceDelete();

    // 强制删除所有相关模型...
    $flight->history()->forceDelete();


<a name="query-scopes"></a>
## 查询作用域

<a name="global-scopes"></a>
### 全局作用域

全局作用域允许我们为给定模型的所有查询添加条件约束。Laravel 自带的 [软删除功能](#soft-deleting) 就使用了全局作用域来从数据库中拉出所有没有被删除的模型。编写自定义的全局作用域可以提供一种方便的、简单的方式，来确保给定模型的每个查询都有特定的条件约束。

#### 编写全局作用域

自定义全局作用域很简单，首先定义一个实现 `Illuminate\Database\Eloquent\Scope` 接口的类，该接口要求你实现一个方法：`apply`。需要的话可以在 `apply` 方法中添加 `where` 条件到查询：

    <?php

    namespace App\Scopes;

    use Illuminate\Database\Eloquent\Scope;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;

    class AgeScope implements Scope
    {
        /**
         * 应用作用域
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

> {tip} Laravel 没有规定你需要把这些类放置于哪个文件夹，你可以自由在 `app` 文件夹下创建 `Scopes` 文件夹来存放。

####  应用全局作用域

要将全局作用域分配给模型，需要重写给定模型的 `boot` 方法并使用 `addGlobalScope` 方法：

    <?php

    namespace App;

    use App\Scopes\AgeScope;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 数据模型的启动方法
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();

            static::addGlobalScope(new AgeScope);
        }
    }

添加作用域后，如果使用 `User::all()` 查询则会生成如下SQL语句：

    select * from `users` where `age` > 200

#### 匿名的全局作用域

Eloquent 还允许我们使用闭包定义全局作用域，这在实现简单作用域的时候特别有用，这样的话，我们就没必要定义一个单独的类了：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;

    class User extends Model
    {
        /**
         * 数据模型的启动方法
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

我们还可以通过以下方式，利用 `age` 标识符来移除全局作用：

    User::withoutGlobalScope('age')->get();

#### 移除全局作用域

如果想要在给定查询中移除指定全局作用域，可以使用 `withoutGlobalScope`：

    User::withoutGlobalScope(AgeScope::class)->get();

如果你想要移除某几个或全部全局作用域，可以使用 `withoutGlobalScopes` 方法：

    User::withoutGlobalScopes()->get();

    User::withoutGlobalScopes([FirstScope::class, SecondScope::class])->get();

<a name="local-scopes"></a>
###  本地作用域

本地作用域允许我们定义通用的约束集合以便在应用中复用。例如，你可能经常需要获取最受欢迎的用户，要定义这样的一个作用域，只需简单在对应 Eloquent 模型方法前加上一个 `scope` 前缀，作用域总是返回查询构建器：

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

#### 利用查询范围

一旦定义了范围，则可以在查询模型时调用范围方法。在进行方法调用时不需要加上 `scope` 前缀。你甚至可以链式调用不同的范围，如：

    $users = App\User::popular()->active()->orderBy('created_at')->get();

#### 动态范围

有时候，你可能希望定义一个可接受参数的范围。这时只需给你的范围加上额外的参数即可。范围参数应该被定义在 `$query` 参数之后：

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

现在，你可以在范围调用时传递参数：

    $users = App\User::ofType('admin')->get();

<a name="events"></a>
## 事件

Eloquent 模型会触发许多事件，让你在模型的生命周期的多个时间点进行监控：
`creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`. 

事件让你每当有特定的模型类在数据库保存或更新时，执行代码。

当一个新模型被初次保存将会触发 `creating` 以及 `created` 事件。如果一个模型已经存在于数据库且调用了 `save` 方法，将会触发 `updating` 和 `updated` 事件。在这两种情况下都会触发 `saving` 和 `saved` 事件。

开始前，在你的 Eloquent 模型上定义一个 `$events` 属性，将 Eloquent 模型的生命周期的多个点映射到你的 [服务提供者](/docs/{{version}}/providers) 。


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
         * 模型的时间映射。
         *
         * @var array
         */
        protected $events = [
            'saved' => UserSaved::class,
            'deleted' => UserDeleted::class,
        ];
    }

<a name="observers"></a>
### 观察者


如果你在一个给定的模型中监听许多事件，您可以使用观察者将所有监听器变成一个类。观察者类里的方法名应该反映Eloquent想监听的事件。 每种方法接收 model 作为其唯一的参数。 Laravel不包括观察者默认目录，所以你可以创建任何你喜欢你的目录来存放：
 
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
要注册一个观察者，需要用模型中的`observe`方法去观察。你可以在你的服务提供商之一的`boot`方法中注册观察者。在这个例子中，我们将在`AppServiceProvider`注册观察者：

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
| [@sml2h3](https://github.com/sml2h3)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/12218_1488347757.jpeg?imageView2/1/w/200/h/200">  |  翻译  | [欢迎访问个人博客→疯狂极客](https://www.fkgeek.com) |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org
