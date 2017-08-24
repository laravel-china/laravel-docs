# Laravel 的路由中间件

- [简介](#简介)
- [创建中间件](#创建中间件)
- [注册中间件](#注册中间件)
    - [全局中间件](#全局中间件)
    - [为路由指定中间件](#为路由指定中间件)
    - [中间件组](#中间件组)
- [中间件参数](#中间件参数)
- [Terminable 中间件](#terminable-中间件)

<a name="introduction"></a>
## Introduction

中间件提供了一套简便机制来过滤进入应用程序的 HTTP 请求。例如，Laravel中包含可以认证用户权限的中间件。若用户未认证，则跳转到登录界面。相反，若用户认证通过，则允许该请求继续传递到应用程序。

当然，除了认证权限外，中间件还可用来处理各种任务。例如，CORS 中间件负责所有应用程序的相应添加合适的头部信息；日志中间件可记录所有传入应用程序的请求。

Laravel 已内置了一些中间件，含权限认证中间件、CSRF 防御等。这些中间件都放在 `app/Http/Middleware`  目录下。

<a name="创建中间件"></a>
## 创建中间件

执行 Artisan 命令 `make:middleware` ，创建中间件。

    php artisan make:middleware CheckAge

该命令将在项目的 `app/Http/Middleware` 目录下，新建一个 `CheckAge` 类。在该中间件中，我们只通过 `age` 参数大于 200 的请求。否则，将用户请求重定向到 `home` URI。

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckAge
    {
        /**
         * 处理传入请求
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

如你所见，如果 `age` 参数小于或等于 `200` 时，中间件将返回 HTTP 重定向给客户端。否则，该请求将通过进入应用程序。为了请求能够传递到应用该程序（即中间件「通过」请求），只需调用回调函数 `$next`,并将 `$request` 作为参数即可。

最好将中间件看作是一系列的「层」，HTTP 请求在触及应用程序前先经过它们。每层都可认证请求，甚至可以完全拒绝请求继续访问。

### 前置/后置中间件

在请求前或后运行中间件取决于其本身。例如，下面这个中间件将会在请求被应用处理 **前** 执行。

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

相反，这个中间件将会在请求被应用处理 **后** 执行。

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

<a name="注册中间件"></a>
## 注册中间件

<a name="全局中间件"></a>
### 全局中间件

如果想要让每个进入应用的 HTTP 请求都通过某个中间件，只要将该中间件列入 `app/Http/Kernel.php` 类的 `$middleware` 属性中即可。

<a name="为路由指定中间件"></a>
### 为路由指定中间件

如果你想要为特定的路由分配中间件，首先需在 `app/Http/Kernel.php` 文件中为该中间件指定一个 `键名`。`$routeMiddleware` 属性，默认包含 Laravel 框架所有的内置中间件。只需将要添加的中间件附加其后，然后为其指定 `键名` 即可。

    // App\Http\Kernel 类内...

    protected $routeMiddleware = [
        'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    ];

一旦在 HTTP kernel 中定义了中间件，便可使用 `middleware` 方法来为路由指定中间件了。

    Route::get('admin/profile', function () {
        //
    })->middleware('auth');

可为路由指定多个中间件。

    Route::get('/', function () {
        //
    })->middleware('first', 'second');

也可使用完整类名指定中间件。

    use App\Http\Middleware\CheckAge;

    Route::get('admin/profile', function () {
        //
    })->middleware(CheckAge::class);

<a name="中间件组"></a>
### 中间件组

有些时候可能想要将某些中间归类，然后通过统一的 `键名` 来指定它们。只要使用 HTTP kernel 中的 `$middlewareGroups` 属性便可实现。

Laravel 自带开箱即用的 `web` 和 `api` 中间件组，当中包含了一些可能应用到 UI 和 API 路由的中间件。

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

类似于独立中间件，使用相同的语法为路由和控制器方法指定中间件组。再次说明，中间件组简便了一次性为路由指定多个中间件。

    Route::get('/', function () {
        //
    })->middleware('web');

    Route::group(['middleware' => ['web']], function () {
        //
    });

> {tip} 开箱即用的 `web` 中间件组自动应用到 `RouteServiceProvider` 中定义的 `routes/web.php` 文件。

<a name="中间件参数"></a>
## 中间件参数

中间件可接收额外的参数。例如，当你的应用在执行特性的操作前，需要认证用户是否拥有该操作的「角色」。可创建一个 `checkRole` 中间件来接收角色名字作为额外参数。

额外的中间件参数可在 `$next` 后传入中间件。

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckRole
    {
        /**
         * 处理传入请求
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

定义路由时，用：来分隔中间件名和指定的中间件参数。多个参数使用逗号分开。

    Route::put('post/{id}', function ($id) {
        //
    })->middleware('role:editor');

<a name="Terminable 中间"></a>
## Terminable 中间件

有时候中间件想要在 HTTP 响应返回到浏览器后执行一些动作。例如，Laravel 内置的「session」中间件在响应发送到浏览器后存储 session 数据。为实现这种需求，只需在你的中间件中定义 `terminate` 方法即可。

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
            // 存储 session 数据...
        }
    }

`terminate` 方法必须接收 request 和 response 两个参数。一旦定义了 terminable 中间件，须在 `app/Http/Kernel.php` 文件中的路由列表或全局中间件中添加它。

当在中间件调用 `terminate` 方法时，Laravel 将从 [服务容器](/docs/5.5/container) 中解析一个新的实例。如果想要在调用 `handle` 和 `terminate` 方法时使用相同的实例，需使用容器的 `singleton` 方法向容器注册中间件。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@半夏](https://laravel-china.org/users/6928)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/6928_1479451835.jpeg?imageView2/1/w/100/h/100">  |  翻译  | [@半夏](https://github.com/mintgreen1108) at Github  |

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： http://d.laravel-china.org
