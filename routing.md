# HTTP 路由

- [基本路由](#basic-routing)
- [路由参数](#route-parameters)
    - [必选路由参数](#required-parameters)
    - [可选路由参数](#parameters-optional-parameters)
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
- [获取当前路由信息](#accessing-the-current-route)

<a name="basic-routing"></a>
## 基本路由

最基本的路由只需要一个 URI 与一个 `闭包`，这里提供了一个非常简单优雅的定义路由的方法：

    Route::get('foo', function () {
        return 'Hello World';
    });

#### 默认路由文件

所有的 Laravel 路由都在 `routes` 目录中的路由文件中定义，这些文件都由框架自动加载。 `routes/web.php` 文件中定义你的 web 页面路由。这些路由都会应用 `web` 中间件组，其提供了诸如 `Session` 和 `CSRF` 保护等特性。定义在 `routes/api.php` 中的路由都是无状态的，并且会应用 `api` 中间件组。

大多数的应用，都是从 `routes/web.php` 文件开始定义路由。

#### 可用的路由方法

我们可以注册路由来响应所有的 HTTP 方法

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

有的时候你可能需要注册一个可响应多个 HTTP 方法的路由，这时可以使用 `match` 方法，也可以使用 `any` 方法注册一个实现响应所有 HTTP 的请求的路由：

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

#### CSRF 保护

任何指向 `web` 中 `POST`, `PUT` 或 `DELETE` 路由的 HTML 表单请求都应该包含一个 CSRF 令牌，否则，这个请求将会被拒绝。更多的关于 CSRF 的说明在 [CSRF documentation](/docs/{{version}}/csrf)：

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

<a name="route-parameters"></a>
## 路由参数

<a name="required-parameters"></a>
### 必选路由参数

当然，有时我们需要在路由中捕获一些 URL 片段。例如，我们需要从 URL 中捕获用户的 ID ，我们可以这样定义路由参数：

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

也可以在路由中定义多个参数：

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });


路由的参数通常都会被放在 `{}` 内，并且参数名只能为字母，当运行路由时，参数会通过路由闭包来传递。 

> **注意：** 路由参数不能包含 `-` 字符。请用下划线 (`_`) 替换。

<a name="parameters-optional-parameters"></a>
### 可选路由参数

当需要指定一个路由参数为可选时，可以在参数后面加上 `?` 来实现，但是相应的变量必须有默认值：

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="named-routes"></a>
## 命名路由

命名路由可以方便的生成 URLs 或者重定向，可以在定义路由后使用 `name` 方法实现：

    Route::get('user/profile', function () {
        //
    })->name('profile');

还可以为控制器动作指定路由名称:

    Route::get('user/profile', 'UserController@showProfile')->name('profile');

#### 为命名路由生成 URLs

为路由指定了名称后，我们可以使用全局辅助函数 `route` 来在生成 URLs 或者重定向到该条路由：  

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

如果是有定义参数的命名路由，可以把参数作为 `route` 函数的第二个参数传入，指定的参数将会自动插入到 URL 中对应的位置：

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

<a name="route-groups"></a>
## 路由组

路由组允许共享路由属性，例如中间件和命名空间等，我们没有必要为每个路由单独设置共有属性，共有属性会以数组的形式放到 `Route::group` 方法的第一个参数中。

<a name="route-group-middleware"></a>
### 中间件

要给路由组中给所有定义的路由分配中间件，可以在路由组中使用 `middleware` 键，中间件将会依照列表内指定的顺序运行：

    Route::group(['middleware' => 'auth'], function () {
        Route::get('/', function ()    {
            // Uses Auth Middleware
        });

        Route::get('user/profile', function () {
            // Uses Auth Middleware
        });
    });

<a name="route-group-namespaces"></a>
### 命名空间

另一个常见的例子是，指定相同的 PHP 命名空间给控制器组。可以使用 `namespace` 参数来指定组内所有控制器的公共命名空间：

    Route::group(['namespace' => 'Admin'], function() {
        // Controllers Within The "App\Http\Controllers\Admin" Namespace
    });

请记住，默认 `RouteServiceProvider` 会在命名空间组中引入你的路由文件，让你不用指定完整的 `App\Http\Controllers` 命名空间前缀就能注册控制器路由，因此，我们在定义的时候只需要指定命名空间 `App\Http\Controllers` 以后的部分。

<a name="route-group-sub-domain-routing"></a>
### 子域名路由

路由组也可以用作子域名的通配符，子域名可以像 URIs 一样当作路由组的参数，因此允许把捕获的子域名一部分用于我们的路由或控制器。路由组中子域名的属性可以使用路由组属性的 `domain` 键。

    Route::group(['domain' => '{account}.myapp.com'], function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### 路由前缀

通过路由组数组属性中的 `prefix` 键可以给每个路由组中的路由加上指定的 URI 前缀，例如，我们可以给路由组中所有的 URIs 加上路由前缀 `admin` :

    Route::group(['prefix' => 'admin'], function () {
        Route::get('users', function ()    {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-model-binding"></a>
## 路由模型绑定

当注入模型 ID 到路由控制器时，我们通常需要查询这个 ID 对应的模型，Laravel 路由模型绑定提供了一个方便的方法自动将模型注入到我们的路由中，例如，除了注入一个用户的 ID，你也可以注入与指定 ID 相符的完整 `User` 类实例。

<a name="implicit-binding"></a>
### 隐式绑定

Laravel 会自动解析定义在路由或控制器动作（变量名匹配路由片段）中的 Eloquent 模型类型声明，例如：

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

在这个例子中，由于类型声明了 Eloquent 模型 `App\User`，对应的变量名 `$user` 会匹配路由片段中的 `{user}`，这样，Laravel 会自动注入与请求 URI 中传入的 ID 对应的用户模型实例。

如果数据库中找不到对应的模型实例，将会自动生成产生一个 404 HTTP 响应。

#### 自定义键名

如果你想要隐式模型绑定除 `id` 以为的数据库字段，你可以重写 Eloquent 模型类的 `getRouteKeyName` 方法：

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="explicit-binding"></a>
### 显式绑定

要注册显式绑定，你需要在 RouteServiceProvider::boot 方法中使用路由的 model 方法来为给定参数指定绑定类：

#### 绑定参数至模型
    public function boot()
    {
        parent::boot();

        Route::model('user', 'App\User');
    }

接着，定义包含 `{user}` 参数的路由

    $router->get('profile/{user}', function(App\User $user) {
        //
    });

因为我们已经绑定 `{user}` 参数至 `App\User` 模型，所以 `User` 实例会被注入至该路由。所以，举个例子，一个至 `profile/1` 的请求会注入 ID 为 1 的 `User` 实例。

> **注意：**如果在数据库不存在对应 ID 的数据，就会自动抛出一个 404 异常。

#### 自定义解析逻辑

如果你想要使用自定义的解析逻辑，需要使用 `Route::bind` 方法，传递到 `bind` 方法的闭包会获取到 URI 请求参数中的值，并且返回你想要在该路由中注入的类实例：

    $router->bind('user', function ($value) {
        return App\User::where('name', $value)->first();
    });

<a name="form-method-spoofing"></a>
## 表单方法伪造

HTML 表单没有支持 `PUT`、`PATCH` 或 `DELETE` 动作。所以在从 HTML 表单中调用被定义的 `PUT`、`PATCH` 或 `DELETE` 路由时，你将需要在表单中增加隐藏的 `_method` 字段。 `_method` 字段送出的值将被作为 HTTP 的请求方法使用：

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

你也可以使用辅助函数 `methid_field` 来生成隐藏的输入字段 `_method`：

    {{ method_field('PUT') }}

<a name="accessing-the-current-route"></a>
## 获取当前路由信息

你可以使用 `Route` 上的 `current`, `currentRouteName`, and `currentRouteAction` 方法来访问处理当前输入请求的路由信息：

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

更多 API 请参考 [underlying class of the Route facade](http://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) 和 [Route instance](http://laravel.com/api/{{version}}/Illuminate/Routing/Route.html)
