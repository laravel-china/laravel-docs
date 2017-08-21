# Laravel 的路由中间件

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

Laravel 中间件提供了一种方便的机制来过滤进入应用的 HTTP 请求，例如，Laravel 包含验证用户身份权限的中间件。如果用户没有通过身份验证，中间件会重定向到登录页，引导用户登录。反之，中间件将允许该请求继续传递到应用程序。

当然，除了身份验证以外，中间件还可以被用来执行各式各样的任务，如：CORS 中间件负责为所有即将离开应用的响应添加适当的头信息；日志中间件可以记录所有传入应用的请求。

Laravel 已经内置了一些中间件，包括身份验证、CSRF 保护等。所有的中间件都放在 `app/Http/Middleware` 目录内。

<a name="defining-middleware"></a>
## 创建中间件

使用 `make:middleware` 这个 Artisan 命令创建新的中间件：

    php artisan make:middleware CheckAge

该命令将会在 `app/Http/Middleware` 目录内新建一个 `CheckAge` 类。在这个中间件内，我们仅允许请求的 `age` 参数大于 200 时访问该路由，否则，会将用户请求重定向到 `home` URI 。

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

如你所见，若请求参数 `age` 小于等于 `200`，中间件将返回给客户端 HTTP 重定向，反之应用程序才会继续处理该请求。若将请求继续传递到应用程序（即允许通过中间件验证），只需将 `$request` 作为参数调用 `$next` 回调函数。

最好将中间件想象为一系列的「层」，HTTP 请求必须经过它们才会触发您的应用程序。每一层都可以检测接收的请求，甚至可以完全拒绝请求访问您的应用。

### 前置中间件 / 后置中间件

中间件运行在请求之前或之后取决于中间件本身。例如，以下中间件会在请求被应用处理 **之前** 执行一些任务

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

然而，这个中间件会在请求被应用处理 **之后** 执行它的任务：

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

如果你希望访问你应用的每个 HTTP 请求都经过某个中间件，只需将该中间件类列入 `app/Http/Kernel.php` 类里的 `$middleware` 属性。

<a name="assigning-middleware-to-routes"></a>
### 为路由指定中间件

如果你想为特殊的路由指定中间件，首先应该在 `app/Http/Kernel.php` 文件内为该中间件指定一个 `键值`。默认情况下 `Kernel` 类的 `$routeMiddleware` 属性已经包含了 Laravel 内置的中间件条目。加入自定义的中间件，只需把它附加到此列表并指定你定义的`键值`即可。例如：

    // App\Http\Kernel 类内

    protected $routeMiddleware = [
        'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    ];

一旦在 HTTP kernel 文件内定义了中间件，即可使用 `middleware` 方法将中间件分配给路由：

    Route::get('admin/profile', function () {
        //
    })->middleware('auth');

为路由指定多个中间件：

    Route::get('/', function () {
        //
    })->middleware('first', 'second');

也可使用完整类名指定中间件：

    use App\Http\Middleware\CheckAge;

    Route::get('admin/profile', function () {
        //
    })->middleware(CheckAge::class);

<a name="middleware-groups"></a>
### 中间件组

有时您可能想要将多个中间件分组到同一个`键值`下，从而使它们更方便地分配给路由，你可以使用 HTTP kernel 的 `$middlewareGroups` 属性来实现。

Laravel 带有开箱即用的 `web` 和 `api` 中间件，包含了可能应用到 Web UI 和 API 路由的通用中间件：

    /**
     * 应用的路由中间件组
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

中间件组可以像单个中间件一样使用相同的语法指定给路由和控制器。重申，路由组仅仅是为了使一次将多个中间件指定给某个路由的实现变得更加方便。

    Route::get('/', function () {
        //
    })->middleware('web');

    Route::group(['middleware' => ['web']], function () {
        //
    });

> {tip} 开箱即用的 `web` 中间件组被自动应用于 `RouteServiceProvider` 中定义的 `routes/web.php` 路由组。

<a name="middleware-parameters"></a>
## 中间件参数

中间件也可以接受其他附加的参数。例如，如果应用需要在运行特定操作前验证该用户具备该操作的权限的「角色」，你可以新建一个 `CheckRole` 中间件，该中间件接收「角色」名字作为附加参数。

附加的中间件参数将在 `$next` 参数之后被传入：

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
                // Redirect...
            }

            return $next($request);
        }

    }

定义路由时，指定中间件参数可以通过冒号 `:` 来隔开中间件与参数，多个参数可以使用逗号分隔：

    Route::put('post/{id}', function ($id) {
        //
    })->middleware('role:editor');

<a name="terminable-middleware"></a>
## Terminable 中间件

有些时候中间件需要在 HTTP 响应发送到浏览器后运行来处理一些任务。比如，Laravel 内置的「session」中间件存储的 session 数据是在响应被发送到浏览器之后才进行写入的。想实现这一点，你需要在中间件中定义一个 `terminable` 方法，它会在响应发送后自动被调用：

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

`terminable` 方法必需接收请求及响应两个参数。一旦定义了 terminable 中间件，你便需要将它增加到 HTTP kernel 文件的全局中间件清单列表中。

中间件的 `terminate` 调用时，Laravel 会从 [服务容器](/docs/{{version}}/container) 中解析一个全新的中间件实例。如果你想在 `handle` 和 `terminate` 被调用时使用同一个中间件实例，可使用容器的 `singleton` 方法向容器注册中间件。


## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@王凯波](http://weibo.com/wangkaibo)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/1924_1487053084.jpeg?imageView2/1/w/100/h/100">  |  翻译  | 面向工资编程 😆 [@wangkaibo](https://github.com/wangkaibo/)  |

--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/3810/laravel-54-document-translation-come-and-join-the-translation)。
> 
> 文档永久地址： http://d.laravel-china.org