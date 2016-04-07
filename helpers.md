# 辅助方法

- [简介](#introduction)
- [可用方法](#available-methods)

<a name="introduction"></a>
## 简介

Laravel 包含一群多样化的 PHP 辅助方法函数。许多在 Laravel 自身框架中使用；如果觉得实用，也可以在你的应用当中使用。

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
[array_only](#method-array-only)
[array_pluck](#method-array-pluck)
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
[elixir](#method-elixir)
[public_path](#method-public-path)
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
[trans](#method-trans)
[trans_choice](#method-trans-choice)
</div>

### 网址

<div class="collection-method-list" markdown="1">
[action](#method-action)
[asset](#method-asset)
[secure_asset](#method-secure-asset)
[route](#method-route)
[url](#method-url)
</div>

### 其他

<div class="collection-method-list" markdown="1">
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[collect](#method-collect)
[config](#method-config)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[dd](#method-dd)
[env](#method-env)
[event](#method-event)
[factory](#method-factory)
[method_field](#method-method-field)
[old](#method-old)
[redirect](#method-redirect)
[request](#method-request)
[response](#method-response)
[session](#method-session)
[value](#method-value)
[view](#method-view)
[with](#method-with)
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

如果给定的键不存在于该数组，`array_add` 函数将给定的键值对加到数组中：

    $array = array_add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>
#### `array_collapse()` {#collection-method}

`array_collapse` 函数将数组的每一个数组折成单一数组：

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

`array_dot` 函数把多维数组扁平化成一维数组，并用「点」式语法表示深度：

    $array = array_dot(['foo' => ['bar' => 'baz']]);

    // ['foo.bar' => 'baz'];

<a name="method-array-except"></a>
#### `array_except()` {#collection-method}

`array_except` 函数从数组移除给定的键值对：

    $array = ['name' => 'Desk', 'price' => 100];

    $array = array_except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-first"></a>
#### `array_first()` {#collection-method}

`array_first` 函数返回数组中第一个通过为真测试的元素：

    $array = [100, 200, 300];

    $value = array_first($array, function ($key, $value) {
        return $value >= 150;
    });

    // 200

可传递第三个参数作为默认值。当没有任何数值通过测试时将返回该数值：

    $value = array_first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `array_flatten()` {#collection-method}

`array_flatten` 函数将多维数组扁平化成一维。

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $array = array_flatten($array);

    // ['Joe', 'PHP', 'Ruby'];

<a name="method-array-forget"></a>
#### `array_forget()` {#collection-method}

`array_forget` 函数以「点」式语法从深度嵌套数组移除给定的键值对：

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `array_get()` {#collection-method}

`array_get` 函数使用「点」式语法从深度嵌套数组取回给定的值：

    $array = ['products' => ['desk' => ['price' => 100]]];

    $value = array_get($array, 'products.desk');

    // ['price' => 100]

`array_get` 函数同样接受默认值，当指定的键找不到时返回：

    $value = array_get($array, 'names.john', 'default');

<a name="method-array-has"></a>
#### `array_has()` {#collection-method}

`array_has` 函数使用「点」式语法检查给定的项目是否存在于数组中：

    $array = ['products' => ['desk' => ['price' => 100]]];

    $hasDesk = array_has($array, ['products.desk']);

    // true

<a name="method-array-only"></a>
#### `array_only()` {#collection-method}

`array_only` 函数从数组返回给定的键值对：

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $array = array_only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `array_pluck()` {#collection-method}

`array_pluck` 函数从数组拉出一列给定的键值对：

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $array = array_pluck($array, 'developer.name');

    // ['Taylor', 'Abigail'];

你也能指定以什么做为结果列的键值：

    $array = array_pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail'];

<a name="method-array-pull"></a>
#### `array_pull()` {#collection-method}

`array_pull` 函数从数组移除并返回给定的键值对：

    $array = ['name' => 'Desk', 'price' => 100];

    $name = array_pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

<a name="method-array-set"></a>
#### `array_set()` {#collection-method}

`array_set` 函数使用「点」式语法在深度嵌套数组中写入值：

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-sort"></a>
#### `array_sort()` {#collection-method}

`array_sort` 函数借由给定闭包结果排序数组：

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

`array_sort_recursive` 函数使用 `sort` 函数递归排序数组：

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

`array_where` 函数使用给定的闭包过滤数组：

    $array = [100, '200', 300, '400', 500];

    $array = array_where($array, function ($key, $value) {
        return is_string($value);
    });

    // [1 => 200, 3 => 400]

<a name="method-head"></a>
#### `head()` {#collection-method}

`head` 函数返回给定数组的第一个元素：

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()` {#collection-method}

`last` 函数返回给定数组的最后一个元素：

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="paths"></a>
## 路径

<a name="method-app-path"></a>
#### `app_path()` {#collection-method}

`app_path` 函数获取 `app` 文件夹的完整路径：

    $path = app_path();

你同样可以使用 `app_path` 函数产生针对给定文件相对于 app 目录的完整路径：

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {#collection-method}

`base_path` 函数获取项目根目录的完整路径：

    $path = base_path();

你同样可以使用 `base_path` 函数产生针对给定文件相对于项目根目录的完整路径：

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {#collection-method}

`config_path` 函数获取应用配置目录的完整路径：

    $path = config_path();

<a name="method-database-path"></a>
#### `database_path()` {#collection-method}

`database_path` 函数获取应用数据库目录的完整路径：

    $path = database_path();

<a name="method-elixir"></a>
#### `elixir()` {#collection-method}

`elixir` 函数获取加上版本号的 [Elixir](/docs/{{version}}/elixir) 文件路径：

    elixir($file);

<a name="method-public-path"></a>
#### `public_path()` {#collection-method}

`public_path` 函数获取 `public` 目录的完整路径：

    $path = public_path();

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

`storage_path` 函数获取 `storage` 目录的完整路径：

    $path = storage_path();

你同样可以使用 `storage_path` 函数产生针对给定文件相对于 storage 目录的完整路径：

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## 字符串

<a name="method-camel-case"></a>
#### `camel_case()` {#collection-method}

`camel_case` 函数会将给定的字符串转换成 `驼峰式命名`：

    $camel = camel_case('foo_bar');

    // fooBar

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

`class_basename` 返回不包含命名空间的类名称：

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {#collection-method}

`e` 函数对给定字符串运行 `htmlentities`：

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-ends-with"></a>
#### `ends_with()` {#collection-method}

`ends_with` 函数判断给定字符串结尾是否为指定内容：

    $value = ends_with('This is my name', 'name');

    // true

<a name="method-snake-case"></a>
#### `snake_case()` {#collection-method}

`snake_case` 函数会将给定的字符串转换成 `蛇形命名`：

    $snake = snake_case('fooBar');

    // foo_bar

<a name="method-str-limit"></a>
#### `str_limit()` {#collection-method}

`str_limit` 函数限制字符串的字符数量。该函数接受一个字符串作为第一个参数，以及最大字符数量作为第二参数：

    $value = str_limit('The PHP framework for web artisans.', 7);

    // The PHP...

<a name="method-starts-with"></a>
#### `starts_with()` {#collection-method}

`starts_with` 函数判断字符串开头是否为给定内容：

    $value = starts_with('This is my name', 'This');

    // true

<a name="method-str-contains"></a>
#### `str_contains()` {#collection-method}

`str_contains` 函数判断给定字符串是否包含指定内容：

    $value = str_contains('This is my name', 'my');

    // true

<a name="method-str-finish"></a>
#### `str_finish()` {#collection-method}

`str_finish` 函数添加给定内容到字符串结尾：

    $string = str_finish('this/string', '/');

    // this/string/

<a name="method-str-is"></a>
#### `str_is()` {#collection-method}

`str_is` 函数判断给定的字符串与给定的格式是否符合。星号可作为通配符使用：

    $value = str_is('foo*', 'foobar');

    // true

    $value = str_is('baz*', 'foobar');

    // false

<a name="method-str-plural"></a>
#### `str_plural()` {#collection-method}

`str_plural` 函数转换字符串成复数形。该函数目前仅支持英文：

    $plural = str_plural('car');

    // cars

    $plural = str_plural('child');

    // children

你能提供一整数做为第二参数，获取字符串的单数或复数形：

    $plural = str_plural('child', 2);

    // children

    $plural = str_plural('child', 1);

    // child

<a name="method-str-random"></a>
#### `str_random()` {#collection-method}

`str_random` 函数产生给定长度的随机字符串：

    $string = str_random(40);

<a name="method-str-singular"></a>
#### `str_singular()` {#collection-method}

`str_singular` 函数转换字符串成单数形。该函数目前仅支持英文：

    $singular = str_singular('cars');

    // car

<a name="method-str-slug"></a>
#### `str_slug()` {#collection-method}

`str_slug` 函数从给定字符串产生网址友善的「slug」：

    $title = str_slug("Laravel 5 Framework", "-");

    // laravel-5-framework

<a name="method-studly-case"></a>
#### `studly_case()` {#collection-method}

`studly_case` 函数将给定字符串转换成 `首字大写命名`：

    $value = studly_case('foo_bar');

    // FooBar

<a name="method-trans"></a>
#### `trans()` {#collection-method}

`trans` 函数根据你的[本地化文件](/docs/{{version}}/localization)翻译给定的语句：

    echo trans('validation.required'):

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

`trans_choice` 函数根据后缀变化翻译给定的语句：

    $value = trans_choice('foo.bar', $count);

<a name="urls"></a>
## 网址

<a name="method-action"></a>
#### `action()` {#collection-method}

`action` 函数产生给定控制器行为网址。你不需要输入该控制器的完整命名空间。作为替代，请输入基于 `App\Http\Controllers` 命名空间的控制器类名称：

    $url = action('HomeController@getIndex');

如果该方法支持路由参数，你可以作为第二参数传递：

    $url = action('UserController@profile', ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {#collection-method}

根据目前请求的协定（HTTP 或 HTTPS）产生资源文件网址：

	$url = asset('img/photo.jpg');

<a name="method-secure-asset"></a>
#### `secure_asset()` {#collection-method}

根据 HTTPS 产生资源文件网址：

	echo secure_asset('foo/bar.zip', $title, $attributes = []);

<a name="method-route"></a>
#### `route()` {#collection-method}

`route` 函数产生给定路由名称网址：

    $url = route('routeName');

如果该路由接受参数，你可以作为第二参数传递：

    $url = route('routeName', ['id' => 1]);

<a name="method-url"></a>
#### `url()` {#collection-method}

`url` 函数产生给定路径的完整网址：

    echo url('user/profile');

    echo url('user/profile', [1]);

<a name="miscellaneous"></a>
## 其他

<a name="method-auth"></a>
#### `auth()` {#collection-method}

`auth` 函数返回一个认证器实例。你可以使用它取代 `Auth` facade：

    $user = auth()->user();

<a name="method-back"></a>
#### `back()` {#collection-method}

`back()` 函数产生一个重定向回应让用户回到之前的位置：

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {#collection-method}

`bcrypt` 函数使用 Bcrypt 哈希给定的数值。你可以使用它替代 `Hash` facade：

    $password = bcrypt('my-secret-password');

<a name="method-collect"></a>
#### `collect()` {#collection-method}

`collect` 函数从给定的项目产生[集合](/docs/{{version}}/collections)实例：

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {#collection-method}

`config` 获取设置选项的设置值。设置值可通过「点」式语法读取，其中包含要访问的文件名以及选项名称。可传递一默认值在找不到指定的设置选项时返回该数值：

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

`config` 辅助方法也可以在运行期间，根据给定的键值对指定设置值：

    config(['app.debug' => true]);

<a name="method-csrf-field"></a>
#### `csrf_field()` {#collection-method}

`csrf_field` 函数产生包含 CSRF 令牌内容的 HTML 表单隐藏字段。例如，使用 [Blade 语法](/docs/{{version}}/blade)：

    {!! csrf_field() !!}

<a name="method-csrf-token"></a>
#### `csrf_token()` {#collection-method}

`csrf_token` 函数获取当前 CSRF 令牌的内容：

    $token = csrf_token();

<a name="method-dd"></a>
#### `dd()` {#collection-method}

`dd` 函数输出给定变量并结束脚本运行：

    dd($value);

<a name="method-env"></a>
#### `env()` {#collection-method}

`env` 函数获取环境变量值或返回默认值：

    $env = env('APP_ENV');

    // Return a default value if the variable doesn't exist...
    $env = env('APP_ENV', 'production');

<a name="method-event"></a>
#### `event()` {#collection-method}

`event` 函数配送给定[事件](/docs/{{version}}/events)到所属的侦听器：

    event(new UserRegistered($user));

<a name="method-factory"></a>
#### `factory()` {#collection-method}

`factory` 函数根据给定类、名称以及总数产生模型工厂建构器（model factory builder）。可用于[测试](/docs/{{version}}/testing#model-factories)或[数据填充](/docs/{{version}}/seeding#using-model-factories)：

    $user = factory(App\User::class)->make();

<a name="method-method-field"></a>
#### `method_field()` {#collection-method}

`method_field` 函数产生拟造 HTTP 表单动作内容的 HTML 表单隐藏字段。例如，使用 [Blade 语法](/docs/{{version}}/blade)：

    <form method="POST">
        {!! method_field('delete') !!}
    </form>

<a name="method-old"></a>
#### `old()` {#collection-method}

`old` 函数[获取](/docs/{{version}}/requests#retrieving-input)快闪到 session 的旧有输入数值：

    $value = old('value');

<a name="method-redirect"></a>
#### `redirect()` {#collection-method}

`redirect` 函数返回重定向器实例以进行 [重定向](/docs/{{version}}/responses#redirects)：

    return redirect('/home');

<a name="method-request"></a>
#### `request()` {#collection-method}

`request` 函数获取目前的[请求](/docs/{{version}}/requests)实例或输入的项目：

    $request = request();

    $value = request('key', $default = null)

<a name="method-response"></a>
#### `response()` {#collection-method}

`response` 函数创建一个[回应](/docs/{{version}}/responses)实例或获取一个回应工厂（response factory）实例：

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-session"></a>
#### `session()` {#collection-method}

`session` 函数可被用于获取或设置单一 session 内容：

    $value = session('key');

你可以通过传递键值对给该函数进行内容设置：

    session(['chairs' => 7, 'instruments' => 3]);

该函数在没有传递参数时，将返回 session 实例：

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-value"></a>
#### `value()` {#collection-method}

`value` 函数返回给定数值。而当你传递一个 `闭包` 给该函数，该 `闭包` 将被运行并返回结果：

    $value = value(function() { return 'bar'; });

<a name="method-view"></a>
#### `view()` {#collection-method}

`view` 函数获取[视图](/docs/{{version}}/views) 实例：

    return view('auth.login');

<a name="method-with"></a>
#### `with()` {#collection-method}

`with` 函数返回给定的数值。该函数主要用于链式调用回所保存的 seesion 内容，除此之外不太可能用到：

    $value = with(new Foo)->work();
