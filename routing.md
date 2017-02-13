# Laravel HTTP 路由功能

- [基本路由](#basic-routing)
- [路由参数](#route-parameters)
    - [必选路由参数](#required-parameters)
    - [可选路由参数](#parameters-optional-parameters)
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
- [获取当前路由信息](#accessing-the-current-route)

<a name="basic-routing"></a>
## 基本路由

构建最基本的路由只需要一个 URI 与一个 `闭包`，这里提供了一个非常简单优雅的定义路由的方法：

    Route::get('foo', function () {
        return 'Hello World';
    });

#### 默认路由文件

所有的 Laravel 路由都在 `routes` 目录中的路由文件中定义，这些文件都由框架自动加载。在 `routes/web.php` 文件中定义你的 web 页面路由。这些路由都会应用 `web` 中间件组，其提供了诸如 `Session` 和 `CSRF` 保护等特性。定义在 `routes/api.php` 中的路由都是无状态的，并且会应用 `api` 中间件组。

大多数的应用构建，都是以在 `routes/web.php` 文件定义路由开始的。

#### 可用的路由方法

我们可以注册路由来响应所有的 HTTP 操作：

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

有的时候你可能需要注册一个可响应多个 HTTP 方法的路由，这时你可以使用 `match` 方法，也可以使用 `any` 方法注册一个实现响应所有 HTTP 的请求的路由：

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

#### CSRF 保护

任何指向 `web` 中 `POST`, `PUT` 或 `DELETE` 路由的 HTML 表单请求都应该包含一个 CSRF 令牌，否则，这个请求将会被拒绝。更多的关于 CSRF 的说明在 [CSRF 说明文档](/docs/{{version}}/csrf)：

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

也可以根据需要在路由中定义多个参数：

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

路由的参数通常都会被放在 `{}` 内，并且参数名只能为字母，当运行路由时，参数会通过路由闭包来传递。

> **注意：** 路由参数不能包含 `-` 字符。请用下划线 (`_`) 替换。

<a name="parameters-optional-parameters"></a>
### 可选路由参数

声明路由参数时，如需指定该参数为可选，可以在参数后面加上 `?` 来实现，但是相应的变量必须有默认值：

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### 正则表达式约束

你可以使用 `where` 方法来规范你的路由参数格式。`where` 方法接受参数名称和定义参数约束规则的正则表达式：

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

如果你希望路由参数在全局范围内都遵循一个确定的正则表达式约束，则可以使用 `pattern` 方法。你应该在 `RouteServiceProvider` 的 `boot` 方法里定义这些模式：

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

Pattern 一旦被定义，便会自动应用到所有使用该参数名称的路由上：

    Route::get('user/{id}', function ($id) {
        // 仅在 {id} 为数字时执行...
    });

<a name="named-routes"></a>
## 命名路由

命名路由可以方便的生成 `URL` 或者重定向，你可以在定义路由后使用 `name` 方法实现：

    Route::get('user/profile', function () {
        //
    })->name('profile');

你还可以为控制器方法指定路由名称：

    Route::get('user/profile', 'UserController@showProfile')->name('profile');

#### 为命名路由生成 `URL`

为路由指定了名称后，我们可以使用全局辅助函数 `route` 来生成 `URL` 或者重定向到该条路由：

    // 生成 URL...
    $url = route('profile');

    // 生成重定向...
    return redirect()->route('profile');

如果是有定义参数的命名路由，可以把参数作为 `route` 函数的第二个参数传入，指定的参数将会自动插入到 `URL` 中对应的位置：

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

<a name="route-groups"></a>
## 路由组

路由组允许共享路由属性，例如中间件和命名空间等，我们没有必要为每个路由单独设置共有属性，共有属性会以数组的形式放到 `Route::group` 方法的第一个参数中。

<a name="route-group-middleware"></a>
### 中间件

要给路由组中定义的所有路由分配中间件，可以在路由组中使用 `middleware` 键，中间件将会依照列表内指定的顺序运行：

    Route::group(['middleware' => 'auth'], function () {
        Route::get('/', function ()    {
            // 使用 `Auth` 中间件
        });

        Route::get('user/profile', function () {
            // 使用 `Auth` 中间件
        });
    });

<a name="route-group-namespaces"></a>
### 命名空间

Another common use-case for route groups is assigning the same PHP namespace to a group of controllers using the `namespace` parameter in the group array:

另一个常见的案例是，指定相同的 `PHP` 命名空间给控制器组。可以使用 `namespace` 参数来指定组内所有控制器的公共命名空间：

    Route::group(['namespace' => 'Admin'], function () {
        // Controllers Within The "App\Http\Controllers\Admin" Namespace
    });

Remember, by default, the `RouteServiceProvider` includes your route files within a namespace group, allowing you to register controller routes without specifying the full `App\Http\Controllers` namespace prefix. So, you only need to specify the portion of the namespace that comes after the base `App\Http\Controllers` namespace.

<a name="route-group-sub-domain-routing"></a>
### Sub-Domain Routing

Route groups may also be used to handle sub-domain routing. Sub-domains may be assigned route parameters just like route URIs, allowing you to capture a portion of the sub-domain for usage in your route or controller. The sub-domain may be specified using the `domain` key on the group attribute array:

    Route::group(['domain' => '{account}.myapp.com'], function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### Route Prefixes

The `prefix` group attribute may be used to prefix each route in the group with a given URI. For example, you may want to prefix all route URIs within the group with `admin`:

    Route::group(['prefix' => 'admin'], function () {
        Route::get('users', function ()    {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-model-binding"></a>
## Route Model Binding

When injecting a model ID to a route or controller action, you will often query to retrieve the model that corresponds to that ID. Laravel route model binding provides a convenient way to automatically inject the model instances directly into your routes. For example, instead of injecting a user's ID, you can inject the entire `User` model instance that matches the given ID.

<a name="implicit-binding"></a>
### Implicit Binding

Laravel automatically resolves Eloquent models defined in routes or controller actions whose type-hinted variable names match a route segment name. For example:

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

Since the `$user` variable is type-hinted as the `App\User` Eloquent model and the variable name matches the `{user}` URI segment, Laravel will automatically inject the model instance that has an ID matching the corresponding value from the request URI. If a matching model instance is not found in the database, a 404 HTTP response will automatically be generated.

#### Customizing The Key Name

If you would like model binding to use a database column other than `id` when retrieving a given model class, you may override the `getRouteKeyName` method on the Eloquent model:

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
### Explicit Binding

To register an explicit binding, use the router's `model` method to specify the class for a given parameter. You should define your explicit model bindings in the `boot` method of the `RouteServiceProvider` class:

    public function boot()
    {
        parent::boot();

        Route::model('user', App\User::class);
    }

Next, define a route that contains a `{user}` parameter:

    Route::get('profile/{user}', function (App\User $user) {
        //
    });

Since we have bound all `{user}` parameters to the `App\User` model, a `User` instance will be injected into the route. So, for example, a request to `profile/1` will inject the `User` instance from the database which has an ID of `1`.

If a matching model instance is not found in the database, a 404 HTTP response will be automatically generated.

#### Customizing The Resolution Logic

If you wish to use your own resolution logic, you may use the `Route::bind` method. The `Closure` you pass to the `bind` method will receive the value of the URI segment and should return the instance of the class that should be injected into the route:

    public function boot()
    {
        parent::boot();

        Route::bind('user', function ($value) {
            return App\User::where('name', $value)->first();
        });
    }

<a name="form-method-spoofing"></a>
## Form Method Spoofing

HTML forms do not support `PUT`, `PATCH` or `DELETE` actions. So, when defining `PUT`, `PATCH` or `DELETE` routes that are called from an HTML form, you will need to add a hidden `_method` field to the form. The value sent with the `_method` field will be used as the HTTP request method:

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

You may use the `method_field` helper to generate the `_method` input:

    {{ method_field('PUT') }}

<a name="accessing-the-current-route"></a>
## Accessing The Current Route

You may use the `current`, `currentRouteName`, and `currentRouteAction` methods on the `Route` facade to access information about the route handling the incoming request:

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

Refer to the API documentation for both the [underlying class of the Route facade](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) and [Route instance](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) to review all accessible methods.
