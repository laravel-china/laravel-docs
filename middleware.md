# HTTP 中间件

- [简介](#introduction)
- [创建中间件](#defining-middleware)
- [注册中间件](#registering-middleware)
- [中间件参数](#middleware-parameters)
- [Terminable 中间件](#terminable-middleware)

<a name="introduction"></a>
## 简介

HTTP 中间件提供一个方便的机制来过滤进入应用程序的 HTTP 请求，例如，Laravel 本身使用中间件来检验用户身份验证，如果用户未经过身份验证，中间件会将用户导向登录页面，反之，当用户通过身份验证，中间件将会同意此请求继续往前进。

当然，除了身份验证之外，中间件也可以被用来运行各式各样的任务，CORS 中间件负责替所有即将离开程序的回应加入适当的标头。而日志中间件可以记录所有传入应用程序的请求。

Laravel 框架已经内置一些中间件，包括维护、身份验证、CSRF 保护，等等。所有的中间件都放在 `app/Http/Middleware` 目录内。

<a name="defining-middleware"></a>
## 创建中间件

要创建一个新的中间件，可以使用 `make:middleware` 这个 Artisan 命令：

    php artisan make:middleware OldMiddleware

此命令将会在 `app/Http/Middleware` 目录内置立一个名称为 `OldMiddleware` 的类。在这个中间件内我们只允许请求内的 `age` 变量大于 200 的才能访问路由，否则，我们会将用户重新导向「home」这个 URI。

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class OldMiddleware
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

如你所见，若是 age 小于 200，中间件将会返回 HTTP 重新导向给用户端，否则，请求将会进一步传递到应用程序。只需调用带有 $request 的 $next 方法，即可将请求传递到更深层的应用程序(允许通过中间件)。

HTTP 请求在实际碰触到应用程序之前，最好是可以层层通过许多中间件，每一层都可以对请求进行检查，甚至是完全拒绝请求。

### *前* / *后* 中间件

一个中间件是在请求前还是请求后运行要看中间件自己。这个中间件会在应用程序处理请求**前**运行一些任务：

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

这个中间件则会在应用程序处理请求后运行它的任务：

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

### 全域中间件

若是希望每个 HTTP 请求都经过一个中间件，只要将中间件的类加入到 `app/Http/Kernel.php` 的 `$middleware` 属性清单列表中。

### 为路由指派中间件

如果你要指派中间件给特定的路由，你得先将中间件在 app/Http/Kernel.php 设置一个好记的键，默认情况下，这个文件内的 $routeMiddleware 属性已包含了 Laravel 目前设置的中间件，你只需要在清单列表中加上一组自定义的键即可。

    // 在 App\Http\Kernel 类内...

    protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    ];

中间件一旦在 HTTP kernel 文件内被定义，你即可在路由选项内使用 middleware 键值来指派：

    Route::get('admin/profile', ['middleware' => 'auth', function () {
        //
    }]);

使用一组数组为路由指派多个中间件：

    Route::get('/', ['middleware' => ['first', 'second'], function () {
        //
    }]);

除了使用数组之外，你也可以在路由的定义之后链式调用 `middleware` 方法：

    Route::get('/', function () {
        //
    }])->middleware(['first', 'second']);

<a name="middleware-parameters"></a>
## 中间件参数

中间件也可以接收自定义传参，例如，如果应用程序要在运行特定操作之前，检查通过验证的用户是否具备该操作的「角色」，可以创建 `RoleMiddleware` 来接收角色名称作为额外的传参。

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
                // Redirect...
            }

            return $next($request);
        }

    }

在路由中可使用冒号 `:` 来区隔中间件名称与指派参数，多笔参数可使用逗号作为分隔：

    Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
        //
    }]);

<a name="terminable-middleware"></a>
## Terminable 中间件

有些时候中间件需要在 HTTP 回应已被发送到用户端之后才运行，例如，Laravel 内置的「session」中间件，保存 session 数据是在回应已被发送到用户端 之后 才运行。为了做到这一点，你需要定义中间件为「terminable」。

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

`terminate` 方法必须接收请求及回应。一旦定义了 terminable 中间件，你需要将它增加到 HTTP kernel 文件的全域中间件清单列表中。

当在你的中间件调用 `terminate` 方法时，Laravel 会从[服务容器](/docs/{{version}}/container)解析一个全新的中间件实例。如果你希望在 `handle` 及 `terminate` 方法被调用使用一致的中间件实例，只需在容器中使用容器的 `singleton` 方法注册中间件。
