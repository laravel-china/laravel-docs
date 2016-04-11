# Eloquent: 入门

- [简介](#introduction)
- [定义模型](#defining-models)
    - [Eloquent 模型惯例](#eloquent-model-conventions)
- [取回多个模型](#retrieving-multiple-models)
- [取回单个模型或集合](#retrieving-single-models)
    - [取回集合](#retrieving-aggregates)
- [添加和更新模型](#inserting-and-updating-models)
    - [基本添加](#basic-inserts)
    - [基本更新](#basic-updates)
    - [批量赋值](#mass-assignment)
- [删除模型](#deleting-models)
    - [软删除](#soft-deleting)
    - [查找被软删除的模型](#querying-soft-deleted-models)
- [查找范围](#query-scopes)
- [事件](#events)

<a name="introduction"></a>
## 简介

Laravel 的 Eloquent ORM 提供了漂亮、简洁的 ActiveRecord 实现来和数据库交互。每个数据库表有一个对应的「模型」可以用来跟数据表交互。你可以通过模型查找数据表内的数据，以及添加记录到数据表中。

在开始之前，请确认有设置你的数据库链接在 `config/database.php` 文件内。更多数据库的设置信息，请查看[数据库设置](/docs/{{version}}/database#configuration)。

<a name="defining-models"></a>
## 定义模型

开始之前，让我们先创建一个 Eloquent 模型。模型通常放在 `app` 目录，不过你可以自由地把他们放在任何可以通过你的 `composer.json` 自动加载的地方。所有的 Eloquent 模型都继承 `Illuminate\Database\Eloquent\Model` 类。

> [Summer](http://github.com/summerblue): 建议在 app 目录下创建 `Models` 来统一存放。

创建模型实例的最简单的方法是使用 `make:model` [Artisan 命令](/docs/{{version}}/artisan)：

    php artisan make:model User

假设当你生成一个模型时，想要产生一个[数据库迁移](/docs/{{version}}/schema#database-migrations)，可以使用 `--migration` 或 `-m` 选项：

    php artisan make:model User --migration

    php artisan make:model User -m

<a name="eloquent-model-conventions"></a>
### Eloquent 模型惯例

现在，让我们来看一个 `Flight` 模型类的例子，我们将会用它来从 `flights` 数据表取回与保存信息：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        //
    }


#### 数据表名称

请注意，我们并没有告诉 Eloquent `Flight` 模型该使用哪一个数据表。除非明确地指定其他名称，不然类的小写、底线、复数形式会拿来当作数据表的表单名称。所以，这个例子中，Eloquent 将会假设 `Flight` 模型保存记录在 `flights` 数据表。你可以在模型上定义一个 `table` 属性，用来指定自定义的数据表名称：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 与模型关联的数据表。
         *
         * @var string
         */
        protected $table = 'my_flights';
    }

#### 主键

Eloquent 也会假设每个数据表有一个主键字段叫做 `id`。你可以定义一个 `$primaryKey` 属性来重写这个惯例。

#### 时间戳

默认情况下，Eloquent 预期你的数据表会有 `created_at` 和 `updated_at` 字段。如果你不希望让 Eloquent 来自动维护这两个字段，在你的模型内将 `$timestamps` 属性设置为 `false`：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 指定是否模型应该被戳记时间。
         *
         * @var bool
         */
        public $timestamps = false;
    }

如果你需要自定义你的时间戳格式，在你的模型内设置 `$dateFormat` 属性。这个属性决定日期如何在数据库中保存，以及当模型被序列化成数组或是 JSON 时的格式：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 模型的日期字段的保存格式。
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

#### 数据库连接

默认情况下，所有的 Eloquent 模型会使用你应用程序中默认的数据库连接设置。如果你想为模型指定不同的连接，可以使用 `$connection` 属性：

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

<a name="retrieving-multiple-models"></a>
## 取回多个模型

一旦你创建了一个模型并且将模型 [关连到数据表](/docs/{{version}}/schema)，你就可以从数据库中获取数据。把每个 Eloquent 模型想像成强大的[查询构造器](/docs/{{version}}/queries)，让你可以流畅地查找与模型关联的数据表。例如：

    <?php

    namespace App\Http\Controllers;

    use App\Flight;
    use App\Http\Controllers\Controller;

    class FlightController extends Controller
    {
        /**
         * 显示所有可以的航班清单。
         *
         * @return Response
         */
        public function index()
        {
            $flights = Flight::all();

            return view('flight.index', ['flights' => $flights]);
        }
    }

#### 访问字段的值

假设你有一个 Eloquent 模型的实例，你可以通过相对应的属性来访问模型的字段值。例如，让我们遍历查找所返回的每个 `Flight` 实例，并且输出 `name` 字段的值：

    foreach ($flights as $flight) {
        echo $flight->name;
    }

#### 增加额外的限制

Eloquent 的 `all` 方法会返回在模型数据表中所有的结果。由于每个 Eloquent 模型可以当作一个[查询构造器](/docs/{{version}}/queries)，所以你可以在查找中增加规则，然后通过 `get` 方法来获取结果：

    $flights = App\Flight::where('active', 1)
                   ->orderBy('name', 'desc')
                   ->take(10)
                   ->get();

> **注意：** 由于 Eloquent 模型是查询构造器，应该阅读所有 [查询构造器](/docs/{{version}}/queries)可用的方法。你可以在你的 Eloquent 查找中使用这其中的任何方法。

#### 集合

像是 `all` 以及 `get` 之类可以取回多个结果的 Eloquent 方法，将会返回一个 `Illuminate\Database\Eloquent\Collection` 实例。`Collection` 类提供[多样的辅助方法](/docs/{{version}}/eloquent-collections#available-methods) 用来处理你的 Eloquent 结果。当然，你也可以简单地像数组一样遍历你的集合：

    foreach ($flights as $flight) {
        echo $flight->name;
    }

#### 分块结果

如果你需要处理上千笔 Eloquent 查找结果，可以使用 `chunk` 命令。`chunk` 方法将会获取一个 Eloquent 模型的「分块」，将它们送到指定的 `闭包 (Closure)` 进行处理。当你在处理大量的结果时，使用 `chunk` 方法可以节省内存：

    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });

传递到方法的第一个参数是表示你希望每次「分块」要接收的数据数量。闭包则作为第二个参数传递，它将会在每次从数据取出分块时被调用。

<a name="retrieving-single-models"></a>
## 取回单个模型／集合

当然，除了从指定的数据表取回所有记录，你也可以通过 `find` 和 `first` 取回单个的记录。这些方法返回单个模型的实例，而不是返回模型的集合：

    // 借由主键取回一个模型...
    $flight = App\Flight::find(1);

    // 取回符合查找限制的第一个模型 ...
    $flight = App\Flight::where('active', 1)->first();

#### 找不到的异常

有时候你可能希望在找不到模型时抛出异常，这在路由或是控制器内特别有用。`findOrFail` 以及 `firstOrFail` 方法会取回查找的第一个结果。而如果没有找到结果，将会抛出一个 `Illuminate\Database\Eloquent\ModelNotFoundException`：

    $model = App\Flight::findOrFail(1);

    $model = App\Flight::where('legs', '>', 100)->firstOrFail();

如果没有捕捉到异常，会自动地送回 HTTP `404` 响应给用户，所以当使用这些方法时，没有必要明确的编写检查以返回 `404` 响应：

    Route::get('/api/flights/{id}', function ($id) {
        return App\Flight::findOrFail($id);
    });

<a name="retrieving-aggregates"></a>
### 取回集合

当然，你也可以使用 `count`、`sum`、`max`，和其他[查询构造器](/docs/{{version}}/queries)提供的[聚合函数](/docs/{{version}}/queries#aggregates)。这些方法会返回适当的纯量值，而不是一个完整的模型实例：

    $count = App\Flight::where('active', 1)->count();

    $max = App\Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## 添加和更新模型

<a name="basic-inserts"></a>
### 基本添加

要在数据库中创建一条新记录，只要创建一个新模型实例，并在模型上设置属性，再调用 `save` 方法：

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

在这个例子中，我们把进来的 HTTP 请求的 `name` 参数简单地指定给 `App\Flight` 模型实例的 `name` 属性。当我们调用 `save` 方法，就会添加一条记录到数据库中。当 `save` 方法被调用时，`created_at` 以及 `updated_at` 时间戳将会自动被设置，所以不需要手动去设置它们。

<a name="basic-updates"></a>
### 基本更新

`save` 方法也可以用于更新数据库中已经存在的模型。要更新模型，你必须先取回模型，设置任何你希望更新的属性，接着调用 `save` 方法。同样的，`updated_at` 时间戳将会自动被更新，所以不需要手动设置它的值：

    $flight = App\Flight::find(1);

    $flight->name = 'New Flight Name';

    $flight->save();

也可以针对符合指定查找的任意数量模型运行更新。在这个例子中，所有 `active` 并且 `destination` 为 `San Diego` 的航班，将会被标示为延迟：

    App\Flight::where('active', 1)
              ->where('destination', 'San Diego')
              ->update(['delayed' => 1]);

`update` 方法预期收到一个字段与值成对的数组，来代表应该被更新的字段。

<a name="mass-assignment"></a>
### 批量赋值

你也可以使用 `create` 方法来在一行间保存一个新的模型。被添加的模型实例将会从你的方法返回。然而，在这样做之前，你需要在你的模型上指定一个 `fillable` 或 `guarded` 属性，因为所有的 Eloquent 模型有针对批量赋值（Mass-Assignment）做保护。

当用户通过 HTTP 请求传入非预期的参数，并接着这些参数更改了数据库中你不打算要改的字段，就发生了批量赋值（Mass-Assignment）的漏洞。例如，恶意用户可能会通过 HTTP 请求发送 `is_admin` 参数，然后对应到你模型的 `create` 方法，这让该用户把自己升级为一个管理者。

所以，在开始之前，你应该定义你希望哪些模型属性是可以被批量赋值的。你可以在模型上借由 `$fillable` 属性达到这个。例如，让我们来使 `Flight` 模型的 `name` 属性可以被批量赋值：

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

一旦我们已经设置属性为可以被批量赋值的，我们可以使用 `create` 方法来添加一条新记录到数据库。`create` 方法返回已经被保存的模型实例：

    $flight = App\Flight::create(['name' => 'Flight 10']);

`$fillable` 作为一个可以被批量赋值的属性的「白名单」，然而你也可以选择使用 `$guarded`。`$guarded` 属性应该包含一个属性的数组，是你不想要被批量赋值的。所有不在数组里面的其他属性将会是可以被批量赋值的。所以，`$guarded` 的功能像是一个「黑名单」。当然，你应该使用 `$fillable` 或 `$guarded` - 而不是两者：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 不可以被批量赋值的属性。
         *
         * @var array
         */
        protected $guarded = ['price'];
    }

在上面的例子当中，所有属性 **除了 `price`** 以外，都可以被批量赋值。

#### 其他创建的方法

还有两种其他方法，你可以用来通过属性批量赋值创建你的模型：`firstOrCreate` 和 `firstOrNew`。`firstOrCreate` 方法将会使用指定的字段／值对，来尝试寻找数据库中的记录。如果在数据库找不到模型，将会用指定的属性来添加一条记录。

`firstOrNew` 方法类似 `firstOrCreate`，会尝试使用指定的属性在数据库中寻找符合的纪录。然而，假设找不到模型，将会返回一个新的模型实例。请注意 `firstOrnew` 返回的模型还尚未保存到数据库。你需要通过手动调用 `save` 方法来保存它：

    // 用属性取回航班，或如果不存在则创建它...
    $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

    // 用属性取回航班，或实例化一个新实例...
    $flight = App\Flight::firstOrNew(['name' => 'Flight 10']);

<a name="deleting-models"></a>
## 删除模型

要删除模型，必须在模型实例上调用 `delete` 方法：

    $flight = App\Flight::find(1);

    $flight->delete();

#### 通过键来删除现有的模型

在上面的例子中，我们在调用 `delete` 方法之前，先从数据库取回了模型。然而，如果你知道模型中的主键，你可以不取回模型就直接删除它。如果要这么做，请调用 `destroy` 方法：

    App\Flight::destroy(1);

    App\Flight::destroy([1, 2, 3]);

    App\Flight::destroy(1, 2, 3);

#### 通过查找来删除模型

当然，你也可以在一组模型上运行删除查找。在这个例子中，我们将会删除所有被标示为不活跃的航班：

    $deletedRows = App\Flight::where('active', 0)->delete();

<a name="soft-deleting"></a>
### 软删除

除了实际从数据库中移除记录，Eloquent 也可以「软删除」模型。当模型被软删除时，它们不是真的从数据库中被移除。而是 `deleted_at` 属性被设置在模型上并添加到数据库。如果模型有一个非空值的 `deleted_at`，代表模型已经被软删除了。要在模型启动软删除，必须在模型上使用 `Illuminate\Database\Eloquent\SoftDeletes` trait 并添加 `deleted_at` 字段到你的 `$dates` 属性：

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

当然，你应该添加 `deleted_at` 字段到你的数据表。Laravel [结构产生器](/docs/{{version}}/migrations)包含了一个辅助的方法用来创建这个字段：

    Schema::table('flights', function ($table) {
        $table->softDeletes();
    });

现在，当你在模型上调用 `delete` 方法，`deleted_at` 字段将会被设置成目前的日期和时间。而且，当查找有启用软删除的模型时，被软删除的模型将会自动从所有的查找结果中排除。

要确认指定的模型实例是否已经被软删除，可以使用 `trashed` 方法：

    if ($flight->trashed()) {
        //
    }

<a name="querying-soft-deleted-models"></a>
### 查找被软删除的模型

#### 包含被软删除的模型

如上面所述，被软删除的模型将会自动从所有的查找结果中排除。然而，你可以借由在查找上调用 `withTrashed` 方法， 强迫被软删除的模型出现在查找结果：

    $flights = App\Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();

`withTrashed` 方法也可以被用在[关联](/docs/{{version}}/eloquent-relationships)查找：

    $flight->history()->withTrashed()->get();

#### Where 子句注意事项

当在你的软删除模型的查找增加 `orWhere` 子句时，总是使用[高端 where 子句](http://laravel.com/docs/5.1/queries#advanced-where-clauses)将 `WHERE` 子句的逻辑分组。例如：

    User::where(function($query) {
            $query->where('name', '=', 'John')
                  ->orWhere('votes', '>', 100);
            })
            ->get();

这会产生以下的 SQL：

    select * from `users` where `users`.`deleted_at` is null and (`name` = 'John' or `votes` > 100)

如果 `orWhere` 子句没有被分组，它会在包含软删除的纪录产生以下的 SQL：

    select * from `users` where `users`.`deleted_at` is null and `name` = 'John' or `votes` > 100

#### 只获取被软删除的模型

`onlyTrashed` 方法会 **只** 获取被软删除的模型：

    $flights = App\Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();

#### 恢复被软删除的模型

有时候你可能希望「取消删除」一个被软删除的模型。要恢复一个被软删除的模型回到有效状态，必须在模型实例上使用 `restore` 方法：

    $flight->restore();

你也可以在查找上使用 `restore` 方法来快速地恢复多个模型：

    App\Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();

与 `withTrashed` 方法类似，`restore` 方法也可以被用在[关联](/docs/{{version}}/eloquent-relationships):

    $flight->history()->restore();

#### 永久地删除模型

有时候你可能需要真正地从数据库移除模型。要永久地从数据库移除一个被软删除的模型，必须使用 `forceDelete` 方法：

    // 强制删除单个模型实例...
    $flight->forceDelete();

    // 强制删除所有相关模型...
    $flight->history()->forceDelete();

<a name="query-scopes"></a>
## 查找范围

范围（Scopes）让你定义限制的共用集合，它可以轻松地在你的应用程序重复使用。例如，你可能需要频繁地获取所有被认为是「受欢迎的」用户。要定义范围，必须简单地在 Eloquent 模型方法前面加上前缀 `scope`：

范围应总是返回查询构造器的实例：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 限制查找只包括受欢迎的用户。
         *
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }

        /**
         * 限制查找只包括活跃的用户。
         *
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeActive($query)
        {
            return $query->where('active', 1);
        }
    }

#### 利用查找范围

一旦定义了范围，你可以在查找模型时调用范围方法。然而，当你调用方法时，你不需要加上 `scope` 前缀。你甚至能串接不同的范围调用，例如：

    $users = App\User::popular()->active()->orderBy('created_at')->get();

#### 动态范围

有时候，你可能希望定义一个接受参数的范围。只要在你的范围加上额外的参数即可。范围参数应该被定义在 `$query` 参数之后：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 限制查找只包括指定类型的用户。
         *
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeOfType($query, $type)
        {
            return $query->where('type', $type);
        }
    }

现在，你可以在调用范围时传递参数：

    $users = App\User::ofType('admin')->get();

<a name="events"></a>
## 事件

Eloquent 模型会触发许多事件，让你可以借由以下的方法，在模型的生命周期的多个时间点进行操作：`creating`、`created`、`updating`、`updated`、`saving`、`saved`、`deleting`、`deleted`、`restoring`、`restored`。事件让你可以在每次特定的模型类在数据库被保存或更新时，简单地运行代码。

<a name="basic-usage"></a>
### 基本用法

当一个新模型初次被保存，将会触发 `creating` 以及 `created` 事件。如果一个模型已经存在于数据库而且调用了 `save` 方法，将会触发 `updating` 和 `updated` 事件。然而，在这两个状况下，都将会触发 `saving` 和 `saved` 事件。

例如，让我们来在[服务提供者](/docs/{{version}}/providers)中定义一个 Eloquent 事件侦听器。在我们的事件侦听器中，我们会在指定的模型上调用 `isValid` 方法，并在模型无效的时候返回 `false`。从 Eloquent 事件侦听器返回 `false` 会取消 `save` 和 `update` 的操作：

    <?php

    namespace App\Providers;

    use App\User;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 启动所有应用程序服务。
         *
         * @return void
         */
        public function boot()
        {
            User::creating(function ($user) {
                if ( ! $user->isValid()) {
                    return false;
                }
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
