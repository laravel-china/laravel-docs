# Laravel 的路由中间件

- [简介](#introduction)
- [定义中间件](#defining-middleware)
- [注册中间件](#registering-middleware)
    - [全局中间件](#global-middleware)
    - [为路由分配中间件](#assigning-middleware-to-routes)
    - [中间件组](#middleware-groups)
- [中间件参数](#middleware-parameters)
- [Terminable 中间件](#terminable-middleware)

<a name="introduction"></a>
## 简介

Laravel 中间件提供了一种方便的机制来过滤进入应用的 HTTP 请求。例如，Laravel 内置了一个中间件来验证用户的身份认证。如果用户没有通过身份认证，中间件会将用户重定向到登录界面。但是，如果用户被认证，中间件将允许该请求进一步进入该应用。

当然，除了身份认证以外，还可以编写另外的中间件来执行各种任务。例如：CORS 中间件可以负责为所有离开应用的响应添加合适的头部信息；日志中间件可以记录所有传入应用的请求。

Laravel 自带了一些中间件，包括身份验证、CSRF 保护等。所有这些中间件都位于 `app/Http/Middleware` 目录。

<a name="defining-middleware"></a>
## 定义中间件

运行Artisan 命令 `make:middleware` 创建新的中间件：

    php artisan make:middleware CheckAge

该命令将会在 `app/Http/Middleware` 目录内新建一个 `CheckAge` 类。在这个中间件里，我们仅允许提供的参数 `age` 大于 200 的请求访问该路由。否则，我们会将用户重定向到 `home` 。

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckAge
    {
        /**
         * 处理传入的请求
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->age <= 200) {
                return redirect('home');
            }

            return $next($request);
        }

    }

如你所见，若给定的 `age` 小于等于 `200`，那中间件将返回一个 HTTP 重定向到客户端；否则，请求将进一步传递到应用中。要让请求继续传递到应用程序中（即允许「通过」中间件验证的），只需使用 `$request` 作为参数去调用回调函数 `$next` 。

最好将中间件想象为一系列 HTTP 请求必须经过才能触发你应用的「层」。每一层都会检查请求（是否符合某些条件），（如果不符合）甚至可以（在请求访问你的应用之前）完全拒绝掉。

### 前置 & 后置中间件

中间件是在请求之前或之后运行取决于中间件本身。例如，以下的中间件会在应用处理请求 **之前** 执行一些任务：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class BeforeMiddleware
    {
        public function handle($request, Closure $next)
        {
            // 执行动作

            return $next($request);
        }
    }

而下面（这种写法的）中间件会在应用处理请求 **之后** 执行其任务：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AfterMiddleware
    {
        public function handle($request, Closure $next)
        {
            $response = $next($request);

            // 执行动作

            return $response;
        }
    }

<a name="registering-middleware"></a>
## 注册中间件

<a name="global-middleware"></a>
### 全局中间件

如果你想让中间件在你应用的每个 HTTP 请求期间运行，只需在 `app/Http/Kernel.php` 类中的 `$middleware` 属性里列出这个中间件类 。

<a name="assigning-middleware-to-routes"></a>
### 为路由分配中间件

如果要为特定的路由分配中间件，

如果想为特殊的路由指定中间件，首先应该在 `app/Http/Kernel.php` 文件内为该中间件指定一个 `键`。默认情况下，`Kernel` 类的 `$routeMiddleware` 属性包含 Laravel 内置的中间件条目。要加入自定义的，只需把它附加到列表后并为其分配一个自定义 `键` 即可。例如：

    // 在 App\Http\Kernel 类中

    protected $routeMiddleware = [
        'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    ];

一旦在 Kernel 中定义了中间件，就可使用 `middleware` 方法将中间件分配给路由：

    Route::get('admin/profile', function () {
        //
    })->middleware('auth');

你还可以为路由分配多个中间件：

    Route::get('/', function () {
        //
    })->middleware('first', 'second');

分配中间件时，你还可以传递完整的类名：

    use App\Http\Middleware\CheckAge;

    Route::get('admin/profile', function () {
        //
    })->middleware(CheckAge::class);

<a name="middleware-groups"></a>
### 中间件组

有时你可能想用单一的 `键` 为几个中间件分组，使其更容易分配到路由。可以使用 Kernel 类的 `$middlewareGroups` 属性来实现。

Laravel 自带的 `web` 和 `api` 中间件组包含了你可能会应用到 Web UI 和 API 路由的常见的中间件：

    /**
     * 应用程序的路由中间件组
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],

        'api' => [
            'throttle:60,1',
            'auth:api',
        ],
    ];

可以使用与单个中间件相同的语法将中间件组分配给路由和控制器操作。重申一遍，中间件组只是更方便地实现了一次为路由分配多个中间件。

    Route::get('/', function () {
        //
    })->middleware('web');

    Route::group(['middleware' => ['web']], function () {
        //
    });

> {tip} 无需任何操作，`RouteServiceProvider` 会自动将 `web` 中间件组应用于你的的 `routes/web.php` 文件。

<a name="middleware-parameters"></a>
## 中间件参数

中间件也可以接受额外的参数。例如，如果应用需要在运行特定操作前验证经过身份认证的用户是否具备给定的「角色」，你可以新建一个 `CheckRole` 中间件，由它来接收「角色」名称作为附加参数。

附加的中间件参数应该在 `$next` 参数之后被传递：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckRole
    {
        /**
         * 处理传入的请求
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // 重定向...
            }

            return $next($request);
        }

    }

定义路由时通过一个 `:` 来隔开中间件名称和参数来指定中间件参数。多个参数就使用逗号分隔：

    Route::put('post/{id}', function ($id) {
        //
    })->middleware('role:editor');

<a name="terminable-middleware"></a>
## Terminable 中间件

有时中间件可能需要在 HTTP 响应发送到浏览器之后处理一些工作。比如，Laravel 内置的「session」中间件会在响应发送到浏览器之后将会话数据写入存储器中。如果你在中间件中定义一个 `terminate` 方法，则会在响应发送到浏览器后自动调用：

    <?php

    namespace Illuminate\Session\Middleware;

    use Closure;

    class StartSession
    {
        public function handle($request, Closure $next)
        {
            return $next($request);
        }

        public function terminate($request, $response)
        {
            // Store the session data...
        }
    }

`terminate` 方法应该同时接收和响应。一旦定义了这个中间件，你应该将它添加到路由列表或 `app/Http/Kernel.php` 文件的全局中间件中。

在你的中间件上调用 `terminate` 调用时，Laravel 会从 [服务容器](/docs/{{version}}/container) 中解析出一个新的中间件实例。如果要在调用 `handle` 和 `terminate` 方法时使用同一个中间件实例，就使用容器的 `singleton` 方法向容器注册中间件。


## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@半夏](https://laravel-china.org/users/6928) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/6928_1479451835.jpeg?imageView2/1/w/100/h/100"> |  翻译  | [@半夏](https://github.com/mintgreen1108) |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  Review  | Stay Hungry. Stay Foolish. |


---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
