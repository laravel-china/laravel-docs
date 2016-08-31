# HTTP 路由

- [基本路由](#basic-routing)
- [路由参数](#route-parameters)
    - [基础路由参数](#required-parameters)
    - [可选的路由参数](#parameters-optional-parameters)
    - [正则表达式限制参数](#parameters-regular-expression-constraints)
- [命名路由](#named-routes)
- [路由群组](#route-groups)
    - [中间件](#route-group-middleware)
    - [命名空间](#route-group-namespaces)
    - [子域名路由](#route-group-sub-domain-routing)
    - [路由前缀](#route-group-prefixes)
- [CSRF 保护](#csrf-protection)
    - [介绍](#csrf-introduction)
    - [异常 URIs](#csrf-excluding-uris)
    - [X-CSRF-Token](#csrf-x-csrf-token)
    - [X-XSRF-Token](#csrf-x-xsrf-token)
- [路由模型绑定](#route-model-binding)
- [跨站请求伪造](#form-method-spoofing)
- [获取当前路由信息](#accessing-the-current-route)

<a name="basic-routing"></a>
## 基本路由

你可以在 `app/Http/routes.php` 文件中定义应用程序的大多数路由，该文件将会被 `App\Providers\RouteServiceProvider` 类加载。最基本的 Laravel 路由仅接受 URI 和一个`闭包`：

    Route::get('foo', function () {
        return 'Hello World';
    });

#### 默认路由文件

默认情况下，`routes.php` 文件包含单个路由和一个路由群组，该路由群组包含的所有路由都使用了中间件组 `web`，而这个中间件组为路由提供了 Session 状态和 CSRF 保护功能。大部分情况下，我们会将所有路由定义在这个文件中。

#### 可供使用的路由方法

我们可以注册路由来响应任何方法的 HTTP 请求：

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

有时候你可能需要注册一个可响应多个 HTTP 动作的路由。这时可通过 `Route` [facade](/docs/{{version}}/facades) 的 `match` 方法来实现：

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

<a name="route-parameters"></a>
## 路由参数

<a name="required-parameters"></a>
### 必选参数

有时候你可能需要从 URI 中获取一些参数。例如，从 URL 获取用户的 ID。这时可通过自定义路由参数来获取：

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

你可以依照路由需要，定义任意数量的路由参数：

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

路由的参数都会被放在「大括号」内。当运行路由时，参数会通过路由`闭包`来传递。

> **注意：** 路由参数不能包含 `-` 字符。请用下划线 (`_`) 替换。

<a name="parameters-optional-parameters"></a>
### 可选的路由参数

有时候你需要指定可选的路由参数，可以在参数名称后面加上 `?` 来实现：

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### 正则表达式限制参数

你可以使用 `where` 方法来限制路由参数格式。`where` 方法接受参数的名称和定义参数应该如何被限制的正则表达式：

    Route::get('user/{name}', function ($name) {
        //
    })
    ->where('name', '[A-Za-z]+');

    Route::get('user/{id}', function ($id) {
        //
    })
    ->where('id', '[0-9]+');

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })
    ->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

<a name="parameters-global-constraints"></a>
#### 全局限制

如果你希望路由参数可以总是遵循正则表达式，则可以使用 `pattern` 方法。你应该在 `RouteServiceProvider` 的 `boot` 方法里定义这些模式：

    /**
     * 定义你的路由模型绑定，模式过滤器等。
     *
     * @param  \Illuminate\Routing\Router  $router
     * @return void
     */
    public function boot(Router $router)
    {
        $router->pattern('id', '[0-9]+');

        parent::boot($router);
    }

模式一旦被定义，便会自动应用到所有使用该参数名称的路由上：

    Route::get('user/{id}', function ($id) {
        // Only called if {id} is numeric.
    });

<a name="named-routes"></a>
## 命名路由

命名路由让你可以更方便的为特定路由生成 URL 或进行重定向。你可以使用 `as` 数组键指定名称到路由上：

    Route::get('user/profile', ['as' => 'profile', function () {
        //
    }]);

还可以指定路由名称到控制器动作：

    Route::get('user/profile', [
        'as' => 'profile', 'uses' => 'UserController@showProfile'
    ]);

除了可以在路由的数组定义中指定路由名称外，你也可以在路由定义后方链式调用 `name` 方法：

    Route::get('user/profile', 'UserController@showProfile')->name('profile');

#### 路由群组和命名路由

如果你使用了 [路由群组](#route-groups)，那么你可以在路由群组的属性数组中指定一个 `as` 关键字，这将允许你为路由群组中的所有路由设置相同的前缀名称：

    Route::group(['as' => 'admin::'], function () {
        Route::get('dashboard', ['as' => 'dashboard', function () {
            // 路由名称为「admin::dashboard」
        }]);
    });

#### 对命名路由生成 URLs

一旦你在指定的路由中分配了名称，则可通过 `route` 函数来使用路由名称生成 URLs 或重定位：

    $url = route('profile');

    $redirect = redirect()->route('profile');

如果路由定义了参数，那么你可以把参数作为第二个参数传递给 `route` 方法。指定的参数将自动加入到 URL 中：

    Route::get('user/{id}/profile', ['as' => 'profile', function ($id) {
        //
    }]);

    $url = route('profile', ['id' => 1]);

<a name="route-groups"></a>
## 路由群组

路由群组允许你共用路由属性，例如：中间件、命名空间，你可以利用路由群组统一为多个路由设置共同属性，而不需在每个路由上都设置一次。共用属性被指定为数组格式，当作 `Route::group` 方法的第一个参数。

为了了解更多路由群组的相关内容，我们可通过几个常用样例来熟悉这些特性。

<a name="route-group-middleware"></a>
### 中间件

指定中间件到所有群组内的路由中，则可以在群组属性数组里使用 `middleware` 参数。中间件将会依照列表内指定的顺序运行：

    Route::group(['middleware' => 'auth'], function () {
        Route::get('/', function ()    {
            // 使用 Auth 中间件
        });

        Route::get('user/profile', function () {
            // 使用 Auth 中间件
        });
    });

<a name="route-group-namespaces"></a>
### 命名空间

另一个常见的例子是，指定相同的 PHP 命名空间给控制器群组。可以使用 `namespace` 参数来指定群组内所有控制器的命名空间：


    Route::group(['namespace' => 'Admin'], function()
    {
        // 控制器在「App\Http\Controllers\Admin」命名空间

        Route::group(['namespace' => 'User'], function()
        {
            // 控制器在「App\Http\Controllers\Admin\User」命名空间
        });
    });

请记住，默认 `RouteServiceProvider` 会在命名空间群组内导入你的 `routes.php` 文件，让你不用指定完整的 `App\Http\Controllers` 命名空间前缀就能注册控制器路由。所以，我们只需要指定在基底 `App\Http\Controllers` 根命名空间之后的部分命名空间。

<a name="route-group-sub-domain-routing"></a>
### 子域名路由

路由群组也可以被用来做处理通配符的子域名。子域名可以像路由 URIs 分配路由参数，让你在路由或控制器中获取子域名参数。使用路由群组属性数组上的 `domain` 指定子域名变量名称：

    Route::group(['domain' => '{account}.myapp.com'], function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### 路由前缀

通过路由群组数组属性中的 `prefix`，在路由群组内为每个路由指定的 URI 加上前缀。例如，你可能想要在路由群组中将所有的路由 URIs 加上前缀 `admin`：

    Route::group(['prefix' => 'admin'], function () {
        Route::get('users', function ()    {
            // 符合「/admin/users」URL
        });
    });

你也可以使用 `prefix` 参数去指定路由群组中共用的参数：

    Route::group(['prefix' => 'accounts/{account_id}'], function () {
        Route::get('detail', function ($account_id)    {
            // 符合 accounts/{account_id}/detail URL
        });
    });

<a name="csrf-protection"></a>
## CSRF 保护

<a name="csrf-introduction"></a>
### 介绍

Laravel 提供简单的方法保护你的应用程序不受到 [跨网站请求伪造](http://en.wikipedia.org/wiki/Cross-site_request_forgery) 攻击。跨网站请求伪造是一种恶意的攻击，破坏份子伪造 `已通过身份检验的用户身份` 来运行未经授权的命令。

Laravel 会自动生成一个 CSRF token 给每个用户的 Session。该 token 用来验证用户是否为实际发出请求的用户。可以使用 `csrf_field` 辅助函数来生成一个包含 CSRF token 的 `_token` 隐藏表单字段：

    <?php echo csrf_field(); ?>

`csrf_field` 辅助函数会生成以下的 HTML：

    <input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">

当然，也可以在 Blade [模板引擎](/docs/{{version}}/blade) 中使用：

    {{ csrf_field() }}

你不需要手动验证 POST、PUT 或 DELETE 请求的 CSRF token。`VerifyCsrfToken` [HTTP 中间件](/docs/{{version}}/middleware) 将自动验证请求与 session 中的 token 是否相符。

<a name="csrf-excluding-uris"></a>
### 不受 CSRF 保护的 URIs

有时候你可能会希望一组 URIs 不要被 CSRF 保护。例如，你如果使用 [Stripe](https://stripe.com) 处理付款，并且利用他们的 webhook 系统，你需要从 Laravel CSRF 保护中排除 webhook 的处理路由。

可以在 `VerifyCsrfToken` 中间件中增加 `$except` 属性来排除 URIs：

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

    class VerifyCsrfToken extends BaseVerifier
    {
        /**
         * URIs 应被 CSRF 验证执行。
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
        ];
    }

<a name="csrf-x-csrf-token"></a>
### X-CSRF-TOKEN

除了检查 CSRF token 是否被当作 POST 参数之外，在 Laravel `VerifyCsrfToken` 中间件也会检查请求标头中的 `X-CSRF-TOKEN`。例如，你可以将其保存在 meta 标签中：

    <meta name="csrf-token" content="{{ csrf_token() }}">

一旦你创建了 `meta` 标签，你就可以使用 jQuery 之类的函数库将 token 加入到所有的请求标头。基于 AJAX 的应用，提供了简单、方便的 CSRF 保护：

    $.ajaxSetup({
            headers: {
                'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
            }
    });

<a name="csrf-x-xsrf-token"></a>
### X-XSRF-TOKEN

Laravel 也会在 `XSRF-TOKEN` cookie 中保存 CSRF token。你也可以使用 cookie 的值来设置 `X-XSRF-TOKEN` 请求标头。一些 JavaScript 框架会自动帮你处理，例如：Angular。你不大可能会需要手动去设置这个值。

<a name="route-model-binding"></a>
## 路由模型绑定

Laravel 路由模型绑定提供了一个方便的方式来注入类实例到你的路由中。例如，除了注入一个用户的 ID，你也可以注入与指定 ID 相符的完整 `User` 类实例。

### 隐式绑定

Laravel 会自动解析定义在路由或控制器动作（变量名匹配路由片段）中的 Eloquent 模型类型声明，例如：

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

在这个例子中，由于类型声明了 Eloquent 模型 `App\User`，对应的变量名 `$user` 会匹配路由片段中的 `{user}`，这样，Laravel 会自动注入与请求 URI 中传入的 ID 对应的用户模型实例。

如果数据库中找不到对应的模型实例，会会自动生成 HTTP 404 响应。

#### 自定义键名

如果你想要隐式模型绑定使用数据表的其它字段（默认使用 `id`），可以重写 Eloquent 模型类的 `getRouteKeyName` 方法：

    /**
     * 从路由中获取到键
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

### 显式绑定

要注册显式绑定，需要使用路由的 `model` 方法来为给定参数指定绑定类。应该在 `RouteServiceProvider::boot` 方法中定义模型绑定：

#### 绑定参数至模型

    public function boot(Router $router)
    {
        parent::boot($router);

        $router->model('user', 'App\User');
    }

接着，定义包含 `{user}` 参数的路由：

    $router->get('profile/{user}', function(App\User $user) {
        //
    });

因为我们已经绑定 `{user}` 参数至 `App\User` 模型，所以 `User` 实例会被注入至该路由。所以，举个例子，一个至 `profile/1` 的请求会注入 ID 为 1 的 `User` 实例。

> **注意：**如果符合的模型不存在于数据库中，就会自动抛出一个 404 异常。

#### 自定义解析逻辑

如果你想要使用自定义的解析逻辑，需要使用 `Route::bind` 方法，传递到 `bind` 方法的闭包会获取到 URI 请求参数中的值，并且返回你想要在该路由中注入的类实例：

    $router->bind('user', function ($value) {
        return App\User::where('name', $value)->first();
    });

#### 自定义未找到的行为

如果你想要指定自己的模型未找到行为，将封装该行为的闭包作为第三个参数传递给 `model` 方法：

    $router->model('user', 'App\User', function () {
        throw new NotFoundHttpException;
    });

<a name="form-method-spoofing"></a>
## 请求方法伪造

HTML 表单没有支持 `PUT`、`PATCH` 或 `DELETE` 动作。所以在从 HTML 表单中调用被定义的 `PUT`、`PATCH` 或 `DELETE` 路由时，你将需要在表单中增加隐藏的 `_method` 字段。跟随 `_method` 字段送出的值将被作为 HTTP 的请求方法使用：

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

你也可以使用 `methid_field` 辅助函数来生成隐藏的输入字段 `_method`：

    <?php echo method_field('PUT'); ?>

当然，也可以使用 Blade [模板引擎](/docs/{{version}}/blade)：

    {{ method_field('PUT') }}

<a name="accessing-the-current-route"></a>
## 获取当前路由信息

`Route::current()` 方法会返回当前请求的 `Illuminate\Routing\Route` 实例，你可使用此实例进行各种操作：

    $route = Route::current();

    $name = $route->getName();

    $actionName = $route->getActionName();

你可以利用 `Route` facade 的 `currentRouteName` 和 `currentRouteAction` 方法，来获取当前路由的路由命名和控制器动作。

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

更多关于 route 的方法，请查看 API 文档 [Route facade](http://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) 和 [Route 实例](http://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) 。
