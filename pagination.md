# 分页

- [简介](#introduction)
- [基本使用](#basic-usage)
    - [对查询语句构造器进行分页](#paginating-query-builder-results)
    - [对 Eloquent 模型进行分页](#paginating-eloquent-results)
    - [手动创建分页](#manually-creating-a-paginator)
- [显示分页结果](#displaying-pagination-results)
    - [转换至 JSON](#converting-results-to-json)
- [将分页结果显示在视图中](#customizing-the-pagination-view)
- [分页实例方法](#paginator-instance-methods)

<a name="introduction"></a>
## 简介

在其他框架中，分页是非常让人苦恼的， Laravel 为 [查询构造器](/docs/{{version}}/queries) 和 [Eloquent ORM](/docs/{{version}}/eloquent) 集成提供了方便，使用数据库分页更容易。生成的分页样式兼容 [Bootstrap CSS 框架](http://getbootstrap.com/)。

<a name="basic-usage"></a>
## 基本使用

<a name="paginating-query-builder-results"></a>
### 对查询语句构造器进行分页

有几种方法可以对项目进行分页。最简单的是在 [查询构造器](/docs/{{version}}/queries) 和 [Eloquent ORM](/docs/{{version}}/eloquent) 中使用 `paginate` 方法。 `paginate` 方法能够自动判定当前页面正确的数量限制和偏移数。默认情况下，当前页数由 HTTP 请求所带的  `?page` 参数来决定。当然，该值由 Laravel 自动检测，并自动插入由分页器生成的链接。

让我们先来看看如何在查询上调用 `paginate` 方法。传递给 `paginate` 的唯一参数就是你每页想要显示的数目，这个参数规定每页显示多少条数据。在下面这个例子中，我们就是要在每页显示 15 条数据：


    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         *  显示应用中的所有用户.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->paginate(15);

            return view('user.index', ['users' => $users]);
        }
    }

> {note} 目前，使用 `groupBy` 的分页操作不能被 Laravel 有效执行，如果你需要在分页结果中使用 `groupBy` ，推荐你手动查询数据库然后创建分页器。

#### 简单分页

如果你只需要在分页视图中简单的显示 “下一页” 和 “上一页” 链接，你可以选择使用 `simplePaginate` 方法来进行更高效的查找。当你不需要在页面上显示页码时，这对于大数据来说将会非常有用：

    $users = DB::table('users')->simplePaginate(15);

<a name="paginating-eloquent-results"></a>
### 基于 Eloquent 模型分页

你可能需要对 [Eloquent](/docs/{{version}}/eloquent) 分页。在这个例子中，我们将把 `User` 模型进行分页并且设置其每页有 `15` 条数据，如你所见，语法跟查询语句构造器的分页语法几乎一样：

    $users = App\User::paginate(15);

当然，你可以在设置一些约束后再使用 `paginate` 分页，我们使用 `where` 举例：

    $users = User::where('votes', '>', 100)->paginate(15);

或者也可以在使用 Eloquent 模型进行分页时使用 `simplePaginate` 方法：

    $users = User::where('votes', '>', 100)->simplePaginate(15);

<a name="manually-creating-a-paginator"></a>
### 手动创建分页

有时候你可能会希望从项目的数组中手动创建一个分页实例。这时可以依据你的需求决定创建 `Illuminate\Pagination\Paginator` 或者 `Illuminate\Pagination\LengthAwarePaginator` 。

 `Paginator` 类不需要知道数据的总条数；然而也正是因为这点，导致它无法提供获取最后一页的方法。`LengthAwarePaginator` 和 `Paginator` 参数几乎相同；但是它却需要知道数据的总条数。

换句话说，`Paginator` 对应于查询语句构造器和 Eloquent 的 `simplePaginate` 方法。而 `LengthAwarePaginator` 则等同于 `paginate` 方法。

> {note} 当手动创建一个分页器实例时，你应该手动「切割」传递给分页器的数组。如果你不知道如何做到这一点，请查阅 PHP 的 [array_slice](http://php.net/manual/en/function.array-slice.php) 函数。

<a name="displaying-pagination-results"></a>
## 显示分页结果

当你调用查询构建器或 Eloquent 查询上的 `simplePaginate` 或 `paginate`方法时,你将会获取一个分页实例，当调用 `paginate` 方法时，你将获取 `Illuminate\Pagination\LengthAwarePaginator` ，而调用方法 `simplePaginate` 方法时，将会获取 `Illuminate\Pagination\Paginator` 实例。这些对象提供相关方法描述这些结果集，除了这些帮助函数外，分页器实例本身就是迭代器，可以像数组一样对其进行循环调用。总之，一旦你获取到结果，就可以对结果进行显示，并使用  [Blade](/docs/{{version}}/blade) 渲染页面的链接：

    <div class="container">
        @foreach ($users as $user)
            {{ $user->name }}
        @endforeach
    </div>

    {{ $users->links() }}

`links` 方法将给予查找结果中其它页面的链接。每一个链接中都已经包含正确的 `?page` 查找字符串变量。请记住，由 `links` 方法生成的 HTML 兼容于 [Bootstrap CSS 框架](https://getbootstrap.com)。

#### 自定义分页链接

`setPath` 方法允许你生成分页链接时自定义分页使用的 URI 。例如，如果你想要分页器生成例如 `http://example.com/custom/url?page=N` 类似的链接，你可以使用 `setPath` 方法将 `custom/url` 加到分页中：

    Route::get('users', function () {
        $users = App\User::paginate(15);

        $users->setPath('custom/url');

        //
    });

#### 添加参数到分页链接

你可以使用 `appends` 方法添加所需要的参数到分页链接中。例如，要加入 `&sort=votes` 到每个分页链接时，你应该这样使用 `appends` 方法：

    {{ $users->appends(['sort' => 'votes'])->links() }}

如果你想加入一个有 “hash fragment” 的分页链接网址, 则可以使用  `fragment` 方法。例如，要在每个分页链接的最后加入 `#foo` , 应该像这样使用 `fragment` 方法：

    {{ $users->fragment('foo')->links() }}

<a name="converting-results-to-json"></a>
### 转换至 JSON

Laravel 的分页类实现了 `Illuminate\Contracts\Support\Jsonable` 的 `toJson` 方法，所以可以很容易的将你的分页结果转换成 JSON 。你可以将一个分页实例转换为 JSON，只需从一个路由或控制器中返回它即可：

    Route::get('users', function () {
        return App\User::paginate();
    });

分页器的 JSON 将包括分页相关的信息，如 `total`, `current_page`, `last_page` 等等。 该实例数据可通过 JSON 数组中的 `data` 键来获取。下方是从路由返回的分页实例转换成 JSON 的一个例子：

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
## 在视图中显示分页结果

默认情况下，渲染显示分页链接使用 Bootstrap CSS 框架。不过，如果你不使用 Bootstrap 你可以自由定义您自己的视图来呈现这些链接，当我们在分页实例中调用 `links` 方法，将视图名称传递给方法的第一个参数：

    {{ $paginator->links('view.name') }}

自定义分页视图的最简单的方法是通过 `vendor:publish` 命令创建到 `resources/views/vendor` 目录：

    php artisan vendor:publish --tag=laravel-pagination

这个名令将会在 `resources/views/vendor/pagination` 目录中创建视图。`default.blade.php` 文件为默认的分页模板，你可以编辑这个模板以修改分页的 HTML 样式。

<a name="paginator-instance-methods"></a>
## 分页实例方法

每个分页实例通过以下方法提供了额外的分页信息：

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
