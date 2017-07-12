# 分页

- [简介](#introduction)
- [基本使用](#basic-usage)
    - [对查询语句构造器进行分页](#paginating-query-builder-results)
    - [对 Eloquent 模型进行分页](#paginating-eloquent-results)
    - [手动创建分页](#manually-creating-a-paginator)
- [将分页结果显示在视图中](#displaying-results-in-a-view)
- [转换至 JSON](#converting-results-to-json)

<a name="introduction"></a>
## 简介

在其它的框架中，分页是非常让人苦恼的。而在 Laravel 中却是很轻而易举的。 Laravel 可以快速生成基于当前页面链接的智能「请求范围」，并且生成兼容 [Bootstrap CSS 框架](http://getbootstrap.com/) 的 HTML。

<a name="basic-usage"></a>
## 基本使用

<a name="paginating-query-builder-results"></a>
### 对查询语句构造器进行分页

有几种方法可以对项目进行分页。最简单的是使用 `paginate` 方法。在使用 [查询语句构造器](/docs/{{version}}/queries) 或是 [Eloquent 查找](/docs/{{version}}/eloquent) 时。由 Laravel 提供的 `paginate` 方法能够自动判定当前页面正确的数量限制和偏移数。默认状况下，当前页数由 HTTP 请求所带的 `?page` 参数来决定。当然，该值由 Laravel 自动检测，并自动插入由分页器生成的链接。

首先，让我们来看看如何在数据库查找时使用 `paginate` 方法。在这个例子中，传递给 `paginate` 唯一的参数是你想在「每页」显示的数据数。我们在此指定每页显示 `15` 条数据：

    <?php

    namespace App\Http\Controllers;

    use DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 显示应用的所有用户。
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->paginate(15);

            return view('user.index', ['users' => $users]);
        }
    }

> **注意：**目前， Laravel 的分页无法有效操作含有 `groupBy` 的语句。如果你需要对使用 `groupBy` 的结果做分页，建议你先进行数据库查找后再手动制作分页。

#### 「简易分页」

如果在你的视图中只需要显示简单的「下一步」和「上一步」链接，你可以选择使用 `simplePaginate` 方法来进行更高效的查找。当你不需要在页面上显示页码时，这对于大数据来说将会非常有用：

    $users = DB::table('users')->simplePaginate(15);

<a name="paginating-eloquent-results"></a>
### 对 Eloquent 模型进行分页

你也可以对 Eloquent 进行分页。在这个例子中，我们将对 `User` 模型进行分页并且设置其每页有 `15` 条数据。如你所见，语法跟查询语句构造器的分页语法几乎一样：

    $users = App\User::paginate(15);

当然，你也可以对 `paginate` 设置其它限制的查找，如 `where` 条件：

    $users = User::where('votes', '>', 100)->paginate(15);

或者也可以在使用 Eloquent 模型进行分页时使用 `simplePaginate` 方法：

    $users = User::where('votes', '>', 100)->simplePaginate(15);

<a name="manually-creating-a-paginator"></a>
### 手动创建分页

有时候你可能会希望从项目的数组中手动创建一个分页实例。这时可以依据你的需求决定创建 `Illuminate\Pagination\Paginator` 或是 `Illuminate\Pagination\LengthAwarePaginator`。

`Paginator` 类不需要知道数据的总条数；然而也正是因为这点，导致它无法提供获取最后一页的方法。`LengthAwarePaginator` 与 `Paginator` 的参数几乎相同；但是它却需要知道数据的总条数。

换句话说， `Paginator` 对应于查询语句构造器和 Eloquent 的 `simplePaginate` 方法，而 `LengthAwarePaginator` 则等同于 `paginate` 方法。

当手动创建一个分页器实例时，你应该手动「切割」传递给分页器的数组。如果你不知道如何做到这一点，请查阅 PHP 的 [array_slice](http://php.net/manual/en/function.array-slice.php) 函数。

<a name="displaying-results-in-a-view"></a>
## 将分页结果显示在视图中

当在查询语句构造器或 Eloquent 中使用 `simplePaginate` 方法或 `paginate` 方法时，你会得到一个分页器的实例。当使用 `paginate` 方法时，将得到 `Illuminate\Pagination\LengthAwarePaginator` 实例。当使用 `simplePaginate` 方法时，则会得到 `Illuminate\Pagination\Paginator` 实例。这些对象提供几种方法用来描述结果集。除了这些辅助函数之外，分页器的实例也是个迭代器，并且可以像数组一样来使用循环取值。

总之，一旦你获取到结果，就可以对结果进行显示，并使用 [Blade 模板](/docs/{{version}}/blade) 渲染页面的链接：

    <div class="container">
        @foreach ($users as $user)
            {{ $user->name }}
        @endforeach
    </div>

    {{ $users->links() }}

`links` 方法将给予查找结果中其它页面的链接。每一个链接中都已经包含正确的 `?page` 查找字符串变量。请记住，由 `links` 方法生成的 HTML 兼容于 [Bootstrap CSS 框架](https://getbootstrap.com)。

#### 自定义分页器的 URI

`setPath` 方法允许你在生成链接时自定义 URI 。例如，如果你希望分页器生成像 `http://example.com/custom/url?page=N` 这样的链接，则应该使用 `setPath` 方法将 `custom/url` 加到分页中：

    Route::get('users', function () {
        $users = App\User::paginate(15);

        $users->setPath('custom/url');

        //
    });

#### 加入参数到分页链接中

你可以使用 `appends` 方法添加所需要的参数到分页链接中。例如，要加入 `&sort=votes` 到每个分页链接时，你应该这样使用 `appends` 方法：

    {!! $users->appends(['sort' => 'votes'])->links() !!}

如果你想加入一个有「哈希片段」的分页器链接网址，则可以使用 `fragment` 方法。例如，要在每个分页链接的最后加入 `#foo` ，应该像这样使用 `fragment` 方法：

    {!! $users->fragment('foo')->links() !!}

#### 其它辅助函数

你也可以通过以下方法获得额外的分页信息：

- `$results->count()`
- `$results->currentPage()`
- `$results->firstItem()`
- `$results->hasMorePages()`
- `$results->lastItem()`
- `$results->lastPage() (Not available when using simplePaginate)`
- `$results->nextPageUrl()`
- `$results->perPage()`
- `$results->previousPageUrl()`
- `$results->total() (Not available when using simplePaginate)`
- `$results->url($page)`

<a name="converting-results-to-json"></a>
## 转换至 JSON

Laravel 的分页类实现了 `Illuminate\Contracts\Support\JsonableInterface` 的 `toJson` 方法，所以可以很容易的将你的分页结果转换成 JSON。

你可以将一个分页器实例转换为 JSON，只需从一个路由或控制器中返回它即可：

    Route::get('users', function () {
        return App\User::paginate();
    });

分页器的 JSON 将包括分页相关的信息，如 `total` ， `current_page` ， `last_page` ，等等。该实例数据可通过 JSON 数组中的 `data` 键来获取。下方是从路由返回的分页器实例转换成 JSON 的一个例子：

#### 分页结果转为 JSON 的例子

    {
       "total": 50,
       "per_page": 15,
       "current_page": 1,
       "last_page": 4,
       "next_page_url": "http://laravel.app?page=2",
       "prev_page_url": null,
       "from": 1,
       "to": 15,
       "data":[
            {
                // Result Object
            },
            {
                // Result Object
            }
       ]
    }





--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译。
> 
> 文档永久地址： http://d.laravel-china.org