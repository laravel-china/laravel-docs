# HTTP 中间件

- [简介](#introduction)
- [创建中间件](#defining-middleware)
- [注册中间件](#registering-middleware)
    - [全局中间件](#global-middleware)
    - [为路由指定中间件](#assigning-middleware-to-routes)
    - [中间件组](#middleware-groups)
- [中间件参数](#middleware-parameters)
- [Terminable 中间件](#terminable-middleware)

<a name="introduction"></a>
## 简介

HTTP 中间件提供了一个方便的机制来过滤进入应用程序的 HTTP 请求，例如，Auth 中间件验证用户的身份，如果用户未通过身份验证，中间件将会把用户导向登录页面，反之，当用户通过了身份验证，中间件将会通过此请求并接着往下执行。

当然，除了身份验证之外，中间件也可以被用来运行各式各样的任务，如：CORS 中间件负责替所有即将离开程序的响应加入适当的标头；而日志中间件则可以记录所有传入应用程序的请求。

Laravel 框架已经内置了一些中间件，包括维护、身份验证、CSRF 保护，等等。所有的中间件都放在 `app/Http/Middleware` 目录内。

<a name="defining-middleware"></a>
## 创建中间件

要创建一个新的中间件，则可以使用 `make:middleware` 这个 Artisan 命令：

    php artisan make:middleware AgeMiddleware

此命令将会在 `app/Http/Middleware` 目录内设定一个名称为 `AgeMiddleware` 的类。在这个中间件内我们只允许请求的年龄 `age` 变量大于 200 时才能访问路由，否则，我们会将用户重定向到首页「home」这个 URI 上。

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AgeMiddleware
    {
        /**
         * 运行请求过滤器。
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->input('age') <= 200) {
                return redirect('home');
            }

            return $next($request);
        }

    }

如你所见，若是 age 小于 200，中间件将会返回 HTTP 重定向给用户端，否则，请求将会进一步传递到应用程序。只需调用带有 `$request` 的 `$next` 方法，即可将请求传递到更深层的应用程序（相当于允许通过中间件）。

HTTP 请求在实际碰触到应用程序之前，最好是可以层层通过中间件。每一层都可以对请求进行检查，甚至完全拒绝请求。

### 前置中间件 / 后置中间件

「前置中间件（BeforeMiddleware）」运行于请求处理之前：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class BeforeMiddleware
    {
        public function handle($request, Closure $next)
        {
            // 运行动作

            return $next($request);
        }
    }


> 译者注： 前置中间件运行的时间点是在每一个请求处理之前，可以参阅此文章加深理解：[如何查看 Laravel 5 的所有数据库请求](https://phphub.org/topics/2018)

这个中间件会在应用程序处理请求 **后** 运行它的任务：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AfterMiddleware
    {
        public function handle($request, Closure $next)
        {
            $response = $next($request);

            // 运行动作

            return $response;
        }
    }

<a name="registering-middleware"></a>
## 注册中间件

<a name="global-middleware"></a>
### 全局中间件

若是希望每个 HTTP 请求都经过一个中间件，只要将中间件的类加入到 `app/Http/Kernel.php` 的 `$middleware` 属性清单列表中。

<a name="assigning-middleware-to-routes"></a>
### 为路由指派中间件

如果你要指派中间件给特定路由，你得先在 `app/Http/Kernel.php` 给中间件设置一个好记的 `键`，默认情况下，这个文件内的 `$routeMiddleware` 属性已包含了 Laravel 目前设置的中间件，你只需要在清单列表中加上一组自定义的键即可。

    // 在 App\Http\Kernel 类内...

    protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    ];

中间件一旦在 HTTP `kernel` 文件内被定义，即可在路由选项内使用 `middleware` 键值指定：

    Route::get('admin/profile', ['middleware' => 'auth', function () {
        //
    }]);

使用一组数组为路由指定多个中间件：

    Route::get('/', ['middleware' => ['first', 'second'], function () {
        //
    }]);

除了使用数组之外，你也可以在路由的定义之后链式调用 `middleware` 方法：

    Route::get('/', function () {
        //
    })->middleware(['first', 'second']);

你可以使用完整类名作为路由指派中间件。

    use App\Http\Middleware\FooMiddleware;

    Route::get('admin/profile', ['middleware' => FooMiddleware::class, function () {
        //
    }]);

<a name="middleware-groups"></a>
### 中间件组

有时候你可能想要通过指定一个键名的方式将相关中间件分到一个组里面，从而更方便将其分配到路由中，这可以通过使用 HTTP Kernel 的 `$middlewareGroups` 实现。

Laravel 自带了开箱即用的 `web` 和 `api` 两个中间件组以包含可以应用到 Web UI 和 API 路由的通用中间件：

    /**
     * 应用程序的中间件组
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
        ],

        'api' => [
            'throttle:60,1',
            'auth:api',
        ],
    ];

中间件组可以被分配给路由和控制器动作，使用和单个中间件分配同样的语法。再次申明，中间件组的目的只是让一次分配给路由多个中间件的实现更加简单：

    Route::group(['middleware' => ['web']], function () {
        //
    });

请注意，默认情况下，`RouteServiceProvider` 已经为 `routes.php` 文件指定了 `web` 中间件组。

<a name="middleware-parameters"></a>
## 中间件参数

中间件也可以接收自定义传参，例如，要在运行特定操作前检查已验证用户是否具备该操作的「角色」，可以创建 `RoleMiddleware` 来接收角色名称作为额外的传参。

附加的中间件参数将会在 `$next` 参数之后被传入中间件：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class RoleMiddleware
    {
        /**
         * 运行请求过滤
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

在路由中可使用冒号 `:` 来区隔中间件名称与指派参数，多个参数可使用逗号作为分隔：

    Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
        //
    }]);

<a name="terminable-middleware"></a>
## Terminable 中间件

有些时候中间件需要在 HTTP 响应被发送到浏览器之后才运行，例如，Laravel 内置的「session」中间件存储的 session 数据是在响应被发送到浏览器之后才进行写入的。想要做到这一点，你需要定义中间件为「terminable」。

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
            // 保存 session 数据...
        }
    }

`terminate` 方法必须接收请求及响应。一旦定义了 terminable 中间件，你便需要将它增加到 HTTP kernel 文件的全局中间件清单列表中。

当在你的中间件调用 `terminate` 方法时，Laravel 会从 [服务容器](/docs/{{version}}/container) 解析一个全新的中间件实例。

如果你希望在 `handle` 及 `terminate` 方法被调用时使用一致的中间件实例，只需在容器中使用容器的 `singleton` 方法注册中间件。



--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译。
> 
> 文档永久地址： http://d.laravel-china.org