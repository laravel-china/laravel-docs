# Laravel 发行说明

- [版本控制方案](#versioning-scheme)
- [支持策略](#support-policy)
- [Laravel 5.5](#laravel-5.5)

<a name="versioning-scheme"></a>
## 版本控制方案

Laravel 的版本控制方案继续以下约定： `主版本号.次版本号.修订号`。较小的框架发布每六个月一次 (每年一月和七月)，修订号发布可能每周一次。修订号版本应该 **永不** 做破坏性变更。

在应用程序中或者包中引用 Laravel 框架或者其中的组件时，你必须总是使用一个版本约束，例如 `5.5.*`，因为次版本号变化框架会做破坏性变更。我们努力保证你升级一个新的次版本号时使用 1 天或者更少的时间。

主版本号之间的迭代往往需要多年，每次迭代都代表了框架的架构和底层结构发生了改变。目前，在开发过程中没有主版本变更的发布。

#### 为什么不使用语义版本控制?

一方面所有 Laravel 可选的组件 (Cashier, Dusk, Valet, Socialite, etc.) **规定** 使用语义版本控制。但是，框架本身没有使用。原因是语义版本是一种「简化」的方法，用于确定两段代码是否兼容。即使是使用语义版本时，你仍然在安装升级包时运行自动化测试，以了解是否有任何东西 *实际上* 与代码不兼容。

相反，Laravel 框架使用的版本控制方案更适合实际的发布。此外，修订版本的发布 **永不** 包含破坏性变更，为了不接收到破坏性变更，你必须保证 `主版本号.次版本号.*` 的约定。

<a name="support-policy"></a>
## 支持策略

LTS 发布版, 例如 Laravel 5.1, 会有 2 年的 bug 修复和 3 年的安全修复支持。这些发布版提供最长时间的支持和维护。

普通版本，只提供 6 个月的 bug 修复和 1 年的安全修复支持。


> [Laravel 的发布路线图](https://laravel-china.org/articles/2594/laravel-release-roadmap) - by [Summer](https://github.com/summerblue)

<a name="laravel-5.5"></a>
## Laravel 5.5

Laravel 5.5 对 Laravel 5.4 继续改进，添加了：

* 包自动检测
* 控制台命令自动注册
* 队列工作链
* 可渲染邮件样式
* 自定义异常报告
* 更多一致异常处理
* 数据库测试改进
* 简单自定义验证规则
* React 前端预配置，`Route::view` 和 `Route::redirect` 方法
* Memcached 和 Redis「锁」机制
* 按需通知功能
* Dusk 的隐私模式 Chrome 支持
* 方便的 Blade 改进，
* 改进受信端口支持
* 以及更多

除此之外， Laravel 5.5 同时发布 [Laravel Horizon](http://horizon.laravel.com)，一个漂亮的新队列监控和配置系统用来管理你的 Redis 队列。

> {tip} 这份文档总结了最值得注意的框架变更，如果需要了解更多，请查看 [on GitHub](https://github.com/laravel/framework/blob/5.5/CHANGELOG-5.5.md) 变更日志。

### Laravel Horizon

Horizon 提供了一个漂亮的界面和代码驱动配置你的框架 Redis 队列。它允许你很容易监控队列系统关键指标，例如吞吐量，运行时间和失败任务。

你所有的配置都存放一个简单的配置文件中，让你可以对配置文件进行版本控制，以更好的和你的团队能够协作。

更多关于 Horizon 的信息，请查看 [完整的 Horizon 文档](/docs/{{version}}/horizon)

### 包自动检测

在之前的框架版本中，安装一个包通常需要几个步骤，例如添加服务提供者到 `app` 配置文件并注册相关门面。现在，在 Laravel 5.5 中框架可以为你自动检测和注册服务提供者和门面。

例如，你可以通过安装 `barryvdh/laravel-debugbar` 这个包进行实验，通过 Composer 来安装它，debug 条可以在你的应用中启用但你无需在配置文件中添加配置它：

    composer require barryvdh/laravel-debugbar

包开发者只需要添加他们的服务提供者和门面到他们的包的 `composer.json` 文件中：

    "extra": {
        "laravel": {
            "providers": [
                "Laravel\\Tinker\\TinkerServiceProvider"
            ]
        }
    },

更多信息，请查看详细文档 [package development](/docs/{{version}}/packages).

### 控制台命令自动注册

当创建一个新的控制台命令，你不再需要必须手动把它们列入到你的 Console kernel 的 `$commands` 属性中，一个新的 `load` 方法会在 kernel 中调用 `commands` 方法，它将扫描给定的目录，并自动注册控制台命令：

    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');

        // ...
    }

### 新前端预配置

虽然基本的 Vue 脚手架依然包含在 Laravel 5.5 中，几个新的前端配置选项现在也是可用的，一个新创建的应用，你可能会想更换 Vue 脚手架为 React 脚手架，那么使用 `preset` 命令就可以：

    php artisan preset react

或者，你可以移除 JavaScript 和 CSS 框脚手架完全的不使用它们，可以使用 `none`，这个预设配置会为你提供简单的 Sass 和简单的 JavaScript：

    php artisan preset none

> {note} 这个命令只能用在新创建的框架中，不应该在一个已存在的应用中使用它。

### 队列工作链

工作链允许你指定一个队列工作列表，然后这个框架会按照规定顺序允许队列任务，如果一个工作序列失败，则会终止这个工作不再运行。执行队列工作链，你可以在 dispatchable 时使用 `withChain` 方法：

    ProvisionServer::withChain([
        new InstallNginx,
        new InstallPhp
    ])->dispatch();

### 验证规则对象

验证规则对象提供一个新的将自定义验证规则添加到应用程序的紧凑方式。在以前的框架版本中，`Validator::extend` 方法要想使用必须通过匿名函数添加一个自定义的验证规则。

然而，这会增加麻烦，在 Laravel 5.5 中一个新的 `make:rule` Artisan 命令会创建一个新的验证规则在 `app/Rules` 目录中：

    php artisan make:rule ValidName

一个验证对象只含有两个方法：`passes` 和 `message`。`passes` 方法接收属性值和属性名，并应该返回布尔值，根据是否这个属性值通过或没通过验证。`message` 方法应该返回验证错误信息在验证未通过时：


    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;

    class ValidName implements Rule
    {
        /**
         * 确定验证通过规则
         *
         * @param  string  $attribute
         * @param  mixed  $value
         * @return bool
         */
        public function passes($attribute, $value)
        {
            return strlen($value) === 6;
        }

        /**
         * 获取验证错误信息
         *
         * @return string
         */
        public function message()
        {
            return 'The name must be six characters long.';
        }
    }

一旦规则定义，你就能简单的通过一个验证实例对象，将其应用在验证规则中：

    use App\Rules\ValidName;

    $this->validate($request, [
        'name' => ['required', new ValidName],
    ]);

### 受信端口集成

当引用程序运行在负载均衡架构中终止 TLS / SSL，你会注意到你的应用有时不能创建 HTTPS 链接。这通常是因为你的应用流量被转发到负载均衡的 80 端口并且不知道它应该生成安全链接。

为了解决这个问题，许多 Laravel 安装使用 [Trusted Proxies](https://github.com/fideloper/TrustedProxy) Chris Fidao 编写的包. 因为这是一个常见情况，Chris 的包现在已经默认集成在 Laravel 5.5 中了。

一个新的的中间件 `App\Http\Middleware\TrustProxies` 已经默认包含在 Lravel 5.5 中，这个中间件允许你快速自定义受信任的代理：

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Http\Request;
    use Fideloper\Proxy\TrustProxies as Middleware;

    class TrustProxies extends Middleware
    {
        /**
         * 这个应用程序的可信代理
         *
         * @var array
         */
        protected $proxies;

        /**
         * 当前代理头映射
         *
         * @var array
         */
        protected $headers = [
            Request::HEADER_FORWARDED => 'FORWARDED',
            Request::HEADER_X_FORWARDED_FOR => 'X_FORWARDED_FOR',
            Request::HEADER_X_FORWARDED_HOST => 'X_FORWARDED_HOST',
            Request::HEADER_X_FORWARDED_PORT => 'X_FORWARDED_PORT',
            Request::HEADER_X_FORWARDED_PROTO => 'X_FORWARDED_PROTO',
        ];
    }

### 按需通知

有时候你可能需要发送一个通知给一些人而不是程序的 「用户」，使用新的 `Notification::route` 方法，你可以在发送之前指定路由选择：

    Notification::route('mail', 'taylor@laravel.com')
                ->route('nexmo', '5555555555')
                ->send(new InvoicePaid($invoice));

### 可渲染邮件样式

邮件现在可直接从路由返回，允许你在浏览器快速预览你的邮件样式：

    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

### 自定义异常报告

在之前的版本中，你可能不得不求助于异常处理中的「类型检查」，以便为给定异常呈现自定义响应。在实际中，你可能有在 `render` 方法中写这样的异常处理代码：

    /**
     * 渲染一个异常信息给一个 HTTP 响应
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof SpecialException) {
            return response(...);
        }

        return parent::render($request, $exception);
    }

在 Laravel 5.5 中，你可以现在定义一个 `render` 方法直接在你的异常类里，这个方法允许你自定义异常响应的渲染逻辑，这有助于避免异常处理程序中的条件逻辑积累。如果你想要定制改异常的报告逻辑，你可以定义一个 `report` 方法到这个类：


    <?php

    namespace App\Exceptions;

    use Exception;

    class SpecialException extends Exception
    {
        /**
         * 报告异常
         *
         * @return void
         */
        public function report()
        {
            //
        }

        /**
         * 渲染异常
         *
         * @param  \Illuminate\Http\Request
         * @return void
         */
        public function render($request)
        {
            return response(...);
        }
    }

### 请求验证

`Illuminate\Http\Request` 对象现在提供一个 `validate` 方法, 允许你在匿名函数或控制器中验证传入的请求：

    use Illuminate\Http\Request;

    Route::get('/comment', function (Request $request) {
        $request->validate([
            'title' => 'required|string',
            'body' => 'required|string',
        ]);

        // ...
    });

### 一致异常处理

验证异常处理现在贯穿整个框架。以前，在框架多个位置需要定制，以更改 JSON 验证错误响应的默认格式。此外，Laravel 5.5 中 JSON 验证响应默认格式现在遵守以下约定：

    {
        "message": "The given data was invalid.",
        "errors": {
            "field-1": [
                "Error 1",
                "Error 2"
            ],
            "field-2": [
                "Error 1",
                "Error 2"
            ],
        }
    }

所有的 JSON 验证错误格式化能够在一个方法中控制定义，这个方法在 `App\Exceptions\Handler` 类中。举个例子，下面的自定义将使用 Laravel 5.4 约定进行格式化 JSON 验证响应：

    use Illuminate\Validation\ValidationException;

    /**
     * 将验证异常转换为 JSON 响应
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Validation\ValidationException  $exception
     * @return \Illuminate\Http\JsonResponse
     */
    protected function invalidJson($request, ValidationException $exception)
    {
        return response()->json($exception->errors(), $exception->status);
    }

### Cache 锁

框架现在为 Redis 和 Memcached 缓存提供 获得 和 释放 原子「锁」。这提供了一种简单的方法来获取任意锁，而不必担心竞争条件。举个例子，在执行一个任务之前，你可能希望获得一个锁，以使其他进程在你执行时不尝试相同的任务：

    if (Cache::lock('lock-name', 60)->get()) {
        // 获得锁 60 秒，继续处理……

        Cache::lock('lock-name')->release();
    } else {
        // 无法获得锁……
    }

或者，你可以将一个匿名函数传递给 `get` 方法。这个匿名函数只有在获得锁是才会执行：

    Cache::lock('lock-name', 60)->get(function () {
        // 获得锁 60 秒
    });

除此之外，你可以 「阻塞」直到锁可用为止:

    if (Cache::lock('lock-name', 60)->block(10)) {
        // 等待最长10秒的时间，锁可用
    }

### Blade 改进

编写一个自定义条件指令时，定义它有时很复杂。因此 Blade 现在提供一个 `Blade::if` 方法允许你使用匿名函数快速自定义一个条件指令。举个例子，定义一个检查当前应用程序环境的自定义条件，我们可以在我们的 `AppServiceProvider` 的 `boot` 方法这样做：

    use Illuminate\Support\Facades\Blade;

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

一旦定义了自定义条件，我们就可以很容易地在模板上使用它：

    @env('local')
        // 应用是本地环境...
    @else
        // 应用不是本地环境...
    @endenv

除了增加容易定制的条件判断指令外，还新添加了快捷的快速检查当前用户的身份验证：

    @auth
        // 当前用户已经登录...
    @endauth

    @guest
        // 当前用户未登录...
    @endguest

### 新的路由选择方法

如果你定义了一个路由然后重定向到另一个 URI，你现在可以使用 `Route::redirect` 方法。这个方法提供方便快捷的重定向工作，而不必为执行简单的重定向定义完整的路由或控制器：

    Route::redirect('/here', '/there', 301);

如果你的路由仅仅需要返回一个视图，你可以现在使用 `Route::view` 方法，像 `redirect` 方法一样，这个方法提供方便快捷的视图显示而不必为执行简单的重定向定义完整的路由或控制器。`view` 方法接受一个 URI 作为第一个参数，第二个参数是视图的名称，除此之外，你可以提供一个数组数据作为可选参数传递给视图：

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@dongm2ez](https://github.com/dongm2ez)  | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/9032795?v=3&s=460?imageView2/1/w/100/h/100">  |  翻译  | 欢迎在 [Github](https://github.com/dongm2ez) 上关注我 |
