# 用户认证

- [介绍](#introduction)
    - [数据库注意事项](#introduction-database-considerations)
- [认证快速入门](#authentication-quickstart)
    - [路由](#included-routing)
    - [视图](#included-views)
    - [认证](#included-authenticating)
    - [获取已认证之用户](#retrieving-the-authenticated-user)
    - [限制路由访问](#protecting-routes)
    - [错误尝试限制](#authentication-throttling)
- [手动认证用户](#authenticating-users)
    - [记住用户](#remembering-users)
    - [其它认证方法](#other-authentication-methods)
- [HTTP 基础认证](#http-basic-authentication)
     - [无状态 HTTP 基础认证](#stateless-http-basic-authentication)
- [重设密码](#resetting-passwords)
    - [重设数据库](#resetting-database)
    - [路由](#resetting-routing)
    - [视图](#resetting-views)
    - [重设密码后](#after-resetting-passwords)
- [社会化认证](#social-authentication)
- [添加自定义认证驱动](#adding-custom-authentication-drivers)
- [事件](#events)

<a name="introduction"></a>
## 介绍

Laravel 让用户认证变得非常简单。几乎所有的认证行为都可以通过配置信息 `config/auth.php` 来控制。

<a name="introduction-database-considerations"></a>
### 数据库注意事项

默认的 Laravel 在 `app` 文件夹中会含有 `App\User` [Eloquent 模型](/docs/{{version}}/eloquent)。这个模型将使用默认的 Eloquent 认证来驱动。如果你的应用程序没有使用 Eloquent，请选择使用 Laravel 查询构造器的 `database` 认证驱动。

为 `App\User` 模型创建数据库表结构时，确认密码字段最少有 60 字符长。

`users` 数据表中必须含有 nullable 、100 字符长的 `remember_token` 字段，这个字段将会被用来保存「记住我」 session 的令牌。只要在创建迁移时，使用 `$table->rememberToken()`，即可轻松加入这个字段。

<a name="authentication-quickstart"></a>
## 认证快速入门

Laravel 带有两个认证控制器，它们被放置在 `App\Http\Controllers\Auth` 命名空间内，`AuthController` 处理用户注册及认证，而 `PasswordController` 负责处理重置用户的密码。

这些控制器使用了 trait 来包含所需要的方法，对于大多数的应用程序而言，你并不需要修改这些控制器。

<a name="included-routing"></a>
### 路由

默认没有 [路由](/docs/{{version}}/routing) 指向这些认证控制器，需要自己添加到 `app/Http/routes.php` 中。

    // 认证路由...
    Route::get('auth/login', 'Auth\AuthController@getLogin');
    Route::post('auth/login', 'Auth\AuthController@postLogin');
    Route::get('auth/logout', 'Auth\AuthController@getLogout');

    // 注册路由...
    Route::get('auth/register', 'Auth\AuthController@getRegister');
    Route::post('auth/register', 'Auth\AuthController@postRegister');

<a name="included-views"></a>
### 视图

虽然这些认证控制器被包含在框架中，你仍需提供 [视图](/docs/{{version}}/views) 给控制器渲染，而视图需要被放置在 `resources/views/auth` 文件夹中，你可以任意的自定义这些视图。

登录视图应该被放在 `resources/views/auth/login.blade.php` 而注册视图则放在 `resources/views/auth/register.blade.php`。

#### 认证表单例子

    <!-- resources/views/auth/login.blade.php -->

    <form method="POST" action="/auth/login">
        {!! csrf_field() !!}

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            Password
            <input type="password" name="password" id="password">
        </div>

        <div>
            <input type="checkbox" name="remember"> Remember Me
        </div>

        <div>
            <button type="submit">Login</button>
        </div>
    </form>

#### 注册表单例子

    <!-- resources/views/auth/register.blade.php -->

    <form method="POST" action="/auth/register">
        {!! csrf_field() !!}

        <div>
            Name
            <input type="text" name="name" value="{{ old('name') }}">
        </div>

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            Password
            <input type="password" name="password">
        </div>

        <div>
            Confirm Password
            <input type="password" name="password_confirmation">
        </div>

        <div>
            <button type="submit">Register</button>
        </div>
    </form>

<a name="included-authenticating"></a>
### 认证

现在你已经为认证控制器设置好了路由及视图，你可以在应用程序注册并认证新用户了。

只需要在浏览器简单地访问定义的路由就行了，认证控制器早已包含了处理认证现有用户，及保存新用户在数据库的逻辑了（通过他们各自的 traits）。

当用户成功认证后，他们将被重定向到 `/home` URI，而你需要向路由注册此 URI 来处理这个请求，也可以自定义认证后转向的 URI，只需修改 `AuthController` 的 `redirectPath` 属性即可：

    protected $redirectPath = '/dashboard';

用户认证失败后，将会被重定向到 `/auth/login` URI。你可以设置 `AuthController` 的 `loginPath` 属性来自定义认证失败后的重定向位置：

    protected $loginPath = '/login';

`loginPath` 并不会改变当用户访问受保护的路由时所重定向的路径。该路径是由 `App\Http\Middleware\Authenticate` 中间件的 `handle` 方法所控制的。

#### 自定义表单字段

如果想要修改注册时的表单字段，或是自定义如何将新用户的记录写入数据库，你可以修改 `AuthController` 类，这个类负责验证和创建新的用户。

`AuthController` 的 `validator` 方法包含了对新用户的验证规则。你可以随意的修改这个方法。

`AuthController` 的 `create` 方法负责使用 [Eloquent ORM](/docs/{{version}}/eloquent) 来创建新的 `App\User` 纪录到数据库。你可以根据需求任意修改这个方法。

<a name="retrieving-the-authenticated-user"></a>
### 获取已认证的用户信息

可以通过 `Auth` facade 来访问认证的用户。

    $user = Auth::user();

也有另外一种方法可以访问认证过的用户，就是通过 `Illuminate\Http\Request` 实例：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class ProfileController extends Controller
    {
        /**
         * 更新用户的数据
         *
         * @param  Request  $request
         * @return Response
         */
        public function updateProfile(Request $request)
        {
            if ($request->user()) {
                // $request->user() 返回认证过的用户的实例...
            }
        }
    }

#### 检查用户是否登录

使用 `Auth` facade 的 `check` 方法来检查用户是否登录，如果已经登录，将会返回 `true`：

    if (Auth::check()) {
        // 这个用户已经登录...
    }

在允许该用户访问特定的路由或控制器之前，可以使用中间件来检查用户是否认证过。要想得到更多信息，请阅读 [限制路由访问](/docs/{{version}}/authentication#protecting-routes) 的文档。

<a name="protecting-routes"></a>
### 限制路由访问

[路由中间件](/docs/{{version}}/middleware) 用于限定认证过的用户访问指定的路由，Laravel 提供了 `auth` 中间件来达到这个目的，而这个中间件被定义在 `app\Http\Middleware\Authenticate.php` 中，只需要将它应用到路由定义中：

    // 使用路由闭包...
    Route::get('profile', ['middleware' => 'auth', function() {
        // 只有认证过的用户能进来这里...
    }]);

    // 使用控制器...
    Route::get('profile', [
        'middleware' => 'auth',
        'uses' => 'ProfileController@show'
    ]);

如果使用 [控制器类](/docs/{{version}}/controllers)，可以在构造器中调用 `middleware` 方法：

    public function __construct()
    {
        // 执行 auth 认证
        $this->middleware('auth');
    }

<a name="authentication-throttling"></a>
### 错误尝试限制

Laravel 内置的 `AuthController` 类提供 `Illuminate\Foundation\Auth\ThrottlesLogins` trait 允许你在应用程序中限制登录次数。

默认情况下，如果用户在进行几次尝试后仍不能提供正确的凭证，将在一分钟内无法进行登录。这个限制会特别针对用户的用户名称 / 邮件地址和他们的 IP 地址：

    <?php

    namespace App\Http\Controllers\Auth;

    use App\User;
    use Validator;
    use App\Http\Controllers\Controller;
    use Illuminate\Foundation\Auth\ThrottlesLogins;
    use Illuminate\Foundation\Auth\AuthenticatesAndRegistersUsers;

    class AuthController extends Controller
    {
        use AuthenticatesAndRegistersUsers, ThrottlesLogins;

        // 重置 AuthController 类...
    }

<a name="authenticating-users"></a>
## 手动认证用户

当然，不一定要使用 Laravel 内置的认证控制器，你可以选择删除这些控制器，然后创建自定义的控制器。

我们将通过 `Auth` [facade](/docs/{{version}}/facades) 访问 Laravel 的认证服务，接下来让我们看一下 `Auth` 的 `attempt` 方法：

    <?php

    namespace App\Http\Controllers;

    use Auth;
    use Illuminate\Routing\Controller;

    class AuthController extends Controller
    {
        /**
         * 处理认证
         *
         * @return Response
         */
        public function authenticate()
        {
            // 尝试登录
            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                // 认证通过...
                return redirect()->intended('dashboard');
            }
        }
    }

`attempt` 方法会接受一个数组来作为第一个参数，这个数组的值可用来寻找数据库里的用户数据，所以在上面的例子中，用户通过 `email` 字段被取回，如果用户被找到了，数据库里经过哈希的密码将会与数组中哈希的 `password` 值比对，如果两个值一样的话就会开启一个通过认证的 session 给用户。

如果认证成功，`attempt` 方法将会返回 `true`，反之则为 `false`。

重定向器上的 `intended` 方法将会重定向用户回原本想要进入的页面，也可以传入一个回退 URI 至这个方法，以避免要转回的页面不可使用。

可以加入除用户的邮箱及密码外的额外条件进行认证查找。例如，我们要确认用户是否被标示为 `active`：

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // 此用户是已激活的、没被停权的、且存在的用户
    }

要想让用户注销，你可以使用 `Auth` facade 的 `logout` 方法。这个方法会清除所有认证后加入到用户 session 的数据：

    Auth::logout();

> **注意：** 在这些例子中，`email` 不是一个一定要有的选项，它仅仅是被用来当作例子，你可以用任何字段，如「手机号码」，只要它在数据库的意义等同于「用户名」。

<a name="remembering-users"></a>
## 记住用户

如果你想要提供「记住我」的功能，你需要传入一个布尔值到 `attempt` 方法的第二个参数，在用户注销前 session 值都会被一直保存。

`users` 数据表一定要包含一个 `remember_token` 字段，这是用来保存「记住我」令牌的。

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // 这个用户被记住了...
    }

可以使用 `viaRemember` 方法来检查这个用户是否使用「记住我」 cookie 来做认证：

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### 其它认证方法

#### 用「用户实例」做认证

如果你需要使用存在的用户实例来登录，你需要调用 `login` 方法，并传入使用实例，这个对象必须是由 `Illuminate\Contracts\Auth\Authenticatable` [contract](/docs/{{version}}/contracts) 所实现。当然，`App/User` 模型已经实现了这个接口：

    Auth::login($user);

#### 用用户 ID 做认证

使用 `loginUsingId` 方法来登录指定 ID 用户，这个方法接受要登录用户的主键：

    Auth::loginUsingId(1);

#### 仅在本次认证用户

可以使用 `once` 方法来针对一次性认证用户，没有任何的 session 或 cookie 会被使用，这个对于构建无状态的 API 非常的有用，`once` 方法跟 `attempt` 方法拥有同样的传入参数：

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP 基础认证

[HTTP 基础认证](http://en.wikipedia.org/wiki/Basic_access_authentication) 提供一个快速的方法来认证用户，不需要任何「登录」页面。开始之前，先增加 `auth.basic` [中间件](/docs/{{version}}/middleware)到你的路由，`auth.basic` 中间件已经被包含在 Laravel 框架中，所以你不需要定义它：

    Route::get('profile', ['middleware' => 'auth.basic', function() {
        // 只有认证过的用户可进入...
    }]);

一旦中间件被增加到路由上，当使用浏览器进入这个路由时，你将自动的被提示需要提供凭证。默认情况下，`auth.basic` 中间件将会使用用户的 `email` 字段当作「用户名」。

#### FastCGI 的注意事项

如果是正在使用 FastCGI，则 HTTP 的基础认证可能无法正常运作，你需要将下面这几行加入你 `.htaccess` 文件中：

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### 无状态 HTTP 基础认证

你可以使用 HTTP 基础认证而不用在 session 中设置用户认证用的 cookie，这个功能对 API 认证来说非常有用。为了达到这个目的，[定义一个中间件](/docs/{{version}}/middleware) 并调用 `onceBasic` 方法。如果从 `onceBasic` 方法没有返回任何响应的话，这个请求会直接传进应用程序中：

    <?php

    namespace Illuminate\Auth\Middleware;

    use Auth;
    use Closure;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * 处理请求
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

接着，[注册这个路由中间件](/docs/{{version}}/middleware#registering-middleware)，然后将它增加在一个路由上：

    Route::get('api/user', ['middleware' => 'auth.basic.once', function() {
        // 只有认证过的用户可以进入...
    }]);

<a name="resetting-passwords"></a>
## 重设密码

<a name="resetting-database"></a>
### 重设数据库

很多 Web 应用程序都会提供用户重设密码功能，Laravel 提供发送重置密码邮件和实现密码重设功能，而避免让你重造车轮。

开始之前，请先确认 `App\User` 模型实现了 `Illuminate\Contracts\Auth\CanResetPassword` contract。当然，原有的 `App\User` 早已实现了这个接口，并且使用 `Illuminate\Auth\Passwords\CanResetPassword` trait 引入实现这个接口所需要的方法。

#### 生成重置令牌的数据表迁移文件

接下来，必须要创建一个用来保存密码重置令牌的数据表，而这个数据表的迁移已经包含在 Laravel 中了，就在 `database/migrations` 文件夹里。所以，你要做的就是做一次迁移：

    php artisan migrate

<a name="resetting-routing"></a>
### 路由

Laravel 包含了 `Auth\PasswordController`，虽然它含有所有重置用户密码的逻辑，但是你仍需要指定路由到这个控制器上：

    // 密码重置链接的路由...
    Route::get('password/email', 'Auth\PasswordController@getEmail');
    Route::post('password/email', 'Auth\PasswordController@postEmail');

    // 密码重置的路由...
    Route::get('password/reset/{token}', 'Auth\PasswordController@getReset');
    Route::post('password/reset', 'Auth\PasswordController@postReset');

<a name="resetting-views"></a>
### 视图

除了定义 `PasswordController` 路由，还需要提供视图给这个控制器。不过不用担心，我们已经提供了例子视图来帮助你开始，你也可以随意的定义表单风格。

#### 重置密码链接的请求表单例子

你需要提供 HTML 视图给密码重置的请求表单。这些视图被放在 `resources/views/auth/password.blade.php`。这个表单提供了单个的字段来给用户输入邮箱，让他们可以收到密码重置链接：

    <!-- resources/views/auth/password.blade.php -->

    <form method="POST" action="/password/email">
        {!! csrf_field() !!}

        @if (count($errors) > 0)
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        @endif

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            <button type="submit">
                发送重置密码邮件
            </button>
        </div>
    </form>

当用户送出重置密码的请求，他们会收到一封有链接到 `PasswordController` 的 `getReset` 方法（通常是路由到 `/password/reset`）的邮箱。

你将需要为邮箱创造一个 `resources/views/emails/password.blade.php` 视图。这个视图会接收一个带有密码重置令牌的 `$token` 变量，这个变量含有密码重置令牌来匹配用户的密码重置请求，举例如下：

    <!-- 文件 resources/views/emails/password.blade.php -->

    点击此处重置你的密码：{{ url('password/reset/'.$token) }}

#### 密码重置表单的例子

当用户点击了邮箱的链接来重置密码，将会显示一个密码重置表单，这个视图被放在 `resources/views/auth/reset.blade.php`。

这里有个密码重置表单的例子：

    <!-- resources/views/auth/reset.blade.php -->

    <form method="POST" action="/password/reset">
        {!! csrf_field() !!}
        <input type="hidden" name="token" value="{{ $token }}">

        @if (count($errors) > 0)
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        @endif

        <div>
            电子邮箱
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            密码
            <input type="password" name="password">
        </div>

        <div>
            重新输入密码
            <input type="password" name="password_confirmation">
        </div>

        <div>
            <button type="submit">
                重置密码
            </button>
        </div>
    </form>

<a name="after-resetting-passwords"></a>
### 重设密码后

在你定义好路由跟视图之后，你就可以使用浏览器访问这个路由了。

`PasswordController` 已经包含了发送密码重置链接邮箱，及更新密码到数据库的逻辑。

密码重置以后，这个用户会自动登录并重定向到 `/home`。要自定义重定向地址，只需定义 `PasswordController` 的 `redirectTo` 属性即可：

    protected $redirectTo = '/dashboard';

> **注意：** 默认情况下，密码重置令牌会在一个小时后过期，你可以更改 `config/auth.php` 的 `reminder.expire` 选项，来修改这个设置。

<a name="social-authentication"></a>
## 社会化认证

除了传统的表单认证之外，Laravel 同样提供了简单方便的方法来认证 OAuth 提供者，这个方法使用了  [Laravel Socialite](https://github.com/laravel/socialite)。Socialite 目前支持 Facebook、Twitter、LinkedIn、Google、GitHub 跟 Bitbucket 的认证。

开始使用 Socialite 前，请先添加依赖包至你的 `composer.json`：

    composer require laravel/socialite

### 配置信息

安装 Socialite 之后，到 `config/app.php` 配置文件中注册 `Laravel\Socialite\SocialiteServiceProvider`：

    'providers' => [
        // 其它服务提供者...

        Laravel\Socialite\SocialiteServiceProvider::class,
    ],

此外再添加 `Socialite` facade 到 `app` 配置文件的 `aliases` 数组中：

    'Socialite' => Laravel\Socialite\Facades\Socialite::class,

你将需要添加凭证来使用 OAuth 服务，这些凭证需要被放在 `config/services.php` 配置文件中，并根据你应用程序的需求，增加 `facebook`、`twitter`、`linkedin`、`google`、`github` 或 `bitbucket` 的键，例如：

    'github' => [
        'client_id' => 'your-github-app-id',
        'client_secret' => 'your-github-app-secret',
        'redirect' => 'http://your-callback-url',
    ],

### 基础应用

接下来，你已经准备好开始认证用户了！需要两个路由: 一个用来重定向用户到 OAuth 提供者，另一个在认证后接收提供者的回调。我们将会借由 `Socialite` [facade](/docs/{{version}}/facades) 访问 Socialite：

    <?php

    namespace App\Http\Controllers;

    use Socialite;
    use Illuminate\Routing\Controller;

    class AuthController extends Controller
    {
        /**
         * 重定向用户到 GitHub 认证页。
         *
         * @return Response
         */
        public function redirectToProvider()
        {
            return Socialite::driver('github')->redirect();
        }

        /**
         * 从 GitHub 得到用户信息
         *
         * @return Response
         */
        public function handleProviderCallback()
        {
            $user = Socialite::driver('github')->user();

            // $user->token;
        }
    }

`redirect` 方法会负责处理并发送用户到 OAuth 提供者，而 `user` 方法会从提供者返回的请求来获取用户信息。在重定向用户之前，你也可以使用 `scopes` 方法来设置请求的 「作用域」。这个方法将重写所有已经存在的作用域：

    return Socialite::driver('github')
                ->scopes(['scope1', 'scope2'])->redirect();

当然，你需要指定路由到控制器方法：

    Route::get('auth/github', 'Auth\AuthController@redirectToProvider');
    Route::get('auth/github/callback', 'Auth\AuthController@handleProviderCallback');

一些 OAuth 提供者支持在重定向的请求中自定义参数。若要在请求中加入任何自定义参数，只要调用 `with` 方法并带上一个关联数组：

    return Socialite::driver('google')
                ->with(['hd' => 'example.com'])->redirect();

#### 获取用户详情信息

一旦你有了用户的实例，你就可以获取用户更详细的信息:

    $user = Socialite::driver('github')->user();

    // OAuth Two 提供者
    $token = $user->token;

    // OAuth One 提供者
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // 所有提供者
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();

<a name="adding-custom-authentication-drivers"></a>
## 添加自定义的认证驱动

如果你不是使用传统的关系型数据库来保存用户，你将需要扩充 Laravel 来添加你自己的认证驱动。我们将使用 `Auth` facade 的 `extend` 方法来定义自定义驱动。你应该将 `extend` 放置在 [服务提供者](/docs/{{version}}/providers) 中：

    <?php

    namespace App\Providers;

    use Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * 运行服务注册后的启动进程。
         *
         * @return void
         */
        public function boot()
        {
            Auth::extend('riak', function($app) {
                // 返回 Illuminate\Contracts\Auth\UserProvider 的实例...
                return new RiakUserProvider($app['riak.connection']);
            });
        }

        /**
         * 在容器注册绑定。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

在你用 `extend` 方法注册这个驱动后，你可以在 `config/auth.php` 转换到新的驱动。

### 用户提供者 Contract

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

`retrieveById` 函数通常获取一个代表用户的值，例如 MySQL 中自增的 ID。`Authenticatable` 的实现通过 ID 匹配的方法来取回和返回。

`retrieveByToken` 函数借助用户唯一的 `$identifier` 和「记住我」`$token` 来获取用户。如同之前的方法，`Authenticatable` 的实现应该被返回。

`updateRememberToken` 方法使用新的 `$token` 更新了 `$user` 的 `remember_token` 字段。这个新的令牌可以是全新的令牌（当使用「记住我」尝试登录成功时），或是 null（当用户注销时）。

`retrieveByCredentials` 方法获取了从 `Auth::attempt` 方法发送过来的凭证数组（当想要登录时）。这个方法应该要 「查找」所使用的持久化存储系统来匹配这些凭证。通常，这个方法会运行一个带着「where」`$credentials['username']` 条件的查找。这个方法接着需要返回一个 `UserInterface` 的实现。**此方法不应该企图做任何密码验证或认证操作。**

`validateCredentials` 方法需要比较 `$user` 和 `$credentials` 来认证这个用户。例如，这个方法可能会比较 `$user->getAuthPassword()` 字符串及 `Hash::make` 后的 `$credentials['password']`。这个方法应该只验证用户的凭证并返回一个布尔值。

### 可验证之 Contract

现在我们已经介绍了 `UserProvider` 的每个方法，让我们看一下 `Authenticate` contract。这个提供者需要 `retrieveById` 和 `retrieveByCredentials` 方法来返回这个接口的实现：

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

这个接口很简单。`getAuthIdentifier` 方法需要返回用户的「主键」。在 MySQL 中，这个主键是指自动增加的主键。而 `getAuthPassword` 应该要返回用户哈希后的密码。这个接口允许认证系统和任何用户类运作，不用管你在使用何种 ORM 或存储抽象层。默认情况下，Laravel 的 `app` 文件夹中会包含 `User` 类来实现此接口，所以你可以观察这个类以作为实现的例子。

<a name="events"></a>
## 事件

Laravel 提供了在认证过程中的各种 [事件](/docs/{{version}}/events)。你可以在 `EventServiceProvider` 中对这些事件做监听：

    /**
     * 为你的应用程序注册任何事件。
     *
     * @param  \Illuminate\Contracts\Events\Dispatcher  $events
     * @return void
     */
    public function boot(DispatcherContract $events)
    {
        parent::boot($events);

        // 在每次尝试认证时触发...
        $events->listen('auth.attempt', function ($credentials, $remember, $login) {
            //
        });

        // 登录成功时触发...
        $events->listen('auth.login', function ($user, $remember) {
            //
        });

        // 注销时触发...
        $events->listen('auth.logout', function ($user) {
            //
        });
    }


