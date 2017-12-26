# Laravel 发行说明

- [版本控制方案](#versioning-scheme)
- [支持策略](#support-policy)
- [Laravel 5.5](#laravel-5.5)

<a name="versioning-scheme"></a>
## 版本控制方案

Laravel 的版本控制方案继续以下约定： `主版本号.次版本号.修订号`。次版本号的框架会在每六个月发布一次 (二月和八月)，修订号发布可能会每周发布一次。而修订号版本**不**应该包含破坏性更改。

当你从应用程序中或者在包中引用 Laravel 框架或其组件时，应始终使用版本约束，例如 `5.5.*`，因为 Laravel 的次版本号会包括突破性更改。但是，我们会努力确保你可以在一天或更短时间内完成更新。

主版本号之间的迭代往往需要多年，每次迭代都代表了框架的架构和底层结构发生了改变。而目前并没有准备开发主版本号的计划。

#### 为什么 Laravel 不使用语义版本控制？

一方面，Laravel 所有可选的组件（Cashier、Dusk、Valet、Socialite 等等）都使用语义版本控制。然而 Laravel 框架本身并没有这样做。原因是语义版本控制是确定两段代码是否兼容的「简化」方法。即使是使用语义版本控制，你仍然必须安装升级包并运行你的自动化测试组件，来确定*事实上*是否有任何异常与代码不兼容。

相反，Laravel 框架使用的版本控制方案更适合实际的发布。此外，因为修订版本的发布**不**包含破坏性变更，只要你的版本遵循 `主版本号.次版本号.*` 的约定， 你就不会接收到破坏性变更。

<a name="support-policy"></a>
## 支持策略

对于 LTS 版本，例如 Laravel 5.1，提供两年的错误修复和三年的安全修复。这些版本提供最长时间的支持和维护。对于一般版本，则只是提供六个月的错误修复和为期一年的安全修复。


> [Laravel 的发布路线图](https://laravel-china.org/articles/2594/laravel-release-roadmap) - by [Summer](https://github.com/summerblue)

<a name="laravel-5.5"></a>
## Laravel 5.5

Laravel 5.5 对 Laravel 5.4 中的包进行了改进，其中添加了：包自动发现、API 资源／转换、控制台命令自动注册、队列任务链、队列任务速率限制、基于时间任务的尝试、可渲染的邮件、自定义异常报告、异常处理规范化、数据库测试改进、更简单自定义验证规则、前端预配置、`Route::view` 和 `Route::redirect` 方法、Memcached 和 Redis 缓存驱动程序的「锁定」、按需通知功能、Dusk 支持 Chrome 的 headless 模式 、方便的 Blade 快捷键、改进可信代理支持等。

此外， Laravel 5.5 同时发布了 [Laravel Horizon](http://horizon.laravel.com)，一个用来管理你的 Redis 队列的漂亮的队列仪表板和配置系统。

> {tip} 这份文档总结了最值得注意的框架变更，如果需要了解更多，请查看 [GitHub](https://github.com/laravel/framework/blob/5.5/CHANGELOG-5.5.md) 上更全面的变更日志。

### Laravel Horizon

Horizon 为你的 Laravel Redis 队列提供了一个漂亮的仪表版和代码驱动配置。Horizon 允许你轻松监控队列系统的关键指标，例如任务吞吐量、运行时间和失败任务。

所有的配置都存放一个简单的配置文件中，让你的整个团队可以协同工作。

更多关于 Horizon 的信息，请查看 [完整的 Horizon 文档](/docs/{{version}}/horizon)。

### 包自动发现

在之前的 Laravel 版本中，安装包通常需要几个步骤，例如添加服务提供器到 `app` 配置文件并注册相关的 facades。现在，从 Laravel 5.5 开始，Laravel 可以自动检测并注册服务提供器和 facades。

例如，你可以通过 `barryvdh/laravel-debugbar` 安装这个包来体验一下。用 Composer 来安装之后，无需任何配置，就可以直接使用 debugbar：

    composer require barryvdh/laravel-debugbar

包的开发者只需要将他们的服务提供器和门面添加到他们的包的 `composer.json` 文件中：

    "extra": {
        "laravel": {
            "providers": [
                "Laravel\\Tinker\\TinkerServiceProvider"
            ]
        }
    },

更多信息，请查看详细文档 [包开发](/docs/{{version}}/packages).

### API 资源

构建 API 时，你可能需要一个位于 Eloquent 模型和实际返回给用户响应之间的 JSON 转换层。Laravel 的资源类允许你以轻松地方式将你的模型和模型集合转换为 JSON。而这个资源类代表了需要转换为 JSON 结构的单个模型。例如，这里是一个简单的用户资源类：

````php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\Resource;

class User extends Resource
{
    /**
     * 将资源转换为数组。
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
````

当然这只是 API 资源的最基本的例子。 Laravel 还提供各种方法来帮助你构建资源和资源集合。有关更多信息，请查看有关 API 资源的 [完整文档](/docs/{{version}}/eloquent-resources)。

### 控制台命令自动注册

当创建一个新的控制台命令，不再需要必须手动把它们列入到你的控制台内核的 `$commands` 属性中。在内核的 `commands` 方法中调用一个新的 `load` 方法，它会扫描给定的目录来获取控制台命令并自动注册它们：

    /**
     * 注册应用程序的命令。
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');

        // ...
    }

### 新前端预配置

虽然基础的 Vue 脚手架依然包含在 Laravel 5.5 中，不同的是，现在有了几种新的前端预设选项可用的。一个新的 Laravel 应用中，你可以使用 `preset` 命令将 Vue 脚手架更换为 React 脚手架：

    php artisan preset react

或者，你可以使用 `preset none` 完全删除 JavaScript 和 CSS 框脚手架。这个预设配置会为你的应用提供简单的 Sass 文件和实用的 JavaScript：

    php artisan preset none

> {note} 这个命令只能用在新创建的框架中，而不应该在现有的应用中使用它。

### 队列任务链

任务链允许你指定按顺序运行的队列任务列表。如果队列中的一个任务失败，则其余任务便不会再运行。要执行队列的任务链，你可以在分配任务时使用 `withChain` 方法：

    ProvisionServer::withChain([
        new InstallNginx,
        new InstallPhp
    ])->dispatch();

### Queued 任务速率限制

如果你的应用程序与 Redis 进行交互，那你就可以根据时间或并发来调整排队的任务。当你队列的任务与限制速率的 API 进行交互时，这个功能就派上用场了。例如，你可以限制给定类型的任务只能每 60 秒运行 10 次：

````Php
Redis::throttle('key')->allow(10)->every(60)->then(function () {
    // 任务逻辑...
}, function () {
    // 无法获得锁...

    return $this->release(10);
});
````

> {tip} 在上面的示例中，`key` 是唯一可以标识要限制的任务类型的字符串。 例如，你可能会根据任务的类名和其运行的 Eloquent 模型的 ID 来构建这个 key。

或者，你也可以指定同时处理给定任务的最大工作进程数。当队列的任务正在修改一次只能由一个任务修改的资源时，我们可以将给定类型的任务限制为一次只能由一个工作进程处理：

````Php
Redis::funnel('key')->limit(1)->then(function () {
    // 任务逻辑...
}, function () {
    // 无法获得锁...

    return $this->release(10);
});
````

### 基于时间的任务尝试

作为在任务失败之前定义任务可能尝试多少次的替代方法，现在你可以定义任务的超时时间。这允许在给定时间范围内尝试次数的任务。将 `retryUntil` 方法添加到任务类中来定义任务的超时时间：

````Php
/**
 * 确定任务的超时时间。
 *
 * @return \DateTime
 */
public function retryUntil()
{
    return now()->addSeconds(5);
}
````

> {tip} 你也可以在队列的事件监听器上定义一个 `retryUntil` 方法。

### 验证规则对象

验证规则对象为你的应用程序提供了一种新的、紧凑的方式来添加自定义验证规则。在以前的 Laravel 版本中，`Validator::extend` 方法用于通过闭包添加自定义验证规则。但是，某种程度上这样挺麻烦的。在 Laravel 5.5 中，可以用新的 Artisan 命令 `make:rule` 在 `app/Rules` 目录中创建一个新的验证规则：

    php artisan make:rule ValidName

验证对象只有两个方法：`passes` 和 `message`。`passes` 方法接收属性值和名称，并根据属性值是否有效返回 `true` 或 `false`。`message` 方法返回验证失败时应使用的验证错误消息：


    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;

    class ValidName implements Rule
    {
        /**
         * 确定验证规则是否通过
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

一旦定义了规则，你就可以简单地通过传递一个规则对象的实例与其他验证规则来使用它：

    use App\Rules\ValidName;

    $this->validate($request, [
        'name' => ['required', new ValidName],
    ]);

### 可信代理集成

在应用程序后面的负载均衡器上运行到期的 TLS / SSL 证书时，你会注意到你的应用有时不能创建 HTTPS 链接。这通常是因为你的应用正在从 80 端口转发流量的负载均衡器不知道安全链接应该被生成。

为了解决这个问题，很多 Laravel 用户安装了Chris Fidao 的包 [Trusted Proxies](https://github.com/fideloper/TrustedProxy) 。因为这是一个常见情况，Chris 的包现在已经默认集成在 Laravel 5.5 中了。

默认情况下，Laravel 5.5 中包含了一个新的中间件 `App\Http\Middleware\TrustProxies`。这个中间件允许你快速自定义受信任的代理：

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
         * 当前的代理头映射
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

有时候你可能需要发送通知给不在应用中存储的 「用户」，使用新的 `Notification::route` 方法，你可以在发送之前指定临时的路由信息：

    Notification::route('mail', 'taylor@laravel.com')
                ->route('nexmo', '5555555555')
                ->send(new InvoicePaid($invoice));

### 可渲染的邮件

现在可直接从路由返回邮件，让你可以在浏览器快速预览你的邮件样式：

    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

### 自定义异常报告

在之前的 Laravel 版本中，你可能不得不在异常处理程序中使用「类型检查」，来为给定异常呈现自定义响应。例如，你可能有在处理异常程序的 `render` 方法中写了这样的代码：

    /**
     * 渲染异常信息给一个 HTTP 响应
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

在 Laravel 5.5 中，你可以直接在异常中定义一个 `render` 方法。这个方法允许你将自定义响应呈现逻辑直接放置在异常上来避免异常处理程序中的条件逻辑积累。如果你想要自定义异常的报告逻辑，你可以在这个类中定义 `report` 方法：


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

`Illuminate\Http\Request` 对象现在提供一个 `validate` 方法, 允许你快速验证传入路由闭包或控制器的请求：

    use Illuminate\Http\Request;

    Route::get('/comment', function (Request $request) {
        $request->validate([
            'title' => 'required|string',
            'body' => 'required|string',
        ]);

        // ...
    });

### 异常处理规范化

现在在整个框架中，验证异常处理的响应消息是一致的。以前，框架中有多个位置需要将默认的验证错误响应的 JSON 格式更改为自定义。现在，Laravel 5.5 中验证响应默认 JSON 格式现在遵守以下约定：

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

所有验证错误的  JSON 格式可以通过在 `App\Exceptions\Handler` 类中定义单个方法来控制。例如，下面将使用 Laravel 5.4 方式进行来自定义验证响应的  JSON 格式：

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

### 缓存锁

Redis 和 Memcached 缓存驱动程序现在支持获取和释放原子「锁」。这提供了一种在不考虑竞争条件的情况下获取任意锁的简单方法。举个例子，在执行任务之前，你可能希望获得给定值的锁定，来确保没有其他进程在执行相同的任务：

    if (Cache::lock('lock-name', 60)->get()) {
        // 获得锁 60 秒，继续处理……

        Cache::lock('lock-name')->release();
    } else {
        // 无法获得锁……
    }

或者，你可以将给 `get` 方法传递一个闭包。在判断可以锁定给定值并且在执行闭包后自动释放锁定的情况下，闭包才会执行：

    Cache::lock('lock-name', 60)->get(function () {
        // 获得锁 60 秒
    });

此外，你也可以直接对给定值进行 「阻塞」直到锁可用为止:

    if (Cache::lock('lock-name', 60)->block(10)) {
        // 等待最长10秒的时间，锁可用
    }

### Blade 改进

编写一个自定义指令有时候比定义简单的自定义条件语句更复杂。因此 Blade 现在提供一个 `Blade::if` 方法，它允许你使用闭包快速定义自定义条件指令。举个例子，定义一个检查当前应用程序环境的自定义条件，我们可以在我们的 `AppServiceProvider` 的 `boot` 方法这样做：

    use Illuminate\Support\Facades\Blade;

    /**
     * 执行注册后引导服务
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

定义完之后，你可以在模版中这样使用：

    @env('local')
        // 应用是本地环境...
    @else
        // 应用不是本地环境...
    @endenv

除了能够轻松定义 Blade 条件指令之外，5.5 还添加了指令 `@auth` 和 `@guest` 来快速检查当前用户的身份验证状态：

    @auth
        // 当前用户已经登录...
    @endauth

    @guest
        // 当前用户未登录...
    @endguest

### 新路由方法

如果要定义重定向到另一个 URI 的路由，可以使用 `Route::redirect` 方法。这个方法算是一种快捷方式。这样你就不必定义完整的路由或控制器来执行简单的重定向：

    Route::redirect('/here', '/there', 301);

同样的效果，如果你的路由只需要返回一个视图，使用 `Route::view` 方法。`view` 方法接受一个 URI 作为第一个参数，视图名称作为其第二个参数。另外，你可以提供一个数组作为可选的第三个参数传递给视图：

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
### 「Sticky」数据库连接

#### `sticky` 选项

配置读／写数据库连接时，可以使用新的 `sticky` 配置选项：

````
'mysql' => [
    'read' => [
        'host' => '192.168.1.1',
    ],
    'write' => [
        'host' => '196.168.1.2'
    ],
    'sticky'    => true,
    'driver'    => 'mysql',
    'database'  => 'database',
    'username'  => 'root',
    'password'  => '',
    'charset'   => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix'    => '',
],
````

`sticky` 选项是一个可选的值，可用于在当前请求周期内立即读取已写入数据库的记录。如果启用了 `sticky` 选项，并且在当前请求周期内对数据库执行了「写入」操作，任何进一步的「读取」操作将使用「写入」连接。这确保了在该请求周期期间写入的任何数据可以在同一请求期间立即从数据库读回。你可以决定这是否是你应用程序所需的行为。

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@dongm2ez](https://github.com/dongm2ez)  | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/9032795?v=3&s=460?imageView2/1/w/100/h/100">  |  翻译  | 欢迎在 [Github](https://github.com/dongm2ez) 上关注我 |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  Review  | Stay Hungry. Stay Foolish. |


---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
