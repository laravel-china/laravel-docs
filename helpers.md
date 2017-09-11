# Laravel 的辅助函数列表

- [简介](#introduction)
- [可用方法](#available-methods)

<a name="introduction"></a>
## 简介

Laravel 包含各种各样的全局「辅助」PHP 函数，这些方法中的很多方法都在 Laravel 框架中使用；如果你觉得方便，你可以在你的应用中自由的使用它们。

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
[array_wrap](#method-array-wrap)
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
[kebab_case](#method-kebab-case)
[snake_case](#method-snake-case)
[str_limit](#method-str-limit)
[starts_with](#method-starts-with)
[str_after](#method-str-after)
[str_before](#method-str-before)
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
[secure_url](#method-secure-url)
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
[cache](#method-cache)
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
[old](#method-old)
[redirect](#method-redirect)
[report](#method-report)
[request](#method-request)
[response](#method-response)
[retry](#method-retry)
[session](#method-session)
[tap](#method-tap)
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

如果给定的健不在数组中，那么 `array_add` 函数将会把给定健值对添加到数组中：

    $array = array_add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>
#### `array_collapse()` {#collection-method}

`array_collapse` 函数把数组中的每一个数组合并成单个数组：

    $array = array_collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-divide"></a>
#### `array_divide()` {#collection-method}

`array_divide` 函数返回两个数组，一个包含原始数组的健，另一个包含原始数组的值：

    list($keys, $values) = array_divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `array_dot()` {#collection-method}

`array_dot` 函数将多维数组平坦化为使用「点」符号表示深度的一维数组：

    $array = array_dot(['foo' => ['bar' => 'baz']]);

    // ['foo.bar' => 'baz'];

<a name="method-array-except"></a>
#### `array_except()` {#collection-method}

`array_except` 函数从数组中删除指定的健值对：

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

也可以将默认值作为第三个参数传递给方法。如果没有值通过测试，则返回默认值：

    $value = array_first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `array_flatten()` {#collection-method}

`array_flatten` 函数将多维数组平坦化为一维数组。

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $array = array_flatten($array);

    // ['Joe', 'PHP', 'Ruby'];

<a name="method-array-forget"></a>
#### `array_forget()` {#collection-method}

`array_forget` 函数使用「点」表示法从一个深度嵌套的数组中删除给定的健值对：

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `array_get()` {#collection-method}

`array_get` 函数使用「点」符号从深度嵌套的数组中检索一个值：

    $array = ['products' => ['desk' => ['price' => 100]]];

    $value = array_get($array, 'products.desk');

    // ['price' => 100]

`array_get` 函数也接受一个默认值，如果没有找到指定的健，则返回默认值：

    $value = array_get($array, 'names.john', 'default');

<a name="method-array-has"></a>
#### `array_has()` {#collection-method}

`array_has` 使用「点」表示法检查数组中是否存在指定的项目：

    $array = ['product' => ['name' => 'desk', 'price' => 100]];

    $hasItem = array_has($array, 'product.name');

    // true

    $hasItems = array_has($array, ['product.price', 'product.discount']);

    // false

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

`array_only` 函数只返回给定的数组中指定的健值对：

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $array = array_only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `array_pluck()` {#collection-method}

`array_pluck` 函数将从数组中提取出一列给定的健值对：

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $array = array_pluck($array, 'developer.name');

    // ['Taylor', 'Abigail'];

你也可以指定生成的列表的想要的健是什么：

    $array = array_pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail'];

<a name="method-array-prepend"></a>
#### `array_prepend()` {#collection-method}

`array_prepend` 函数将一个项目推到数组的开始位置：

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

`array_set` 函数使用「点」表示法在深度嵌套的数组中设置一个值：

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-sort"></a>
#### `array_sort()` {#collection-method}

`array_sort` 函数根据给定的闭包的结果对数组进行排序：

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

`array_sort_recursive` 使用 `sort` 函数递归排序数组：

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

    $array = array_where($array, function ($value, $key) {
        return is_string($value);
    });

    // [1 => 200, 3 => 400]

<a name="method-array-wrap"></a>
#### `array_wrap()` {#collection-method}

`array_wrap` 函数将给定的值包装成一个数组。如果给定的值已经是一个数组，则不会被改变：

    $string = 'Laravel';

    $array = array_wrap($string);

    // [0 => 'Laravel']

<a name="method-head"></a>
#### `head()` {#collection-method}

`head` 函数返回给定数组中的第一个元素：

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()` {#collection-method}

`last` 函数返回给定数组中的最后一个元素：

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="paths"></a>
## 路径

<a name="method-app-path"></a>
#### `app_path()` {#collection-method}

`app_path` 返回 `app` 目录的完整路径。你还可以使用 `app_path` 函数来生成相对于 `app` 目录的文件完整路径：

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {#collection-method}

`base_path` 函数返回项目根目录的完整路径。你还可以使用 `base_path` 函数生成指定文件相对于项目根目录的完整路径：

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {#collection-method}

`config_path` 函数返回应用程序配置目录的完整路径：

    $path = config_path();

<a name="method-database-path"></a>
#### `database_path()` {#collection-method}

`database_path` 函数返回应用程序数据库目录的完整路径：

    $path = database_path();

<a name="method-mix"></a>
#### `mix()` {#collection-method}

`mix` 函数获取 [版本化 Mix 文件](/docs/{{version}}/mix) 文件的路径：

    mix($file);

<a name="method-public-path"></a>
#### `public_path()` {#collection-method}

`public_path` 函数返回 `public` 目录的完整路径：

    $path = public_path();

<a name="method-resource-path"></a>
#### `resource_path()` {#collection-method}

`resource_path` 函数返回 `resources` 目录的完整路径。你还可以使用 `resource_path` 函数来生成相对于资源目录的指定文件的完整路径：

    $path = resource_path();

    $path = resource_path('assets/sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

`storage_path` 函数返回 `storage` 目录的完整路径。你还可以使用 `storage_path` 来生成相对于储存目录的指定文件的完整路径：

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## 字符串

<a name="method-camel-case"></a>
#### `camel_case()` {#collection-method}

`camel_case` 函数将给定的值符传转换为 `驼峰命名`：

    $camel = camel_case('foo_bar');

    // fooBar

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

`class_basename` 返回给定类删除命名空间的类名：

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {#collection-method}

`e` 函数使用 PHP 函数 `htmlspecialchars` 并且 `double_encode` 选项设置为 `false`：

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-ends-with"></a>
#### `ends_with()` {#collection-method}

`ends_with` 函数判断给定的字符串结尾是否是指定的内容：

    $value = ends_with('This is my name', 'name');

    // true

<a name="method-kebab-case"></a>
#### `kebab_case()` {#collection-method}

`lebab_case` 函数将给定的字符串转换为 `短横线隔开式`：

    $value = kebab_case('fooBar');

    // foo-bar


<a name="method-snake-case"></a>
#### `snake_case()` {#collection-method}

`snake_case` 函数将给定的字符串转换为 `蛇形命名`：

    $snake = snake_case('fooBar');

    // foo_bar

<a name="method-str-limit"></a>
#### `str_limit()` {#collection-method}

`str_limit` 函数限制字符串的字符数。该函数第一个参数接受一个字符串，第二个参数作为允许的最大字符数。

    $value = str_limit('The PHP framework for web artisans.', 7);

    // The PHP...

<a name="method-starts-with"></a>
#### `starts_with()` {#collection-method}

`starts_with` 函数判断给定的字符串的开头是否是指定值：

    $value = starts_with('This is my name', 'This');

    // true

<a name="method-str-after"></a>
#### `str_after()` {#collection-method}

`str_after` 函数返回字符串中指定值之后的所有内容：

    $value = str_after('This is: a test', 'This is:');

    // ' a test'

<a name="method-str-before"></a>
#### `str_before()` {#collection-method}

`str_before` 函数返回字符串指定值之前的所有内容：

    $value = str_before('Test :it before', ':it before');

    // 'Test '

<a name="method-str-contains"></a>
#### `str_contains()` {#collection-method}

`str_contains` 函数判断字符串是否包含指定的值：

    $value = str_contains('This is my name', 'my');

    // true

你还可以传递一个值的数组，来判断字符串是否包任意指定内容：

    $value = str_contains('This is my name', ['my', 'foo']);

    // true

<a name="method-str-finish"></a>
#### `str_finish()` {#collection-method}

`str_finish` 函数添加一个如果没有以指定值结尾的字符串后面：

    $string = str_finish('this/string', '/');
    $string2 = str_finish('this/string/', '/');

    // this/string/

<a name="method-str-is"></a>
#### `str_is()` {#collection-method}

`str_is` 函数判断指定的字符串是否匹配指定的格式。星号可以作为通配符使用：

    $value = str_is('foo*', 'foobar');

    // true

    $value = str_is('baz*', 'foobar');

    // false

<a name="method-str-plural"></a>
#### `str_plural()` {#collection-method}

`str_plural` 函数将字符串转换为复数形式。这个函数目前仅支持英文：

    $plural = str_plural('car');

    // cars

    $plural = str_plural('child');

    // children

你可以给函数的第二个参数传递一个整数，来检索字符串的单数形式或者复数形式：

    $plural = str_plural('child', 2);

    // children

    $plural = str_plural('child', 1);

    // child

<a name="method-str-random"></a>
#### `str_random()` {#collection-method}

`str_random` 函数生成一个指定长度的随机字符串。这个函数数用 PHP 的 `random_bytes` 函数：

    $string = str_random(40);

<a name="method-str-singular"></a>
#### `str_singular()` {#collection-method}

`str_singular` 函数将字符串转换为单数形式。这个函数目前仅支持英文：

    $singular = str_singular('cars');

    // car

<a name="method-str-slug"></a>
#### `str_slug()` {#collection-method}

`str_slug` 函数根据给定的字符串生成一个友好的「slug」URL：

    $title = str_slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-studly-case"></a>
#### `studly_case()` {#collection-method}

`studly_case` 函数将给定的字符串转换为 `首字母大写`：

    $value = studly_case('foo_bar');

    // FooBar

<a name="method-title-case"></a>
#### `title_case()` {#collection-method}

`title_case` 函数将给定的字符串转换为 `每个单词首字母大写`;

    $title = title_case('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-trans"></a>
#### `trans()` {#collection-method}

`trans` 函数使用你的 [本地化文件](/docs/{{version}}/localization) 来翻译给定的语句：

    echo trans('validation.required'):

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

`trans_choice` 函数根据给定数量来决定翻译指定语句是复数形式还是单数形式：

    $value = trans_choice('foo.bar', $count);

<a name="urls"></a>
## URLs

<a name="method-action"></a>
#### `action()` {#collection-method}

`action` 函数为指定的控制器动作生成一个 URL。你不需要传递完整的控制器命名空间。只需要传递相对于 `App\Http\Controllers` 的命名空间：

    $url = action('HomeController@getIndex');

如果该方法接受路由参数，你可以使用第二个参数传递：

    $url = action('UserController@profile', ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {#collection-method}

使用当前请求的协议（ HTTP 或 HTTPS ）为资源文件生成一个 URL：

    $url = asset('img/photo.jpg');

<a name="method-secure-asset"></a>
#### `secure_asset()` {#collection-method}

使用 HTTPS 协议生成资源文件的 URL:

    echo secure_asset('foo/bar.zip', $title, $attributes = []);

<a name="method-route"></a>
#### `route()` {#collection-method}

`route` 函数为给定的命名路由生成一个 URL：

    $url = route('routeName');

如果路由接受参数，则可以使用第二个参数传递给方法：

    $url = route('routeName', ['id' => 1]);

默认情况下，`route` 函数生成的是绝对 URL。如果你想生成一个相对 URL，你可以第三个值传递 `false`：

    $url = route('routeName', ['id' => 1], false);

<a name="method-secure-url"></a>
#### `secure_url()` {#collection-method}

`secure_url` 函数为给定的路径生成一个完整的 HTTPS URL 路径：

    echo secure_url('user/profile');

    echo secure_url('user/profile', [1]);

<a name="method-url"></a>
#### `url()` {#collection-method}

`url` 函数生成给定的路径的完整 URL：

    echo url('user/profile');

    echo url('user/profile', [1]);

如果没有提供路径，则返回 `Illuminate\Routing\UrlGenerator` 实例：

    echo url()->current();
    echo url()->full();
    echo url()->previous();

<a name="miscellaneous"></a>
## 其他

<a name="method-abort"></a>
#### `abort()` {#collection-method}

`abort` 函数将会跑出一个 HTTP 异常并且由异常处理程序处理：

    abort(401);

你还可以提供异常的响应文本：

    abort(401, 'Unauthorized.');

<a name="method-abort-if"></a>
#### `abort_if()` {#collection-method}

如果给定的布尔值为 `true` 则 `abort_if` 函数将抛出一个 HTTP 异常：

    abort_if(! Auth::user()->isAdmin(), 403);

<a name="method-abort-unless"></a>
#### `abort_unless()` {#collection-method}

如果给定的布尔值为 `false` 则 `abort_unless` 函数将抛出一个 HTTP 异常：

    abort_unless(Auth::user()->isAdmin(), 403);

<a name="method-auth"></a>
#### `auth()` {#collection-method}

为例方便起见 `auth` 函数返回一个认证实例。你可以使用它来替代 `Auth` facade：

    $user = auth()->user();

<a name="method-back"></a>
#### `back()` {#collection-method}

`back()` 函数会生成用户之前位置的一个重定向响应：

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {#collection-method}

`bcrypt` 使用 Bcrypt 对给定的值进行散列。你可以使用它替代 `Hash` facade：

    $password = bcrypt('my-secret-password');

<a name="method-cache"></a>
#### `cache()` {#collection-method}

`cache` 函数可以用来从缓存中获取值。如果缓存中不存在给定的健，则返回默认值：

    $value = cache('key');

    $value = cache('key', 'default');

你可以通过健值对的数组来添加项目到缓冲中。你还应该传递一个以分钟为单位缓存过期时间：

    cache(['key' => 'value'], 5);

    cache(['key' => 'value'], Carbon::now()->addSeconds(10));

<a name="method-collect"></a>
#### `collect()` {#collection-method}

`collect` 函数根据给定的数组创建一个 [集合](/docs/{{version}}/collections) 实例：

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {#collection-method}

`config` 函数用来获取配置信息的值，可以使用「点」语法访问配置值，其中要包含文件名和选项名。可以指定一个默认值，如果选项不存在则返回默认值：

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

`config` 辅助函数也可以通过传递一个健值对数组在运行的时候配置信息：

    config(['app.debug' => true]);

<a name="method-csrf-field"></a>
#### `csrf_field()` {#collection-method}

`csrf_field` 函数生成包含 CSRF 令牌值的 HTML `hidden` 表单字段。例如，使用 [Blade 语法](/docs/{{version}}/blade)：

    {{ csrf_field() }}

<a name="method-csrf-token"></a>
#### `csrf_token()` {#collection-method}

`csrf_token` 函数获取当前 CSRF 令牌的值：

    $token = csrf_token();

<a name="method-dd"></a>
#### `dd()` {#collection-method}

`dd` 函数输出给定的值并结束脚本运行：

    dd($value);

    dd($value1, $value2, $value3, ...);

如果你不想终止脚本运行，请改用 `dump` 函数：

    dump($value);

<a name="method-dispatch"></a>
#### `dispatch()` {#collection-method}

`dispatch` 函数将一个新的任务推送到 Laravel [任务列队](/docs/{{version}}/queues)

    dispatch(new App\Jobs\SendEmails);

<a name="method-env"></a>
#### `env()` {#collection-method}

`env` 函数获取环境变量的值或者返回默认值：

    $env = env('APP_ENV');

    // 如果环境变量不存在则返回默认值...
    $env = env('APP_ENV', 'production');

<a name="method-event"></a>
#### `event()` {#collection-method}

`event` 函数将给定的 [事件](/docs/{{version}}/events) 派发到所属侦听器：

    event(new UserRegistered($user));

<a name="method-factory"></a>
#### `factory()` {#collection-method}

`factory` 函数根据给定的类、名称和数量创建一个模型工厂构建器。可以在 [测试](/docs/{{version}}/database-testing#writing-factories) or [数据填充](/docs/{{version}}/seeding#using-model-factories) 中使用：

    $user = factory(App\User::class)->make();

<a name="method-info"></a>
#### `info()` {#collection-method}

`info` 函数将信息写入日志：

    info('Some helpful information!');

上下文数据的数组也可以传递给函数：

    info('User login attempt failed.', ['id' => $user->id]);

<a name="method-logger"></a>
#### `logger()` {#collection-method}

`logger` 函数可以将一个 `debug` 级别的消息写入到日志中：

    logger('Debug message');

上下文数据的数组也可以传递给函数：

    logger('User has logged in.', ['id' => $user->id]);

如果没有传值给函数则返回 [日志](/docs/{{version}}/errors#logging) 的实例：

    logger()->error('You are not allowed here.');

<a name="method-method-field"></a>
#### `method_field()` {#collection-method}

`method_field` 函数生成一个模拟 HTTP 动作的 HTML `hidden` 表单字段。例如，使用 [Blade 语法](/docs/{{version}}/blade)：

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>

<a name="method-old"></a>
#### `old()` {#collection-method}

`old` 函数 [获取](/docs/{{version}}/requests#retrieving-input) 一个旧的 session 闪存输入值：

    $value = old('value');

    $value = old('value', 'default');

<a name="method-redirect"></a>
#### `redirect()` {#collection-method}

`redirect` 函数返回一个重定向 HTTP 响应，如果没有没有传入参数，则返回重定向实例：

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>
#### `report()` {#collection-method}

`report` 函数将使用异常处理程序的 `report` 方法抛出异常：

    report($e);

<a name="method-request"></a>
#### `request()` {#collection-method}

`request` 函数返回当前 [请求](/docs/{{version}}/requests) 实例或者获取输入项：

    $request = request();

    $value = request('key', $default = null)

<a name="method-response"></a>
#### `response()` {#collection-method}

`response` 函数创建一个 [响应](/docs/{{version}}/responses) 实例，或者获取响应工厂实例：

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>
#### `retry()` {#collection-method}

`retry` 函数尝试执行给定的回调，直到到达给定的最大尝试次数。如果回调没有派出异常并且有返回值则返回返回值。如果回调抛出异常，它将自动重试。如果超过最大尝试次数，则抛出异常。

    return retry(5, function () {
        // 在 100ms 左右尝试 5 次... 
    }, 100);

<a name="method-session"></a>
#### `session()` {#collection-method}

`session` 函数可以用来获取或者设置 Session 值：

    $value = session('key');

你可以通过健值对数组传递给函数来设置 Session 值：

    session(['chairs' => 7, 'instruments' => 3]);

如果没有传递值给函数，则返回 Session 实例：

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-tap"></a>
#### `tap()` {#collection-method}

`tap` 函数接受两个参数：`$value` 和一个闭包。传入的 `$value` 将会作为闭包函数的传参，处理完后成为 `tap` 的返回值。闭包的返回值是无关紧要（不需要 `return` 关键词）。

    $user = tap(User::first(), function ($user) {
        $user->name = 'taylor';

        $user->save();
    });

如果没有传递闭包给 `tap` 函数，你可以调用给定 `$value` 上任何方法。不管方法中定义的实际返回值是什么，你调用的方法返回值始终 `$value`。例如，Eloquent `update` 一般返回一个整数。而我们可以通过 `tap` 函数链式调用 `update` 的方式返回模型本身：

    $user = tap($user)->update([
        'name' => $name,
        'email' => $email
    ]);

<a name="method-value"></a>
#### `value()` {#collection-method}

`value` 函数可以简单的返回它的值。然而，如果将 `闭包` 传递给函数，则运行这个 `闭包` 并返回结果：

    $value = value(function () {
        return 'bar';
    });

<a name="method-view"></a>
#### `view()` {#collection-method}

`view` 函数获取一个 [视图](/docs/{{version}}/views) 实例：

    return view('auth.login');

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [Seven Du](https://github.com/medz) | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/5564821?s=300"> | 翻译 | 基于 Laravel 的社交开源系统 [ThinkSNS+](https://github.com/slimkit/thinksns-plus) 欢迎 Star。  |



--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org
