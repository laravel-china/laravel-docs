# Laravel 的辅助函数列表

- [简介](#introduction)
- [可用方法](#available-methods)

<a name="introduction"></a>
## 简介

Laravel 包含各种各样的全局「辅助」PHP 函数，框架本身也大量地使用了这些功能；如果你觉得方便，你可以在你的应用中自由的使用它们。

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

### 数组 & 对象

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
[array_random](#method-array-random)
[array_set](#method-array-set)
[array_sort](#method-array-sort)
[array_sort_recursive](#method-array-sort-recursive)
[array_where](#method-array-where)
[array_wrap](#method-array-wrap)
[data_fill](#method-data-fill)
[data_get](#method-data-get)
[data_set](#method-data-set)
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

[\__](#method-__)
[camel_case](#method-camel-case)
[class_basename](#method-class-basename)
[e](#method-e)
[ends_with](#method-ends-with)
[kebab_case](#method-kebab-case)
[preg_replace_array](#method-preg-replace-array)
[snake_case](#method-snake-case)
[starts_with](#method-starts-with)
[str_after](#method-str-after)
[str_before](#method-str-before)
[str_contains](#method-str-contains)
[str_finish](#method-str-finish)
[str_is](#method-str-is)
[str_limit](#method-str-limit)
[str_plural](#method-str-plural)
[str_random](#method-str-random)
[str_replace_array](#method-str-replace-array)
[str_replace_first](#method-str-replace-first)
[str_replace_last](#method-str-replace-last)
[str_singular](#method-str-singular)
[str_slug](#method-str-slug)
[str_start](#method-str-start)
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
[app](#method-app)
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[blank](#method-blank)
[broadcast](#method-broadcast)
[cache](#method-cache)
[class_uses_recursive](#method-class-uses-recursive)
[collect](#method-collect)
[config](#method-config)
[cookie](#method-cookie)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[dd](#method-dd)
[decrypt](#method-decrypt)
[dispatch](#method-dispatch)
[dispatch_now](#method-dispatch-now)
[dump](#method-dump)
[encrypt](#method-encrypt)
[env](#method-env)
[event](#method-event)
[factory](#method-factory)
[filled](#method-filled)
[info](#method-info)
[logger](#method-logger)
[method_field](#method-method-field)
[now](#method-now)
[old](#method-old)
[optional](#method-optional)
[policy](#method-policy)
[redirect](#method-redirect)
[report](#method-report)
[request](#method-request)
[rescue](#method-rescue)
[resolve](#method-resolve)
[response](#method-response)
[retry](#method-retry)
[session](#method-session)
[tap](#method-tap)
[today](#method-today)
[throw_if](#method-throw-if)
[throw_unless](#method-throw-unless)
[trait_uses_recursive](#method-trait-uses-recursive)
[transform](#method-transform)
[validator](#method-validator)
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

如果给定的健不在数组中，那么 `array_add` 函数将会把给定的键/值对添加到数组中：

````
$array = array_add(['name' => 'Desk'], 'price', 100);

// ['name' => 'Desk', 'price' => 100]
````


<a name="method-array-collapse"></a>
#### `array_collapse()` {#collection-method}

`array_collapse` 函数将多个单数组合并成一个数组：

````
$array = array_collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
````

<a name="method-array-divide"></a>
#### `array_divide()` {#collection-method}

`array_divide` 函数返回两个数组，一个包含原始数组的健，另一个包含原始数组的值：

````
list($keys, $values) = array_divide(['name' => 'Desk']);

// $keys: ['name']

// $values: ['Desk']
````

<a name="method-array-dot"></a>
#### `array_dot()` {#collection-method}

`array_dot` 函数将多维数组平铺到一维数组中，该数组使用「点」符号表示深度：

````
$array = array_dot(['foo' => ['bar' => 'baz']]);

// ['foo.bar' => 'baz'];
````

<a name="method-array-except"></a>
#### `array_except()` {#collection-method}

`array_except` 函数从数组中删除给定的键/值对：

````
$array = ['name' => 'Desk', 'price' => 100];

$array = array_except($array, ['price']);

// ['name' => 'Desk']
````

<a name="method-array-first"></a>
#### `array_first()` {#collection-method}

`array_first` 函数返回数组中第一个通过指定测试的元素：

````
$array = [100, 200, 300];

$value = array_first($array, function ($value, $key) {
    return $value >= 150;
});

// 200
````

将默认值作为第三个参数传递给该方法。如果没有值通过测试，则返回该值：

````
$value = array_first($array, $callback, $default);
````

<a name="method-array-flatten"></a>
#### `array_flatten()` {#collection-method}

`array_flatten` 函数将多维数组平铺为一维数组。

````
$array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

$array = array_flatten($array);

// ['Joe', 'PHP', 'Ruby'];
````

<a name="method-array-forget"></a>
#### `array_forget()` {#collection-method}

`array_forget` 函数使用「点」符号从深度嵌套数组中移除给定的键/值对：

````
$array = ['products' => ['desk' => ['price' => 100]]];

array_forget($array, 'products.desk');

// ['products' => []]
````

<a name="method-array-get"></a>
#### `array_get()` {#collection-method}

`array_get` 函数使用「点」符号从深度嵌套的数组中检索值：

````
$array = ['products' => ['desk' => ['price' => 100]]];

$value = array_get($array, 'products.desk');

// ['price' => 100]
````

`array_get` 函数也接受一个默认值，如果没有找到指定的健，则返回该值：

````
$value = array_get($array, 'names.john', 'default');
````

<a name="method-array-has"></a>
#### `array_has()` {#collection-method}

`array_has` 函数使用「点」符号检查数组中是否存在给定的项目或项目组：

````
$array = ['product' => ['name' => 'desk', 'price' => 100]];

$hasItem = array_has($array, 'product.name');

// true

$hasItems = array_has($array, ['product.price', 'product.discount']);

// false
````

<a name="method-array-last"></a>
#### `array_last()` {#collection-method}

`array_last` 函数返回数组中最后一个通过指定测试的元素：

````
$array = [100, 200, 300, 110];

$value = array_last($array, function ($value, $key) {
    return $value >= 150;
});

// 300
````

将默认值作为第三个参数传递给该方法。如果没有值通过测试，则返回该值：

````
$last = array_last($array, $callback, $default);
````

<a name="method-array-only"></a>

#### `array_only()` {#collection-method}

`array_only` 函数仅返回给定数组中指定的键/值对：

````
$array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

$array = array_only($array, ['name', 'price']);

// ['name' => 'Desk', 'price' => 100]
````

<a name="method-array-pluck"></a>
#### `array_pluck()` {#collection-method}

`array_pluck` 函数从数组中检索给定键的所有值：

````
$array = [
    ['developer' => ['id' => 1, 'name' => 'Taylor']],
    ['developer' => ['id' => 2, 'name' => 'Abigail']],
];

$array = array_pluck($array, 'developer.name');

// ['Taylor', 'Abigail'];
````

你也可以指定生成的列表的键：

````
$array = array_pluck($array, 'developer.name', 'developer.id');

// [1 => 'Taylor', 2 => 'Abigail'];
````

<a name="method-array-prepend"></a>
#### `array_prepend()` {#collection-method}

`array_prepend` 函数将一个项目推到数组的开头：

````
$array = ['one', 'two', 'three', 'four'];

$array = array_prepend($array, 'zero');

// $array: ['zero', 'one', 'two', 'three', 'four']
````

你可以指定用于该值的键：

````
$array = ['price' => 100];

$array = array_prepend($array, 'Desk', 'name');

// ['name' => 'Desk', 'price' => 100]
````

<a name="method-array-pull"></a>

#### `array_pull()` {#collection-method}

`array_pull` 函数返回并从数组中删除键/值对：

````
$array = ['name' => 'Desk', 'price' => 100];

$name = array_pull($array, 'name');

// $name: Desk

// $array: ['price' => 100]
````

将默认值作为第三个参数传递给该方法。如果键不存在，则返回该值：

````
$value = array_pull($array, $key, $default);
````

<a name="method-array-random"></a>

#### `array_random()` {#collection-method}

`array_random` 函数从数组中返回一个随机值：

````
$array = [1, 2, 3, 4, 5];

$random = array_random($array);

// 4 - (随机获取)
````

你也可以指定要返回的随机数的数量作为第二个可选参数。一旦你指定了第二个参数，即使数量为 1，这个函数也会返回一个数组：

````
$items = array_random($array, 2);

// [2, 5] - (随机获取)
````

<a name="method-array-set"></a>
#### `array_set()` {#collection-method}

`array_set` 函数使用「点」符号在深度嵌套的数组中设置一个值：

````
$array = ['products' => ['desk' => ['price' => 100]]];

array_set($array, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 200]]]
````

<a name="method-array-sort"></a>
#### `array_sort()` {#collection-method}

`array_sort` 函数按照其值排序数组：

````
$array = ['Desk', 'Table', 'Chair'];

$sorted = array_sort($array);

// ['Chair', 'Desk', 'Table']
````

你也可以按给定的闭包返回的结果对数组进行排序：

````
$array = [
    ['name' => 'Desk'],
    ['name' => 'Table'],
    ['name' => 'Chair'],
];

$sorted = array_values(array_sort($array, function ($value) {
    return $value['name'];
}));

/*
    [
        ['name' => 'Chair'],
        ['name' => 'Desk'],
        ['name' => 'Table'],
    ]
*/
````

<a name="method-array-sort-recursive"></a>

#### `array_sort_recursive()` {#collection-method}

`array_sort_recursive` 函数使用 `sort` 函数递归排序数组：

````
$array = [
    ['Roman', 'Taylor', 'Li'],
    ['PHP', 'Ruby', 'JavaScript'],
];

$sorted = array_sort_recursive($array);

/*
    [
        ['Li', 'Roman', 'Taylor'],
        ['JavaScript', 'PHP', 'Ruby'],
    ]
*/
````

<a name="method-array-where"></a>
#### `array_where()` {#collection-method}

`array_where` 函数使用给定的闭包来过滤数组：

````
$array = [100, '200', 300, '400', 500];

$array = array_where($array, function ($value, $key) {
    return is_string($value);
});

// [1 => 200, 3 => 400]
````

<a name="method-array-wrap"></a>
#### `array_wrap()` {#collection-method}

`array_wrap` 函数将给定的值包装成一个数组。如果给定的值已经是一个数组，则不会被改变：

````
$string = 'Laravel';

$array = array_wrap($string);

// [0 => 'Laravel']
````

<a name="method-data-fill"></a>
#### `data_fill()` {#collection-method}

`data_fill` 函数使用「点」符号在嵌套数组或对象内设置缺少的值：

````
$data = ['products' => ['desk' => ['price' => 100]]];

data_fill($data, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 100]]]

data_fill($data, 'products.desk.discount', 10);

// ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]
````

该函数也接受星号「*」作为通配符，并相应地填写目标：

````
$data = [
    'products' => [
        ['name' => 'Desk 1', 'price' => 100],
        ['name' => 'Desk 2'],
    ],
];

data_fill($data, 'products.*.price', 200);

/*
    [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 200],
        ],
    ]
*/
````

<a name="method-data-get"></a>
#### `data_get()` {#collection-method}

`data_get` 函数使用「点」符号从嵌套数组或对象中检索值：

````
$data = ['products' => ['desk' => ['price' => 100]]];

$price = data_get($data, 'products.desk.price');

// 100
````

`data_get` 函数还接受默认值作为第三个参数，如果找不到指定的键，将返回该值：

````
$discount = data_get($data, 'products.desk.discount', 0);

// 0
````

<a name="method-data-set"></a>
#### `data_set()` {#collection-method}

`data_set` 函数使用「点」符号在嵌套数组或对象内设置一个值：

````
$data = ['products' => ['desk' => ['price' => 100]]];

data_set($data, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 200]]]
````

这个函数也接受通配符「*」，并相应地在目标上设置值：

````
    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 150],
        ],
    ];

    data_set($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 200],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */
````

默认情况下，所有现有的值都会被覆盖。如果你只想设置一个不存在值，你可以传递 `false` 作为第三个参数：

````
$data = ['products' => ['desk' => ['price' => 100]]];

data_set($data, 'products.desk.price', 200, false);

// ['products' => ['desk' => ['price' => 100]]]
````

<a name="method-head"></a>
#### `head()` {#collection-method}

`head` 函数返回给定数组中的第一个元素：

````
$array = [100, 200, 300];

$first = head($array);

// 100
````

<a name="method-last"></a>
#### `last()` {#collection-method}

`last` 函数返回给定数组中的最后一个元素：

````
$array = [100, 200, 300];

$last = last($array);

// 300
````

<a name="paths"></a>
## 路径

<a name="method-app-path"></a>
#### `app_path()` {#collection-method}

`app_path` 返回 `app` 目录的完整路径。你还可以使用 `app_path` 函数来生成相对于 `app` 目录的文件完整路径：

````
$path = app_path();

$path = app_path('Http/Controllers/Controller.php');
````

<a name="method-base-path"></a>
#### `base_path()` {#collection-method}

`base_path` 函数返回项目根目录的完整路径。你还可以使用 `base_path` 函数生成指定文件相对于项目根目录的完整路径：

````
$path = base_path();

$path = base_path('vendor/bin');
````

<a name="method-config-path"></a>
#### `config_path()` {#collection-method}

`config_path` 函数返回应用程序配置目录的完整路径。你也可以使用 `config_path` 函数来生成应用程序配置目录中给定文件的完整路径：

````
$path = config_path();
````

<a name="method-database-path"></a>
#### `database_path()` {#collection-method}

`database_path` 函数返回应用程序数据库目录的完整路径。你也可以使用 `database_path` 函数来生成数据库目录中给定文件的完整路径：

````
$path = database_path();
````

<a name="method-mix"></a>
#### `mix()` {#collection-method}

`mix` 函数获取 [版本化 Mix 文件](/docs/{{version}}/mix) 的路径：

````
mix($file);
````

<a name="method-public-path"></a>
#### `public_path()` {#collection-method}

`public_path` 函数返回 `public` 目录的完整路径。你也可以使用 `public_path` 函数来生成 `public` 目录中给定文件的完整路径：

````
$path = public_path();
````

<a name="method-resource-path"></a>
#### `resource_path()` {#collection-method}

`resource_path` 函数返回 `resources` 目录的完整路径。你也可以使用 `resource_path` 函数来生成相对于资源目录的指定文件的完整路径：

````
$path = resource_path();

$path = resource_path('assets/sass/app.scss');
````

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

`storage_path` 函数返回 `storage` 目录的完整路径。你也可以使用 `storage_path` 来生成相对于储存目录的指定文件的完整路径：

````
$path = storage_path();

$path = storage_path('app/file.txt');
````

<a name="strings"></a>
## 字符串

<a name="method-__"></a>
#### `__()` {#collection-method}

`__` 函数使用你的 [本地化文件](/docs/{{version}}/localization) 来翻译给定的翻译字符串或翻译键：

````
echo __('Welcome to our application');

echo __('messages.welcome');
````

如果指定的翻译字符串或键不存在，则 `__` 函数会简单地返回给定的值。所以，按照上面的例子，如果翻译键 `messages.welcome` 不存在，`__` 方法会将其直接返回。

<a name="method-camel-case"></a>
#### `camel_case()` {#collection-method}

`camel_case` 函数将给定的值符传转换为「驼峰命名」：

````
$camel = camel_case('foo_bar');

// fooBar
````

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

`class_basename` 返回给定类删除命名空间的类名：

````
$class = class_basename('Foo\Bar\Baz');

// Baz
````

<a name="method-e"></a>
#### `e()` {#collection-method}

`e` 函数将 `double_encode` 选项设置为 `false` 来运行 PHP 的 `htmlspecialchars` 函数：

````
echo e('<html>foo</html>');

// &lt;html&gt;foo&lt;/html&gt;
````

<a name="method-ends-with"></a>
#### `ends_with()` {#collection-method}

`ends_with` 函数判断给定的字符串是否以给定的值结尾：

````
$value = ends_with('This is my name', 'name');

// true
````

<a name="method-kebab-case"></a>
#### `kebab_case()` {#collection-method}

`kebab_case` 函数将给定的字符串转换为「短横线命名」：

````
$value = kebab_case('fooBar');

// foo-bar
````

<a name="method-preg-replace-array"></a>

#### `preg_replace_array()`

`preg_replace_array` 函数使用数组顺序替换字符串中的给定模式：

```
$string = '活动将在 :start 和 :end 之间进行';

$replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

// 活动将在 8:30 至 9:00 之间进行
```

<a name="method-snake-case"></a>

#### `snake_case()` {#collection-method}

`snake_case` 函数将给定的字符串转换为「蛇形命名」：

````
$snake = snake_case('fooBar');

// foo_bar
````

<a name="method-starts-with"></a>

#### `starts_with()` {#collection-method}

`starts_with` 函数判断给定的字符串的开头是否是指定值：

```
$value = starts_with('This is my name', 'This');

// true

```

<a name="method-str-after"></a>

#### `str_after()` {#collection-method}

`str_after` 函数返回字符串中指定值之后的所有内容：

```
$slice = str_after('This is my name', 'This is');

// ' my name'
```


<a name="method-str-before"></a>
#### `str_before()` {#collection-method}

`str_before` 函数返回字符串中给定值之前的所有内容：

````
$slice = str_before('This is my name', 'my name');

// 'This is '
````

<a name="method-str-contains"></a>
#### `str_contains()` {#collection-method}

`str_contains` 函数判断给定的字符串是否包含给定的值：

````
$value = str_contains('This is my name', 'my');

// true
````

你也可以传递一个值的数组来判断给定的字符串是否包含任何值：

````
$value = str_contains('This is my name', ['my', 'foo']);

// true
````

<a name="method-str-finish"></a>
#### `str_finish()` {#collection-method}

`str_finish` 函数将给定字符串以给定值结尾返回（如果它尚未以给定值结尾）：

````
$string = str_finish('this/string', '/');

// this/string/

$string2 = str_finish('this/string/', '/');

// this/string/
````

<a name="method-str-is"></a>
#### `str_is()` {#collection-method}

`str_is` 函数判断给定的字符串是否匹配给定的模式。星号可以用来表示通配符：

````
$value = str_is('foo*', 'foobar');

// true

$value = str_is('baz*', 'foobar');

// false
````

<a name="method-str-limit"></a>

#### `str_limit()` {#collection-method}

`str_limit` 函数按给定的长度截断给定的字符串：

```
$truncated = str_limit('The quick brown fox jumps over the lazy dog', 20);

// The quick brown fox...
```

你也可以传递第三个参数来改变将被追加到最后的字符串：

```
$truncated = str_limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

// The quick brown fox (...)
```

<a name="method-str-plural"></a>

#### `str_plural()` {#collection-method}

`str_plural` 函数将字符串转换为复数形式。这个函数目前仅支持英文：

````
$plural = str_plural('car');

// cars

$plural = str_plural('child');

// children
````

你可以提供一个整数作为函数的第二个参数来检索字符串的单数或复数形式：

````
$plural = str_plural('child', 2);

// children

$plural = str_plural('child', 1);

// child
````

<a name="method-str-random"></a>
#### `str_random()` {#collection-method}

`str_random` 函数生成一个指定长度的随机字符串。这个函数数用 PHP 的 `random_bytes` 函数：

    $string = str_random(40);
#### `str_replace_array()` {#collection-method}

`str_replace_array` 函数使用数组顺序替换字符串中的给定值：

````
$string = 'The event will take place between ? and ?';

$replaced = str_replace_array('?', ['8:30', '9:00'], $string);

// The event will take place between 8:30 and 9:00
````

<a name="method-str-replace-first"></a>
#### `str_replace_first()` {#collection-method}

`str_replace_first` 函数替换字符串中给定值的第一个匹配项：

````
$replaced = str_replace_first('the', 'a', 'the quick brown fox jumps over the lazy dog');

// a quick brown fox jumps over the lazy dog
````

<a name="method-str-replace-last"></a>
#### `str_replace_last()` {#collection-method}

`str_replace_last` 函数替换字符串中最后一次出现的给定值：

````
$replaced = str_replace_last('the', 'a', 'the quick brown fox jumps over the lazy dog');

// the quick brown fox jumps over a lazy dog
````

<a name="method-str-singular"></a>

#### `str_singular()` {#collection-method}

`str_singular` 函数将字符串转换为单数形式。这个函数目前仅支持英文：

````
$singular = str_singular('cars');

// car
````

<a name="method-str-slug"></a>
#### `str_slug()` {#collection-method}

`str_slug` 函数根据给定的字符串生成一个 URL 友好的「slug」：

````
$title = str_slug('Laravel 5 Framework', '-');

// laravel-5-framework
````

<a name="method-str-start"></a>

#### `str_start()` {#collection-method}

`str_start` 函数将给定值的单个实例添加到字符串（如果它尚未以值开始）：

````
$adjusted = str_start('this/string', '/');

// /this/string

$adjusted = str_start('/this/string/', '/');

// /this/string
````

<a name="method-studly-case"></a>

#### `studly_case()` {#collection-method}

`studly_case` 函数将给定的字符串转换为「变种驼峰命名」：

````
$value = studly_case('foo_bar');

// FooBar
````

<a name="method-title-case"></a>
#### `title_case()` {#collection-method}

`title_case` 函数将给定的字符串转换为「首字母大写」：

````
$title = title_case('a nice title uses the correct case');

// A Nice Title Uses The Correct Case
````

<a name="method-trans"></a>
#### `trans()` {#collection-method}

`trans` 函数使用你的 [本地化文件](/docs/{{version}}/localization) 来翻译给定的翻译字符串或翻译键：

````
echo trans('messages.welcome');
````

如果指定的翻译键不存在，则 `trans` 方法会简单地返回给定的键。所以，就上面的例子而言，如果翻译键不存在， `trans` 方法会返回 `messages.welcome`。

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

`trans_choice` 函数根据词形变化来翻译给定的翻译键：

````
echo trans_choice('messages.notifications', $unreadCount);
````

如果指定的翻译键不存在，`trans_choice` 方法会简单地返回给定的键。所以，按照上面的例子，如果翻译键不存在，`trans_choice` 方法会返回 `messages.notifications`。


<a name="urls"></a>
## URLs

<a name="method-action"></a>
#### `action()` {#collection-method}

`action` 函数为指定的控制器动作生成一个 URL。你不需要传递完整的控制器命名空间。只需要传递相对于 `App\Http\Controllers` 的命名空间的控制器类名称：

````
$url = action('HomeController@getIndex');
````

如果该方法接受路由参数，则可以将它们作为方法的第二个参数传递：

````
$url = action('UserController@profile', ['id' => 1]);
````

<a name="method-asset"></a>
#### `asset()` {#collection-method}

`asset` 函数使用当前请求的协议（ HTTP 或 HTTPS ）为资源文件生成 URL：

````
$url = asset('img/photo.jpg');
````

<a name="method-secure-asset"></a>
#### `secure_asset()` {#collection-method}

`secure_asset` 函数使用 HTTPS 协议为资源文件生成 URL:

````
echo secure_asset('foo/bar.zip');
````

<a name="method-route"></a>
#### `route()` {#collection-method}

`route` 函数为给定的命名路由生成一个 URL：

````
$url = route('routeName');
````

如果路由接受参数，则可以将它们作为方法的第二个参数传递：

````
$url = route('routeName', ['id' => 1]);
````

默认情况下，`route` 函数生成的是绝对 URL。如果你想生成一个相对 URL，你可以传递 `false` 作为第三个参数：

````
$url = route('routeName', ['id' => 1], false);
````

<a name="method-secure-url"></a>
#### `secure_url()` {#collection-method}

`secure_url` 函数为给定的路径生成一个标准的 HTTPS URL：

````
echo secure_url('user/profile');

echo secure_url('user/profile', [1]);
````

<a name="method-url"></a>
#### `url()` {#collection-method}

`url` 函数生成给定路径的标准 URL：

````
echo url('user/profile');

echo url('user/profile', [1]);
````

如果没有提供路径，则返回 `Illuminate\Routing\UrlGenerator` 实例：

````
echo url()->current();
echo url()->full();
echo url()->previous();
````

<a name="miscellaneous"></a>
## 其他

<a name="method-abort"></a>
#### `abort()` {#collection-method}

`abort` 函数抛出 [异常处理](/docs/{{version}}/errors#the-exception-handler) 程序呈现的 [HTTP 异常](/docs/{{version}}/errors#http-exceptions)：

````
abort(401);
````

你也可以提供额外的响应文本和自定义响应标头：

````
abort(403, 'Unauthorized.', $headers);
````

<a name="method-abort-if"></a>
#### `abort_if()` {#collection-method}

如果给定的布尔表达式计算结果为 `true`， `abort_if` 函数将抛出一个 HTTP 异常：

````
abort_if(! Auth::user()->isAdmin(), 403);
````

和 `abort` 方法一样，你也可以提供异常的响应文本作为第三个参数，并提供一个自定义响应头数组作为第四个参数。

<a name="method-abort-unless"></a>

#### `abort_unless()` {#collection-method}

如果给定的布尔表达式计算结果为 `false`，`abort_unless` 函数将抛出一个 HTTP 异常：

````
abort_unless(Auth::user()->isAdmin(), 403);
````

和 `abort` 方法一样，你也可以提供异常的响应文本作为第三个参数，并提供一个自定义响应头数组作为第四个参数。

<a name="method-app"></a>

#### `app()` {#collection-method}

`app` 函数返回 [服务容器](/docs/{{version}}/container) 实例

````
$container = app();
````

你可以传递一个类或接口名称来从容器中解析它：

````
$api = app('HelpSpot\API');
````

<a name="method-auth"></a>
#### `auth()` {#collection-method}

`auth` 函数返回一个 [认证](/docs/{{version}}/authentication) 实例。为了方便起见，你可以使用它来替代 `Auth` facade：

````
$user = auth()->user();
````

如果需要，你可以指定你想要访问的认证实例：

````
$user = auth('admin')->user();
````

<a name="method-back"></a>
#### `back()` {#collection-method}

`back()` 函数生成一个 [重定向 HTTP 响应](/docs/{{version}}/responses#redirects) 到用户之前的位置：

````
return back($status = 302, $headers = [], $fallback = false);

return back();
````

<a name="method-bcrypt"></a>
#### `bcrypt()` {#collection-method}

`bcrypt` 使用 Bcrypt 对给定的值进行散列。你可以使用它替代 `Hash` facade：

````
$password = bcrypt('my-secret-password');
````

<a name="method-broadcast"></a>

#### `broadcast()` {#collection-method}

`broadcast` 函数将 [广播](/docs/{{version}}/broadcasting) 给定的 [事件](/docs/{{version}}/events) 到它的监听器：

````
broadcast(new UserRegistered($user));
````

<a name="method-blank"></a>

#### `blank()` {#collection-method}

`blank` 函数判断给定的值是否为「空」：

````
blank('');
blank('   ');
blank(null);
blank(collect());

// true

blank(0);
blank(true);
blank(false);

// false
````

要使用与 `blank` 相反的功能，请看 [filled](/docs/{{version}}/helpers#method-filled) 方法。


<a name="method-cache"></a>
#### `cache()` {#collection-method}

`cache` 函数可以用来从缓存中获取值。如果缓存中不存在给定的健，则返回一个可选的默认值：

````
$value = cache('key');

$value = cache('key', 'default');
````

你可以通过将一组键/值对传递给函数来将其添加到缓存中。与此同时，你还应该传递有效的分钟数或持续时间作为缓存过期时间：

````
cache(['key' => 'value'], 5);

cache(['key' => 'value'], Carbon::now()->addSeconds(10));
````

<a name="method-class-uses-recursive"></a>

#### `class_uses_recursive()` {#collection-method}

`class_uses_recursive` 函数返回一个类使用的所有 traits，包括任何子类使用的 traits：

````
$traits = class_uses_recursive(App\User::class);
````

<a name="method-collect"></a>

#### `collect()` {#collection-method}

`collect` 函数根据给定的数组创建一个 [集合](/docs/{{version}}/collections) 实例：

````
$collection = collect(['taylor', 'abigail']);
````

<a name="method-config"></a>
#### `config()` {#collection-method}

`config` 函数获取 [配置](/docs/{{version}}/configuration) 变量的值。可以使用「点」语法访问配置值，其中包括文件的名称和希望访问的选项。如果配置选项不存在，则可以指定一个默认值并返回：

````
$value = config('app.timezone');

$value = config('app.timezone', $default);
````

可以在运行时通过传递一组键/值对来设置配置变量：

````
config(['app.debug' => true]);
````

<a name="method-cookie"></a>

#### `cookie()` {#collection-method}

`cookie` 函数创建一个新的 [cookie](/docs/{{version}}/requests#cookies) 实例：

````
$cookie = cookie('name', 'value', $minutes);
````

<a name="method-csrf-field"></a>

#### `csrf_field()` {#collection-method}

`csrf_field` 函数生成包含 CSRF 令牌值的 HTML `hidden` 表单字段。例如，使用 [Blade 语法](/docs/{{version}}/blade)：

````
{{ csrf_field() }}
````

<a name="method-csrf-token"></a>
#### `csrf_token()` {#collection-method}

`csrf_token` 函数获取当前 CSRF 令牌的值：

````
$token = csrf_token();
````

<a name="method-dd"></a>
#### `dd()` {#collection-method}

`dd` 函数输出给定的值并结束脚本运行：

````
dd($value);

dd($value1, $value2, $value3, ...);
````

如果你不想终止脚本运行，请改用 `dump` 函数。

<a name="method-decrypt"></a>

#### `decrypt()` {#collection-method}

`decrypt` 函数使用 Laravel 的 [加密器](/docs/{{version}}/encryption) 来解密给定的值：

````
$decrypted = decrypt($encrypted_value);
````

<a name="method-dispatch"></a>

#### `dispatch()` {#collection-method}

`dispatch` 函数将给定的 [任务](/docs/{{version}}/queues#creating-jobs) 推送到 Laravel [任务列队](/docs/{{version}}/queues) 中：

````
dispatch(new App\Jobs\SendEmails);
````

<a name="method-dispatch-now"></a>

#### `dispatch_now()` {#collection-method}

`dispatch_now` 函数立即运行给定的 [任务](/docs/{{version}}/queues#creating-jobs)，并从其 `handle` 方法返回值：

````
$result = dispatch_now(new App\Jobs\SendEmails);
````

<a name="method-dump"></a>

#### `dump()` {#collection-method}

`dump` 函数打印给定的变量：

````
dump($value);

dump($value1, $value2, $value3, ...);
````

如果要在打印变量后停止执行脚本，请改用 [`dd`](#method-dd) 函数。

<a name="method-encrypt"></a>

#### `encrypt()` {#collection-method}

`encrypt` 函数使用 Laravel 的 [加密器](/docs/{{version}}/encryption) 对给定的值进行加密：

````
$encrypted = encrypt($unencrypted_value);
````

<a name="method-env"></a>

#### `env()` {#collection-method}

`env` 函数获取 [环境变量](/docs/{{version}}/configuration#environment-configuration) 的值或者返回默认值：

````
$env = env('APP_ENV');

// 如果环境变量不存在则返回默认值...
$env = env('APP_ENV', 'production');
````

<a name="method-event"></a>
#### `event()` {#collection-method}

`event` 函数将给定的 [事件](/docs/{{version}}/events) 分派给它的监听器：

````
event(new UserRegistered($user));
````

<a name="method-factory"></a>
#### `factory()` {#collection-method}

`factory` 函数根据给定的类、名称和数量创建一个模型工厂构建器。可以在 [测试](/docs/{{version}}/database-testing#writing-factories) or [数据填充](/docs/{{version}}/seeding#using-model-factories) 中使用：

````
$user = factory(App\User::class)->make();
````

<a name="method-filled"></a>
#### `filled()` {#collection-method}

`filled` 函数判断给定的值是否不为「空」：

````
filled(0);
filled(true);
filled(false);

// true

filled('');
filled('   ');
filled(null);
filled(collect());

// false
````

要使用与 `filled` 相反的功能，请看 [blank](/docs/{{version}}/helpers#method-blank) 方法。


<a name="method-info"></a>
#### `info()` {#collection-method}

`info` 函数将信息写入日志：

````
info('Some helpful information!');
````

有前后关系的数组也可以传递给函数：

````
info('User login attempt failed.', ['id' => $user->id]);
````

<a name="method-logger"></a>
#### `logger()` {#collection-method}

`logger` 函数可以将一个 `debug` 级别的消息写入到 [日志](/docs/{{version}}/errors#logging) 中：

````
logger('Debug message');
````

有前后关系的数组也可以传递给函数：

````
logger('User has logged in.', ['id' => $user->id]);
````

如果没有传值给函数则返回 [日志](/docs/{{version}}/errors#logging) 的实例：

````
logger()->error('You are not allowed here.');
````

<a name="method-method-field"></a>
#### `method_field()` {#collection-method}

`method_field` 函数生成一个 HTML `hidden` 表单字段，其中包含表单的 HTTP 动作的欺骗值。例如，使用 [Blade 语法](/docs/{{version}}/blade)：

````
<form method="POST">
    {{ method_field('DELETE') }}
</form>
````

<a name="method-now"></a>
#### `now()` {#collection-method}

`now` 函数为当前时间创建一个新的 `Illuminate\Support\Carbon` 实例:

````
$now = now();
````

<a name="method-old"></a>
#### `old()` {#collection-method}

`old` 函数 [获取](/docs/{{version}}/requests#retrieving-input) 会话中闪存的 [旧输入](/docs/{{version}}/requests#old-input) 值：

````
$value = old('value');

$value = old('value', 'default');
````

<a name="method-optional"></a>
#### `optional()` {#collection-method}

`optional` 函数可以接受任何参数，并且允许你访问该对象的属性或者调用方法。如果给定的对象是 `null` ， 那么属性和方法会简单地返回 `null` 而不是产生一个错误：

````
return optional($user->address)->street;

{!! old('name', optional($user)->name) !!}
````

<a name="method-policy"></a>
#### `policy()` {#collection-method}

`policy` 方法为给定的类获取一个 [策略](/docs/{{version}}/authorization#creating-policies) 实例：

````
$policy = policy(App\User::class);
````

<a name="method-redirect"></a>
#### `redirect()` {#collection-method}

`redirect` 函数返回一个 [重定向 HTTP 响应](/docs/{{version}}/responses#redirects)，如果没有没有传入参数，则返回重定向实例：

````
return redirect('/home');

return redirect()->route('route.name');
````

<a name="method-report"></a>
#### `report()` {#collection-method}

`report` 函数将使用 [异常处理程序](/docs/{{version}}/errors#the-exception-handler) 的 `report` 方法抛出异常：

````
report($e);
````

<a name="method-request"></a>
#### `request()` {#collection-method}

`request` 函数返回当前 [请求](/docs/{{version}}/requests) 实例或者获取输入项：

````
$request = request();

$value = request('key', $default = null)
````

<a name="method-rescue"></a>

#### `rescue()` {#collection-method}

`rescue` 函数执行给定的闭包并捕获执行期间发生的任何异常。所有被捕获的异常将被发送到你的 [异常处理程序](/docs/{{version}}/errors#the-exception-handler) 的 `report` 方法。要注意的是，该请求将继续处理：

````
return rescue(function () {
    return $this->method();
});
````

你也可以将第二个参数传递给 `rescue` 方法。如果在执行闭包时发生异常，这个参数将是应该返回的默认值：

````
return rescue(function () {
    return $this->method();
}, false);

return rescue(function () {
    return $this->method();
}, function () {
    return $this->failure();
});
````

<a name="method-resolve"></a>
#### `resolve()` {#collection-method}

`resolve` 函数使用 [服务容器](/docs/{{version}}/container) 将给定的类或接口名称解析为其实例：

````
$api = resolve('HelpSpot\API');
````

<a name="method-response"></a>

#### `response()` {#collection-method}

`response` 函数创建 [响应](/docs/{{version}}/responses) 实例或者获取响应工厂实例：

````
return response('Hello World', 200, $headers);

return response()->json(['foo' => 'bar'], 200, $headers);
````

<a name="method-retry"></a>
#### `retry()` {#collection-method}

`retry` 函数尝试执行给定的回调，直到到达给定的最大尝试次数。如果回调没有抛出异常，则返回值将被返回。如果回调抛出异常，它将自动重试。如果超过最大尝试次数，则会抛出异常：

````
return retry(5, function () {
    // 在 100ms 左右尝试 5 次...
}, 100);
````

<a name="method-session"></a>
#### `session()` {#collection-method}

`session` 函数可以用来获取或者设置 [Session](/docs/{{version}}/session) 值：

````
$value = session('key');
````

你可以通过将一组键/值对传递给该函数来设置值：

````
session(['chairs' => 7, 'instruments' => 3]);
````

如果没有传递值给函数，则返回 Session 实例：

````
$value = session()->get('key');

session()->put('key', $value);
````

<a name="method-tap"></a>
#### `tap()` {#collection-method}

`tap` 函数接受两个参数：一个任意的 `$value` 和一个闭包。`$value` 将被传递给闭包，然后由 `tap` 函数返回。不需要在闭包中使用 `return` 返回值。

````
$user = tap(User::first(), function ($user) {
    $user->name = 'taylor';

    $user->save();
});
````

如果没有闭包被传递给 `tap` 函数，你可以调用给定 `$value` 的任何方法。而你调用的方法的返回值始终为 `$value` ，无论方法在其定义中实际返回的是什么。例如，Eloquent 的 `update` 方法通常会返回一个整数。但是，我们可以强制通过 `tap` 函数链式调用 `update` 方法来返回模型本身：

```
$user = tap($user)->update([
    'name' => $name,
    'email' => $email,
]);
```

<a name="method-today"></a>

#### `today()` {#collection-method}

`today` 函数为当前日期创建一个新的 `Illuminate\Support\Carbon` 实例:

````
$today = today();
````

<a name="method-throw-if"></a>

#### `throw_if()` {#collection-method}

如果给定的布尔表达式计算结果为 `true`，`throw_if` 函数抛出给定的异常：

````
throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

throw_if(
    ! Auth::user()->isAdmin(),
    AuthorizationException::class,
    'You are not allowed to access this page'
);
````

<a name="method-throw-unless"></a>
#### `throw_unless()` {#collection-method}

如果给定的布尔表达式计算结果为`false`，则 `throw_unless`函数会抛出给定的异常：

````
throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

throw_unless(
    Auth::user()->isAdmin(),
    AuthorizationException::class,
    'You are not allowed to access this page'
);
````

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()` {#collection-method}

`trait_uses_recursive` 函数返回一个类使用的所有 trait：

````
$traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);
````

<a name="method-transform"></a>
#### `transform()` {#collection-method}

如果给定的值不为空，那么 `transform` 函数对给定的值执行闭包并返回其结果：

````
$callback = function ($value) {
    return $value * 2;
};

$result = transform(5, $callback);

// 10
````

默认值或闭包也可以作为方法的第三个参数传递。如果给定值为空白，则返回该值：

````
$result = transform(null, $callback, 'The value is blank');

// The value is blank
````

<a name="method-validator"></a>
#### `validator()` {#collection-method}

`validator` 函数用给定的参数创建一个新的 [验证器](/docs/{{version}}/validation) 实例。为方便起见，你可以使用它来代替 `Validator` facade ：

````
$validator = validator($data, $rules, $messages);
````

<a name="method-value"></a>

#### `value()` {#collection-method}

`value` 函数返回给定的值。但是，如果将一个闭包传递给该函数，则将执行该闭包并返回其结果：

````
$result = value(true);

// true

$result = value(function () {
    return false;
});

// false
````

<a name="method-view"></a>
#### `view()` {#collection-method}

`view` 函数获取一个 [视图](/docs/{{version}}/views) 实例：

````
return view('auth.login');
````

<a name="method-with"></a>
#### `with()` {#collection-method}

`with` 函数会返回给定的值。如果传入一个闭包作为该函数的第二个参数，会返回闭包执行的结果：

````
$callback = function ($value) {
    return (is_numeric($value)) ? $value * 2 : 0;
};

$result = with(5, $callback);

// 10

$result = with(null, $callback);

// 0

$result = with(5, null);

// 5
````

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [Seven Du](https://github.com/medz) | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/5564821?s=300"> | 翻译 | 基于 Laravel 的社交开源系统 [ThinkSNS+](https://github.com/slimkit/thinksns-plus) 欢迎 Star。 |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | Review | Stay Hungry. Stay Foolish. |


---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
