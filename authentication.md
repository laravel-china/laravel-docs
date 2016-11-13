# 用户认证

- [简介](#introduction)
    - [数据库注意事项](#introduction-database-considerations)
- [认证快速入门](#authentication-quickstart)
    - [路由](#included-routing)
    - [视图](#included-views)
    - [认证](#included-authenticating)
    - [获取已认证之用户](#retrieving-the-authenticated-user)
    - [限制路由访问](#protecting-routes)
    - [登入限流](#login-throttling)
- [手动认证用户](#authenticating-users)
    - [记住用户](#remembering-users)
    - [其它认证方法](#other-authentication-methods)
- [HTTP 基础认证](#http-basic-authentication)
     - [无状态 HTTP 基础认证](#stateless-http-basic-authentication)
- [增加自定义 Guard](#adding-custom-guards)
- [增加自定义用户 Provider](#adding-custom-user-providers)
    - [用户 Provider Contract](#the-user-provider-contract)
    - [用户认证 Contract](#the-authenticatable-contract)
- [事件](#events)

<a name="introduction"></a>
## 简介

> {tip} **想要快速起步？** 在一个全新的 Laravel 应用中运行 `php artisan make:auth` 命令，然后可以用浏览器访问 `http://your-app.dev/register` 或者你在程序中定义的其他 url。这个简单的命令就可以搭建好整个认证系统的脚手架。

Laravel 中实现用户认证非常简单。实际上，几乎所有东西都已经为你配置好了。配置文件位于 `config/auth.php`，其中包含了用于调整认证服务行为的、标注好注释的选项配置。

在其核心代码中，Laravel 的认证组件由 `guards` 和 `providers` 组成，Guard 定义了用户在每个请求中如何实现认证，例如，Laravel 通过 `session` guard 来维护 Session 存储的状态和 Cookie。

Provider 定义了如何从持久化存储中获取用户信息，Laravel 底层支持通过 Eloquent 和数据库查询构建器两种方式来获取用户，如果需要的话，你还可以定义额外的 Provider。

如果看到这些名词觉得很困惑，大可不必太过担心，因为对绝大多数应用而言，只需使用默认认证配置即可，不需要做什么改动。

<a name="introduction-database-considerations"></a>
### 数据库注意事项

默认的 Laravel 在 `app` 文件夹中会含有 `App\User` [Eloquent 模型](/docs/{{version}}/eloquent)。这个模型将使用默认的 Eloquent 认证来驱动。如果你的应用程序没有使用 Eloquent，请选择使用 Laravel 查询构造器的 `database` 认证驱动。

为 `App\User` 模型创建数据库表结构时，确认密码字段最少必须 60 字符长。保持字段原定的 255 字符长是个好选择。

`users` 数据表中必须含有 nullable 、100 字符长的 `remember_token` 字段。当用户登录应用并勾选「记住我」时，这个字段将会被用来保存「记住我」 session 的令牌。

<a name="authentication-quickstart"></a>
## 认证快速入门

Laravel 带有两个认证控制器，它们被放置在 `App\Http\Controllers\Auth` 命名空间内，`RegisterController` 处理用户注册，`LoginController` 处理用户认证，`ForgotPasswordController` 处理重置密码的 e-mail 链接，`ResetPasswordController` 包含重置密码的逻辑。每个控制器都使用 trait 来包含必要的方法。对于大部分应用，都不必去修改这些控制器。
这些控制器使用了 trait 来包含所需要的方法，对于大多数的应用程序而言，你并不需要修改这些控制器。

<a name="included-routing"></a>
### 路由

Laravel 通过运行如下命令可快速生成认证所需要的路由和视图：

    php artisan make:auth

该命令应该在新安装的应用下使用，它会生成 layout 布局视图，注册和登录视图，以及所有的认证路由，同时生成 `HomeController` ，用来处理登录成功后会跳转到该控制器下的请求。

<a name="included-views"></a>
### 视图

正如上面所提到的，`php artisan make:auth` 命令会在 `resources/views/auth` 目录下创建所有认证需要的视图。

`make:auth` 命令还创建了 `resources/views/layouts` 目录，该目录下包含了应用的基础布局文件。所有这些视图都基于 Bootstrap CSS 框架，你也可以根据需要对其进行自定义。

<a name="included-authenticating"></a>
### 认证

现在你已经为自带的认证控制器设置好了路由和视图，接下来我们来实现新用户注册和登录认证。你可以在浏览器中访问定义好的路由，认证控制器已经（通过 trait）包含了注册及登录逻辑。

#### 自定义路径

当一个用户成功进行登录认证后，默认将会跳转到 `/home`，你可以通过在 `LoginController`，`RegisterController` 和 `ResetPasswordController` 中设置 `redirectTo` 属性来自定义登录认证成功之后的跳转路径：

    protected $redirectTo = '/';

当一个用户登录认证失败后，默认将会自动跳转回登录表单对应的页面。

#### 自定义用户名

Laravel默认使用 `email` 字段来认证。如果你想用其他字段认证，可以在 `LoginController` 里面定义一个 `username` 方法

public function username()
{
    return 'username';
}

#### 自定义 Guard

你还可以自定义实现用户认证的 「guard」，要实现这一功能，需要在 `LoginController`，`RegisterController` 和 `ResetPasswordController` 中定义 `guard` 方法，该方法需要返回一个 guard 实例：

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### 自定义验证 / 存储

要修改新用户注册所必需的表单字段，或者自定义新用户字段如何存储到数据库，你可以修改 `RegisterController` 类。该类负责为应用验证输入参数和创建新用户。

`RegisterController` 的 `validator` 方法包含了新用户的验证规则，你可以按需要自定义该方法。

`RegisterController` 的 `create` 方法负责使用 [Eloquent ORM](/docs/{{version}}/eloquent) 在数据库中创建新的 `App\User` 记录。当然，你也可以基于自己的需求自定义该方法。

<a name="retrieving-the-authenticated-user"></a>
### 获取已认证的用户信息

可以通过 `Auth` facade 来访问认证的用户。

    use Illuminate\Support\Facades\Auth;

    // 获取当前已通过认证的用户...
    $user = Auth::user();

    // 获取当前已通过认证的用户id...
    $id = Auth::id();
    
    

也有另外一种方法可以访问认证过的用户，就是通过 `Illuminate\Http\Request` 实例，请注意类型提示的类会被自动注入：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class ProfileController extends Controller
    {
        /**
         * Update the user's profile.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            if ($request->user()) {
                // $request->user() 返回认证过的用户的实例...
            }
        }
    }

#### 检查用户是否登录

使用 `Auth` facade 的 `check` 方法来检查用户是否登录，如果已经登录，将会返回 `true`：

    use Illuminate\Support\Facades\Auth;

    if (Auth::check()) {
        // 这个用户已经登录...
    }

> {tip} 尽管可以使用 `check` 方法来检查用户是否登录，在允许该用户访问特定的路由或控制器之前，可以使用中间件来检查用户是否认证过。要想得到更多信息，请阅读 [限制路由访问](/docs/{{version}}/authentication#protecting-routes) 的文档。

<a name="protecting-routes"></a>
### 限制路由访问

[路由中间件](/docs/{{version}}/middleware) 用于限定认证过的用户访问指定的路由，Laravel 提供了 `auth` 中间件来达到这个目的，而这个中间件被定义在 `app\Http\Middleware\Authenticate.php` 中。因为这个中间件已经在 HTTP kernel 中注册了，只需要将它应用到路由定义中即可使用：

    Route::get('profile', ['middleware' => 'auth', function() {
        // 只有认证过的用户能进来这里...
    }]);

如果使用 [控制器类](/docs/{{version}}/controllers)，可以在构造器中调用 `middleware` 方法，来代替在路由中直接定义：

    public function __construct()
    {
        $this->middleware('auth');
    }

#### 指定一个Guard

添加 `auth` 中间件到路由后，还需要指定使用哪个 guard 来实现认证。指定的 guard 对应配置文件 `auth.php` 中 `guards` 数组的某个键：

    public function __construct()
    {
        $this->middleware('auth:api');
    }

<a name="authentication-throttling"></a>
### 登录限流

Laravel 内置的 `LoginController` 类提供 `Illuminate\Foundation\Auth\ThrottlesLogins` trait 允许你在应用程序中限制登录次数。默认情况下，如果用户在进行几次尝试后仍不能提供正确的凭证，将在一分钟内无法进行登录。这个限制会特别针对用户的用户名称 / 邮件地址和他们的 IP 地址。

<a name="authenticating-users"></a>
## 手动认证用户

当然，不一定要使用 Laravel 内置的认证控制器。如果选择删除这些控制器，可以直接调用 Laravel 的认证类来实现用户认证管理。不用担心，很简单。

我们可以利用 `Auth` [facade](/docs/{{version}}/facades) 来访问 Laravel 的认证服务，因此需要确认在类的顶部导入 `Auth` facade。接下来让我们看一下 `Auth` 的 `attempt` 方法：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Auth;

    class AuthController extends Controller
    {
        /**
         * Handle an authentication attempt.
         *
         * @return Response
         */
        public function authenticate()
        {
            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                // Authentication passed...
                return redirect()->intended('dashboard');
            }
        }
    }

`attempt` 方法会接受一个数组来作为第一个参数，这个数组的值可用来寻找数据库里的用户数据，所以在上面的例子中，用户通过 `email` 字段被取出，如果用户被找到了，数据库里经过哈希的密码将会与数组中哈希的 `password` 值比对，如果两个值一样的话就会开启一个通过认证的 session 给用户。

如果认证成功，`attempt` 方法将会返回 `true`，反之则为 `false`。

重定向器上的 `intended` 方法将会重定向用户回原本想要进入的页面，也可以传入一个回退 URI 至这个方法，以避免要转回的页面不可使用。

#### 指定额外条件

可以加入除用户的邮箱及密码外的额外条件进行认证查找。例如，我们要确认用户是否被标示为 `active`：

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }

> {note} 在这些例子中，`email` 不是一个一定要有的选项，它仅仅是被用来当作例子，你可以用任何字段，只要它在数据库的意义等同于「用户名」。

#### 访问指定 Guard 实例

可以通过 `Auth` facade 的 `guard` 方法来指定使用特定的 guard 实例。这样可以实现在应用不同部分管理用户认证时使用完全不同的认证模型或者用户表。

传递给 `guard` 方法 guard 名称必须是 `auth.php` 配置文件中 guards 的值之一：

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### 注销用户

要想让用户注销，你可以使用 `Auth` facade 的 `logout` 方法。这个方法会清除所有认证后加入到用户 session 的数据：

    Auth::logout();

<a name="remembering-users"></a>
## 记住用户

如果你想要提供「记住我」的功能，你需要传入一个布尔值到 `attempt` 方法的第二个参数，在用户注销前 session 值都会被一直保存。`users` 数据表一定要包含一个 `remember_token` 字段，这是用来保存「记住我」令牌的。

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // 这个用户被记住了...
    }

> {tip} 如果使用 Laravel 内置的 `LoginController`，合适的「记住我」逻辑已经通过 traits 实现。

可以使用 `viaRemember` 方法来检查这个用户是否使用「记住我」 cookie 来做认证：

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### 其它认证方法

#### 用「用户实例」做认证

如果你需要使用存在的用户实例来登录，你需要调用 `login` 方法，并传入使用实例，这个对象必须是由 `Illuminate\Contracts\Auth\Authenticatable` [contract](/docs/{{version}}/contracts) 所实现。当然，`App/User` 模型已经实现了这个接口：

    Auth::login($user);

    // 登录并且「记住」用户
    Auth::login($user, true);

你也可以指定 guard 实例：

    Auth::guard('admin')->login($user);

#### 用用户 ID 做认证

使用 `loginUsingId` 方法来登录指定 ID 用户，这个方法接受要登录用户的主键：

    Auth::loginUsingId(1);

    // 登录并且「记住」用户
    Auth::loginUsingId(1, true);

#### 仅在本次认证用户

可以使用 `once` 方法来针对一次性认证用户，没有任何的 session 或 cookie 会被使用，这个对于构建无状态的 API 非常的有用，`once` 方法跟 `attempt` 方法拥有同样的传入参数：

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP 基础认证

[HTTP 基础认证](http://en.wikipedia.org/wiki/Basic_access_authentication) 提供一个快速的方法来认证用户，不需要任何「登录」页面。开始之前，先增加 `auth.basic` [中间件](/docs/{{version}}/middleware) 到你的路由，`auth.basic` 中间件已经被包含在 Laravel 框架中，所以你不需要定义它：

    Route::get('profile', function() {
        // 只有认证过的用户可进入...
    })->middleware('auth.basic');

一旦中间件被增加到路由上，当使用浏览器进入这个路由时，将自动的被提示需要提供凭证。默认情况下，`auth.basic` 中间件将会使用用户的 `email` 字段当作「用户名」。

#### FastCGI 的注意事项

如果是正在使用 FastCGI，则 HTTP 的基础认证可能无法正常运作，你需要将下面这几行加入你 `.htaccess` 文件中：

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### 无状态 HTTP 基础认证

你可以使用 HTTP 基础认证而不用在 session 中设置用户认证用的 cookie，这个功能对 API 认证来说非常有用。为了达到这个目的，[定义一个中间件](/docs/{{version}}/middleware) 并调用 `onceBasic` 方法。如果从 `onceBasic` 方法没有返回任何响应的话，这个请求会直接传进应用程序中：

    <?php

    namespace Illuminate\Auth\Middleware;

    use Illuminate\Support\Facades\Auth;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

接着，[注册这个路由中间件](/docs/{{version}}/middleware#registering-middleware)，然后将它增加在一个路由上：

    Route::get('api/user', function() {
        // 只有认证过的用户可以进入...
    })->middleware('auth.basic.once');

## 增加自定义的 Guard

你可以使用 `Auth` 的 `extend` 方法来自定义认证 Guard，你需要在 [服务提供者](/docs/{{version}}/providers) 中放置此代码调用。因为 Laravel 已经提供 `AuthServiceProvider`，可以把代码放入其中：

    <?php

    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
    use Illuminate\Support\Facades\Auth;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::extend('jwt', function($app, $name, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\Guard...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }

正如上面的代码所示，`extend` 方法传参进去的回调需要返回 `Illuminate\Contracts\Auth\Guard` 的实现，这个接口类有几个方法你需要实现。定制好 Guard 以后，你需要在配置信息中开启使用：

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="adding-custom-user-providers"></a>
## 添加自定义用户提供者

如果你没有使用传统的关系型数据库存储用户信息，则需要使用自己的认证用户提供者来扩展 Laravel。我们使用 `Auth` facade 上的 `provider` 方法定义自定义该提供者：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::provider('riak', function($app, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...

                return new RiakUserProvider($app->make('riak.connection'));
            });
        }
    }

通过 `provider` 方法注册用户提供者后，你可以在配置文件 `config/auth.php` 中切换到新的用户提供者。首先，在该配置文件定义一个使用新驱动的 `providers` 数组：

    'providers' => [
        'users' => [
            'driver' => 'riak',
        ],
    ],

然后，可以在你的 `guards` 配置中使用这个提供者：

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

### UserProvider 契约

`Illuminate\Contracts\Auth\UserProvider` 的实现只负责获取 `Illuminate\Contracts\Auth\Authenticatable` 的实现， 且不受限于永久保存系统，例如 MySQL, Riak 等等。这两个接口允许 Laravel 认证机制继续作用，而不用管用户如何保存或是使用什么样类型的类实现它。

让我们来看看 `Illuminate\Contracts\Auth\UserProvider` contract：

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

`retrieveById` 函数通常获取一个代表用户的值，例如 MySQL 中自增的 ID。`Authenticatable` 的实现通过 ID 匹配的方法来取出和返回。

`retrieveByToken` 函数借助用户唯一的 `$identifier` 和「记住我」`$token` 来获取用户。如同之前的方法，`Authenticatable` 的实现应该被返回。

`updateRememberToken` 方法使用新的 `$token` 更新了 `$user` 的 `remember_token` 字段。这个新的令牌可以是全新的令牌（当使用「记住我」尝试登录成功时），或是 null（当用户注销时）。

`retrieveByCredentials` 方法获取了从 `Auth::attempt` 方法发送过来的凭证数组（当想要登录时）。这个方法应该要 「查找」所使用的持久化存储系统来匹配这些凭证。通常，这个方法会运行一个带着「where」`$credentials['username']` 条件的查找。这个方法接着需要返回一个 `UserInterface` 的实现。**此方法不应该企图做任何密码验证或认证操作。**

`validateCredentials` 方法需要比较 `$user` 和 `$credentials` 来认证这个用户。例如，这个方法可能会比较 `$user->getAuthPassword()` 字符串及 `Hash::make` 后的 `$credentials['password']`。这个方法应该只验证用户的凭证并返回一个布尔值。

### 可验证之 Contract

现在我们已经介绍了 `UserProvider` 的每个方法，让我们看一下 `Authenticate` contract。这个提供者需要 `retrieveById` 和 `retrieveByCredentials` 方法来返回这个接口的实现：

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }


这个接口很简单。`getAuthIdentifierName` 方法需要返回「主键名字」。`getAuthIdentifier` 方法需要返回用户的「主键」。在 MySQL 中，这个主键是指自动增加的主键。而 `getAuthPassword` 应该要返回用户哈希后的密码。这个接口允许认证系统和任何用户类运作，不用管你在使用何种 ORM 或存储抽象层。默认情况下，Laravel 的 `app` 文件夹中会包含 `User` 类来实现此接口，所以你可以观察这个类以作为实现的例子。

<a name="events"></a>
## 事件

Laravel 提供了在认证过程中的各种 [事件](/docs/{{version}}/events)。你可以在 `EventServiceProvider` 中对这些事件做监听：

    /**
     * 为应用程序的事件监听器的映射
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],

        'Illuminate\Auth\Events\Authenticated' => [
            'App\Listeners\LogAuthenticated',
        ],

        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],

        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],

        'Illuminate\Auth\Events\Lockout' => [
            'App\Listeners\LogLockout',
        ],
    ];
