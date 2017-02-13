# Laravel 的分页功能

- [简介](#introduction)
- [基本使用](#basic-usage)
    - [对查询语句构造器进行分页](#paginating-query-builder-results)
    - [对 Eloquent 模型进行分页](#paginating-eloquent-results)
    - [手动创建分页](#manually-creating-a-paginator)
- [显示分页结果](#displaying-pagination-results)
    - [将结果转换为 JSON](#converting-results-to-json)
- [自定义分页视图](#customizing-the-pagination-view)
- [分页器实例方法](#paginator-instance-methods)

<a name="introduction"></a>
## 简介

在其他的框架中，分页是令人非常烦恼的。 Laravel 的分页器为 [查询构造器](/docs/{{version}}/queries) 和 [Eloquent ORM](/docs/{{version}}/eloquent) 集成提供了方便，并提供基于数据库结果集的分页开箱即用。分页器生产的 HTML 兼容 [Bootstrap CSS framework](https://getbootstrap.com/).

<a name="basic-usage"></a>
## 基本使用

<a name="paginating-query-builder-results"></a>
### 对查询语句构造器进行分页

有几种方法可以对项目进行分页。最简单的是在 [查询语句构造器](/docs/{{version}}/queries) 或 [Eloquent 查询](/docs/{{version}}/eloquent) 中使用 `paginate` 方法。 `paginate` 方法会自动基于当前用户查看的当前页面来设置适当的限制和偏移。默认情况下，当前页面通过 HTTP 请求所带的 `?page` 参数的值来检测。当然，这个值会被 Laravel 自动检测，并且自动插入到由分页器生产的链接中。

首先，让我们先来看看如何在查询上调用 `paginate` 方法。在这个例子中，传递给 `paginate` 方法的唯一参数是你希望在「每页」显示的数据数。在这种情况下，让我们指定希望每页显示 `15` 条数据：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 显示应用中所有的用户
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->paginate(15);

            return view('user.index', ['users' => $users]);
        }
    }

> {note} 目前，使用 `groupBy` 语句的分页操作无法由 Laravel 有效执行。如果你需要在一个分页结果集中使用 `groupBy`，建议你查询数据库并手动创建分页器。

#### 「简单分页」

如果你只需要在你的分页视图中显示简单的「上一页」和「下一页」的链接，你可以使用 `simplePaginate` 方法来执行更高效的查询。当你在渲染视图时不需要显示页码链接，这对于大数据集非常有用。

    $users = DB::table('users')->simplePaginate(15);

<a name="paginating-eloquent-results"></a>
### 对 Eloquent 模型进行分页

你也可以对 [Eloquent](/docs/{{version}}/eloquent) 查询进行分页。在这个例子中，我们将对 `User` 模型进行分页并且每页显示 `15` 条数据。正如你看到的，语法几乎与基于查询语句构造器的分页相同：

    $users = App\User::paginate(15);

当然，你可以在对查询设置了其他限制之后调用 `paginate` 方法，例如 `where` 子句：

    $users = User::where('votes', '>', 100)->paginate(15);

你也可以在 Eloquent 模型进行分页使用 `simplePaginate` 方法：

    $users = User::where('votes', '>', 100)->simplePaginate(15);

<a name="manually-creating-a-paginator"></a>
### 手动创建分页

有时候你可能希望手动创建一个分页实例，并传递其到项目数组中。你可以依据你的需求创建 `Illuminate\Pagination\Paginator` 或 `Illuminate\Pagination\LengthAwarePaginator` 实例。

`Paginator` 类不需要知道结果集中的数据项总数；然而，由于这个，该类没有用于检索最后一页索引的方法。`LengthAwarePaginator` 接收的参数几乎和 `Paginator` 一样；但是，它需要计算结果集中的数据项总数。

换一种说法，`Paginator` 对应于查询语句构造器和 Eloquent 的 `simplePaginate` 方法，而 `LengthAwarePaginator` 对应于 `paginate` 方法。

> {note} 当手动创建分页器实例时，你应该手动「切割」传递给分页器的结果集。如果你不确定如何去做到这一点，查阅 PHP 的函数 [array_slice](https://secure.php.net/manual/en/function.array-slice.php) 。

<a name="displaying-pagination-results"></a>
## 显示分页结果

当调用 `paginate` 方法的时候，你将会接收到一个 `Illuminate\Pagination\LengthAwarePaginator` 实例。而当你调用 `simplePaginate` 方法时，你将会接收到一个 `Illuminate\Pagination\Paginator` 实例。这些对象提供了一些描述结果集的方法。除了这些辅助方法，分页器是迭代器并且可以作为数组循环。因此，一旦检索到结果集，你可以使用 [Blade](/docs/{{version}}/blade) 模板显示结果集并渲染页面链接：

    <div class="container">
        @foreach ($users as $user)
            {{ $user->name }}
        @endforeach
    </div>

    {{ $users->links() }}

`links` 方法将会渲染结果集中的其他页链接。这些链接中每一个都已经包含了 `?page` 查询字符串变量。记住，`links` 方法生产的 HTML 兼容 [Bootstrap CSS framework](https://getbootstrap.com)

#### 自定义分页器的 URI

`withPath` 方法允许你在生成分页链接时自定义 URI 。例如，如果你想分页器生成的链接如 `http://example.com/custom/url?page=N`，你应该传递 `custom/url` 到 `withPath` 方法：

    Route::get('users', function () {
        $users = App\User::paginate(15);

        $users->withPath('custom/url');

        //
    });

#### 附加参数到分页链接中

你可以使用 `append` 方法附加查询参数到分页链接中。例如，要附加 `sort=votes` 到每个分页链接，你应该这样调用 `append` 方法：

    {{ $users->appends(['sort' => 'votes'])->links() }}

如果你希望附加「哈希片段」到分页器的链接中，你应该使用 `fragment` 方法。例如，要附加 `#foo` 到每个分页链接的末尾，应该这样调用 `fragment` 方法：

    {{ $users->fragment('foo')->links() }}

<a name="converting-results-to-json"></a>
### 将结果转换为 JSON

Laravel 分页器结果类实现了 `Illuminate\Contracts\Support\Jsonable` 接口契约并且提供 `toJson` 方法，所以它很容易将你的分页结果集转换为 Json。你也可以简单地通过从路由或者控制器动作返回分页实例并将其转换为 JSON ：

    Route::get('users', function () {
        return App\User::paginate();
    });

从分页器获取的 JSON 将包含元信息，如： `total`, `current_page`, `last_page` 等等。实际的结果对象将通过 JSON 数组中的 `data` 键来获取。 以下是一个从路由返回分页器实例创建的 JSON 示例：

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

<a name="customizing-the-pagination-view"></a>
## 自定义分页视图

在默认情况下，视图渲染显示的分页链接都兼容 Bootstrap CSS 框架。但是，如果你不适用 Bootstrap，你可以自定义你自己的视图去渲染这些链接。当在分页器实例中调用 `links` 方法，传递视图名称作为方法的第一参数：

    {{ $paginator->links('view.name') }}

    // 传递数据到视图中...
    {{ $paginator->links('view.name', ['foo' => 'bar']) }}

然而，自定义分页视图最简单的方法是通过 `vendor:publish` 命令将它们导出到你的 `resources/views/vendor` 目录：

    php artisan vendor:publish --tag=laravel-pagination

这个命令将视图放置在 `resources/views/vendor/pagination` 目录中。这个目录下的 `default.blade.php` 文件对应于默认分页视图。你可以简单地编辑这个文件以修改分页的 HTML 。

<a name="paginator-instance-methods"></a>
## 分页器实例方法

每个分页器实例通过以下方法提供额外的分页信息：

- `$results->count()`
- `$results->currentPage()`
- `$results->firstItem()`
- `$results->hasMorePages()`
- `$results->lastItem()`
- `$results->lastPage() (当使用 simplePagination 时无效)`
- `$results->nextPageUrl()`
- `$results->perPage()`
- `$results->previousPageUrl()`
- `$results->total() (当使用 simplePagination 时无效)`
- `$results->url($page)`

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@cjjian](https://laravel-china.org/users/3848)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/3848_1477641871.png?imageView2/1/w/100/h/100">  |  翻译  | Nothing is impossible. [@Jiajian Chan](https://github.com/jcc/) at Github  |
