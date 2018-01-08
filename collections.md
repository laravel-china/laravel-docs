# Laravel 的集合 Collection

- [简介](#introduction)
    - [创建集合](#creating-collections)
- [可用的方法](#available-methods)
- [高阶消息传递](#higher-order-messages)

<a name="introduction"></a>
## 简介

`Illuminate\Support\Collection` 类提供了一个更具可读性的、更便于处理数组数据的封装。具体例子看下面的代码。我们使用了 `collect` 函数从数组中创建新的集合实例，对其中的每个元素运行 `strtoupper` 函数之后再移除所有的空元素：

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });

正如你看到的，`Collection` 类允许你链式调用其方法，以达到在底层数组上优雅地执行 map 和 reject 操作。一般来说，集合是不可改变的，这意味着每个 `Collection` 方法都会返回一个全新的 `Collection` 实例。

<a name="creating-collections"></a>
### 创建集合

如上所述，辅助函数 `collect` 会为给定的数组返回一个新的 `Illuminate\Support\Collection` 实例。也就是说，创建一个集合就这么简单：

    $collection = collect([1, 2, 3]);

> {tip} 默认情况下， [Eloquent](/docs/{{version}}/eloquent) 查询的结果返回的内容都是 `Collection` 实例。

<a name="available-methods"></a>
## 可用的方法

这个文档接下来的内容，我们会探讨 `Collection` 类每个可用的方法。记住，所有方法都可以以方法链的形式优雅地操纵数组。而且，几乎所有的方法都会返回新的 `Collection` 实例，允许你在必要时保存集合的原始副本。

<style>
    #collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[collapse](#method-collapse)
[combine](#method-combine)
[contains](#method-contains)
[containsStrict](#method-containsstrict)
[count](#method-count)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffKeys](#method-diffkeys)
[each](#method-each)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectByKeys](#method-intersectByKeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[map](#method-map)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[min](#method-min)
[mode](#method-mode)
[nth](#method-nth)
[only](#method-only)
[partition](#method-partition)
[pipe](#method-pipe)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[reverse](#method-reverse)
[search](#method-search)
[shift](#method-shift)
[shuffle](#method-shuffle)
[slice](#method-slice)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[splice](#method-splice)
[split](#method-split)
[sum](#method-sum)
[take](#method-take)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[values](#method-values)
[when](#method-when)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[zip](#method-zip)

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

<a name="method-all"></a>
#### `all()` {#collection-method .first-collection-method}

`all` 方法返回该集合表示的底层 `数组`：

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-average"></a>
#### `average()` {#collection-method}

[`avg`](#method-avg) 方法的别名。

<a name="method-avg"></a>
#### `avg()` {#collection-method}

`avg` 方法返回给定键的 [平均值](https://en.wikipedia.org/wiki/Average)：

    $average = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->avg('foo');

    // 20

    $average = collect([1, 1, 2, 4])->avg();

    // 2

<a name="method-chunk"></a>
#### `chunk()` {#collection-method}

`chunk` 方法将集合拆成多个指定大小的小集合：

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->toArray();

    // [[1, 2, 3, 4], [5, 6, 7]]

这个方法特别适用在使用网格系统时的 [视图](/docs/{{version}}/views)，如 [Bootstrap](http://getbootstrap.com/css/#grid)。想像你有一个 [Eloquent](/docs/{{version}}/eloquent) 模型的集合要在网格中显示：

```HTML
@foreach ($products->chunk(3) as $chunk)
    <div class="row">
        @foreach ($chunk as $product)
            <div class="col-xs-4">{{ $product->name }}</div>
        @endforeach
    </div>
@endforeach
```

<a name="method-collapse"></a>
#### `collapse()` {#collection-method}

`collapse` 方法将多个数组的集合合并成一个数组的集合：

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-combine"></a>
#### `combine()` {#collection-method}

`combine` 方法可以将一个集合的值作为「键」，再将另一个数组或者集合的值作为「值」合并成一个集合：

    $collection = collect(['name', 'age']);

    $combined = $collection->combine(['George', 29]);

    $combined->all();

    // ['name' => 'George', 'age' => 29]

<a name="method-contains"></a>
#### `contains()` {#collection-method}

`contains` 方法判断集合是否包含给定的项目：

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    // true

    $collection->contains('New York');

    // false

你也可以用 `contains` 方法匹配一对键/值，即判断给定的配对是否存在于集合中：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->contains('product', 'Bookcase');

    // false

最后，你也可以传递一个回调到 `contains` 方法来执行自己的真实测试：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function ($value, $key) {
        return $value > 5;
    });

    // false

`contains` 方法在检查项目值时使用「宽松」比较，意味着具有整数值的字符串将被视为等于相同值的整数。 相反 [`containsStrict`](https://laravel.com/docs/5.5/collections#method-containsstrict) 方法则是使用「严格」比较进行过滤。

<a name="method-containsstrict"></a>
#### `containsStrict()` {#collection-method}

此方法和 [`contains`](#method-contains) 方法类似，但是它却是使用了「严格」比较来比较所有值。

<a name="method-count"></a>
#### `count()` {#collection-method}

`count` 方法返回该集合内的项目总数：

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-diff"></a>
#### `diff()` {#collection-method}

`diff` 方法将集合与其它集合或纯 PHP 数组进行值的比较，然后返回原集合中存在而给定集合中不存在的值：

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

<a name="method-diffassoc"></a>
#### `diffAssoc()` {#collection-method}

`diffAssoc` 该方法与另外一个集合或基于它的键和值的 PHP 数组进行比较。这个方法会返回原集合不存在于给定集合中的键值对 ：

    $collection = collect([
        'color' => 'orange',
        'type' => 'fruit',
        'remain' => 6
    ]);

    $diff = $collection->diffAssoc([
        'color' => 'yellow',
        'type' => 'fruit',
        'remain' => 3,
        'used' => 6
    ]);

    $diff->all();

    // ['color' => 'orange', 'remain' => 6]

<a name="method-diffkeys"></a>
#### `diffKeys()` {#collection-method}

`diffKeys` 方法与另外一个集合或 PHP 数组的「键」进行比较，然后返回原集合中存在而给定的集合中不存在「键」所对应的键值对：

    $collection = collect([
        'one' => 10,
        'two' => 20,
        'three' => 30,
        'four' => 40,
        'five' => 50,
    ]);

    $diff = $collection->diffKeys([
        'two' => 2,
        'four' => 4,
        'six' => 6,
        'eight' => 8,
    ]);

    $diff->all();

    // ['one' => 10, 'three' => 30, 'five' => 50]

<a name="method-each"></a>
#### `each()` {#collection-method}

`each` 迭代集合中的内容并将其传递到回调函数中：

    $collection = $collection->each(function ($item, $key) {
        //
    });

如果你想要中断对内容的迭代，那就从回调中返回 `false`：

    $collection = $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-every"></a>
#### `every()` {#collection-method}

`every` 方法可用于验证集合中每一个元素都通过给定的真实测试：

    collect([1, 2, 3, 4])->every(function ($value, $key) {
        return $value > 2;
    });

    // false

<a name="method-except"></a>
#### `except()` {#collection-method}

`except` 方法返回集合中除了指定键以外的所有项目：

    $collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

    $filtered = $collection->except(['price', 'discount']);

    $filtered->all();

    // ['product_id' => 1]

与 `except` 相反的方法，请查看 [only](#method-only)。

<a name="method-filter"></a>
#### `filter()` {#collection-method}

`filter` 方法使用给定的回调函数过滤集合的内容，只留下那些通过给定真实测试的内容：

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [3, 4]

如果没有提供回调函数，集合中所有返回 `false` 的元素都会被移除：

    $collection = collect([1, 2, 3, null, false, '', 0, []]);

    $collection->filter()->all();

    // [1, 2, 3]

与 `filter` 相反的方法，可以查看 [reject](#method-reject)。

<a name="method-first"></a>
#### `first()` {#collection-method}

`first` 方法返回集合中通过给定真实测试的第一个元素：

    collect([1, 2, 3, 4])->first(function ($value, $key) {
        return $value > 2;
    });

    // 3

你也可以不传入参数使用 `first` 方法以获取集合中第一个元素。如果集合是空的，则会返回 `null`：

    collect([1, 2, 3, 4])->first();

    // 1

<a name="method-flatmap"></a>
#### `flatMap()` {#collection-method}

`flatMap` 方法遍历集合并将其中的每个值传递到给定的回调。可以通过回调修改每个值的内容再返回出来，从而形成一个新的被修改过内容的集合。然后你就可以用 `all()` 打印修改后的数组：

    $collection = collect([
        ['name' => 'Sally'],
        ['school' => 'Arkansas'],
        ['age' => 28]
    ]);

    $flattened = $collection->flatMap(function ($values) {
        return array_map('strtoupper', $values);
    });

    $flattened->all();

    // ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];

<a name="method-flatten"></a>
#### `flatten()` {#collection-method}

`flatten` 方法将多维集合转为一维的：

    $collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];

你还可以选择性地传入「深度」参数：

    $collection = collect([
        'Apple' => [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ],
        'Samsung' => [
            ['name' => 'Galaxy S7', 'brand' => 'Samsung']
        ],
    ]);

    $products = $collection->flatten(1);

    $products->values()->all();

    /*
        [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
            ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
        ]
    */

在这个例子里，调用 `flatten` 方法时不传入深度参数的话也会将嵌套数组转成一维的，然后返回 `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`。传入深度参数能让你限制设置返回数组的层数。

<a name="method-flip"></a>
#### `flip()` {#collection-method}

`flip` 方法将集合中的键和对应的数值进行互换：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` {#collection-method}

`forget` 方法通过给定的键来移除掉集合中对应的内容：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $collection->forget('name');

    $collection->all();

    // ['framework' => 'laravel']

> {note} 与大多数集合的方法不同，`forget` 不会返回修改过后的新集合；它会直接修改原来的集合。

<a name="method-forpage"></a>
#### `forPage()` {#collection-method}

`forPage` 方法返回给定页码上显示的项目的新集合。这个方法接受页码作为其第一个参数和每页显示的项目数作为其第二个参数。

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunk = $collection->forPage(2, 3);

    $chunk->all();

    // [4, 5, 6]

<a name="method-get"></a>
#### `get()` {#collection-method}

`get` 方法返回给定键的项目。如果该键不存在，则返回 `null`：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('name');

    // taylor

你可以选择性地传递默认值作为第二个参数：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('foo', 'default-value');

    // default-value

你甚至可以将回调函数当作默认值。如果指定的键不存在，就会返回回调的结果：

    $collection->get('email', function () {
        return 'default-value';
    });

    // default-value

<a name="method-groupby"></a>
#### `groupBy()` {#collection-method}

`groupBy` 方法根据给定的键对集合内的项目进行分组：

    $collection = collect([
        ['account_id' => 'account-x10', 'product' => 'Chair'],
        ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ['account_id' => 'account-x11', 'product' => 'Desk'],
    ]);

    $grouped = $collection->groupBy('account_id');

    $grouped->toArray();

    /*
        [
            'account-x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'account-x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

除了传入一个字符串的「键」，你还可以传入一个回调。该回调应该返回你希望用来分组的键的值。

    $grouped = $collection->groupBy(function ($item, $key) {
        return substr($item['account_id'], -3);
    });

    $grouped->toArray();

    /*
        [
            'x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

<a name="method-has"></a>
#### `has()` {#collection-method}

`has` 方法判断集合中是否存在给定的键：

    $collection = collect(['account_id' => 1, 'product' => 'Desk']);

    $collection->has('product');

    // true

<a name="method-implode"></a>
#### `implode()` {#collection-method}

`implode` 方法合并集合中的项目。其参数取决于集合中项目的类型。如果集合包含数组或对象，你应该传入你希望连接的属性的键，以及你希望放在值之间用来「拼接」的字符串：:

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair


如果集合包含简单的字符串或数值，只需要传入「拼接」用的字符串作为该方法的唯一参数即可：

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()` {#collection-method}

`intersect` 方法从原集合中删除不在给定「数组」或集合中的任何值。最终的集合会保留原集合的键：

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

<a name="method-intersectByKeys"></a>
#### `intersectByKeys()` {#collection-method}

`intersectByKeys` 方法删除原集合中不存在于给定「数组」或集合中的任何键。

    $collection = collect([
        'serial' => 'UX301', 'type' => 'screen', 'year' => 2009
    ]);

    $intersect = $collection->intersectByKeys([
        'reference' => 'UX404', 'type' => 'tab', 'year' => 2011
    ]);

    $intersect->all();

    // ['type' => 'screen', 'year' => 2009]

<a name="method-isempty"></a>
#### `isEmpty()` {#collection-method}

如果集合是空的，`isEmpty` 方法返回 `true`，否则返回 `false`：

    collect([])->isEmpty();

    // true

<a name="method-isnotempty"></a>
#### `isNotEmpty()` {#collection-method}

如果集合不是空的，`isNotEmpty` 方法会返回 `true`：否则返回 `false`：

    collect([])->isNotEmpty();

    // false

<a name="method-keyby"></a>
#### `keyBy()` {#collection-method}

`keyBy` 方法以给定的键作为集合的键。如果多个项目具有相同的键，则只有最后一个项目会显示在新集合中：

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'desk'],
        ['product_id' => 'prod-200', 'name' => 'chair'],
    ]);

    $keyed = $collection->keyBy('product_id');

    $keyed->all();

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

你也可以传入一个回调方法，回调返回的值会作为该集合的键：

    $keyed = $collection->keyBy(function ($item) {
        return strtoupper($item['product_id']);
    });

    $keyed->all();

    /*
        [
            'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

<a name="method-keys"></a>
#### `keys()` {#collection-method}

`keys` 方法返回集合的所有键：

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keys = $collection->keys();

    $keys->all();

    // ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` {#collection-method}

`last` 方法返回集合中通过给定真实测试的最后一个元素：

    collect([1, 2, 3, 4])->last(function ($value, $key) {
        return $value < 3;
    });

    // 2


你也可以不传入参数调用 `last` 方法来获取集合中最后一个元素。如果集合是空的，返回 `null`：

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-map"></a>
#### `map()` {#collection-method}

`map` 方法遍历集合并将每一个值传入给定的回调。该回调可以任意修改项目并返回，从而形成新的被修改过项目的集合：

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });

    $multiplied->all();

    // [2, 4, 6, 8, 10]

> {note} 像其他集合方法一样，`map` 返回一个新的集合实例；它不会修改它所调用的集合。如果你想改变原集合，得使用 [`transform`](#method-transform) 方法。


<a name="method-mapwithkeys"></a>
#### `mapWithKeys()` {#collection-method}

`mapWithKeys` 方法遍历集合并将每个值传入给定的回调。回调应该返回包含一个键值对的关联数组：

    $collection = collect([
        [
            'name' => 'John',
            'department' => 'Sales',
            'email' => 'john@example.com'
        ],
        [
            'name' => 'Jane',
            'department' => 'Marketing',
            'email' => 'jane@example.com'
        ]
    ]);

    $keyed = $collection->mapWithKeys(function ($item) {
        return [$item['email'] => $item['name']];
    });

    $keyed->all();

    /*
        [
            'john@example.com' => 'John',
            'jane@example.com' => 'Jane',
        ]
    */

<a name="method-max"></a>
#### `max()` {#collection-method}

`max` 方法返回给定键的最大值：

    $max = collect([['foo' => 10], ['foo' => 20]])->max('foo');

    // 20

    $max = collect([1, 2, 3, 4, 5])->max();

    // 5

<a name="method-median"></a>
#### `median()` {#collection-method}

`median` 方法返回给定键的 [中间值](https://en.wikipedia.org/wiki/Median) ：

    $median = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->median('foo');

    // 15

    $median = collect([1, 1, 2, 4])->median();

    // 1.5

<a name="method-merge"></a>
#### `merge()` {#collection-method}

`merge` 方法将给定数组或集合合并到原集合。如果给定项目中的字符串键与原集合中的字符串键匹配，给定的项目的值将会覆盖原集合中的值：

    $collection = collect(['product_id' => 1, 'price' => 100]);

    $merged = $collection->merge(['price' => 200, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'price' => 200, 'discount' => false]

如果给定的项目的键是数字，这些值将被追加到集合的末尾：

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-min"></a>
#### `min()` {#collection-method}

`min` 方法返回给定键的最小值：

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

    // 10

    $min = collect([1, 2, 3, 4, 5])->min();

    // 1

<a name="method-mode"></a>
#### `mode()` {#collection-method}

`mode` 方法返回给定键的 [众数值](https://en.wikipedia.org/wiki/Mode_(statistics))：

    $mode = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->mode('foo');

    // [10]

    $mode = collect([1, 1, 2, 4])->mode();

    // [1]

<a name="method-nth"></a>
#### `nth()` {#collection-method}

`nth` 方法创建由每隔 n 个元素组成一个新的集合：

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->nth(4);

    // ['a', 'e']
你也可以选择传入一个偏移位置作为第二个参数

    $collection->nth(4, 1);

    // ['b', 'f']

<a name="method-only"></a>
#### `only()` {#collection-method}

`only` 方法返回集合中给定键的所有项目：

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->only(['product_id', 'name']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

与 `only` 相反的方法，请查看 [except](#method-except)。

<a name="method-partition"></a>
#### `partition()` {#collection-method}

`partition` 方法可以和PHP 中的 `list` 方法结合使用，来分开通过指定条件的元素以及那些不通过指定条件的元素：

    $collection = collect([1, 2, 3, 4, 5, 6]);

    list($underThree, $aboveThree) = $collection->partition(function ($i) {
        return $i < 3;
    });

<a name="method-pipe"></a>
#### `pipe()` {#collection-method}

`pipe` 方法将集合传给给定的回调并返回结果：

    $collection = collect([1, 2, 3]);

    $piped = $collection->pipe(function ($collection) {
        return $collection->sum();
    });

    // 6

<a name="method-pluck"></a>
#### `pluck()` {#collection-method}

`pluck` 方法获取集合中给定键对应的所有值：

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $plucked = $collection->pluck('name');

    $plucked->all();

    // ['Desk', 'Chair']

你也可以通过传入第二个参数来指定生成的集合的键：

    $plucked = $collection->pluck('name', 'product_id');

    $plucked->all();

    // ['prod-100' => 'Desk', 'prod-200' => 'Chair']

<a name="method-pop"></a>
#### `pop()` {#collection-method}

`pop` 方法移除并返回集合中的最后一个项目：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop();

    // 5

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-prepend"></a>
#### `prepend()` {#collection-method}

`prepend` 方法将给定的值添加到集合的开头：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

你也可以传递第二个参数来设置前置项的键：

    $collection = collect(['one' => 1, 'two' => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two' => 2]

<a name="method-pull"></a>
#### `pull()` {#collection-method}

`pull` 方法把给定键对应的值从集合中移除并返回：

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` {#collection-method}

`push` 方法把给定值添加到集合的末尾：

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` {#collection-method}

`put` 方法在集合内设置给定的键值对：

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {#collection-method}

`random` 方法从集合中返回一个随机项：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (retrieved randomly)

你可以选择性传入一个整数到 `random` 来指定要获取的随机项的数量。只要你显式传递你希望接收的数量时，则会返回项目的集合：

    $random = $collection->random(3);

    $random->all();

    // [0 => 1, 1 => 2, 4 => 5] - (retrieved randomly)

<a name="method-reduce"></a>
#### `reduce()` {#collection-method}

`reduce` 方法将每次迭代的结果传递给下一次迭代直到集合减少为单个值：

    $collection = collect([1, 2, 3]);

    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });

    // 6

第一次迭代时 `$carry` 的数值为 `null`；你也可以通过传入第二个参数到 `reduce` 来指定它的初始值：

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);

    // 10

<a name="method-reject"></a>
#### `reject()` {#collection-method}

`reject` 方法使用指定的回调过滤集合。如果回调返回 `true` ，就会把对应的项目从集合中移除：

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [1, 2]

与 `reject` 相反的方法，查看 [`filter`](#method-filter) 。

<a name="method-reverse"></a>
#### `reverse()` {#collection-method}

`reverse` 方法倒转集合中项目的顺序：

    $collection = collect([1, 2, 3, 4, 5]);

    $reversed = $collection->reverse();

    $reversed->all();

    // [5, 4, 3, 2, 1]

<a name="method-search"></a>
#### `search()` {#collection-method}

`search` 方法搜索给定的值并返回它的键。如果找不到，则返回 `false`：

    $collection = collect([2, 4, 6, 8]);

    $collection->search(4);

    // 1

搜索使用「宽松」比较完成，这意味着具有整数值的字符串会被认为等于相同值的整数。要使用「严格」比较，就传入 `true` 作为该方法的第二个参数：

    $collection->search('4', true);

    // false

另外，你可以通过回调来搜索第一个通过真实测试的项目：

    $collection->search(function ($item, $key) {
        return $item > 5;
    });

    // 2

<a name="method-shift"></a>
#### `shift()` {#collection-method}

`shift` 方法移除并返回集合的第一个项目：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->shift();

    // 1

    $collection->all();

    // [2, 3, 4, 5]

<a name="method-shuffle"></a>
#### `shuffle()` {#collection-method}

`shuffle` 方法随机排序集合中的项目：

    $collection = collect([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    // [3, 2, 5, 1, 4] - (generated randomly)

<a name="method-slice"></a>
#### `slice()` {#collection-method}

`slice` 方法返回集合中给定值后面的部分：

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

如果你想限制返回内容的大小，就将期望的大小作为第二个参数传递给方法：

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

默认情况下，返回的内容将会保留原始键。假如你不希望保留原始的键，你可以使用 `values` 方法来重新建立索引。

<a name="method-sort"></a>
#### `sort()` {#collection-method}

`sort` 方法对集合进行排序。排序后的集合保留着原数组的键，所以在这个例子中我们使用 [`values`](#method-values) 方法来把键重置为连续编号的索引。

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

如果你有更高级的排序需求，你可以传入回调来用你自己的算法进行`排序`。请参阅 PHP 文档的 [`usort`](http://php.net/manual/en/function.usort.php#refsect1-function.usort-parameters)，这是集合的 `sort` 方法在底层所调用的。

> {tip} 如果要对嵌套数组或对象的集合进行排序，参考 [`sortBy`](#method-sortby) 和 [`sortByDesc`](#method-sortbydesc) 方法。


<a name="method-sortby"></a>
#### `sortBy()` {#collection-method}

`sortBy` 方法以给定的键对集合进行排序。排序后的集合保留了原数组键，所以在这个例子中，我们使用 [`values`](#method-values) 方法将键重置为连续编号的索引：

    $collection = collect([
        ['name' => 'Desk', 'price' => 200],
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
    ]);

    $sorted = $collection->sortBy('price');

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'price' => 100],
            ['name' => 'Bookcase', 'price' => 150],
            ['name' => 'Desk', 'price' => 200],
        ]
    */

你还可以传入自己的回调以决定如何对集合的值进行排序：

    $collection = collect([
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $sorted = $collection->sortBy(function ($product, $key) {
        return count($product['colors']);
    });

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'colors' => ['Black']],
            ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
            ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
        ]
    */

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {#collection-method}

这个方法与 [`sortBy`](#method-sortby) 方法一样，但是会以相反的顺序来对集合进行排序：

<a name="method-splice"></a>
#### `splice()` {#collection-method}

`splice` 方法删除并返回从给定值后的内容，原集合也会受到影响：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

你可以传入第二个参数以限制被删除内容的大小：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

此外，你可以传入含有新项目的第三个参数来代替集合中删除的项目：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]

<a name="method-split"></a>
#### `split()` {#collection-method}

`split` 方法将集合按给定的值拆分：

    $collection = collect([1, 2, 3, 4, 5]);

    $groups = $collection->split(3);

    $groups->toArray();

    // [[1, 2], [3, 4], [5]]

<a name="method-sum"></a>
#### `sum()` {#collection-method}

`sum` 方法返回集合内所有项目的总和：

    collect([1, 2, 3, 4, 5])->sum();

    // 15

如果集合包含嵌套数组或对象，则应该传入一个键来指定要进行求和的值：

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    // 1272

另外，你也可以传入回调来决定要用集合中的哪些值进行求和：

    $collection = collect([
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $collection->sum(function ($product) {
        return count($product['colors']);
    });

    // 6

<a name="method-take"></a>
#### `take()` {#collection-method}

`take` 方法返回给定数量项目的新集合：

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

你也可以传入负整数从集合末尾开始获取指定数量的项目：

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-tap"></a>
#### `tap()` {#collection-method}

`tap` 方法将集合传递给回调，在特定点「tap」集合。此举能让你对集合中的项目执行某些操作，而不影响集合本身：

    collect([2, 4, 3, 1, 5])
        ->sort()
        ->tap(function ($collection) {
            Log::debug('Values after sorting', $collection->values()->toArray());
        })
        ->shift();

    // 1

<a name="method-times"></a>
#### `times()` {#collection-method}

静态 `times` 方法通过回调在给定次数内创建一个新的集合：

    $collection = Collection::times(10, function ($number) {
        return $number * 9;
    });

    $collection->all();

    // [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]

使用这个方法可以与工厂结合使用创建出 [Eloquent](/docs/{{version}}/eloquent) 模型：

    $categories = Collection::times(3, function ($number) {
        return factory(Category::class)->create(['name' => 'Category #'.$number]);
    });

    $categories->all();

    /*
        [
            ['id' => 1, 'name' => 'Category #1'],
            ['id' => 2, 'name' => 'Category #2'],
            ['id' => 3, 'name' => 'Category #3'],
        ]
    */

<a name="method-toarray"></a>
#### `toArray()` {#collection-method}

`toArray` 方法将集合转换成 PHP `数组`。如果集合的值是 [Eloquent](/docs/{{version}}/eloquent) 模型，那也会被转换成数组：

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> {note} `toArray` 也会将所有集合的嵌套对象转换为数组。如果你想获取原数组，就改用 [`all`](#method-all) 方法。

<a name="method-tojson"></a>
#### `toJson()` {#collection-method}

`toJson` 方法将集合转换成 JSON 字符串：

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk", "price":200}'

<a name="method-transform"></a>
#### `transform()` {#collection-method}

`transform` 方法迭代集合并对集合内的每个项目调用给定的回调。而集合的内容也会被回调返回的值取代：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });

    $collection->all();

    // [2, 4, 6, 8, 10]

> {note} 与大多数集合的方法不同，`transform` 会修改集合本身。如果你想创建新的集合，就改用 [`map`](#method-map) 方法。

<a name="method-union"></a>
#### `union()` {#collection-method}

`union` 方法将给定的数组添加到集合中。如果给定的数组中含有与原集合一样的键，则原集合的值不会被改变：

    $collection = collect([1 => ['a'], 2 => ['b']]);

    $union = $collection->union([3 => ['c'], 1 => ['b']]);

    $union->all();

    // [1 => ['a'], 2 => ['b'], 3 => ['c']]

<a name="method-unique"></a>
#### `unique()` {#collection-method}

`unique` 方法返回集合中所有唯一的项目。返回的集合保留着原数组的键，所以在这个例子中，我们使用 [`values`](#method-values) 方法来把键重置为连续编号的索引。

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

处理嵌套数组或对象时，你可以指定用来决定唯一性的键：

    $collection = collect([
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]);

    $unique = $collection->unique('brand');

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ]
    */

你也可以传入自己的回调来确定项目的唯一性：

    $unique = $collection->unique(function ($item) {
        return $item['brand'].$item['type'];
    });

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
            ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
        ]
    */

在检查项目值时 `unique` 方法使用的是「宽松」比较，意味着具有整数值的字符串将被视为等于相同值的整数。使用 [`uniqueStrict`](#method-uniquestrict) 可以进行「严格」比较 。

<a name="method-uniquestrict"></a>
#### `uniqueStrict()` {#collection-method}

这个方法的使用和 [`unique`](#method-unique) 方法类似，只是使用了「严格」比较来过滤。

<a name="method-values"></a>
#### `values()` {#collection-method}

`values` 方法返回键被重置为连续编号的新集合：

    $collection = collect([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200]
    ]);

    $values = $collection->values();

    $values->all();

    /*
        [
            0 => ['product' => 'Desk', 'price' => 200],
            1 => ['product' => 'Desk', 'price' => 200],
        ]
    */

<a name="method-when"></a>
#### `when()` {#collection-method}

`when` 方法当传入的第一个参数为 `true` 的时，将执行给定的回调：

    $collection = collect([1, 2, 3]);

    $collection->when(true, function ($collection) {
        return $collection->push(4);
    });

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-where"></a>
#### `where()` {#collection-method}

`where` 方法通过给定的键值过滤集合：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->where('price', 100);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

比较数值的时候，`where` 方法使用「宽松」比较，意味着具有整数值的字符串将被认为等于相同值的整数。使用 [`whereStrict`](#method-wherestrict) 方法来进行「严格」比较过滤。

<a name="method-wherestrict"></a>
#### `whereStrict()` {#collection-method}

这个方法与 [`where`](#method-where) 方法一样；但是会以「严格」比较来匹配所有值：

<a name="method-wherein"></a>
#### `whereIn()` {#collection-method}

`whereIn` 方法通过给定的键值数组来过滤集合：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Bookcase', 'price' => 150],
            ['product' => 'Desk', 'price' => 200],
        ]
    */

`whereIn` 方法在检查项目值时使用「宽松」比较，意味着具有整数值的字符串将被视为等于相同值的整数。你可以使用 [`whereInStrict`](#method-whereinstrict) 做「严格」比较。

<a name="method-whereinstrict"></a>
#### `whereInStrict()` {#collection-method}

此方法的使用和 [`whereIn`](#method-wherein) 方法类似，只是使用了「严格」比较来匹配所有值。

<a name="method-wherenotin"></a>
#### `whereNotIn()` {#collection-method}

`whereNotIn` 通过集合中不包含的给定键值对进行：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereNotIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

`whereNotIn` 方法在检查项目值时使用「宽松」比较，意味着具有整数值的字符串将被视为等于相同值的整数。你可以使用 [`whereNotInStrict`](#method-wherenotinstrict) 做比较 `严格` 的匹配。

<a name="method-wherenotinstrict"></a>
#### `whereNotInStrict()` {#collection-method}

此方法的使用和 [`whereNotIn`](#method-wherenotin) 方法类似，只是使用了「严格」比较来匹配所有值。

<a name="method-zip"></a>
#### `zip()` {#collection-method}

`zip` 方法将给定数组的值与相应索引处的原集合的值合并在一起：

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]

<a name="higher-order-messages"></a>
## 高阶消息传递

集合也提供对「高阶消息传递」的支持，即对集合执行常见操作的快捷方式。支持高阶消息传递的集合方法有：`average`， `avg`， `contains`， `each`， `every`， `filter`， `first`， `flatMap`， `map`， `partition`， `reject`， `sortBy`， `sortByDesc` 和 `sum`。

每个高阶消息都能作为集合实例的动态属性来访问。例如，使用 `each` 高阶消息传递在集合中的每个对象上调用一个方法：

    $users = User::where('votes', '>', 500)->get();

    $users->each->markAsVip();

同样，我们可以使用 `sum` 高阶消息来收集集合中「投票」总数：

    $users = User::where('group', 'Development')->get();

    return $users->sum->votes;

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@逸侨](http://yiqiao.me)  | <img class="avatar-66 rm-style" src="https://avatars0.githubusercontent.com/u/6688591">  |  翻译  | 学的慢，但一直再走~。QQ 群：[319347221](//shang.qq.com/wpa/qunwpa?idkey=ee3bb560b41ebdb59188263f353926494f1248c7137d3338dda180fa1ce25354) 欢迎大家加入，个人Blog：[yiqiao](http://yiqiao.me) |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  Review  | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
