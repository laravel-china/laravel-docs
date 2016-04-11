# 分页

- [简介](#introduction)
- [基本使用](#basic-usage)
    - [对查询语句构造器分页](#paginating-query-builder-results)
    - [对 Eloquent 模型分页](#paginating-eloquent-results)
    - [手动创建分页](#manually-creating-a-paginator)
- [将分页结果显示在视图中](#displaying-results-in-a-view)
- [转换至 JSON](#converting-results-to-json)

<a name="introduction"></a>
## 简介

在其它的框架中，分页是非常让人苦恼的。而在 Laravel 中是很轻而易举的。 Laravel 可以快速产生基于当前页面的智能「范围」，并且产生的 HTML 兼容于 [Bootstrap CSS 框架](http://getbootstrap.com/)。

<a name="basic-usage"></a>
## 基本使用

<a name="paginating-query-builder-results"></a>
### 对查询语句构造器分页

有几种方法对项目进行分页。最简单的是使用 `paginate` 方法在使用[查询语句构造器](/docs/{{version}}/queries)或是 [Eloquent 查找](/docs/{{version}}/eloquent)时。由 Laravel 提供的 `paginate` 方法自动判定当前页面正确的数量限制和偏移数。默认状况下，当前页数由 HTTP 请求所带的 `?page` 参数来决定。当然，该值由 Laravel 自动检测，并自动带入由分页器产生的链接。

首先，让我们来看看在数据库查找时使用 `paginate` 方法。在这个例子中，传递给 `paginate` 唯一的参数是你想在「每页」显示的数据数。在这个例子中，我们指定每页显示 `15` 笔数据：

    <?php

    namespace App\Http\Controllers;

    use DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show all of the users for the application.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->paginate(15);

            return view('user.index', ['users' => $users]);
        }
    }

> **注意：**目前， Laravel 的分页无法有效操作含有 `groupBy` 语句。如果你需要对使用 `groupBy` 的结果做分页，建议你查找数据库后再手动制作分页。

#### 「简易分页」

如果在你的视图只需要显示简单的「下一步」和「上一步」链接，你可以选择使用 `simplePaginate` 方法来进行更高效的查找。如果你不需要在页面上显示每个页码时，这对于大型数据非常有用：

    $users = DB::table('users')->simplePaginate(15);

<a name="paginating-eloquent-results"></a>
### 对 Eloquent 模型分页

你也可以对 Eloquent 进行分页。在这个例子中，我们将对 `User` 模型进行分页并且设置每页有 `15` 笔数据。正如你所看到的，语法与对查询语句构造器进行分页几乎是一样的：

    $users = App\User::paginate(15);

当然，你可以对 `paginate` 设置其它限制的查找，如 `where` 条件：

    $users = User::where('votes', '>', 100)->paginate(15);

你也可以在使用 Eloquent 模型进行分页时使用 `simplePaginate` 方法：

    $users = User::where('votes', '>', 100)->simplePaginate(15);

<a name="manually-creating-a-paginator"></a>
### 手动创建分页

有时候你可能会希望从项目的数组中手动创建一个分页实例。你可以依据你的需求决定创建 `Illuminate\Pagination\Paginator` 或是 `Illuminate\Pagination\LengthAwarePaginator` 。

`Paginator` 类不需要知道数据的总笔数；然而因为这点，它也无法提供获取最后一页的方法。`LengthAwarePaginator` 与 `Paginator` 的参数几乎相同；但是它需要数据的总笔数。

换句话说， `Paginator` 对应于查询语句构造器和 Eloquent 的 `simplePaginate` 方法，而 `LengthAwarePaginator` 相等于 `paginate` 方法。

当手动创建一个分页器实例时，你应该手动「切割」传递给分页器的数组。如果你不确定如何做到这一点，请查阅 PHP 的 [array_slice](http://php.net/manual/en/function.array-slice.php) 函数。

<a name="displaying-results-in-a-view"></a>
## 将分页结果显示在视图中

当在查询语句构造器或 Eloquent 中使用 `simplePaginate` 方法或使用 `paginate` 方法，你会得到一个分页器的实例。当使用 `paginate` 方法时，将得到 `Illuminate\Pagination\LengthAwarePaginator` 的实例。当使用 `simplePaginate` 方法时，会得到 `Illuminate\Pagination\Paginator` 的实例。这些对象提供几种方法用来描述结果集。除了这些辅助方法，分页器的实例也是个迭代器，并且可以像数组一样使用循环取值。

总之，一旦你已经获取结果，你可以显示结果，并使用 [Blade 模板](/docs/{{version}}/blade)渲染页面的链接：

    <div class="container">
        @foreach ($users as $user)
            {{ $user->name }}
        @endforeach
    </div>

    {!! $users->render() !!}

`render` 方法将给予查找结果中其它页面的链接。每一个链接中都已经包含正确的 `?page` 查找字符串变量。请记住，由 `render` 方法产生的 HTML 皆兼容于 [Bootstrap CSS 框架](https://getbootstrap.com)。

> **注意：**当在 Blade 模版中使用 `render` 方法时，一定要使用 `{！ ！}` 语法，HTML 链接才不会被转义。

#### 自定义分页器的 URI

`setPath` 方法允许你在产生链接时自定义 URI 。例如，如果你希望分页器产生像 `http://example.com/custom/url?page=N` ，你应该使用 `setPath` 方法将 `custom/url` 将加到分页中：

    Route::get('users', function () {
        $users = App\User::paginate(15);

        $users->setPath('custom/url');

        //
    });

#### 加入参数到分页链接中

你可以使用 `appends` 方法添加所需要的参数到分页链接中。例如，要加入 `&sort=votes` 到每个分页链接时，你应该这样使用 `appends` 方法：

    {!! $users->appends(['sort' => 'votes'])->render() !!}

如果你想加入一个有「哈希片段」的分页器链接网址，你可以使用 `fragment` 方法。例如，要在每个分页链接的最后加入 `#foo` ，应该这样使用 `fragment` 方法：

    {!! $users->fragment('foo')->render() !!}

#### 其它辅助方法

你也可以通过以下方法获得额外的分页信息：

- `$results->count()`
- `$results->currentPage()`
- `$results->hasMorePages()`
- `$results->lastPage() (在 simplePaginate 中无法使用)`
- `$results->nextPageUrl()`
- `$results->perPage()`
- `$results->previousPageUrl()`
- `$results->total()（在 simplePaginate 中无法使用）`
- `$results->url($page)`

<a name="converting-results-to-json"></a>
## 转换至 JSON

Laravel 的分页类实现了 `Illuminate\Contracts\Support\JsonableInterface` 的 `toJson` 方法，所以很容易将你的分页结果转换成 JSON 。

你可以将一个分页器实例转换为 JSON ，只需要简单地从一个路由或控制器中返回它：

    Route::get('users', function () {
        return App\User::paginate();
    });

分页器的 JSON 将包括分页相关的信息，如 `total` ， `current_page` ， `last_page` ，等等。该实例数据可通过 JSON 数组中的 `data` 键中获取。下方是从路由返回的分页器实例转换成 JSON 的一个例子：

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
