# 集合

- [简介](#introduction)
- [创建集合](#creating-collections)
- [可用的方法](#available-methods)

<a name="introduction"></a>
## 简介

`Illuminate\Support\Collection` 类提供一个流畅、便利的封装来操控数组数据。举个例子，查看下列的代码。我们将用 `collect` 辅助方法从数组创建一个新的集合实例，对每一个元素运行 `strtoupper` 函数，然后移除所有的空元素：

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });


如你所见，`Collection` 类允许你链式调用它的方法以对底层的数组流畅地进行映射与删减。一般来说，每一个 `Collection` 方法会返回一个全新的 `Collection` 实例，你可以放心地进行链接调用。

<a name="creating-collections"></a>
## 创建集合

如上所述，`collect` 辅助方法会用传入的数组返回一个新的 `Illuminate\Support\Collection` 实例。所以要创建一个集合就这么简单：

    $collection = collect([1, 2, 3]);

默认 [Eloquent](/docs/{{version}}/eloquent) 模型的集合总是以 `Collection` 实例返回；然而，你可以随意的在你应用程序的任何适当场合使用 `Collection` 类。

<a name="available-methods"></a>
## 可用的方法

在这份文档剩余的部份，我们将会探讨每一个 `Collection` 类上可用的方法。要记得的是，所有方法都能被链式调用调用，几乎所有的方法都会返回新的 `Collection` 实例，让你保留原版的集合以备不时之需。

你可以从这张数据库表中选择任一方法看使用例子：

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
[avg](#method-avg)
[chunk](#method-chunk)
[collapse](#method-collapse)
[contains](#method-contains)
[count](#method-count)
[diff](#method-diff)
[each](#method-each)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[isEmpty](#method-isempty)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[map](#method-map)
[max](#method-max)
[merge](#method-merge)
[min](#method-min)
[only](#method-only)
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
[sum](#method-sum)
[take](#method-take)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[unique](#method-unique)
[values](#method-values)
[where](#method-where)
[whereLoose](#method-whereloose)
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

`all` 方法简单地返回该集合所代表的底层数组：

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-avg"></a>
#### `avg()` {#collection-method}

`avg` 方法返回集合中所有项目的平均值：

    collect([1, 2, 3, 4, 5])->avg();

    // 3

如果集合包含了嵌套数组或对象，你可以通过传递键来指定使用哪些值计算平均值：

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->avg('pages');

    // 636

<a name="method-chunk"></a>
#### `chunk()` {#collection-method}

`chunk` 方法将集合拆成多个指定大小的较小集合：

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->toArray();

    // [[1, 2, 3, 4], [5, 6, 7]]

这个方法在有用到网格系统如 [Bootstrap](http://getbootstrap.com/css/#grid) 的[视图](/docs/{{version}}/views)内特别有用。想像你有一个 [Eloquent](/docs/{{version}}/eloquent) 模型的集合要显示在一个网格内：

    @foreach ($products->chunk(3) as $chunk)
        <div class="row">
            @foreach ($chunk as $product)
                <div class="col-xs-4">{{ $product->name }}</div>
            @endforeach
        </div>
    @endforeach

<a name="method-collapse"></a>
#### `collapse()` {#collection-method}

`collapse` 方法将多个数组组成的集合合成单个数组集合：

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-contains"></a>
#### `contains()` {#collection-method}

`contains` 方法用来判断该集合是否含有指定的项目：

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

最后，你也可以传入一个回调函数到 `contains` 方法内运行你自己的判断式：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function ($key, $value) {
        return $value > 5;
    });

    // false

<a name="method-count"></a>
#### `count()` {#collection-method}

`count` 方法返回该集合内的项目总数：

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-diff"></a>
#### `diff()` {#collection-method}

`diff` 方法拿该集合与其它集合或纯 PHP `数组`进行比较：

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

<a name="method-each"></a>
#### `each()` {#collection-method}

`each` 方法遍历集合中的项目，并将之传入指定的回调函数：

    $collection = $collection->each(function ($item, $key) {
        //
    });

让你的回调函数返回 `false` 以中断循环：

    $collection = $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-every"></a>
#### `every()` {#collection-method}

`every` 方法会创建一个包含每 **第 n 个** 元素的新集合：

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->every(4);

    // ['a', 'e']

你可以选择性的传递偏移值作为第二个参数：

    $collection->every(4, 1);

    // ['b', 'f']

<a name="method-except"></a>
#### `except()` {#collection-method}

`except` 方法返回集合中排除指定键的所有项目：

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->except(['price', 'discount']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

与 `except` 相反的方法请查看 [only](#method-only)。

<a name="method-filter"></a>
#### `filter()` {#collection-method}

`filter` 方法以指定的回调函数筛选集合，只留下那些通过判断测试的项目：

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($item) {
        return $item > 2;
    });

    $filtered->all();

    // [3, 4]

与 `filter` 相反的方法可以查看 [reject](#method-reject)。

<a name="method-first"></a>
#### `first()` {#collection-method}

`first` 方法返回集合中，第一个通过指定测试的元素：

    collect([1, 2, 3, 4])->first(function ($key, $value) {
        return $value > 2;
    });

    // 3

你也可以不传入参数使用 `first` 方法以获取集合中第一个元素。如果集合是空的，则会返回 `null`：

    collect([1, 2, 3, 4])->first();

    // 1

<a name="method-flatten"></a>
#### `flatten()` {#collection-method}

`flatten` 方法将多维集合转为一维集合：

    $collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];

<a name="method-flip"></a>
#### `flip()` {#collection-method}

`flip` 方法将集合中的键和对应的数值进行互换：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` {#collection-method}

`forget` 方法以键自集合移除掉一个项目：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $collection->forget('name');

    $collection->all();

    // [framework' => 'laravel']

> **注意：** 与大多数其它集合的方法不同，`forget` 不会返回修改过后的新集合；它会直接修改调用它的集合。

<a name="method-forpage"></a>
#### `forPage()` {#collection-method}

`forPage` 方法返回含有可以用来在指定页码显示项目的新集合：

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunk = $collection->forPage(2, 3);

    $chunk->all();

    // [4, 5, 6]

这个方法需要页码和每个页面要显示的项目数目。

<a name="method-get"></a>
#### `get()` {#collection-method}

`get` 方法返回指定键的项目。如果该键不存在，则返回 `null`：

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

`groupBy` 方法根据指定的键替集合内的项目分组：

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

除了传入字符串的`键`之外，你也可以传入回调函数。该函数应该返回你希望用来分组的键的值。

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

`has` 方法用来确认集合中是否含有指定的键：

    $collection = collect(['account_id' => 1, 'product' => 'Desk']);

    $collection->has('email');

    // false

<a name="method-implode"></a>
#### `implode()` {#collection-method}

`implode` 方法连接集合中的项目。它的参数依集合中的项目类型而定。

假如集合含有数组或对象，你应该传入你希望连接的属性的键，以及你希望放在数值之间的「黏着」字符串：

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair

假如集合只含有简单的字符串或数字，就只要传入黏着的字符串作该方法唯一的参数：

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()` {#collection-method}

`intersect` 方法移除任何指定`数组`或集合内所没有的数值：

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

如你所见，最后出来的集合将会保留原始集合的键。

<a name="method-isempty"></a>
#### `isEmpty()` {#collection-method}

假如集合是空的，`isEmpty` 方法会返回 `true`：否则返回 `false`：

    collect([])->isEmpty();

    // true

<a name="method-keyby"></a>
#### `keyBy()` {#collection-method}

以指定键的值作为集合项目的键：

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

如果多个项目有同样的键，只有最后一个会出现在新的集合内。

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

`keys` 方法返回该集合所有的键：

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keys = $collection->keys();

    $keys->all();

    // ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` {#collection-method}

`last` 方法返回集合中，最后一个通过指定测试的元素：

    collect([1, 2, 3, 4])->last(function ($key, $value) {
        return $value < 3;
    });

    // 2

你也可以不传入参数使用 `last` 方法以获取集合中最后一个元素。如果集合是空的，则会返回 `null`：

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-map"></a>
#### `map()` {#collection-method}

`map` 方法遍历整个集合并将每一个数值传入指定的回调函数。回调函数可以任意修改并返回项目，于是形成修改过的项目组成的新集合：

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });

    $multiplied->all();

    // [2, 4, 6, 8, 10]

> **注意：** 正如集合大多数其它的方法一样，`map` 返回一个新集合实例；它并没有修改被调用的集合。假如你想改变原始的集合，得使用 [`transform`](#method-transform) 方法。

<a name="method-max"></a>
#### `max()` {#collection-method}

`max` 方法会传指定键的最大值：

    $max = collect([['foo' => 10], ['foo' => 20]])->max('foo');

    // 20

    $max = collect([1, 2, 3, 4, 5])->max();

    // 5

<a name="method-merge"></a>
#### `merge()` {#collection-method}

`merge` 方法将指定的数组合并进集合。数组字符串键与集合字符串键相同的将会覆盖掉集合的数值：

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $merged = $collection->merge(['price' => 100, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]

如果指定数组的键为数字，则数值将会附加在集合的后面：

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-min"></a>
#### `min()` {#collection-method}

`max` 方法会传指定键的最小值：

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

    // 10

    $min = collect([1, 2, 3, 4, 5])->min();

    // 1

<a name="method-only"></a>
#### `only()` {#collection-method}

`only` 方法返回集合中指定键的所有项目：

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->only(['product_id', 'name']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

与 `only` 相反的方法请查看 [except](#method-only)。

<a name="method-pluck"></a>
#### `pluck()` {#collection-method}

`pluck` 方法获取所有集合中指定键的值：

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $plucked = $collection->pluck('name');

    $plucked->all();

    // ['Desk', 'Chair']

你也可以指定要怎么给最后出来的集合分配键：

    $plucked = $collection->pluck('name', 'product_id');

    $plucked->all();

    // ['prod-100' => 'Desk', 'prod-200' => 'Chair']

<a name="method-pop"></a>
#### `pop()` {#collection-method}

`pop` 方法移除并返回集合最后一个项目：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop();

    // 5

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-prepend"></a>
#### `prepend()` {#collection-method}

`prepend` 方法在集合前面增加一个项目：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

你可以传递选择性的第二个参数来设置前置项目的键：

    $collection = collect(['one' => 1, 'two', => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two', => 2]

<a name="method-pull"></a>
#### `pull()` {#collection-method}

`pull` 方法以键从集合中移除并返回一个项目：

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` {#collection-method}

`push` 方法附加一个项目到集合后面：

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` {#collection-method}

`put` 在集合内设置一个指定键和数值：

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {#collection-method}

`random` 方法从集合中随机返回一个项目：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (随机返回)

你可以选择性地传入一个整数到 `random`。如果该整数大于 `1`，则会返回一个集合：

    $random = $collection->random(3);

    $random->all();

    // [2, 4, 5] - (随机返回)

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

`reject` 方法以指定的回调函数筛选集合。该回调函数应该对希望从最终集合移除掉的项目返回 `true`：

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($item) {
        return $item > 2;
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

搜索是用「宽松」比对来进行。要使用严格比对，就传入 `true` 为该方法的第二个参数：

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

    // [3, 2, 5, 1, 4] // (generated randomly)

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

返回的切片将会有以数字索引的新键。假如你希望保留原始的键，传入 `true` 为方法的第三个参数。

<a name="method-sort"></a>
#### `sort()` {#collection-method}

`sort` 方法对集合排序：

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

排序过的集合保有原来的数组键。在这个例子中我们用了 [`values`](#method-values) 方法重设键为连续的数字索引。

要排序内含数组或对象的集合，见 [`sortBy`](#method-sortby) 和 [`sortByDesc`](#method-sortbydesc) 方法。

假如你需要更高级的排序，你可以传入回调函数以你自己的算法进行`排序`。参考 PHP 文档的 [`usort`](http://php.net/manual/en/function.usort.php#refsect1-function.usort-parameters)，这是集合的 `sort` 方法在背后所调用的函数。

<a name="method-sortby"></a>
#### `sortBy()` {#collection-method}

`sortBy` 方法以指定的键排序集合：

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

排序过的集合保有原来的数组键。在这个例子中我们用了 [`values`](#method-values) 方法重设键为连续的数字索引。

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

这个方法与 [`sortBy`](#method-sortby) 有着一样的形式，但是会以相反的顺序来排序集合：

<a name="method-splice"></a>
#### `splice()` {#collection-method}

`splice` 方法移除并返回从指定的索引开始的一小切片项目：

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

<a name="method-sum"></a>
#### `sum()` {#collection-method}

`sum` 方法返回集合内所有项目的总和：

    collect([1, 2, 3, 4, 5])->sum();

    // 15

如果集合包含数组或对象，你应该传入一个键来确认要用哪些数值来计算总合：

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    // 1272

此外，你可以传入自己的回调函数来决定要用哪些数值来计算总合：

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

`take` 方法返回有着指定数量项目的集合：

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

你也可以传入负整数以获取从集合后面来算指定数量的项目：

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-toarray"></a>
#### `toArray()` {#collection-method}

`toArray` 方法将集合转换成纯 PHP `数组`。假如集合的数值是 [Eloquent](/docs/{{version}}/eloquent) 模型，也会被转换成数组：

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> **注意：** `toArray` 也会转换所有内嵌的对象为数组。假如你希望获取原本的底层数组，改用 [`all`](#method-all) 方法。

<a name="method-tojson"></a>
#### `toJson()` {#collection-method}

`toJson` 方法将集合转换成 JSON：

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk","price":200}'

<a name="method-transform"></a>
#### `transform()` {#collection-method}

`transform` 方法遍历集合并对集合内每一个项目调用指定的回调函数。集合的项目将会被回调函数返回的数值取代掉：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });

    $collection->all();

    // [2, 4, 6, 8, 10]

> **注意：** 与大多数其它集合的方法不同，`transform` 会修改集合本身。如果你希望创建新集合，就改用 [`map`](#method-map) 方法。

<a name="method-unique"></a>
#### `unique()` {#collection-method}

`unique` 方法返回集合中所有唯一的项目：

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

排序过的集合保有原来的数组键。在这个例子中我们用了 [`values`](#method-values) 方法重设键为连续的数字索引。

当处理内嵌的数组或对象，你可以指定用来决定唯一性的键：

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

你可以传入自己的回调函数来决定项目的唯一性：

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

<a name="method-values"></a>
#### `values()` {#collection-method}

`values` 方法返回键重设为连续整数的的新集合：

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
<a name="method-where"></a>
#### `where()` {#collection-method}

`where` 方法以一对指定的键／数值筛选集合：

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

`where` 方法以严格比对检查数值。使用 [`whereLoose`](#where-loose) 方法以宽松比对进行筛选。

<a name="method-whereloose"></a>
#### `whereLoose()` {#collection-method}

这个方法与 [`where`](#method-where) 方法有着一样的形式；但是会以「宽松」比对来比对数值：

<a name="method-zip"></a>
#### `zip()` {#collection-method}

`zip` 方法将集合与指定数组同样索引的值合并在一起：

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]
