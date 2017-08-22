# Laravel 的集合 Collection

- [简介](#introduction)
    - [创建集合](#creating-collections)
- [可用的方法](#available-methods)
- [高阶信息传递](#higher-order-messages)

<a name="introduction"></a>
## 简介

`Illuminate\Support\Collection` 类提供一个流畅、便利的封装来操控数组数据。如下面的示例代码，我们用 `collect` 函数从数组中创建新的集合实例，对每一个元素运行 `strtoupper` 函数，然后移除所有的空元素：

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });

如上面的代码示例，`Collection` 类支持链式调用，一般来说，每一个 `Collection` 方法会返回一个全新的 `Collection` 实例，你可以放心地进行链接调用。

<a name="creating-collections"></a>
### 创建集合

如上所述， `collect` 辅助函数会利用传入的数组生成一个新的 `Illuminate\Support\Collection` 实例。所以要创建一个集合就这么简单：

    $collection = collect([1, 2, 3]);

> {tip} 默认 [Eloquent](/docs/{{version}}/eloquent) 模型的查询结果总是以 `Collection` 实例返回。

<a name="available-methods"></a>
## 可用的方法

接下来，我们将会探讨 `Collection` 类的所有方法。要记得的是，所有方法都支持链式调用，几乎所有的方法都会返回新的 `Collection` 实例，让你保留原版的集合以备不时之需。

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
[intersectKey](#method-intersectkey)
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
## 方法清单

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

返回该集合所代表的底层 `数组`：

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-average"></a>
#### `average()` {#collection-method}

[`avg`] 方法的别名。

<a name="method-avg"></a>
#### `avg()` {#collection-method}

返回集合中所有项目的平均值：

    $average = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->avg('foo');

    // 20

    $average = collect([1, 1, 2, 4])->avg();

    // 2

<a name="method-chunk"></a>
#### `chunk()` {#collection-method}

将集合拆成多个指定大小的较小集合：

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->toArray();

    // [[1, 2, 3, 4], [5, 6, 7]]

这个方法在适用于网格系统如 [Bootstrap](http://getbootstrap.com/css/#grid) 的 [视图](/docs/{{version}}/views) 。想像你有一个 [Eloquent](/docs/{{version}}/eloquent) 模型的集合要显示在一个网格内：

    @foreach ($products->chunk(3) as $chunk)
        <div class="row">
            @foreach ($chunk as $product)
                <div class="col-xs-4">{{ $product->name }}</div>
            @endforeach
        </div>
    @endforeach

<a name="method-collapse"></a>
#### `collapse()` {#collection-method}

将多个数组组成的集合合成单个一维数组集合：

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-combine"></a>
#### `combine()` {#collection-method}

将集合的值作为「键」，合并另一个数组或者集合作为「键」对应的值。

    $collection = collect(['name', 'age']);

    $combined = $collection->combine(['George', 29]);

    $combined->all();

    // ['name' => 'George', 'age' => 29]

<a name="method-contains"></a>
#### `contains()` {#collection-method}

判断集合是否含有指定项目：

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    // true

    $collection->contains('New York');

    // false

你可以将一对键/值传入 `contains` 方法，用来判断该组合是否存在于集合内：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);
    
    $collection->contains('product', 'Bookcase');
    
    // false

最后，你也可以传入一个回调函数到 `contains` 方法内运行你自己的判断语句：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function ($value, $key) {
        return $value > 5;
    });
    
    // false

此方法是用宽松的匹配，你可以使用 [`containsStrict`](#method-containsstrict) 做比较 `严格` 的匹配。

<a name="method-containsstrict"></a>
#### `containsStrict()` {#collection-method}

此方法的使用于 [`contains`](#method-contains) 方法类似，只是使用了比较 `严格` 的过滤。

<a name="method-count"></a>
#### `count()` {#collection-method}

返回该集合内的项目总数：

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-diff"></a>
#### `diff()` {#collection-method}

将集合与其它集合或纯 PHP `数组` 进行值的比较，返回第一个集合中存在而第二个集合中不存在的值：

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

<a name="method-diffassoc"></a>
#### `diffAssoc()` {#collection-method}

该方法与另外一个集合或基于它的键和值的 PHP `数组`。此方法将返回原来集合不存在于集合中的 键/值 ：

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

将集合与其它集合或纯 PHP `数组` 的「键」进行比较，返回第一个集合中存在而第二个集合中不存在「键」所对应的键值对：

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

遍历集合中的项目，并将之传入回调函数：

    $collection = $collection->each(function ($item, $key) {
        //
    });

回调函数中返回 `false` 以中断循环：

    $collection = $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-every"></a>
#### `every()` {#collection-method}

判断集合中每一个元素是否都符合指定条件：

    collect([1, 2, 3, 4])->every(function ($value, $key) {
        return $value > 2;
    });
    
    // false

<a name="method-except"></a>
#### `except()` {#collection-method}

返回集合中除了指定键以外的所有项目：

    $collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

    $filtered = $collection->except(['price', 'discount']);

    $filtered->all();

    // ['product_id' => 1]

与 `except` 相反的方法请查看 [only](#method-only)。

<a name="method-filter"></a>
#### `filter()` {#collection-method}

使用回调函数筛选集合，只留下那些通过判断测试的项目：

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

与 `filter` 相反的方法可以查看 [reject](#method-reject)。

<a name="method-first"></a>
#### `first()` {#collection-method}

返回集合第一个通过指定测试的元素：

    collect([1, 2, 3, 4])->first(function ($value, $key) {
        return $value > 2;
    });
    
    // 3

你也可以不传入参数使用 `first` 方法以获取集合中第一个元素。如果集合是空的，则会返回 `null`：

    collect([1, 2, 3, 4])->first();

    // 1

<a name="method-flatmap"></a>
#### `flatMap()` {#collection-method}

对集合内所有子集遍历执行回调，并在最后转为一维集合：

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

将多维集合转为一维集合：

    $collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];

你可以选择性地传入遍历深度的参数：

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

在这个例子里，调用 `flatten` 方法时不传入深度参数会遍历嵌套数组降维成一维数组，生成 `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`，传入深度参数能让你限制降维嵌套数组的层数。

<a name="method-flip"></a>
#### `flip()` {#collection-method}

将集合中的键和对应的数值进行互换：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` {#collection-method}

通过集合的键来移除掉集合中的一个项目：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $collection->forget('name');

    $collection->all();

    // ['framework' => 'laravel']

> {note} 与大多数其它集合的方法不同，`forget` 不会返回修改过后的新集合；它会直接修改调用它的集合。

<a name="method-forpage"></a>
#### `forPage()` {#collection-method}

返回可用来在指定页码上所显示项目的新集合。这个方法第一个参数是页码数，第二个参数是每页显示的个数。

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunk = $collection->forPage(2, 3);

    $chunk->all();

    // [4, 5, 6]

<a name="method-get"></a>
#### `get()` {#collection-method}

返回指定键的项目。如果该键不存在，则返回 `null`：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('name');

    // taylor

你可以选择性地传入一个默认值作为第二个参数：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('foo', 'default-value');

    // default-value

你甚至可以传入回调函数当默认值。如果指定的键不存在，就会返回回调函数的运行结果：

    $collection->get('email', function () {
        return 'default-value';
    });
    
    // default-value

<a name="method-groupby"></a>
#### `groupBy()` {#collection-method}

根据指定的「键」为集合内的项目分组：

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

除了传入字符串的「键」之外，你也可以传入回调函数。该函数应该返回你希望用来分组的键的值。

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

检查集合中是否含有指定的「键」：

    $collection = collect(['account_id' => 1, 'product' => 'Desk']);

    $collection->has('product');

    // true

<a name="method-implode"></a>
#### `implode()` {#collection-method}

`implode` 方法合并集合中的项目。它的参数依集合中的项目类型而定。假如集合含有数组或对象，你应该传入你希望连接的属性的「键」，以及你希望放在数值之间的拼接字符串：:

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);
    
    $collection->implode('product', ', ');
    
    // Desk, Chair


假如集合只含有简单的字符串或数字，则只需要传入拼接的字符串作为该方法的唯一参数即可：

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()` {#collection-method}

移除任何指定 `数组` 或集合内所没有的数值。最终集合保存着原集合的键：

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

<a name="method-intersectkey"></a>
#### `intersectKey()` {#collection-method}

移除任何指定 `数组` 或集合内所没有的键值。

    $collection = collect([
        'serial' => 'UX301', 'type' => 'screen', 'year' => 2009
    ]);
    
    $intersect = $collection->intersectKey([
        'reference' => 'UX404', 'type' => 'tab', 'year' => 2011
    ]);
    
    $intersect->all();
    
    // ['type' => 'screen', 'year' => 2009]

<a name="method-isempty"></a>
#### `isEmpty()` {#collection-method}

如果集合是空的，`isEmpty` 方法会返回 `true`：否则返回 `false`：

    collect([])->isEmpty();

    // true

<a name="method-isnotempty"></a>
#### `isNotEmpty()` {#collection-method}

如果集合不是空的，`isNotEmpty` 方法会返回 `true`：否则返回 `false`：

    collect([])->isNotEmpty();

    // false

<a name="method-keyby"></a>
#### `keyBy()` {#collection-method}

以指定键的值作为集合项目的键。如果几个数据项有相同的键，那在新集合中只显示最后一项：

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

你也可以传入自己的回调函数，该函数应该返回集合的键的值：

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

返回该集合所有的键：

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);
    
    $keys = $collection->keys();
    
    $keys->all();
    
    // ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` {#collection-method}

返回集合中，最后一个通过指定测试的元素：

    collect([1, 2, 3, 4])->last(function ($value, $key) {
        return $value < 3;
    });
    
    // 2


你也可以不传入参数使用 `last` 方法以获取集合中最后一个元素。如果集合是空的，则会返回 `null`：

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-map"></a>
#### `map()` {#collection-method}

遍历整个集合并将每一个数值传入回调函数。回调函数可以任意修改并返回项目，形成修改过的项目组成的新集合：

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });
    
    $multiplied->all();
    
    // [2, 4, 6, 8, 10]

> {note} 正如集合大多数其它的方法一样，`map` 返回一个新集合实例；它并没有修改被调用的集合。假如你想改变原始的集合，得使用 [`transform`](#method-transform) 方法。


<a name="method-mapwithkeys"></a>
#### `mapWithKeys()` {#collection-method}

遍历整个集合并将每一个数值传入回调函数。回调函数返回包含一个键值对的关联数组：

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

计算指定键的最大值：

    $max = collect([['foo' => 10], ['foo' => 20]])->max('foo');

    // 20

    $max = collect([1, 2, 3, 4, 5])->max();

    // 5

<a name="method-median"></a>
#### `median()` {#collection-method}

计算指定键的中位数：

    $median = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->median('foo');

    // 15

    $median = collect([1, 1, 2, 4])->median();

    // 1.5

<a name="method-merge"></a>
#### `merge()` {#collection-method}

合并数组进集合。数组「键」对应的数值会覆盖集合「键」对应的数值：

    $collection = collect(['product_id' => 1, 'price' => 100]);

    $merged = $collection->merge(['price' => 200, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'price' => 200, 'discount' => false]

如果指定数组的「键」为数字，则「值」将会合并到集合的后面：

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-min"></a>
#### `min()` {#collection-method}

计算指定「键」的最小值：

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

    // 10

    $min = collect([1, 2, 3, 4, 5])->min();

    // 1

<a name="method-mode"></a>
#### `mode()` {#collection-method}

计算指定「键」的最常出现的值：

    $mode = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->mode('foo');

    // [10]

    $mode = collect([1, 1, 2, 4])->mode();

    // [1]

<a name="method-nth"></a>
#### `nth()` {#collection-method}

由每隔第 n 个元素组成一个新的集合：

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->nth(4);

    // ['a', 'e']


你也可以选择传入一个偏移量作为第二个参数

    $collection->nth(4, 1);

    // ['b', 'f']

<a name="method-only"></a>
#### `only()` {#collection-method}

返回集合中指定键的所有项目：

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->only(['product_id', 'name']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

与 `only` 相反的方法请查看 [except](#method-only)。

<a name="method-partition"></a>
#### `partition()` {#collection-method}

结合 PHP 中的 `list` 方法来分开符合指定条件的元素以及那些不符合指定条件的元素：

    $collection = collect([1, 2, 3, 4, 5, 6]);

    list($underThree, $aboveThree) = $collection->partition(function ($i) {
        return $i < 3;
    });

<a name="method-pipe"></a>
#### `pipe()` {#collection-method}

将集合传给回调函数并返回结果：

    $collection = collect([1, 2, 3]);

    $piped = $collection->pipe(function ($collection) {
        return $collection->sum();
    });
    
    // 6

<a name="method-pluck"></a>
#### `pluck()` {#collection-method}

获取集合中指定「键」所有对应的值：

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);
    
    $plucked = $collection->pluck('name');
    
    $plucked->all();
    
    // ['Desk', 'Chair']

你也可以指定最终集合的键：

    $plucked = $collection->pluck('name', 'product_id');

    $plucked->all();

    // ['prod-100' => 'Desk', 'prod-200' => 'Chair']

<a name="method-pop"></a>
#### `pop()` {#collection-method}

移除并返回集合最后一个项目：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop();

    // 5

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-prepend"></a>
#### `prepend()` {#collection-method}

在集合前面增加一项数组的值：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

你可以传递第二个参数来设置新增加项的键：

    $collection = collect(['one' => 1, 'two' => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two' => 2]

<a name="method-pull"></a>
#### `pull()` {#collection-method}

把「键」对应的值从集合中移除并返回：

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` {#collection-method}

在集合的后面新添加一个元素：

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` {#collection-method}

在集合内设置一个「键/值」：

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {#collection-method}

`random` 方法从集合中随机返回一个项目：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (retrieved randomly)

你可以选择性地传入一个整数到 `random`。如果该整数大于 `1`，则会返回一个集合：

    $random = $collection->random(3);

    $random->all();

    // [2, 4, 5] - (retrieved randomly)

<a name="method-reduce"></a>
#### `reduce()` {#collection-method}

`reduce` 方法将集合缩减到单个数值，该方法会将每次迭代的结果传入到下一次迭代：

    $collection = collect([1, 2, 3]);

    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });
    
    // 6

第一次迭代时 `$carry` 的数值为 `null`；然而你也可以传入第二个参数进 `reduce` 以指定它的初始值：

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);
    
    // 10

<a name="method-reject"></a>
#### `reject()` {#collection-method}

`reject` 方法以指定的回调函数筛选集合。会移除掉那些通过判断测试（即结果返回 `true`）的项目：

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($value, $key) {
        return $value > 2;
    });
    
    $filtered->all();
    
    // [1, 2]

与 `reject` 相反的方法可以查看 [`filter`](#method-filter) 方法。

<a name="method-reverse"></a>
#### `reverse()` {#collection-method}

`reverse` 方法倒转集合内项目的顺序：

    $collection = collect([1, 2, 3, 4, 5]);

    $reversed = $collection->reverse();

    $reversed->all();

    // [5, 4, 3, 2, 1]

<a name="method-search"></a>
#### `search()` {#collection-method}

`search` 方法在集合内搜索指定的数值并返回找到的键。假如找不到项目，则返回 `false`：

    $collection = collect([2, 4, 6, 8]);

    $collection->search(4);

    // 1

搜索是用「宽松」匹配来进行，也就是说如果字符串值是整数那它就跟这个整数是相等的。要使用严格匹配的话，就传入 `true` 为该方法的第二个参数：

    $collection->search('4', true);

    // false

另外，你可以传入你自己的回调函数来搜索第一个通过你判断测试的项目：

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

`shuffle` 方法随机排序集合的项目：

    $collection = collect([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    // [3, 2, 5, 1, 4] - (generated randomly)

<a name="method-slice"></a>
#### `slice()` {#collection-method}

`slice` 方法返回集合从指定索引开始的一部分切片：

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

如果你想限制返回切片的大小，就传入想要的大小为方法的第二个参数：

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

返回的切片将会保留原始键作为索引。假如你不希望保留原始的键，你可以使用 `values` 方法来重新建立索引。

<a name="method-sort"></a>
#### `sort()` {#collection-method}

对集合排序。排序后的集合保留着原始数组的键，所以在这个例子里我们用 [`values`](#method-values) 方法来把键设置为连续数字的键。

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

假如你需要更高级的排序，你可以传入回调函数以你自己的算法进行`排序`。参考 PHP 文档的 [`usort`](http://php.net/manual/en/function.usort.php#refsect1-function.usort-parameters)，这是集合的 `sort` 方法在背后所调用的函数。

> {tip} 要排序嵌套数组或对象的集合，见 [`sortBy`](#method-sortby) 和 [`sortByDesc`](#method-sortbydesc) 方法。


<a name="method-sortby"></a>
#### `sortBy()` {#collection-method}

以指定的键排序集合。排序后的集合保留了原始数组键，所以在这个例子中我们用 [`values`](#method-values) method 把键设置为连续数字的索引建：

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

你也可以传入自己的回调函数以决定如何排序集合数值：

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

与 [`sortBy`](#method-sortby) 有着一样的形式，但是会以相反的顺序来排序集合：

<a name="method-splice"></a>
#### `splice()` {#collection-method}

返回从指定的索引开始的一小切片项目，原本集合也会被切除：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

你可以传入第二个参数以限制大小：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

此外，你可以传入含有新项目的第三个参数以取代集合中被移除的项目：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]

<a name="method-split"></a>
#### `split()` {#collection-method}

将集合按指定组数分解：

    $collection = collect([1, 2, 3, 4, 5]);

    $groups = $collection->split(3);

    $groups->toArray();

    // [[1, 2], [3, 4], [5]]

<a name="method-sum"></a>
#### `sum()` {#collection-method}

返回集合内所有项目的总和：

    collect([1, 2, 3, 4, 5])->sum();

    // 15

如果集合包含嵌套数组或对象，你应该传入一个「键」来指定要用哪些数值来计算总和：

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);
    
    $collection->sum('pages');
    
    // 1272

此外，你可以传入自己的回调函数来决定要用哪些数值来计算总和：

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

返回有着指定数量项目的集合：

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

你也可以传入负整数以获取从集合后面来算指定数量的项目：

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-tap"></a>
#### `tap()` {#collection-method}

将集合传递给给定的回调，允许在特定点「tap」集合，并在不影响集合本身的情况下执行某些操作：

    collect([2, 4, 3, 1, 5])
        ->sort()
        ->tap(function ($collection) {
            Log::debug('Values after sorting', $collection->values()->toArray());
        })
        ->shift();
    
    // 1

<a name="method-times"></a>
#### `times()` {#collection-method}

静态 `times` 方法通过在一定时间内调用回调来创建一个新的集合：

    $collection = Collection::times(10, function ($number) {
        return $number * 9;
    });
    
    $collection->all();
    
    // [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]

当与工厂结合使用时，该方法可以有效地创建出优秀的模型[Eloquent](/docs/{{version}}/eloquent)：

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

将集合转换成纯 PHP `数组`。假如集合的数值是 [Eloquent](/docs/{{version}}/eloquent) 模型，也会被转换成数组：

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> {note} `toArray` 也会转换所有内嵌的对象为数组。假如你希望获取原本的底层数组，改用 [`all`](#method-all) 方法。

<a name="method-tojson"></a>
#### `toJson()` {#collection-method}

将集合转换成 JSON：

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk", "price":200}'

<a name="method-transform"></a>
#### `transform()` {#collection-method}

遍历集合并对集合内每一个项目调用指定的回调函数。集合的项目将会被回调函数返回的数值取代掉：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });
    
    $collection->all();
    
    // [2, 4, 6, 8, 10]

> {note} 与大多数其它集合的方法不同，`transform` 会修改集合本身。如果你希望创建新集合，就改用 [`map`](#method-map) 方法。

<a name="method-union"></a>
#### `union()` {#collection-method}

将给定的数组合并到集合中，如果数组中含有与集合一样的「键」，集合的键值会被保留：

    $collection = collect([1 => ['a'], 2 => ['b']]);

    $union = $collection->union([3 => ['c'], 1 => ['b']]);

    $union->all();

    // [1 => ['a'], 2 => ['b'], 3 => ['c']]

<a name="method-unique"></a>
#### `unique()` {#collection-method}

`unique` 方法返回集合中所有唯一的项目。返回的集合保留着原始键，所以在这个例子中我们用 [`values`](#method-values) 方法来把键重置为连续数字的键。

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

当处理嵌套数组或对象的时候，你可以指定用来决定唯一性的键：

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

你可以传入自己的回调函数来确定项目的唯一性：

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

此方法是用宽松的匹配，你可以使用 [`uniqueStrict`](#method-uniquestrict) 做比较 `严格` 的匹配。

<a name="method-uniquestrict"></a>
#### `uniqueStrict()` {#collection-method}

此方法的使用于 [`unique`](#method-unique) 方法类似，只是使用了比较 `严格` 的过滤。

<a name="method-values"></a>
#### `values()` {#collection-method}

返回「键」重新被设为「连续整数」的新集合：

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

当第一个参数运算结果为 `true` 的时候，会执行第二个参数传入的闭包：

    $collection = collect([1, 2, 3]);

    $collection->when(true, function ($collection) {
        return $collection->push(4);
    });
    
    $collection->all();
    
    // [1, 2, 3, 4]

<a name="method-where"></a>
#### `where()` {#collection-method}

以一对指定的「键／数值」筛选集合：

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

比较数值的时候用了「宽松」匹配方式，查看 [`whereStrict`](#method-wherestrict) method来用严格比较的方式过滤。

<a name="method-wherestrict"></a>
#### `whereStrict()` {#collection-method}

这个方法与 [`where`](#method-where) 方法有着一样的形式；但是会以「严格」匹配来匹配数值：

<a name="method-wherein"></a>
#### `whereIn()` {#collection-method}

基于参数中的键值数组进行过滤：

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

此方法是用宽松的匹配，你可以使用 [`whereInStrict`](#method-whereinstrict) 做比较 `严格` 的匹配。

<a name="method-whereinstrict"></a>
#### `whereInStrict()` {#collection-method}

此方法的使用于 [`whereIn`](#method-wherein) 方法类似，只是使用了比较 `严格` 的过滤。

<a name="method-wherenotin"></a>
#### `whereNotIn()` {#collection-method}

通过给定数组中不包含的给定键/值过滤集合：

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

此方法是用宽松的匹配，你可以使用 [`whereNotInStrict`](#method-wherenotinstrict) 做比较 `严格` 的匹配。

<a name="method-wherenotinstrict"></a>
#### `whereNotInStrict()` {#collection-method}

此方法的使用于 [`whereNotIn`](#method-wherenotin) 方法类似，只是使用了比较 `严格` 的过滤。

<a name="method-zip"></a>
#### `zip()` {#collection-method}

`zip` 方法将集合与指定数组相同索引的值合并在一起：

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]

<a name="higher-order-messages"></a>
## 高阶信息传递

集合也提供「高阶信息传递支持」，这是对集合执行常见操作的快捷方式。支持高阶信息传递的集合方法有：`average`， `avg`， `contains`， `each`， `every`， `filter`， `first`， `flatMap`， `map`， `partition`， `reject`， `sortBy`， `sortByDesc` 和 `sum`。

每个高阶信息都能作为集合实例的动态属性来访问。例如，我们在集合中使用 `each` 高阶信息传递方法拉哎对每个对象去调用一个方法：

    $users = User::where('votes', '>', 500)->get();

    $users->each->markAsVip();

同样，我们可以使用 `sum` 高阶信息传递的方式来统计出集合中用户总共的「投票数」：

    $users = User::where('group', 'Development')->get();

    return $users->sum->votes;
