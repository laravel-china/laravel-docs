# Laravel HTTP 路由功能

- [基本路由](#basic-routing)
    - [重定向路由](#redirect-routes)
    - [视图路由](#view-routes)
- [路由参数](#route-parameters)
    - [必填参数](#required-parameters)
    - [可选参数](#parameters-optional-parameters)
    - [正则表达式约束](#parameters-regular-expression-constraints)
- [命名路由](#named-routes)
- [路由组](#route-groups)
    - [中间件](#route-group-middleware)
    - [命名空间](#route-group-namespaces)
    - [子域名路由](#route-group-sub-domain-routing)
    - [路由前缀](#route-group-prefixes)
- [路由模型绑定](#route-model-binding)
    - [隐式绑定](#implicit-binding)
    - [显式绑定](#explicit-binding)
- [表单方法伪造](#form-method-spoofing)
- [访问当前路由](#accessing-the-current-route)

<a name="basic-routing"></a>
## 基本路由

构建最基本的路由只需要一个 URI 与一个 `闭包`，这里提供了一个非常简单优雅的定义路由的方法：

    Route::get('foo', function () {
        return 'Hello World';
    });

#### 默认路由文件

所有的 Laravel 路由都在 `routes` 目录中的路由文件中定义，这些文件都由框架自动加载。`routes/web.php` 文件用于定义 web 界面的路由。这里面的路由都会被分配给 `web` 中间件组，它提供了会话状态和 CSRF 保护等功能。定义在 `routes/api.php` 中的路由都是无状态的，并且被分配了 `api` 中间件组。

大多数的应用构建，都是以在 `routes/web.php` 文件定义路由开始的。可以通过在浏览器中输入定义的路由 URL 来访问 `routes/web.php` 中定义的路由。例如，你可以在浏览器中输入 `http://your-app.dev/user` 来访问以下路由：

    Route::get('/user', 'UsersController@index');

`routes/api.php` 文件中定义的路由通过 `RouteServiceProvider` 被嵌套到一个路由组里面。在这个路由组中，会自动添加 URL 前缀 `/api` 到此文件中的每个路由，这样你就无需再手动添加了。你可以在 `RouteServiceProvider` 类中修改此前缀以及其他路由组选项。

#### 可用的路由方法

路由器允许你注册能响应任何 HTTP 请求的路由：

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

有的时候你可能需要注册一个可响应多个 HTTP 请求的路由，这时你可以使用 `match` 方法，也可以使用 `any` 方法注册一个实现响应所有 HTTP 请求的路由：

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

#### CSRF 保护

指向 `web` 路由文件中定义的 `POST`、`PUT` 或 `DELETE` 路由的任何 HTML 表单都应该包含一个 CSRF 令牌字段，否则，这个请求将会被拒绝。可以在 [CSRF 文档](/docs/{{version}}/csrf) 中阅读有关 CSRF 保护的更多信息：

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

<a name="redirect-routes"></a>
### 重定向路由

如果要定义重定向到另一个 URI 的路由，可以使用 `Route::redirect` 方法。这个方法可以快速地实现重定向，而不再需要去定义完整的路由或者控制器。

    Route::redirect('/here', '/there', 301);

<a name="view-routes"></a>
### 视图路由

如果你的路由只需要返回一个视图，可以使用 `Route::view` 方法。它和 `redirect` 一样方便，不需要定义完整的路由或控制器。`view` 方法有三个参数，其中前两个是必填参数，分别是 URL 和视图名称。第三个参数选填，可以传入一个数组，数组中的数据会被传递给视图。

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

<a name="route-parameters"></a>
## 路由参数

<a name="required-parameters"></a>
### 必填参数

当然，有时需要在路由中捕获一些 URL 片段。例如，从 URL 中捕获用户的 ID，可以通过定义路由参数来执行此操作：

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

也可以根据需要在路由中定义多个参数：

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

路由的参数通常都会被放在 `{}` 内，并且参数名只能为字母，同时路由参数不能包含 `-` 符号，如果需要可以用下划线 (`_`) 代替。路由参数会按顺序依次被注入到路由回调或者控制器中，而不受回调或者控制器的参数名称的影响。

<a name="parameters-optional-parameters"></a>
### 可选参数

有时，你可能需要指定一个路由参数，但你希望这个参数是可选的。你可以在参数后面加上 `?` 标记来实现，但前提是要确保路由的相应变量有默认值：

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### 正则表达式约束

你可以使用路由实例上的 `where` 方法约束路由参数的格式。`where` 方法接受参数名称和定义参数应如何约束的正则表达式：

    Route::get('user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

<a name="parameters-global-constraints"></a>
#### 全局约束

如果你希望某个具体的路由参数都遵循同一个正则表达式的约束，就使用 `pattern` 方法在 `RouteServiceProvider` 的 `boot` 方法中定义这些模式：

    /**
     * 定义你的路由模型绑定, pattern 过滤器等。
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');

        parent::boot();
    }

定义好之后，便会自动应用到所有使用该参数名称的路由上：

    Route::get('user/{id}', function ($id) {
        // 仅在 {id} 为数字时执行...
    });

<a name="named-routes"></a>
## 命名路由

命名路由可以方便地为指定路由生成 `URL` 或者重定向。通过在路由定义上链式调用 `name` 方法指定路由名称：

    Route::get('user/profile', function () {
        //
    })->name('profile');

你还可以指定控制器行为的路由名称：

    Route::get('user/profile', 'UserController@showProfile')->name('profile');

#### 为命名路由生成链接

为路由指定了名称后，就可以使用全局辅助函数 `route` 来生成链接或者重定向到该路由：

    // 生成 URL...
    $url = route('profile');

    // 生成重定向...
    return redirect()->route('profile');

如果是有定义参数的命名路由，可以把参数作为 `route` 函数的第二个参数传入，指定的参数将会自动插入到 URL 中对应的位置：

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

#### 检查当前路由

如果你想判断当前请求是否指向了某个路由，你可以调用路由实例上的 `named` 方法。例如，你可以在路由中间件中检查当前路由名称：

    /**
     * 处理一次请求。
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->route()->named('profile')) {
            //
        }

        return $next($request);
    }

<a name="route-groups"></a>
## 路由组

路由组允许你在大量路由之间共享路由属性，例如中间件或命名空间，而不需要为每个路由单独定义这些属性。共享属性应该以数组的形式传入 `Route::group` 方法的第一个参数中。

<a name="route-group-middleware"></a>
### 中间件

要给路由组中所有的路由分配中间件，可以在 `group` 之前调用 `middleware` 方法.中间件会依照它们在数组中列出的顺序来运行：

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // 使用 first 和 second 中间件
        });

        Route::get('user/profile', function () {
            // 使用 first 和 second 中间件
        });
    });

<a name="route-group-namespaces"></a>
### 命名空间

另一个常见用例是使用 `namespace` 方法将相同的 PHP 命名空间分配给路由组的中所有的控制器：

    Route::namespace('Admin')->group(function () {
        // 在 "App\Http\Controllers\Admin" 命名空间下的控制器
    });

请记住，默认情况下，`RouteServiceProvider` 会在命名空间组中引入你的路由文件，让你不用指定完整的 `App\Http\Controllers` 命名空间前缀就能注册控制器路由。因此，你只需要指定命名空间 `App\Http\Controllers` 之后的部分。

<a name="route-group-sub-domain-routing"></a>
### 子域名路由

路由组也可以用来处理子域名。子域名可以像路由 URI 一样被分配路由参数，允许你获取一部分子域名作为参数给路由或控制器使用。可以在 `group` 之前调用 `domain` 方法来指定子域名：

    Route::domain('{account}.myapp.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### 路由前缀

可以用 `prefix` 方法为路由组中给定的 URL 增加前缀。例如，你可以为组中所有路由的 URI 加上 `admin` 前缀：

    Route::prefix('admin')->group(function () {
        Route::get('users', function () {
            // 匹配包含 "/admin/users" 的 URL
        });
    });

<a name="route-model-binding"></a>
## 路由模型绑定

当向路由或控制器行为注入模型 ID 时，就需要查询这个 ID 对应的模型。Laravel 为路由模型绑定提供了一个直接自动将模型实例注入到路由中的方法。例如，你可以注入与给定 ID 匹配的整个 `User` 模型实例，而不是注入用户的 ID。

<a name="implicit-binding"></a>
### 隐式绑定

Laravel 会自动解析定义在路由或控制器行为中与类型提示的变量名匹配的路由段名称的 Eloquent 模型。例如：

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

在这个例子中，由于 `$user` 变量被类型提示为 Eloquent 模型 `App\User`，变量名称又与 URI 中的 `{user}` 匹配，因此，Laravel 会自动注入与请求 URI 中传入的 ID 匹配的用户模型实例。如果在数据库中找不到对应的模型实例，将会自动生成 404 异常。

#### 自定义键名

如果你想要模型绑定在检索给定的模型类时使用除 `id` 之外的数据库字段，你可以在 Eloquent 模型上重写 `getRouteKeyName` 方法：

    /**
     * 为路由模型获取键名。
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="explicit-binding"></a>
### 显式绑定

要注册显式绑定，使用路由器的 `model` 方法来为给定参数指定类。在 `RouteServiceProvider` 类中的 `boot` 方法内定义这些显式模型绑定：

    public function boot()
    {
        parent::boot();

        Route::model('user', App\User::class);
    }

接着，定义一个包含 `{user}` 参数的路由：

    Route::get('profile/{user}', function (App\User $user) {
        //
    });

因为我们已经将所有 `{user}` 参数绑定至 `App\User` 模型，所以 `User` 实例将被注入该路由。例如，`profile/1` 的请求会注入数据库中 ID 为 1 的 `User` 实例。

如果在数据库不存在对应 ID 的数据，就会自动抛出一个 404 异常。

#### 自定义解析逻辑

如果你想要使用自定义的解析逻辑，就使用 `Route::bind` 方法。传递到 `bind` 方法的闭包会接受 URI 中大括号对应的值，并且返回你想要在该路由中注入的类的实例：

    public function boot()
    {
        parent::boot();

        Route::bind('user', function ($value) {
            return App\User::where('name', $value)->first();
        });
    }

<a name="form-method-spoofing"></a>
## 表单方法伪造

HTML 表单不支持 `PUT`、`PATCH` 或 `DELETE` 行为。所以当你要从 HTML 表单中调用定义了 `PUT`、`PATCH` 或 `DELETE` 路由时，你将需要在表单中增加隐藏的 `_method` 输入标签。使用 `_method` 字段的值作为 HTTP 的请求方法：

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

你也可以使用辅助函数 `method_field` 来生成隐藏的 `_method` 输入标签：

    {{ method_field('PUT') }}

<a name="accessing-the-current-route"></a>
## 访问当前路由

你可以使用 `Route` Facade 上的 `current`、`currentRouteName` 和 `currentRouteAction` 方法来访问处理传入请求的路由的信息：

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

想知道所有可访问的方法，可以查看 API 文档，了解 [Route facade](http://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) 和 [Route 实例](http://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) 的基础类。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@卡卡卡么](https://laravel-china.org/users/18466)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/18466_1503308973.jpeg?imageView2/1/w/100/h/100">  |  翻译  |  PHP & Python 开发工程师，[Github](https://github.com/jiannanjiang)，[segmentfault](https://segmentfault.com/u/maketea) 欢迎交流  |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  Review  | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
