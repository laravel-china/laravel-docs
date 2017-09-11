# Laravel 的 API 认证系统 Passport

- [介绍](#introduction)
- [安装](#installation)
    - [前端快速上手](#frontend-quickstart)
    - [部署 Passport](#deploying-passport)
- [配置](#configuration)
    - [令牌的使用期限](#token-lifetimes)
- [发放访问令牌](#issuing-access-tokens)
    - [管理客户端](#managing-clients)
    - [请求令牌](#requesting-tokens)
    - [刷新令牌](#refreshing-tokens)
- [密码授权令牌](#password-grant-tokens)
    - [创建密码授权客户端](#creating-a-password-grant-client)
    - [请求密码授权令牌](#requesting-password-grant-tokens)
    - [请求所有作用域](#requesting-all-scopes)
- [简化授权令牌](#implicit-grant-tokens)
- [客户端授权令牌](#client-credentials-grant-tokens)
- [私人访问令牌](#personal-access-tokens)
    - [创建私人访问令牌的客户端](#creating-a-personal-access-client)
    - [管理私人访问令牌](#managing-personal-access-tokens)
- [路由保护](#protecting-routes)
    - [通过中间件](#via-middleware)
    - [传递访问令牌](#passing-the-access-token)
- [令牌作用域](#token-scopes)
    - [定义作用域](#defining-scopes)
    - [给令牌分派作用域](#assigning-scopes-to-tokens)
    - [检查作用域](#checking-scopes)
- [使用 JavaScript 接入 API](#consuming-your-api-with-javascript)
- [事件](#events)
- [测试](#testing)

<a name="introduction"></a>
## 介绍

在 Laravel 中，实现基于传统表单的登陆和授权已经非常简单，但是如何满足 API 场景下的授权需求呢？在 API 场景里通常通过令牌来实现用户授权，而非维护请求之间的 Session 状态。现在 Laravel 项目中可以使用 Passport 轻而易举地实现 API 授权过程，通过 Passport 可以在几分钟之内为你的应用程序添加完整的 OAuth2 服务端实现。 Passport 基于 [League OAuth2 server](https://github.com/thephpleague/oauth2-server) 该项目的维护人是 [Alex Bilbie](https://github.com/alexbilbie)。

> {note} 本文档假定你已熟悉 OAuth2 。如果你并不了解 OAuth2 ，阅读之前请先熟悉下 OAuth2 的常用术语和基本特征。

<a name="installation"></a>
## 安装

使用 Composer 依赖包管理器安装 Passport ：

    composer require laravel/passport

接下来，将 Passport 的服务提供者注册到配置文件 `config/app.php` 的 `providers` 数组中：

    Laravel\Passport\PassportServiceProvider::class,

Passport 使用服务提供者注册内部的数据库迁移脚本目录，所以上一步完成后，你需要更新你的数据库结构。Passport 的迁移脚本会自动创建应用程序需要的客户端数据表和令牌数据表：

    php artisan migrate

> {note} 如果你不打算使用 Passport 的默认迁移，你应该在 `AppServiceProvider` 的 `register` 方法中调用 `Passport :: ignoreMigrations` 方法。 你可以导出这个默认迁移用 `php artisan vendor:publish --tag=passport-migrations` 命令。

接下来，你需要运行 `passport:install` 命令来创建生成安全访问令牌时用到的加密密钥，同时，这条命令也会创建「私人访问」客户端和「密码授权」客户端：

    php artisan passport:install

上面命令执行后，请将 `Laravel\Passport\HasApiTokens` Trait 添加到 `App\User` 模型中，这个 Trait 会给你的模型提供一些辅助函数，用于检查已认证用户的令牌和使用作用域：

    <?php

    namespace App;

    use Laravel\Passport\HasApiTokens;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;
    }

接下来，需要在 `AuthServiceProvider` 的 `boot` 方法中调用 `Passport::routes` 函数。这个函数会注册一些在访问令牌、客户端、私人访问令牌的发放和吊销过程中会用到的必要路由：

    <?php

    namespace App\Providers;

    use Laravel\Passport\Passport;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * The policy mappings for the application.
         *
         * @var array
         */
        protected $policies = [
            'App\Model' => 'App\Policies\ModelPolicy',
        ];

        /**
         * Register any authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Passport::routes();
        }
    }

最后，需要将配置文件 `config/auth.php` 中 `api` 部分的授权保护项（ `driver` ）改为 `passport` 。此调整会让你的应用程序在接收到 API 的授权请求时使用 Passport 的 `TokenGuard` 来处理：

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],

<a name="frontend-quickstart"></a>
### 前端快速上手

> {note} 如果想要使用 Passport 的 Vue 组件，那么你必须使用 [Vue](https://vuejs.org) Javascript 框架，另外这些组件还用到了 Bootstrap CSS 框架。当然你也可以不使用上面的任何工具，但在实现你自己的前端部分时，Passport 的 Vue 组件仍旧有很高的参考价值。

Passport 配备了一些可以让你的用户自行创建客户端和私人访问令牌的 JSON API。所以，你可以自己花时间来编写一些前端代码来使用这些 API。当然在 Passport 中也已经预制了一些 [Vue](https://vuejs.org) 组件，你可以直接使用这些示例代码，也可以基于这些代码实现自己的前端部分。

使用 Artisan 命令 `vendor:publish` 来发布 Passport 的 Vue 组件：

    php artisan vendor:publish --tag=passport-components

已发布的组件将被放置在 `resources/assets/js/components` 目录中，可以在 `resources/assets/js/app.js` 文件中注册这些已发布的组件：

    Vue.component(
        'passport-clients',
        require('./components/passport/Clients.vue')
    );

    Vue.component(
        'passport-authorized-clients',
        require('./components/passport/AuthorizedClients.vue')
    );

    Vue.component(
        'passport-personal-access-tokens',
        require('./components/passport/PersonalAccessTokens.vue')
    );

这些组件注册后，请保证运行 `npm run dev` 命令重新编译你的代码。一旦你重新编译了代码，你可以直接将这些组件直接放入应用程序的模板中，用于创建客户端和私人访问令牌：

    <passport-clients></passport-clients>
    <passport-authorized-clients></passport-authorized-clients>
    <passport-personal-access-tokens></passport-personal-access-tokens>

<a name="deploying-passport"></a>
### 部署 Passport

第一次部署 Passport 到生产服务器时，可能会需要运行 `passport:keys` 命令。该命令生成 Passport 所需要的加密密钥，来产生访问令牌。这些生成的密钥往往不保存在源码控制中：

    php artisan passport:keys

<a name="configuration"></a>
## 配置

<a name="token-lifetimes"></a>
### 令牌的有效期

默认情况下，Passport 发放的访问令牌是永久有效的，不需要刷新。但是如果你想给访问令牌配置一个短一些的有效期，那你就需要用到 `tokensExpireIn` 和 `refreshTokensExpireIn` 方法了，上述两个方法同样需要在 `AuthServiceProvider` 的 `boot` 方法中调用：

    use Carbon\Carbon;

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::tokensExpireIn(Carbon::now()->addDays(15));

        Passport::refreshTokensExpireIn(Carbon::now()->addDays(30));
    }

<a name="issuing-access-tokens"></a>
## 发放访问令牌

熟悉 OAuth2 的开发者一定知道， OAuth2 中必不可少的部分就是授权码。在获取授权码时，接入应用会重定向一个用户到你的服务端，用户可以选择允许或拒绝向这个客户端发放访问令牌。

<a name="managing-clients"></a>
### 管理客户端

首先，接入应用如果想要与你应用的 API 进行交互，必须先在你的应用程序中注册一个「客户端」。一般来说，这个注册过程需要开发者提供两部分信息：接入应用名称和用户授权后的跳转链接。

#### 命令 `passport:client`

创建客户端最简单的方式是使用 Artisan 命令 `passport:client` ，你可以使用此命令创建自己的客户端，用于测试 OAuth2 的功能。在你执行 `client` 命令时，Passport 会提示输入更多关于你的客户端的信息，最终会提供给你生成的客户端的 ID 和 密钥：

    php artisan passport:client

#### JSON API

考虑到你的用户们并没有办法使用 `client` 命令，Passport 同时提供了用户创建客户端的 JSON API 。这样你就不用再花时间编码来实现客户端创建、更新和删除的相关控制器逻辑了。

然而，你仍旧需要基于 Passport 的 JSON API 开发一套前端界面，方便你的用户管理他们授权的客户端。下面我们会列出所有用于管理客户端的 API，方便起见，我们使用 [Axios](https://github.com/mzabriskie/axios) 展示对 API 的 HTTP 请求。

> {tip} 如果你不想自己重写整个客户端管理的前端界面，可以根据 [前端快速上手](#frontend-quickstart) 在几分钟内组建一套功能完备的前端界面。

#### `GET /oauth/clients`

此接口会返回当前认证用户的所有客户端。主要用途是列出当前用户所有客户端，方便用户修改或删除：

    axios.get('/oauth/clients')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/clients`

此接口用于用户创建新的客户端。它需要两部分数据：客户端的名称、客户端的 `redirect` 链接。当用户允许或拒绝授权请求后，用户都会被重定向到这个 `redirect` 链接。

当客户端创建完成后，会生成此客户端的 ID 和密钥，客户端可以使用这两个值从你的应用程序请求访问令牌。此接口会返回新建客户端实例的信息：

    const data = {
        name: 'Client Name',
        redirect: 'http://example.com/callback'
    };

    axios.post('/oauth/clients', data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });

#### `PUT /oauth/clients/{client-id}`

此接口用于更新客户端信息。它需要两部分数据：客户端的 `name` 和 `redirect` 链接。当用户允许或拒绝授权请求后，用户都会被重定向到这个 `redirect` 链接。此接口会返回被更新客户端实例的信息：

    const data = {
        name: 'New Client Name',
        redirect: 'http://example.com/callback'
    };

    axios.put('/oauth/clients/' + clientId, data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });

#### `DELETE /oauth/clients/{client-id}`

此接口用于删除客户端：

    axios.delete('/oauth/clients/' + clientId)
        .then(response => {
            //
        });

<a name="requesting-tokens"></a>
### 请求令牌

#### 授权时的重定向

客户端创建之后，开发者会使用此客户端的 ID 和密钥向你的应用程序请求一个授权码和访问令牌。首先，接入应用会将用户重定向到你应用程序的 `/oauth/authorize` 路由上，示例如下：

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} 注意，路由 `/oauth/authorize` 已经在 `Passport::routes` 方法中定

#### 确认授权请求

接收到授权请求时，Passport 会显示默认的授权确认页面，用户可以允许或拒绝本次授权请求。用户确认后会被重定向回接入应用程序请求中指定的 `redirect_uri` 链接。`redirect_uri` 必须和客户端创建时提供的 `redirect` 完全一致。

如果你想自定义授权确认页面，可以使用 Artisan 命令 `vendor:publish` 发布 Passport 的视图文件。发布后的视图文件存放路径为 `resources/views/vendor/passport` ：

    php artisan vendor:publish --tag=passport-views

#### 将授权码转换为访问令牌

用户允许授权请求后，用户将会被重定向会接入应用程序，然后接入应用将通过 `POST` 请求向你的应用程序申请访问令牌，此次请求需要携带用户允许授权时产生的授权码。在下面的例子中，我们使用 Guzzle HTTP 库来实现这次 `POST` 请求：

    Route::get('/callback', function (Request $request) {
        $http = new GuzzleHttp\Client;

        $response = $http->post('http://your-app.com/oauth/token', [
            'form_params' => [
                'grant_type' => 'authorization_code',
                'client_id' => 'client-id',
                'client_secret' => 'client-secret',
                'redirect_uri' => 'http://example.com/callback',
                'code' => $request->code,
            ],
        ]);

        return json_decode((string) $response->getBody(), true);
    });


接口 `/oauth/token` 的 JSON 相应中会包含 `access_token` 、`refresh_token` 和 `expires_in` 属性。`expires_in` 的值即当前访问令牌的有效期（单位：秒）。

> {tip} 如上 `/oauth/authorize` 路由，`/oauth/token` 已经在 `Passport::routes` 方法中定义，所以无需再次定义。

<a name="refreshing-tokens"></a>
### 刷新令牌

如果你的应用程序发放了短期访问令牌，用户需要刷新访问令牌时，需要提供与访问令牌同时发放的刷新令牌。在下面的例子中，我们使用 Guzzle HTTP 库来刷新令牌：

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'refresh_token',
            'refresh_token' => 'the-refresh-token',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

接口 `/oauth/token` 会返回一个 JSON 响应，会包含 `access_token` 、`refresh_token` 和 `expires_in` 属性。`expires_in` 属性值即当前访问令牌的有效时间（单位：秒）。

<a name="password-grant-tokens"></a>
## 密码授权令牌

OAuth2 密码授权机制可以让自有应用基于邮箱地址（用户名）和密码获取访问令牌，自有应用比如你的手机客户端。这样就允许自由应用无需跳转步骤即可通过整个 OAuth2 的授权过程。

<a name="creating-a-password-grant-client"></a>
### 创建密码授权客户端

如果想要通过密码授权机制来发布令牌，首先你需要创建一个密码授权客户端。你可以使用带有 `--password` 参数的 `passport:client` 命令。如果你已经运行了 `passport:install` 命令，那无需再单独运行此命令：

    php artisan passport:client --password

<a name="requesting-password-grant-tokens"></a>
### 请求密码授权令牌

当你创建密码授权客户端后，你可以向 `/oauth/token` 接口发起 `POST` 请求来获取访问令牌，请求时需要带有用户的邮箱地址和密码信息。注意，该接口已经在 `Passport::routes` 方法中定义，所以无需再次手动定义。请求成功后，服务端返回的 JSON 响应数据中会带有 `access_token` 和 `refresh_token` 属性：

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

> {tip} 注意：访问令牌默认是永久有效的。但是如果需要你可以 [配置你应用程序的访问令牌有效时间](#configuration)。

<a name="requesting-all-scopes"></a>
### 请求所有作用域

使用密码授权机制时，你可以通过请求作用域 `*` 让你的令牌获取应用程序中定义的所有作用域。在处理使用此令牌发起的请求时，`can` 函数会始终返回 `true` ，这种作用域的授权最好只应用在使用 `password` 授权时发放的令牌中：

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '*',
        ],
    ]);

<a name="implicit-grant-tokens"></a>
## 简化授权令牌

简化授权和通过授权码授权相似; 区别是, 不需要通过授权码去获取令牌而是把令牌直接返回客户端. 主要用在无法安全存储证书场景中，这种授权在 JavaScript 和 移动应用 是最常用的. 开启授权, 在 `AuthServiceProvider` 中调用 `enableImplicitGrant` 方法:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot() 
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::enableImplicitGrant();
    }


调用上面方法开启授权后, 开发者可以通过自己的应用把 client ID 当做参数去请求一个令牌。在你的应用程序 `/oauth/authorize` 的接口中应该有一个重定向请求像下面这样:


    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'token',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });


> {tip} 记住, 这个 `/oauth/authorize` 接口已经定义在 `Passport::routes` 中，所以无需再次手动定义。


<a name="client-credentials-grant-tokens"></a>
## 客户端证书授权令牌


客户端证书授权适用于机器对机器认证，例如，你可以在通过API执行脚本任务中使用此授权。要使用这种授权，你首先需要在 `app/Http/Kernel.php` 的 `$routeMiddleware` 变量中添加新的中间件：


    use Laravel\Passport\Http\Middleware\CheckClientCredentials::class;

    protected $routeMiddleware = [
        'client' => CheckClientCredentials::class,
    ];

然后将这个中间件附加到路由中：


    Route::get('/user', function(Request $request) {
        ...
    })->middleware('client');


要获取令牌，向 `oauth/token` 接口发出请求:


    $guzzle = new GuzzleHttp\Client;

    $response = $guzzle->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'client_credentials',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => 'your-scope',
        ],
    ]);

    echo json_decode((string) $response->getBody(), true);

<a name="personal-access-tokens"></a>
## 私人访问令牌


有些时候你的用户可能想发布一个访问令牌自己使用，又不想经历通常的授权跳转流程，这时候如果能让用户在你的应用程序中自行发放访问令牌，在试验 API 接口或者在简单的授权演示中，是一个不错的解决方案。


> {note} 私人访问令牌总是永久有效的，`tokensExpireIn` 和 `refreshTokensExpireIn` 方法不会影响它的有效期。

<a name="creating-a-personal-access-client"></a>
### 创建私人访问客户端

发布私人访问令牌之前，你需要先创建对应的客户端。你可以使用带 `--personal` 参数的 `passport:client` 命令来创建，如果你已经运行了 `passport:install` 命令，那无需再运行此命令：

    php artisan passport:client --personal

<a name="managing-personal-access-tokens"></a>
### 管理私人访问令牌

创建私人访问客户端后，你可以使用 `User` 模型实例上的 `createToken` 方法来为给定用户发布令牌， `createToken` 方法的第一个参数为令牌名称，第二个参数（可选）是 [作用域](#token-scopes) 列表：

    $user = App\User::find(1);

    // Creating a token without scopes...
    $token = $user->createToken('Token Name')->accessToken;

    // Creating a token with scopes...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

#### JSON API

Passport 中也有用来管理私人访问令牌的 JSON API，你可以基于这些 API 开发一套前端操作界面供给用户管理自己的私人访问令牌。下面我们会列出所有管理私人访问令牌的 API。方便起见，我们使用 [Axios](https://github.com/mzabriskie/axios) 展示对 API 的 HTTP 请求。

> {tip} 如果你不想自己重写整个私人访问令牌管理的前端界面，可以根据 [前端快速上手](#frontend-quickstart) 在几分钟内组建一套功能完备的前端界面。

#### `GET /oauth/scopes`

此接口会返回应用程序中定义的所有 [作用域](#scopes) 。你可以使用此接口将所有的作用域展示给用户，方便他们授权给需要的私人访问令牌：

    axios.get('/oauth/scopes')
        .then(response => {
            console.log(response.data);
        });

#### `GET /oauth/personal-access-tokens` 

此接口返回当前授权用户创建的所有私人访问令牌。主要用途是列出当前用户所有访问令牌，方便用户修改或删除：


    axios.get('/oauth/personal-access-tokens')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/personal-access-tokens`

此接口用来创建私人访问令牌。需要提供两部分数据：令牌的名称（ `name` ） 作用域 （ `scopes` ）：

    const data = {
        name: 'Token Name',
        scopes: []
    };

    axios.post('/oauth/personal-access-tokens', data)
        .then(response => {
            console.log(response.data.accessToken);
        })
        .catch (response => {
            // List errors on response...
        });

#### `DELETE /oauth/personal-access-tokens/{token-id}`

此接口用于删除私人访问令牌：

    axios.delete('/oauth/personal-access-tokens/' + tokenId);

<a name="protecting-routes"></a>
## 路由保护

<a name="via-middleware"></a>
### 通过中间件

Passport 包含一个 [验证保护机制](/docs/{{version}}/authentication#adding-custom-guards) 可以验证请求中的的访问令牌。 前面i将 `api` 中的保护机制改为为 `passport` 后，你只要给需要验证访问令牌的路由添加 `auth:api` 中间件，该机制将发挥作用：

    Route::get('/user', function () {
        //
    })->middleware('auth:api');

<a name="passing-the-access-token"></a>
### 传递访问令牌


接入应用在调用 Passport 保护下的路由时，需要将访问令牌作为 `Bearer` 令牌放在请求头 `Authorization` 中。在下面的例子中，我们使用 Guzzle HTTP 库来实现这次 `POST` 请求：

    $response = $client->request('GET', '/api/user', [
        'headers' => [
            'Accept' => 'application/json',
            'Authorization' => 'Bearer '.$accessToken,
        ],
    ]);

<a name="token-scopes"></a>
## 令牌作用域

<a name="defining-scopes"></a>
### 定义作用域


当 API 客户端接入特定用户时，可以通过作用域来限定其访问权限。例如在你编写的电子商务应用中，一些接入应用可以获取订单的发货状态而不能创建订单。换言之，作用域能够让你的用户限制第三方应用的行为，从而保障自身的利益。

你可以使用 `Passport::tokensCan` 方法来定义 API 的作用域，定义代码需要放置在 `AuthServiceProvider` 的 `boot` 方法中。`tokensCan` 方法接受一个包含作用域名称、描述的数组作为参数。作用域描述将会在授权确认页中直接展示给用户，你可以将其定义为任何你需要的内容：

    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

<a name="assigning-scopes-to-tokens"></a>
### 给令牌分派作用域

#### 授权码机制

使用授权码机制申请访问令牌时，接入应用可以通过 `scope` 字符串参数指定他们需要的作用域。`scope` 包含多个作用域名称时，名称之间使用空格分隔：

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

#### 私人访问令牌

使用 `User` 模型的 `createToken` 方法发放访问令牌时，你可以将需要的作用域数组作为第二个参数传给此方法：

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="checking-scopes"></a>
### 检查作用域

Passport 包含两个检查作用域的中间件，通过访问令牌请求时将会使用这两个中间件来检查是否授予了特定作用域。使用之前，需要将下面的中间件添加到 `app/Http/Kernel.php` 文件的 `$routeMiddleware` 属性中：

    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,

#### 检查所有作用域

路由可以使用 `scopes` 中间件来检查当前请求是否拥有指定的 *所有* 作用域：

    Route::get('/orders', function () {
        // Access token has both "check-status" and "place-orders" scopes...
    })->middleware('scopes:check-status,place-orders');

#### 检查任意作用域

路由可以使用 `scope` 中间件来检查当前请求是否拥有指定的 *任意* 作用域：

    Route::get('/orders', function () {
        // Access token has either "check-status" or "place-orders" scope...
    })->middleware('scope:check-status,place-orders');

#### 检查特定令牌实例的作用域

接入应用使用访问令牌通过你应用程序的验证后，你仍然可以使用当前授权 `User` 实例上的 `tokenCan` 方法来验证此令牌是否拥有指定的作用域：

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });

<a name="consuming-your-api-with-javascript"></a>
## 使用 JavaScript 接入 API

在构建 API 时，如果能通过 JavaScript 应用接入自己的 API 将会给开发过程带来极大的便利。这样你可以与所用人一样使用你自己的应用程序的 API，同样的 API 可以被你自己的 web 应用、移动应用、第三方应用以及你发布到各个包管理平台的 SDK 共同使用。

通常，在你通过 JavaScript 接入你的 API 时，每次请求你的应用程序时都需要手动传递访问令牌，然而，Passport 其中一个中间件可以帮你做这件事，你需要做的仅仅是将 `CreateFreshApiToken` 中间件添加到你的 `web` 中间件组中：

    'web' => [
        // Other middleware...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],


Passport 的这个中间件将会在你所有的对外请求中添加一个 `laravel_token` cookie ，该 cookie 将包含一个加密后的 [JWT](https://jwt.io/) ，Passport 可以根据此数据判断你 JavaScript 应用的授权状态。至此，你可以无需传递访问令牌直接请求应用程序的 API 了：

    axios.get('/user')
        .then(response => {
            console.log(response.data);
        });

当使用上面方法授权时，Axios 会自动带上 `X-CSRF-TOKEN` 请求头传递。另外，默认的 Laravel JavaScript 也会带上 `X-Requested-With` 请求头:

    window.axios.defaults.headers.common = {
        'X-Requested-With': 'XMLHttpRequest',
    };

> {note} 如果你用了其他 JavaScript 框架，需要确保每次对外请求都会带有 `X-CSRF-TOKEN` 和 `X-Requested-With` 请求头。


<a name="events"></a>
## 事件

Passport 在访问令牌和刷新令牌时触发事件。 你可以通过触发这些事件来修改或删除数据库中的其他访问令牌。 你可以在应用程序的 `EventServiceProvider` 中为这些事件附加监听器:

```php
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'Laravel\Passport\Events\AccessTokenCreated' => [
        'App\Listeners\RevokeOldTokens',
    ],

    'Laravel\Passport\Events\RefreshTokenCreated' => [
        'App\Listeners\PruneOldTokens',
    ],
];
```

<a name="testing"></a>
## 测试

Passport 的 `actingAs` 方法可以用于指定当前认证的用户及其授权范围。 `actingAs` 方法第一个参数是一个对象，第二个参数是数组表示申请的授权范围:

    public function testServerCreation()
    {
        Passport::actingAs(
            factory(User::class)->create(),
            ['create-servers']
        );

        $response = $this->post('/api/create-server');

        $response->assertStatus(200);
    }
    
## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|

| [@Kirisky](https://github.com/kirisky)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/18491_1503379318.jpeg?imageView2/1/w/100/h/100">  |  翻译  | 部分关键字翻译参考 [@KevinDiamen](https://github.com/KevinDiamen)  |

| Cloes  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/6187_1477389867.jpg?imageView2/1/w/100/h/100">  |  翻译  |  我的[github](https://github.com/cloes)  |



--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org