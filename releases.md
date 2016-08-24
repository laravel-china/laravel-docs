# Release Notes

- [支持策略](#support-policy)
- [Laravel 5.3](#laravel-5.3)
- [Laravel 5.2](#laravel-5.2)
- [Laravel 5.1.11](#laravel-5.1.11)
- [Laravel 5.1.4](#laravel-5.1.4)
- [Laravel 5.1](#laravel-5.1)
- [Laravel 5.0](#laravel-5.0)
- [Laravel 4.2](#laravel-4.2)
- [Laravel 4.1](#laravel-4.1)

<a name="support-policy"></a>
## 支持策略

Laravel 5.1 LTS 版本会提供两年的 BUG 修复及三年的安全性修复，LTS 版本是 Laravel 能提供的支持及维护最长时间支持的发行版。

对于一般的版本，会提供六个月的 BUG 修复及一年的安全性修复。

> [Laravel 的发布路线图](https://phphub.org/topics/2594) - by [Summer](http://github.com/summerblue)

<a name="laravel-5.3"></a>
## Laravel 5.3

Laravel 5.3 在 5.2 的基础上继续进行优化，提供了大量新功能和新特性：基于驱动的通知系统；通过Laravel Echo提供强大的实时支持；通过Laravel Passport实现无痛的OAuth2服务器；通过Laravel Scout实现全文模型搜索；在Laravel Elixir中支持Webpack；“可邮寄”的对象；明确分离web和api路由；基于闭包的控制台命令；存储上传文件的辅助函数；支持POPO和单动作控制器；以及优化前端脚手架；等等等等。

Laravel 5.3 在 5.2 基础上进行了优化，新特性包括以下：

* [消息通知系统 Laravel Notifications](/docs/5.3/notifications)；
* [事件广播系统 Laravel Echo](/docs/5.3/broadcasting)；
* [Laravel Passport 快速 OAuth2 服务器的扩展包](/docs/5.3/passport)；
* [Laravel Scout 全文搜索引擎](/docs/5.3/scout)；
* Laravel Elixir 开始支持 Webpack；
* 邮件操作 Laravel Mailable；
* `web` 和 `api` 的路由分离；
* 基于闭包的控制台命令；
* 上传文件存储的帮助函数；
* 支持 POPO 和单动作控制；
* 优化默认前端脚手架，等。

### Notifications

> {video} Laracasts 上关于此功能的免费视频 [video tutorial](https://laracasts.com/series/whats-new-in-laravel-5-3/episodes/9)。

Laravel Notifications 提供了简单、优雅的 API 支持你在不同的发送媒介中发送通知，例如邮件、SMS、Slack 等等。

例如，你可以定义一个单据已经支付的通知，然后通过邮件和 SMS 发送这个通知：

    $user->notify(new InvoicePaid($invoice));

[Laravel 社区](http://laravel-notification-channels.com) 已经为通知系统编写了各式的驱动，甚至包括对 iOS 和 Android 通知的支持，更多关于通知系统的信息，请查看 [完整的文档](/docs/5.3/notifications)。

### WebSockets / 事件广播

事件广播在之前版本的 Laravel 中已经存在，Laravel 5.3 现支持对已私有和已存在的 WebSocket 频道添加频道级认证：

    /*
     * 频道认证
     */
    Broadcast::channel('orders.*', function ($user, $orderId) {
        return $user->placedOrder($orderId);
    });

Laravel Echo，可通过 NPM 安装的全新的 JavaScript 包，会和 Laravel 5.3 一起发布，为客户端 JavaScript 应用中监听服务器端事件提供了简单、优雅的 API 接口。

Echo 默认包含对 [Pusher](https://pusher.com) 和 [Socket.io](http://socket.io 的支持：

    Echo.channel('orders.' + orderId)
        .listen('ShippingStatusUpdated', (e) => {
            console.log(e.description);
        });

除了订阅到传统频道上，Laravel Echo 也让频道间的监听变得简单：

    Echo.join('chat.' + roomId)
        .here((users) => {
            //
        })
        .joining((user) => {
            console.log(user.name);
        })
        .leaving((user) => {
            console.log(user.name);
        });

更多信息请查阅 [完整文档](/docs/5.3/broadcasting).

### Laravel Passport (OAuth2 认证服务)

> {video} Laracasts 上关于此功能的免费视频 [video tutorial](https://laracasts.com/series/whats-new-in-laravel-5-3/episodes/13)。

Laravel 5.3 的 Passport 让 API 认证变得简单。Laravel Passport 可以让你在几分钟内为应用程序创建一个完整的 OAuth2 认证服务，Passport 基于 Alex Bilbie 的 [League OAuth2 server](https://github.com/thephpleague/oauth2-server) 实现。

Passport 让发放 OAuth2 令牌（Access Token）变得轻松，你还可以允许用户通过 Web 界面创建 `个人访问令牌`。

为了方便提高开发效率，Passport 内置了一个 Vue 组件，该组件提供了 OAuth2 后台界面功能，允许用户创建客户端、撤销访问令牌，以及更多其他功能：

    <passport-clients></passport-clients>
    <passport-authorized-clients></passport-authorized-clients>
    <passport-personal-access-tokens></passport-personal-access-tokens>

如果你不想使用 Vue 组件，你可以自由的定制用于管理客户端和访问令牌的前端、后台。Passport 提供了一个简单的 JSON API，你可以在前端使用任何 JavaScript 框架与之集成。

Passport 还提供了方便的 API 让你定制「Token 访问域」：

    Passport::tokensCan([
        'place-orders' => 'Place new orders',
        'check-status' => 'Check order status',
    ]);

此外，Passport 还包含了一个用于检查「Token 访问域」访问权限的中间件：

    Route::get('/orders/{order}/status', function (Order $order) {
        // Access token has "check-status" scope...
    })->middleware('scope:check-status');

最后，Passport 还支持从 JavaScript 应用访问你的 API，而不必担心访问令牌传输。

Passport 通过加密 JWT cookies 和同步「CSRF 令牌」来实现此功能，让你专注于业务开发。

更多关于 Passport 信息，请查看 [完整文档](/docs/5.3/passport)。

### 搜索系统 (Laravel Scout)

Laravel Scout 提供了一个简单的、基于驱动的、针对 [Eloquent](/docs/5.3/eloquent) 模型的全文搜索解决方案。

通过模型观察者，Scout 会自动同步更新 Eloquent 的搜索索引，目前，Scout使用 [Algolia](https://www.algolia.com/) 驱动，你可以自由的编写自己驱动来扩展 Scout。

你只需要添加 Searchable trait 到模型中，就能让模型支持搜索：

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;
    }

在你在模型中添加 trait 以后，数据会在 `save` 的时候自动保持同步：

    $order = new Order;

    // ...

    $order->save();

在模型被成功索引以后，可以很轻松的使用全文搜索，你甚至可以为索引的结果进行分页操作：

    return Order::search('Star Trek')->get();

    return Order::search('Star Trek')->where('user_id', 1)->paginate();

更多 Scout 功能，请查阅 [Scout 的完整文档](/docs/5.3/scout)。

### Mailable 对象

> {video} Laracasts 上关于此功能的免费视频 [video tutorial](https://laracasts.com/series/whats-new-in-laravel-5-3/episodes/6)。

Laravel 5.3 Mailable 是一个崭新的 Mail 操作类，通过一种更加优雅的方式发送邮件，而不再需要在闭包中自定义邮件信息。

例如，定义一个简单的邮寄对象用作发送欢迎邮件：

    class WelcomeMessage extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.welcome');
        }
    }

Mailable 对象被创建以后，你可以使用一个简单、优雅的 API 将其发送给用户：

    Mail::to($user)->send(new WelcomeMessage);

Mailable 还支持队列操作，只需要在类声明里实现 `ShouldQueue` 即可：

    class WelcomeMessage extends Mailable implements ShouldQueue
    {
        //
    }

更多关于 Mailable 的信息，请查看 [完整文档](/docs/5.3/mail)。

### 存储上传文件

> {video} Laracasts 上关于此功能的免费视频 [video tutorial](https://laracasts.com/series/whats-new-in-laravel-5-3/episodes/12)。

存储用户上传文件，在 Web 开发中是一个很常见的任务。

Laravel 5.3 提供了一个便捷的 `store` 方法，只需要对上传文件对象调用此方法，并传参准备存储的路径即可：

    /**
     * Update the avatar for the user.
     *
     * @param  Request  $request
     * @return Response
     */
    public function update(Request $request)
    {
        $path = $request->file('avatar')->store('avatars', 's3');

        return $path;
    }

更多上传文件信息，请查看 [完整文档](/docs/{{version}}/filesystem#file-uploads)。


### Webpack 和 Laravel Elixir

Laravel Elixir 6.0 与 Laravel 5.3 共同发布，内置了 Webpack 和 Rollup JavaScript。

默认情况下，Laravel 5.3 的 `gulpfile.js` 使用 Webpack 来编译你的 JavaScript 文件：

    elixir(mix => {
        mix.sass('app.scss')
           .webpack('app.js');
    });

完整文档请见 [Laravel Elixir](/docs/5.3/elixir) 。

### 前端架构

> {video} Laracasts 上关于此功能的免费视频 [video tutorial](https://laracasts.com/series/whats-new-in-laravel-5-3/episodes/4)。

Laravel 5.3 提供了一个更加现代的前端架构。这主要会影响 `make:auth` 命令生成的认证前端脚手架代码，不再从 CDN 中加载前端资源，所有依赖被定义在默认的 package.json 文件中，你可以自行修改。

此外，支持单文件的 Vue 组件现在直接开箱即用， `resources/assets/js/components` 目录下包含了一个简单的示例 `Example.vue`，新的 `resources/assets/js/app.js` 用来配置 JavaScript 类库依赖和 Vue 子模块。

这种架构对开始开发现代的、强大的 JavaScript 应用提供了更好的支持，而不需要要求应用使用任何特定 JavaScript 或者 CSS 框架。

更多信息，请查看对应文档 [前端文档](/docs/5.3/frontend)。

### 路由文件

By default, fresh Laravel 5.3 applications contain two HTTP route files in a new top-level `routes` directory. The `web` and `api` route files provide more explicit guidance in how to split the routes for your web interface and your API. The routes in the `api` route file are automatically assigned the `api` prefix by the `RouteServiceProvider`.

默认情况下，新安装的 Laravel 5.3 应用在新的顶级目录 `routes` 下包含了 `web.php` 和 `api.php` 两个 `HTTP` 路由文件，你也可以按照此方法自行扩展。

API 相关的路由在 `RouteServiceProvider` 中指定了自定添加 `api` 前缀。

### 闭包控制台命令

除了通过命令类定义之外，Artisan 命令现支持在 `app/Console/Kernel.php` 文件中使用简单闭包的方式定义。

在新安装的 Laravel 5.3 应用中，commands 方法会加载 routes/console.php 文件，从而允许你基于闭包、以路由风格定义控制台命令：

    Artisan::command('build {project}', function ($project) {
        $this->info('Building project...');
    });

更多信息请参见 [Artisan 文档](/docs/5.3/artisan#closure-commands)。

### Blade 中的 `$loop` 魔术变量

> {video} Laracasts 上关于此功能的免费视频 [video tutorial](https://laracasts.com/series/whats-new-in-laravel-5-3/episodes/7)。

当我们在 Blade 模板中循环遍历的时候，$loop 魔术变量将会在循环中生效。通过该变量可以访问很多有用的信息，比如当前循环索引值，以及当前循环是第一个还是最后一个：

    @foreach ($users as $user)
        @if ($loop->first)
            This is the first iteration.
        @endif

        @if ($loop->last)
            This is the last iteration.
        @endif

        <p>This is user {{ $user->id }}</p>
    @endforeach

更多信息请查看 [Blade 文档](/docs/5.3/blade#the-loop-variable).

<a name="laravel-5.2"></a>
## Laravel 5.2

Laravel 5.2 continues the improvements made in Laravel 5.1 by adding multiple authentication driver support, implicit model binding, simplified Eloquent global scopes, opt-in authentication scaffolding, middleware groups, rate limiting middleware, array validation improvements, and more.

### Authentication Drivers / "Multi-Auth"

In previous versions of Laravel, only the default, session-based authentication driver was supported out of the box, and you could not have more than one authenticatable model instance per application.

However, in Laravel 5.2, you may define additional authentication drivers as well define multiple authenticatable models or user tables, and control their authentication process separately from each other. For example, if your application has one database table for "admin" users and one database table for "student" users, you may now use the `Auth` methods to authenticate against each of these tables separately.

### Authentication Scaffolding

Laravel already makes it easy to handle authentication on the back-end; however, Laravel 5.2 provides a convenient, lightning-fast way to scaffold the authentication views for your front-end. Simply execute the `make:auth` command on your terminal:

    php artisan make:auth

This command will generate plain, Bootstrap compatible views for user login, registration, and password reset. The command will also update your routes file with the appropriate routes.

> {note} This feature is only meant to be used on new applications, not during application upgrades.

### Implicit Model Binding

Implicit model binding makes it painless to inject relevant models directly into your routes and controllers. For example, assume you have a route defined like the following:

    use App\User;

    Route::get('/user/{user}', function (User $user) {
        return $user;
    });

In Laravel 5.1, you would typically need to use the `Route::model` method to instruct Laravel to inject the `App\User` instance that matches the `{user}` parameter in your route definition. However, in Laravel 5.2, the framework will **automatically** inject this model based on the URI segment, allowing you to quickly gain access to the model instances you need.

Laravel will automatically inject the model when the route parameter segment (`{user}`) matches the route Closure or controller method's corresponding variable name (`$user`) and the variable is type-hinting an Eloquent model class.

### Middleware Groups

Middleware groups allow you to group several route middleware under a single, convenient key, allowing you to assign several middleware to a route at once. For example, this can be useful when building a web UI and an API within the same application. You may group the session and CSRF routes into a `web` group, and perhaps the rate limiter in the `api` group.

In fact, the default Laravel 5.2 application structure takes exactly this approach. For example, in the default `App\Http\Kernel.php` file you will find the following:

    /**
     * The application's route middleware groups.
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
        ],
    ];

Then, the `web` group may be assigned to routes like so:

    Route::group(['middleware' => ['web']], function () {
        //
    });

However, keep in mind the `web` middleware group is *already* applied to your routes by default since the `RouteServiceProvider` includes it in the default middleware group.

### Rate Limiting

A new rate limiter middleware is now included with the framework, allowing you to easily limit the number of requests that a given IP address can make to a route over a specified number of minutes. For example, to limit a route to 60 requests every minute from a single IP address, you may do the following:

    Route::get('/api/users', ['middleware' => 'throttle:60,1', function () {
        //
    }]);

### Array Validation

Validating array form input fields is much easier in Laravel 5.2. For example, to validate that each e-mail in a given array input field is unique, you may do the following:

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users'
    ]);

Likewise, you may use the `*` character when specifying your validation messages in your language files, making it a breeze to use a single validation message for array based fields:

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],

### Bail Validation Rule

A new `bail` validation rule has been added, which instructs the validator to stop validating after the first validation failure for a given rule. For example, you may now prevent the validator from running a `unique` check if an attribute fails an `integer` check:

    $this->validate($request, [
        'user_id' => 'bail|integer|unique:users'
    ]);

### Eloquent Global Scope Improvements

In previous versions of Laravel, global Eloquent scopes were complicated and error-prone to implement; however, in Laravel 5.2, global query scopes only require you to implement a single, simple method: `apply`.

For more information on writing global scopes, check out the full [Eloquent documentation](/docs/{{version}}/eloquent#global-scopes).

<a name="laravel-5.1.11"></a>
## Laravel 5.1.11

Laravel 5.1.11 introduces [authorization](/docs/{{version}}/authorization) support out of the box! Conveniently organize your application's authorization logic using simple callbacks or policy classes, and authorize actions using simple, expressive methods.

For more information, please refer to the [authorization documentation](/docs/{{version}}/authorization).

<a name="laravel-5.1.4"></a>
## Laravel 5.1.4

Laravel 5.1.4 introduces simple login throttling to the framework. Consult the [authentication documentation](/docs/{{version}}/authentication#authentication-throttling) for more information.

<a name="laravel-5.1"></a>
## Laravel 5.1

Laravel 5.1 continues the improvements made in Laravel 5.0 by adopting PSR-2 and adding event broadcasting, middleware parameters, Artisan improvements, and more.

### PHP 5.5.9+

Since PHP 5.4 will enter "end of life" in September and will no longer receive security updates from the PHP development team, Laravel 5.1 requires PHP 5.5.9 or greater. PHP 5.5.9 allows compatibility with the latest versions of popular PHP libraries such as Guzzle and the AWS SDK.

### LTS

 Laravel 5.1 is the first release of Laravel to receive **long term support**. Laravel 5.1 will receive bug fixes for 2 years and security fixes for 3 years. This support window is the largest ever provided for Laravel and provides stability and peace of mind for larger, enterprise clients and customers.

### PSR-2

The [PSR-2 coding style guide](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) has been adopted as the default style guide for the Laravel framework. Additionally, all generators have been updated to generate PSR-2 compatible syntax.

### Documentation

Every page of the Laravel documentation has been meticulously reviewed and dramatically improved. All code examples have also been reviewed and expanded to provide more relevance and context.

### Event Broadcasting

In many modern web applications, web sockets are used to implement realtime, live-updating user interfaces. When some data is updated on the server, a message is typically sent over a websocket connection to be handled by the client.

To assist you in building these types of applications, Laravel makes it easy to "broadcast" your events over a websocket connection. Broadcasting your Laravel events allows you to share the same event names between your server-side code and your client-side JavaScript framework.

To learn more about event broadcasting, check out the [event documentation](/docs/{{version}}/events#broadcasting-events).

### Middleware Parameters

Middleware can now receive additional custom parameters. For example, if your application needs to verify that the authenticated user has a given "role" before performing a given action, you could create a `RoleMiddleware` that receives a role name as an additional argument:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class RoleMiddleware
    {
        /**
         * Run the request filter.
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

Middleware parameters may be specified when defining the route by separating the middleware name and parameters with a `:`. Multiple parameters should be delimited by commas:

    Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
        //
    }]);

For more information on middleware, check out the [middleware documentation](/docs/{{version}}/middleware).

### Testing Overhaul

The built-in testing capabilities of Laravel have been dramatically improved. A variety of new methods provide a fluent, expressive interface for interacting with your application and examining its responses. For example, check out the following test:

    public function testNewUserRegistration()
    {
        $this->visit('/register')
             ->type('Taylor', 'name')
             ->check('terms')
             ->press('Register')
             ->seePageIs('/dashboard');
    }

For more information on testing, check out the [testing documentation](/docs/{{version}}/testing).

### Model Factories

Laravel now ships with an easy way to create stub Eloquent models using [model factories](/docs/{{version}}/database-testing#writing-factories). Model factories allow you to easily define a set of "default" attributes for your Eloquent model, and then generate test model instances for your tests or database seeds. Model factories also take advantage of the powerful [Faker](https://github.com/fzaninotto/Faker) PHP library for generating random attribute data:

    $factory->define(App\User::class, function ($faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->email,
            'password' => str_random(10),
            'remember_token' => str_random(10),
        ];
    });

For more information on model factories, check out [the documentation](/docs/{{version}}/database-testing#writing-factories).

### Artisan Improvements

Artisan commands may now be defined using a simple, route-like "signature", which provides an extremely simple interface for defining command line arguments and options. For example, you may define a simple command and its options like so:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--force}';

For more information on defining Artisan commands, consult the [Artisan documentation](/docs/{{version}}/artisan).

### Folder Structure

To better express intent, the `app/Commands` directory has been renamed to `app/Jobs`. Additionally, the `app/Handlers` directory has been consolidated into a single `app/Listeners` directory which simply contains event listeners. However, this is not a breaking change and you are not required to update to the new folder structure to use Laravel 5.1.

### Encryption

In previous versions of Laravel, encryption was handled by the `mcrypt` PHP extension. However, beginning in Laravel 5.1, encryption is handled by the `openssl` extension, which is more actively maintained.

<a name="laravel-5.0"></a>
## Laravel 5.0

Laravel 5.0 introduces a fresh application structure to the default Laravel project. This new structure serves as a better foundation for building a robust application in Laravel, as well as embraces new auto-loading standards (PSR-4) throughout the application. First, let's examine some of the major changes:

### New Folder Structure

The old `app/models` directory has been entirely removed. Instead, all of your code lives directly within the `app` folder, and, by default, is organized to the `App` namespace. This default namespace can be quickly changed using the new `app:name` Artisan command.

Controllers, middleware, and requests (a new type of class in Laravel 5.0) are now grouped under the `app/Http` directory, as they are all classes related to the HTTP transport layer of your application. Instead of a single, flat file of route filters, all middleware are now broken into their own class files.

A new `app/Providers` directory replaces the `app/start` files from previous versions of Laravel 4.x. These service providers provide various bootstrapping functions to your application, such as error handling, logging, route loading, and more. Of course, you are free to create additional service providers for your application.

Application language files and views have been moved to the `resources` directory.

### Contracts

All major Laravel components implement interfaces which are located in the `illuminate/contracts` repository. This repository has no external dependencies. Having a convenient, centrally located set of interfaces you may use for decoupling and dependency injection will serve as an easy alternative option to Laravel Facades.

For more information on contracts, consult the [full documentation](/docs/{{version}}/contracts).

### Route Cache

If your application is made up entirely of controller routes, you may utilize the new `route:cache` Artisan command to drastically speed up the registration of your routes. This is primarily useful on applications with 100+ routes and will **drastically** speed up this portion of your application.

### Route Middleware

In addition to Laravel 4 style route "filters", Laravel 5 now supports HTTP middleware, and the included authentication and CSRF "filters" have been converted to middleware. Middleware provides a single, consistent interface to replace all types of filters, allowing you to easily inspect, and even reject, requests before they enter your application.

For more information on middleware, check out [the documentation](/docs/{{version}}/middleware).

### Controller Method Injection

In addition to the existing constructor injection, you may now type-hint dependencies on controller methods. The [service container](/docs/{{version}}/container) will automatically inject the dependencies, even if the route contains other parameters:

    public function createPost(Request $request, PostRepository $posts)
    {
        //
    }

### Authentication Scaffolding

User registration, authentication, and password reset controllers are now included out of the box, as well as simple corresponding views, which are located at `resources/views/auth`. In addition, a "users" table migration has been included with the framework. Including these simple resources allows rapid development of application ideas without bogging down on authentication boilerplate. The authentication views may be accessed on the `auth/login` and `auth/register` routes. The `App\Services\Auth\Registrar` service is responsible for user validation and creation.

### Event Objects

You may now define events as objects instead of simply using strings. For example, check out the following event:

    <?php

    class PodcastWasPurchased
    {
        public $podcast;

        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }
    }

The event may be dispatched like normal:

    Event::fire(new PodcastWasPurchased($podcast));

Of course, your event handler will receive the event object instead of a list of data:

    <?php

    class ReportPodcastPurchase
    {
        public function handle(PodcastWasPurchased $event)
        {
            //
        }
    }

For more information on working with events, check out the [full documentation](/docs/{{version}}/events).

### Commands / Queueing

In addition to the queue job format supported in Laravel 4, Laravel 5 allows you to represent your queued jobs as simple command objects. These commands live in the `app/Commands` directory. Here's a sample command:

    <?php

    class PurchasePodcast extends Command implements SelfHandling, ShouldBeQueued
    {
        use SerializesModels;

        protected $user, $podcast;

        /**
         * Create a new command instance.
         *
         * @return void
         */
        public function __construct(User $user, Podcast $podcast)
        {
            $this->user = $user;
            $this->podcast = $podcast;
        }

        /**
         * Execute the command.
         *
         * @return void
         */
        public function handle()
        {
            // Handle the logic to purchase the podcast...

            event(new PodcastWasPurchased($this->user, $this->podcast));
        }
    }

The base Laravel controller utilizes the new `DispatchesCommands` trait, allowing you to easily dispatch your commands for execution:

    $this->dispatch(new PurchasePodcastCommand($user, $podcast));

Of course, you may also use commands for tasks that are executed synchronously (are not queued). In fact, using commands is a great way to encapsulate complex tasks your application needs to perform. For more information, check out the [command bus](/docs/{{version}}/bus) documentation.

### Database Queue

A `database` queue driver is now included in Laravel, providing a simple, local queue driver that requires no extra package installation beyond your database software.

### Laravel Scheduler

In the past, developers have generated a Cron entry for each console command they wished to schedule. However, this is a headache. Your console schedule is no longer in source control, and you must SSH into your server to add the Cron entries. Let's make our lives easier. The Laravel command scheduler allows you to fluently and expressively define your command schedule within Laravel itself, and only a single Cron entry is needed on your server.

It looks like this:

    $schedule->command('artisan:command')->dailyAt('15:00');

Of course, check out the [full documentation](/docs/{{version}}/scheduling) to learn all about the scheduler!

### Tinker / Psysh

The `php artisan tinker` command now utilizes [Psysh](https://github.com/bobthecow/psysh) by Justin Hileman, a more robust REPL for PHP. If you liked Boris in Laravel 4, you're going to love Psysh. Even better, it works on Windows! To get started, just try:

    php artisan tinker

### DotEnv

Instead of a variety of confusing, nested environment configuration directories, Laravel 5 now utilizes [DotEnv](https://github.com/vlucas/phpdotenv) by Vance Lucas. This library provides a super simple way to manage your environment configuration, and makes environment detection in Laravel 5 a breeze. For more details, check out the full [configuration documentation](/docs/{{version}}/installation#environment-configuration).

### Laravel Elixir

Laravel Elixir, by Jeffrey Way, provides a fluent, expressive interface to compiling and concatenating your assets. If you've ever been intimidated by learning Grunt or Gulp, fear no more. Elixir makes it a cinch to get started using Gulp to compile your Less, Sass, and CoffeeScript. It can even run your tests for you!

For more information on Elixir, check out the [full documentation](/docs/{{version}}/elixir).

### Laravel Socialite

Laravel Socialite is an optional, Laravel 5.0+ compatible package that provides totally painless authentication with OAuth providers. Currently, Socialite supports Facebook, Twitter, Google, and GitHub. Here's what it looks like:

    public function redirectForAuth()
    {
        return Socialize::with('twitter')->redirect();
    }

    public function getUserFromProvider()
    {
        $user = Socialize::with('twitter')->user();
    }

No more spending hours writing OAuth authentication flows. Get started in minutes! The [full documentation](/docs/{{version}}/authentication#social-authentication) has all the details.

### Flysystem Integration

Laravel now includes the powerful [Flysystem](https://github.com/thephpleague/flysystem) filesystem abstraction library, providing pain free integration with local, Amazon S3, and Rackspace cloud storage - all with one, unified and elegant API! Storing a file in Amazon S3 is now as simple as:

    Storage::put('file.txt', 'contents');

For more information on the Laravel Flysystem integration, consult the [full documentation](/docs/{{version}}/filesystem).

### Form Requests

Laravel 5.0 introduces **form requests**, which extend the `Illuminate\Foundation\Http\FormRequest` class. These request objects can be combined with controller method injection to provide a boiler-plate free method of validating user input. Let's dig in and look at a sample `FormRequest`:

    <?php

    namespace App\Http\Requests;

    class RegisterRequest extends FormRequest
    {
        public function rules()
        {
            return [
                'email' => 'required|email|unique:users',
                'password' => 'required|confirmed|min:8',
            ];
        }

        public function authorize()
        {
            return true;
        }
    }

Once the class has been defined, we can type-hint it on our controller action:

    public function register(RegisterRequest $request)
    {
        var_dump($request->input());
    }

When the Laravel service container identifies that the class it is injecting is a `FormRequest` instance, the request will **automatically be validated**. This means that if your controller action is called, you can safely assume the HTTP request input has been validated according to the rules you specified in your form request class. Even more, if the request is invalid, an HTTP redirect, which you may customize, will automatically be issued, and the error messages will be either flashed to the session or converted to JSON. **Form validation has never been more simple.** For more information on `FormRequest` validation, check out the [documentation](/docs/{{version}}/validation#form-request-validation).

### Simple Controller Request Validation

The Laravel 5 base controller now includes a `ValidatesRequests` trait. This trait provides a simple `validate` method to validate incoming requests. If `FormRequests` are a little too much for your application, check this out:

    public function createPost(Request $request)
    {
        $this->validate($request, [
            'title' => 'required|max:255',
            'body' => 'required',
        ]);
    }

If the validation fails, an exception will be thrown and the proper HTTP response will automatically be sent back to the browser. The validation errors will even be flashed to the session! If the request was an AJAX request, Laravel even takes care of sending a JSON representation of the validation errors back to you.

For more information on this new method, check out [the documentation](/docs/{{version}}/validation#validation-quickstart).

### New Generators

To complement the new default application structure, new Artisan generator commands have been added to the framework. See `php artisan list` for more details.

### Configuration Cache

You may now cache all of your configuration in a single file using the `config:cache` command.

### Symfony VarDumper

The popular `dd` helper function, which dumps variable debug information, has been upgraded to use the amazing Symfony VarDumper. This provides color-coded output and even collapsing of arrays. Just try the following in your project:

    dd([1, 2, 3]);

<a name="laravel-4.2"></a>
## Laravel 4.2

The full change list for this release by running the `php artisan changes` command from a 4.2 installation, or by [viewing the change file on Github](https://github.com/laravel/framework/blob/4.2/src/Illuminate/Foundation/changes.json). These notes only cover the major enhancements and changes for the release.

> {note} During the 4.2 release cycle, many small bug fixes and enhancements were incorporated into the various Laravel 4.1 point releases. So, be sure to check the change list for Laravel 4.1 as well!

### PHP 5.4 Requirement

Laravel 4.2 requires PHP 5.4 or greater. This upgraded PHP requirement allows us to use new PHP features such as traits to provide more expressive interfaces for tools like [Laravel Cashier](/docs/billing). PHP 5.4 also brings significant speed and performance improvements over PHP 5.3.

### Laravel Forge

Laravel Forge, a new web based application, provides a simple way to create and manage PHP servers on the cloud of your choice, including Linode, DigitalOcean, Rackspace, and Amazon EC2. Supporting automated Nginx configuration, SSH key access, Cron job automation, server monitoring via NewRelic & Papertrail, "Push To Deploy", Laravel queue worker configuration, and more, Forge provides the simplest and most affordable way to launch all of your Laravel applications.

The default Laravel 4.2 installation's `app/config/database.php` configuration file is now configured for Forge usage by default, allowing for more convenient deployment of fresh applications onto the platform.

More information about Laravel Forge can be found on the [official Forge website](https://forge.laravel.com).

### Laravel Homestead

Laravel Homestead is an official Vagrant environment for developing robust Laravel and PHP applications. The vast majority of the boxes' provisioning needs are handled before the box is packaged for distribution, allowing the box to boot extremely quickly. Homestead includes Nginx 1.6, PHP 5.6, MySQL, Postgres, Redis, Memcached, Beanstalk, Node, Gulp, Grunt, & Bower. Homestead includes a simple `Homestead.yaml` configuration file for managing multiple Laravel applications on a single box.

The default Laravel 4.2 installation now includes an `app/config/local/database.php` configuration file that is configured to use the Homestead database out of the box, making Laravel initial installation and configuration more convenient.

The official documentation has also been updated to include [Homestead documentation](/docs/homestead).

### Laravel Cashier

Laravel Cashier is a simple, expressive library for managing subscription billing with Stripe. With the introduction of Laravel 4.2, we are including Cashier documentation along with the main Laravel documentation, though installation of the component itself is still optional. This release of Cashier brings numerous bug fixes, multi-currency support, and compatibility with the latest Stripe API.

### Daemon Queue Workers

The Artisan `queue:work` command now supports a `--daemon` option to start a worker in "daemon mode", meaning the worker will continue to process jobs without ever re-booting the framework. This results in a significant reduction in CPU usage at the cost of a slightly more complex application deployment process.

More information about daemon queue workers can be found in the [queue documentation](/docs/queues#daemon-queue-worker).

### Mail API Drivers

Laravel 4.2 introduces new Mailgun and Mandrill API drivers for the `Mail` functions. For many applications, this provides a faster and more reliable method of sending e-mails than the SMTP options. The new drivers utilize the Guzzle 4 HTTP library.

### Soft Deleting Traits

A much cleaner architecture for "soft deletes" and other "global scopes" has been introduced via PHP 5.4 traits. This new architecture allows for the easier construction of similar global traits, and a cleaner separation of concerns within the framework itself.

More information on the new `SoftDeletingTrait` may be found in the [Eloquent documentation](/docs/eloquent#soft-deleting).

### Convenient Auth & Remindable Traits

The default Laravel 4.2 installation now uses simple traits for including the needed properties for the authentication and password reminder user interfaces. This provides a much cleaner default `User` model file out of the box.

### "Simple Paginate"

A new `simplePaginate` method was added to the query and Eloquent builder which allows for more efficient queries when using simple "Next" and "Previous" links in your pagination view.

### Migration Confirmation

In production, destructive migration operations will now ask for confirmation. Commands may be forced to run without any prompts using the `--force` command.

<a name="laravel-4.1"></a>
## Laravel 4.1

### Full Change List

The full change list for this release by running the `php artisan changes` command from a 4.1 installation, or by [viewing the change file on Github](https://github.com/laravel/framework/blob/4.1/src/Illuminate/Foundation/changes.json). These notes only cover the major enhancements and changes for the release.

### New SSH Component

An entirely new `SSH` component has been introduced with this release. This feature allows you to easily SSH into remote servers and run commands. To learn more, consult the [SSH component documentation](/docs/ssh).

The new `php artisan tail` command utilizes the new SSH component. For more information, consult the `tail` [command documentation](http://laravel.com/docs/ssh#tailing-remote-logs).

### Boris In Tinker

The `php artisan tinker` command now utilizes the [Boris REPL](https://github.com/d11wtq/boris) if your system supports it. The `readline` and `pcntl` PHP extensions must be installed to use this feature. If you do not have these extensions, the shell from 4.0 will be used.

### Eloquent Improvements

A new `hasManyThrough` relationship has been added to Eloquent. To learn how to use it, consult the [Eloquent documentation](/docs/eloquent#has-many-through).

A new `whereHas` method has also been introduced to allow [retrieving models based on relationship constraints](/docs/eloquent#querying-relations).

### Database Read / Write Connections

Automatic handling of separate read / write connections is now available throughout the database layer, including the query builder and Eloquent. For more information, consult [the documentation](/docs/database#read-write-connections).

### Queue Priority

Queue priorities are now supported by passing a comma-delimited list to the `queue:listen` command.

### Failed Queue Job Handling

The queue facilities now include automatic handling of failed jobs when using the new `--tries` switch on `queue:listen`. More information on handling failed jobs can be found in the [queue documentation](/docs/queues#failed-jobs).

### Cache Tags

Cache "sections" have been superseded by "tags". Cache tags allow you to assign multiple "tags" to a cache item, and flush all items assigned to a single tag. More information on using cache tags may be found in the [cache documentation](/docs/cache#cache-tags).

### Flexible Password Reminders

The password reminder engine has been changed to provide greater developer flexibility when validating passwords, flashing status messages to the session, etc. For more information on using the enhanced password reminder engine, [consult the documentation](/docs/security#password-reminders-and-reset).

### Improved Routing Engine

Laravel 4.1 features a totally re-written routing layer. The API is the same; however, registering routes is a full 100% faster compared to 4.0. The entire engine has been greatly simplified, and the dependency on Symfony Routing has been minimized to the compiling of route expressions.

### Improved Session Engine

With this release, we're also introducing an entirely new session engine. Similar to the routing improvements, the new session layer is leaner and faster. We are no longer using Symfony's (and therefore PHP's) session handling facilities, and are using a custom solution that is simpler and easier to maintain.

### Doctrine DBAL

If you are using the `renameColumn` function in your migrations, you will need to add the `doctrine/dbal` dependency to your `composer.json` file. This package is no longer included in Laravel by default.
