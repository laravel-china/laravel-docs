# 辅助函数

- [简介](#introduction)
- [可用方法](#available-methods)

<a name="introduction"></a>
## 简介

Laravel 包含有各种各样的PHP辅助函数，许多都是在 Laravel 自身框架中使用到。如果你觉得实用，也可以在你自己的应用中使用它们。

<a name="available-methods"></a>
## 可用方法

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

### 数组

<div class="collection-method-list" markdown="1">

[array_add](#method-array-add)
[array_collapse](#method-array-collapse)
[array_divide](#method-array-divide)
[array_dot](#method-array-dot)
[array_except](#method-array-except)
[array_first](#method-array-first)
[array_flatten](#method-array-flatten)
[array_forget](#method-array-forget)
[array_get](#method-array-get)
[array_has](#method-array-has)
[array_last](#method-array-last)
[array_only](#method-array-only)
[array_pluck](#method-array-pluck)
[array_prepend](#method-array-prepend)
[array_pull](#method-array-pull)
[array_set](#method-array-set)
[array_sort](#method-array-sort)
[array_sort_recursive](#method-array-sort-recursive)
[array_where](#method-array-where)
[head](#method-head)
[last](#method-last)
</div>

### 路径

<div class="collection-method-list" markdown="1">

[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[mix](#method-mix)
[public_path](#method-public-path)
[resource_path](#method-resource-path)
[storage_path](#method-storage-path)

</div>

### 字符串

<div class="collection-method-list" markdown="1">

[camel_case](#method-camel-case)
[class_basename](#method-class-basename)
[e](#method-e)
[ends_with](#method-ends-with)
[snake_case](#method-snake-case)
[str_limit](#method-str-limit)
[starts_with](#method-starts-with)
[str_contains](#method-str-contains)
[str_finish](#method-str-finish)
[str_is](#method-str-is)
[str_plural](#method-str-plural)
[str_random](#method-str-random)
[str_singular](#method-str-singular)
[str_slug](#method-str-slug)
[studly_case](#method-studly-case)
[title_case](#method-title-case)
[trans](#method-trans)
[trans_choice](#method-trans-choice)

</div>

### URLs

<div class="collection-method-list" markdown="1">

[action](#method-action)
[asset](#method-asset)
[secure_asset](#method-secure-asset)
[route](#method-route)
[url](#method-url)

</div>

### 其他

<div class="collection-method-list" markdown="1">

[abort](#method-abort)
[abort_if](#method-abort-if)
[abort_unless](#method-abort-unless)
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[collect](#method-collect)
[config](#method-config)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[dd](#method-dd)
[dispatch](#method-dispatch)
[env](#method-env)
[event](#method-event)
[factory](#method-factory)
[info](#method-info)
[logger](#method-logger)
[method_field](#method-method-field)
[retry](#method-retry)
[old](#method-old)
[redirect](#method-redirect)
[request](#method-request)
[response](#method-response)
[session](#method-session)
[value](#method-value)
[view](#method-view)

</div>

<a name="method-listing"></a>
## 方法列表

<style>
    #collection-method code {
        font-size: 14px;
    }

    #collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="arrays"></a>
## 数组

<a name="method-array-add"></a>
#### `array_add()` {#collection-method .first-collection-method}

如果给定的键不存在与数组中，`array_add` 就会把给定的键值对添加到数组中：

    $array = array_add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>
#### `array_collapse()` {#collection-method}

`array_collapse` 函数把数组里的每一个数组合并成单个数组：

    $array = array_collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-divide"></a>
#### `array_divide()` {#collection-method}

`array_divide` 函数返回两个数组，一个包含原本数组的键，另一个包含原本数组的值：

    list($keys, $values) = array_divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `array_dot()` {#collection-method}

`array_dot` 函数把多维数组压制成一维数组，并用「点」式语法表示深度：

    $array = array_dot(['foo' => ['bar' => 'baz']]);

    // ['foo.bar' => 'baz'];

<a name="method-array-except"></a>
#### `array_except()` {#collection-method}

`array_except` 函数从数组移除指定的键值对：

    $array = ['name' => 'Desk', 'price' => 100];

    $array = array_except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-first"></a>
#### `array_first()` {#collection-method}

`array_first` 函数返回数组中第一个通过指定测试的元素：

    $array = [100, 200, 300];

    $value = array_first($array, function ($value, $key) {
        return $value >= 150;
    });

    // 200

可传递第三个参数作为默认值。当没有元素通过测试时，将会返回该默认值：

    $value = array_first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `array_flatten()` {#collection-method}

`array_flatten`  函数将多维数组压制成一维数组：

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $array = array_flatten($array);

    // ['Joe', 'PHP', 'Ruby'];

<a name="method-array-forget"></a>
#### `array_forget()` {#collection-method}

`array_forget` 函数以「点」式语法从深度嵌套的数组中移除指定的键值对：

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `array_get()` {#collection-method}

`array_get` 函数使用「点」式语法从深度嵌套的数组中获取指定的值：

    $array = ['products' => ['desk' => ['price' => 100]]];

    $value = array_get($array, 'products.desk');

    // ['price' => 100]

`array_get` 函数同样也接受默认值，如果指定的键找不到时，则返回该默认值：

    $value = array_get($array, 'names.john', 'default');

<a name="method-array-has"></a>
#### `array_has()` {#collection-method}

`array_has` 函数使用「点」式语法检查指定的项目是否存在于数组中：

    $array = ['products' => ['desk' => ['price' => 100]]];

    $hasDesk = array_has($array, 'products.desk');

    // true

<a name="method-array-last"></a>
#### `array_last()` {#collection-method}

`array_last` 函数返回数组中最后一个通过指定测试的元素：

    $array = [100, 200, 300, 110];

    $value = array_last($array, function ($value, $key) {
        return $value >= 150;
    });

    // 300

<a name="method-array-only"></a>
#### `array_only()` {#collection-method}

`array_only` 函数从数组返回指定的键值对：

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $array = array_only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `array_pluck()` {#collection-method}

`array_pluck` 函数从数组拉出一列指定的键值对：

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $array = array_pluck($array, 'developer.name');

    // ['Taylor', 'Abigail'];

你也可以指定要以什么作为结果列的键名：

    $array = array_pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail'];

<a name="method-array-prepend"></a>
#### `array_prepend()` {#collection-method}

`array_prepend` 函数将元素加到数组的头部：

    $array = ['one', 'two', 'three', 'four'];

    $array = array_prepend($array, 'zero');

    // $array: ['zero', 'one', 'two', 'three', 'four']

<a name="method-array-pull"></a>
#### `array_pull()` {#collection-method}

`array_pull` 函数从数组移除指定键值对并返回该键值对：

    $array = ['name' => 'Desk', 'price' => 100];

    $name = array_pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

<a name="method-array-set"></a>
#### `array_set()` {#collection-method}

`array_set` 函数使用「点」式语法在深度嵌套的数组中写入值：

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-sort"></a>
#### `array_sort()` {#collection-method}

`array_sort` 函数根据指定闭包的结果排序数组：

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Chair'],
    ];

    $array = array_values(array_sort($array, function ($value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `array_sort_recursive()` {#collection-method}

`array_sort_recursive` 函数使用 sort 函数递归排序数组：

    $array = [
        [
            'Roman',
            'Taylor',
            'Li',
        ],
        [
            'PHP',
            'Ruby',
            'JavaScript',
        ],
    ];

    $array = array_sort_recursive($array);

    /*
        [
            [
                'Li',
                'Roman',
                'Taylor',
            ],
            [
                'JavaScript',
                'PHP',
                'Ruby',
            ]
        ];
    */

<a name="method-array-where"></a>
#### `array_where()` {#collection-method}

`array_where` 函数使用指定的闭包过滤数组：

    $array = [100, '200', 300, '400', 500];

    $array = array_where($array, function ($value, $key) {
    return is_string($value);
    });

    // [1 => 200, 3 => 400]

<a name="method-head"></a>
#### `head()` {#collection-method}

`head` 函数返回指定数组的第一个元素：

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()` {#collection-method}

`last` 函数返回指定数组的最后一个元素：

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="paths"></a>
## 路径

<a name="method-app-path"></a>
#### `app_path()` {#collection-method}

`app_path` 函数返回 `app` 文件夹的完整路径。你也可以使用 `app_path` 函数生成针对指定文件相对于 app 目录的完整路径：

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {#collection-method}

`base_path` 函数返回项目根目录的完整路径。你也可以使用 `base_path` 函数生成针对指定文件相对于项目根目录的完整路径：

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {#collection-method}

`config_path` 函数返回 `config` 目录的完整路径：

    $path = config_path();

<a name="method-database-path"></a>
#### `database_path()` {#collection-method}

`database_path` 函数返回 `database` 目录的完整路径：

    $path = database_path();

<a name="method-mix"></a>
#### `mix()` {#collection-method}

`mix` 函数获取带有版本号的 [versioned mix file](/docs/{{version}}/mix):

    mix($file);

<a name="method-public-path"></a>
#### `public_path()` {#collection-method}

`public_path` 函数返回 `public` 目录的完整路径：

    $path = public_path();

<a name="method-resource-path"></a>
#### `resource_path()` {#collection-method}

`resource_path` 函数返回 `resources` 目录的完整路径。你也可以使用 `resource_path` 函数生成针对指定文件相对于 `resources` 目录的完整路径：

    $path = resource_path();

    $path = resource_path('assets/sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

`storage_path` 函数返回 `storage` 目录的完整路径。你也可以使用 `storage_path` 函数生成针对指定文件相对于 `storage` 目录的完整路径：

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## 字符串

<a name="method-camel-case"></a>
#### `camel_case()` {#collection-method}

`camel_case` 函数将指定的字符串转换成 `驼峰式命名`：

    $camel = camel_case('foo_bar');

    // fooBar

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

`class_basename` 函数返回不包含命名空间的类名称：

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {#collection-method}

`e` 函数对指定字符串进行 `htmlentities`：

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-ends-with"></a>
#### `ends_with()` {#collection-method}

`ends_with` 函数判断指定字符串结尾是否为指定内容：

    $value = ends_with('This is my name', 'name');

    // true

<a name="method-snake-case"></a>
#### `snake_case()` {#collection-method}

`snake_case` 函数将指定的字符串转换成 `蛇形命名` ：

    $snake = snake_case('fooBar');

    // foo_bar

<a name="method-str-limit"></a>
#### `str_limit()` {#collection-method}

`str_limit` 函数限制字符串的字符个数，该函数接受一个字符串作为第一个参数，第二个参数为允许的最大字符个数：

    $value = str_limit('The PHP framework for web artisans.', 7);

    // The PHP...

<a name="method-starts-with"></a>
#### `starts_with()` {#collection-method}

`starts_with` 函数判断字符串开头是否为指定内容：

    $value = starts_with('This is my name', 'This');

    // true

<a name="method-str-contains"></a>
#### `str_contains()` {#collection-method}

`str_contains` 函数判断字符串是否包含有指定内容：

    $value = str_contains('This is my name', 'my');

    // true

<a name="method-str-finish"></a>
#### `str_finish()` {#collection-method}

`str_finish` 函数添加指定内容到字符串末尾：

    $string = str_finish('this/string', '/');

    // this/string/

<a name="method-str-is"></a>
#### `str_is()` {#collection-method}

`str_is` 函数判断指定的字符串是否匹配指定的格式，星号可作为通配符使用：

    $value = str_is('foo*', 'foobar');

    // true

    $value = str_is('baz*', 'foobar');

    // false

<a name="method-str-plural"></a>
#### `str_plural()` {#collection-method}

`str_plural` 函数把字符串转换成复数形式。该函数目前只支持英文：

    $plural = str_plural('car');

    // cars

    $plural = str_plural('child');

    // children

你可以传入一个整数作为第二个参数，来获取字符串的单数或复数形式：

    $plural = str_plural('child', 2);

    // children

    $plural = str_plural('child', 1);

    // child

<a name="method-str-random"></a>
#### `str_random()` {#collection-method}

`str_random` 函数生成指定长度的随机字符串。该函数使用了 PHP 自带的 `random_bytes` 函数：

    $string = str_random(40);

<a name="method-str-singular"></a>
#### `str_singular()` {#collection-method}

`str_singular` 函数把字符串转换成单数形式。该函数目前只支持英文：

    $singular = str_singular('cars');

    // car

<a name="method-str-slug"></a>
#### `str_slug()` {#collection-method}

`str_slug` 函数根据指定字符串生成 URL 友好的「slug」：

    $title = str_slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-studly-case"></a>
#### `studly_case()` {#collection-method}

`studly_case` 函数把指定字符串转换成 `首字母大写`：

    $value = studly_case('foo_bar');

    // FooBar

<a name="method-title-case"></a>
#### `title_case()` {#collection-method}

`title_case` 函数把指定字符串转换成 `每个单词首字母大写`：

    $title = title_case('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-trans"></a>
#### `trans()` {#collection-method}

`trans` 函数根据你的 [本地化文件](/docs/{{version}}/localization) 翻译指定的语句：

    echo trans('validation.required'):

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

`trans_choice` 函数根据数量翻译指定的语句：

    $value = trans_choice('foo.bar', $count);

<a name="urls"></a>
## URLs

<a name="method-action"></a>
#### `action()` {#collection-method}

`action` 函数根据指定控制器的方法生成 URL，你不需要传入该控制器的完整命名空间。只需要传入相对于 `App\Http\Controllers` 命名空间的控制器类名：

    $url = action('HomeController@getIndex');

如果该方法接受路由参数，可以作为第二个参数传入：

    $url = action('UserController@profile', ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {#collection-method}

根据当前请求的协议（HTTP 或 HTTPS）生成资源文件的 URL：

	$url = asset('img/photo.jpg');

<a name="method-secure-asset"></a>
#### `secure_asset()` {#collection-method}

使用 HTTPS 协议生成资源文件的 URL：

	echo secure_asset('foo/bar.zip', $title, $attributes = []);

<a name="method-route"></a>
#### `route()` {#collection-method}

`route` 函数生成指定路由名称的 URL：

    $url = route('routeName');

如果该路由接受参数，可以作为第二个参数传入：

    $url = route('routeName', ['id' => 1]);

<a name="method-url"></a>
#### `url()` {#collection-method}

`url` 函数生成指定路径的完整 URL：

    echo url('user/profile');

    echo url('user/profile', [1]);

如果没有提供路径参数，将会返回一个 `Illuminate\Routing\UrlGenerator` 实例：

    echo url()->current();
    echo url()->full();
    echo url()->previous();

<a name="miscellaneous"></a>
## 其他

<a name="method-abort"></a>
#### `abort()` {#collection-method}

`abort` 函数抛出一个将被异常处理句柄渲染的 HTTP 异常：

    abort(401);

你也可以传入异常的响应消息：

    abort(401, 'Unauthorized.');

<a name="method-abort-if"></a>
#### `abort_if()` {#collection-method}

`abort_if` 函数如果指定的布尔表达式值为 `true` 则抛出一个 HTTP 异常：

    abort_if(! Auth::user()->isAdmin(), 403);

<a name="method-abort-unless"></a>
#### `abort_unless()` {#collection-method}

`abort_unless` 函数如果指定的布尔表达式值为 `false` 则抛出一个 HTTP 异常：

    abort_unless(Auth::user()->isAdmin(), 403);

<a name="method-auth"></a>
#### `auth()` {#collection-method}

`auth` 函数返回一个 authenticator 实例，可以使用它来代替 Auth facade：

    $user = auth()->user();

<a name="method-back"></a>
#### `back()` {#collection-method}

`back()` 函数生成一个重定向响应让用户返回到之前的位置：

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {#collection-method}

`bcrypt` 函数使用 Bcrypt 算法哈希指定的数值。你可以使用它代替 `Hash` facade：

    $password = bcrypt('my-secret-password');

<a name="method-cache"></a>
#### `cache()` {#cache-method}

`cache` 函数尝试从缓存获取给定 `key` 的值。如果 `key` 不存在则返回默认值：

    $value = cache('key');

    $value = cache('key', 'default');

同时，你也可以传递键值对来设置缓存，第二个参数可以指定缓存的过期时间，单位分钟：

    cache(['key' => 'value'], 5);

    cache(['key' => 'value'], Carbon::now()->addSeconds(10));


<a name="method-collect"></a>
#### `collect()` {#collection-method}

`collect` 函数根据指定的数组生成 [集合](/docs/{{version}}/collections) 实例：

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {#collection-method}

`config` 函数用于获取配置信息的值，配置信息的值可通过「点」式语法访问，其中包含要访问的文件名以及选项名。可传递一个默认值作为第二参数，当配置信息不存在时，则返回该默认值：

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

`config` 辅助函数也可以在运行期间，根据指定的键值对设置指定的配置信息：

    config(['app.debug' => true]);

<a name="method-csrf-field"></a>
#### `csrf_field()` {#collection-method}

`csrf_field` 函数生成包含 CSRF 令牌内容的 HTML 表单隐藏字段。例如，使用 [Blade 语法](/docs/{{version}}/blade)：

    {{ csrf_field() }}

<a name="method-csrf-token"></a>
#### `csrf_token()` {#collection-method}

`csrf_token` 函数获取当前 CSRF 令牌的内容：

    $token = csrf_token();

<a name="method-dd"></a>
#### `dd()` {#collection-method}

`dd` 函数输出指定变量的值并终止脚本运行：

    dd($value);

    dd($value1, $value2, $value3, ...);

如果你不想终止脚本运行，使用 `dump` 函数代替：

    dump($value);

<a name="method-dispatch"></a>
#### `dispatch()` {#collection-method}

`dispatch` 函数把一个新任务推送到 Laravel 的 [任务队列](/docs/{{version}}/queues)中：

    dispatch(new App\Jobs\SendEmails);

<a name="method-env"></a>
#### `env()` {#collection-method}

`env` 函数获取环境变量值或返回默认值：

    $env = env('APP_ENV');

    // Return a default value if the variable doesn't exist...
    $env = env('APP_ENV', 'production');

<a name="method-event"></a>
#### `event()` {#collection-method}

`event` 函数发派指定的 [事件](/docs/{{version}}/events) 到所属的侦听器：

    event(new UserRegistered($user));

<a name="method-factory"></a>
#### `factory()` {#collection-method}

`factory`  函数根据指定类、名称以及数量生成模型工厂构造器（model factory builder）。可用于 [测试](/docs/{{version}}/database-testing#writing-factories) 或 [数据填充](/docs/{{version}}/seeding#using-model-factories)：

    $user = factory(App\User::class)->make();

<a name="method-info"></a>
#### `info()` {#collection-method}

`info` 函数以 `info` 级别写入日志:

    info('Some helpful information!');

<a name="method-logger"></a>
#### `logger()` {#collection-method}

`logger` 函数以 `debug` 级别写入日志:

    logger('Debug message');

同时支持传入数组作为参数：

    logger('User has logged in.', ['id' => $user->id]);

如果没有传入参数，则会返回一个 [日志](/docs/{{version}}/errors#logging) 的实例：

    logger()->error('You are not allowed here.');

<a name="method-method_field"></a>
#### `method_field()` {#collection-method}

`method_field` 函数生成模拟表单 HTTP 动词的 HTML 表单隐藏字段。例如，使用 [Blade 语法](/docs/{{version}}/blade)：

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>

<a name="method-old"></a>
#### `old()` {#collection-method}

`old` 函数 [获取](/docs/{{version}}/requests#retrieving-input) session 内一次性的旧有输入值：

    $value = old('value');

    $value = old('value', 'default');

<a name="method-redirect"></a>
#### `redirect()` {#collection-method}

`redirect` 函数返回一个 HTTP 重定向响应，如果调用时没有传入参数则返回 redirector 实例：

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-request"></a>
#### `request()` {#collection-method}

`request` 函数返回当前 [请求](/docs/{{version}}/requests) 实例或获取输入的项目：

    $request = request();

    $value = request('key', $default = null)

<a name="method-response"></a>
#### `response()` {#collection-method}

`response` 函数创建一个 [响应](/docs/{{version}}/responses) 实例或获取一个 response 工厂实例：

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>
#### `retry()` {#collection-method}

`retry` 函数将会重复调用给定的回调函数，最多调用指定的次数。如果回调函数没有抛出异常并且有值返回，则 `retry` 函数返回该值。如果回调函数抛出异常，`retry` 函数将拦截异常并自动再次调用回调函数，直到调用给定的次数。如果重试次数超出给定次数，拦截的异常将会抛出：

  return retry(5, function () {
      // Attempt 5 times while resting 100ms in between attempts...
  }, 100);

<a name="method-session"></a>
#### `session()` {#collection-method}

`session` 函数可用于获取或设置单个 session 项：

    $value = session('key');

你可以通过传递键值对数组给该函数设置 session 项：

    session(['chairs' => 7, 'instruments' => 3]);

该函数在没有传递参数时，将返回 session 实例：

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-value"></a>
#### `value()` {#collection-method}

`value` 函数返回指定数值。而当你传递一个 `闭包` 给该函数时，该 `闭包` 将被运行并返回该 `闭包` 的运行结果：

    $value = value(function() { return 'bar'; });

<a name="method-view"></a>
#### `view()` {#collection-method}

`view` 函数获取 [视图](/docs/{{version}}/views) 实例：

    return view('auth.login');


| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@zyxcba](https://github.com/cmzz)  | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/6111715?v=3&s=100">  |  翻译  | [淘亿联盟](http://wewx.cn) - 免费的淘宝客优惠券CMS |
