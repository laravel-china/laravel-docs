# API Authentication (Passport)

- [介绍](#introduction)
- [安装](#installation)
    - [前端使用说明](#frontend-quickstart)
- [配置](#configuration)
    - [令牌的使用期限](#token-lifetimes)
    - [Pruning Revoked Tokens](#pruning-revoked-tokens)
- [Issuing Access Tokens](#issuing-access-tokens)
    - [Managing Clients](#managing-clients)
    - [Requesting Tokens](#requesting-tokens)
    - [Refreshing Tokens](#refreshing-tokens)
- [Password Grant Tokens](#password-grant-tokens)
    - [Creating A Password Grant Client](#creating-a-password-grant-client)
    - [Requesting Tokens](#requesting-password-grant-tokens)
    - [Requesting All Scopes](#requesting-all-scopes)
- [Personal Access Tokens](#personal-access-tokens)
    - [Creating A Personal Access Client](#creating-a-personal-access-client)
    - [Managing Personal Access Tokens](#managing-personal-access-tokens)
- [Protecting Routes](#protecting-routes)
    - [Via Middleware](#via-middleware)
    - [Passing The Access Token](#passing-the-access-token)
- [Token Scopes](#token-scopes)
    - [Defining Scopes](#defining-scopes)
    - [Assigning Scopes To Tokens](#assigning-scopes-to-tokens)
    - [Checking Scopes](#checking-scopes)
- [Consuming Your API With JavaScript](#consuming-your-api-with-javascript)

<a name="introduction"></a>
## 介绍

在 Laravel 中，可以非常简单得实现基于传统表单的登陆以及授权，但是如何满足 API 场景下的授权需求呢？在 API 场景中通常通过令牌来实现用户授权，而不是通过维护请求之间的 Session 状态。现在可以使用 Passport 在 Laravel 项目中轻而易举地实现 API 授权过程，通过 Passport 可以在几分钟之内为你的 Laravel 应用程序添加完整的 OAuth2 服务端实现。 Passport 基于 [League OAuth2 server](https://github.com/thephpleague/oauth2-server) 实现，该项目的维护人是 [Alex Bilbie](https://github.com/alexbilbie) 。

> {note} 本文档假定你已熟悉 OAuth2 。如果你对 OAuth2 一无所知，阅读之前请考虑先熟悉下 OAuth2 的常用术语和基本特征。

<a name="installation"></a>
## 安装

使用 Composer 依赖包管理器安装 Passport ：

    composer require laravel/passport

接下来，将 Passport 的服务提供者注册到配置文件 `config/app.php` 的 `providers` 数组中：

    Laravel\Passport\PassportServiceProvider::class,

Passport 将通过服务提供者注册自己内部的数据库迁移脚本目录，所以在上一步添加注册者之后，你需要更新你的数据库结构。Passport 的迁移脚本会自动创建应用程序需要的客户端数据表和令牌数据表：

    php artisan migrate

接下来，你需要运行 `passport:install` 命令。此命令将创建用来生成安全接入令牌的加密密钥，另外，这条命令会创建用于生成接入令牌的「私人接入」客户端和「密码授权」客户端：

    php artisan passport:install

上面命令执行后，请将 `Laravel\Passport\HasApiTokens` Trait 添加到 `App\User` 模型中，这个 Trait 会给你的模型提供一些用于检查已认证用户令牌和使用范围的辅助函数：

    <?php

    namespace App;

    use Laravel\Passport\HasApiTokens;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;
    }

接下来，需要在 `AuthServiceProvider` 的 `boot` 方法中调用 `Passport::routes` 函数。这个函数会注册一些在接入令牌、客户端、私人接入令牌的发放和吊销过程中会用到的必要路由：

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
### 前端使用说明

> {note} 为了使用 Passport 的 Vue 组件，那么你必须使用 [Vue](https://vuejs.org) Javascript 框架，另外这些组件还用到了 Bootstrap CSS 框架。然而，就算你不使用刚刚提到的这些工具，在实现你自己的前端部分时，这些组件仍旧有很高的参考价值。

Passport 配备了一些可以让你的用户自行创建客户端和私人接入令牌的 JSON API。所以，你可以自己花费时间来编写一些前端代码来使用这些 API。但是在 Passport 中也已经预制了一些 [Vue](https://vuejs.org) 组件，你可以直接使用这些示例代码，也可以基于这些代码实现自己的前端部分。

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

这些组件注册后，你可以直接将这些组件直接放入应用程序的模板中，用于创建客户端和私人接入令牌：

    <passport-clients></passport-clients>
    <passport-authorized-clients></passport-authorized-clients>
    <passport-personal-access-tokens></passport-personal-access-tokens>

<a name="configuration"></a>
## 配置

<a name="token-lifetimes"></a>
### 令牌的有效期

默认情况下，Passport 发放的接入令牌是永久有效的，不需要刷新。但是如果你想给接入令牌配置一个短一些的有效期，那你就需要用到 `tokensExpireIn` 和 `refreshTokensExpireIn` 方法了，上述两个方法同样需要在 `AuthServiceProvider` 的 `boot` 方法中调用：

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

<a name="pruning-revoked-tokens"></a>
### 清理已失效的令牌

默认情况下，Passport 不会从数据库中删除已失效的令牌。随着时间增长，数据库中会积累大量已失效的令牌。如果你希望 Passport 自动删除它们，你可以在  `AuthServiceProvider` 的 `boot` 方法中调用 `pruneRevokedTokens` 方法：

    use Laravel\Passport\Passport;

    Passport::pruneRevokedTokens();

这个方法的效果是在用户请求到新的接入令牌或刷新已存在令牌时删除老的已失效令牌，而不是每次调用时立即删除所有的失效令牌。

<a name="issuing-access-tokens"></a>
## 发放令牌

熟悉 OAuth2 的开发者都知道，OAuth2 中必不可少的部分就是授权码。在获取授权码时，应用客户端会重定向一个用户到你的服务端，用户可以选择允许或拒绝向这个客户端发放接入令牌。

<a name="managing-clients"></a>
### 管理客户端

首先，开发者的客户端如果想要与你应用程序的 API 进行交互，必须先在你的应用程序中注册一个「客户端」。一般来说，这个注册过程需要开发者提供两部分信息，一部分是他的应用程序名称，另一部分是用户允许授权后的跳转链接。

#### 命令 `passport:client`

最简单的创建客户端方式是使用 Artisan 命令 `passport:client` ，你可以使用此命令创建自己的客户端，用于测试 OAuth2 的功能。在你执行 `client` 命令时，Passport 会提示输入更多关于你的客户端的信息，最终会提供给你生成的客户端的 ID 和 密钥：

    php artisan passport:client

#### JSON API

考虑到你的用户们应该没有办法使用 `client` 命令，Passport 同时提供了用户创建客户端的 JSON API 。这样你就不用再花时间编码来实现客户端创建、更新和删除的相关控制器了。

然而，你仍旧需要基于 Passport 的 JSON API 开发一套前端界面，方便你的用户管理他们自己的客户端。下面我们会列出所有用于管理客户端的 API，方便起见，我们使用 [Vue](https://vuejs.org) 展示对 API 的 HTTP 请求。

> {tip} 如果你不想自己重写整个客户端管理系统的前端界面，可以根据 [前端使用说明](#frontend-quickstart) 在几分钟内组建一套功能完备的前端界面。

#### `GET /oauth/clients`

此接口会返回当前认证用户的所有客户端。主要用途是列出当前用户所有客户端，方便用户修改或删除：

    this.$http.get('/oauth/clients')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/clients`

此接口用户创建新的客户端。它需要两部分数据：客户端的名称、客户端的 `redirect` 链接。当用户允许或拒绝授权请求后，用户都会被重定向到这个 `redirect` 链接。

当客户端创建完成后，会生成此客户端的 ID 和密钥，客户端会使用这两个值从你的应用程序请求接入令牌。此接口会返回新建客户端的实例信息：

    const data = {
        name: 'Client Name',
        redirect: 'http://example.com/callback'
    };

    this.$http.post('/oauth/clients', data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });

#### `PUT /oauth/clients/{client-id}`

此接口用于更新客户端信息。它需要两部分数据：客户端的名称、客户端的 `redirect` 链接。当用户允许或拒绝授权请求后，用户都会被重定向到这个 `redirect` 链接。此接口会返回被更新客户端的实例信息：

    const data = {
        name: 'New Client Name',
        redirect: 'http://example.com/callback'
    };

    this.$http.put('/oauth/clients/' + clientId, data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });

#### `DELETE /oauth/clients/{client-id}`

此接口用于删除客户端：

    this.$http.delete('/oauth/clients/' + clientId)
        .then(response => {
            //
        });

<a name="requesting-tokens"></a>
### 请求令牌

#### 授权时的重定向

客户端创建之后，开发者会使用此客户端的 ID 和密钥向你的应用程序请求一个授权码和接入令牌。首先，使用者的应用程序会将用户重定向到你应用程序的 `/oauth/authorize` 路由上，示例如下：

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} 注意，路由 `/oauth/authorize` 已经在 `Passport::routes` 方法中定义，所以无需再次定义。

#### 确认授权请求

接收到授权请求后，Passport 会显示默认的授权确认页面，用户可以通过或拒绝本次授权请求。用户确认后会被重定向回消费应用程序请求中指定的 `redirect_uri` 链接。`redirect_uri` 必须和客户端创建时提供的 `redirect` 完全一致。

如果你想自定义授权确认页面，可以使用 Artisan 命令 `vendor:publish` 发布 Passport 的视图文件。发布后的视图文件存放路径为 `resources/views/vendor/passport` ：

    php artisan vendor:publish --tag=passport-views

#### 将授权码转换为接入令牌

如果用户通过授权请求后，用户将会被重定向会消费应用程序，然后消费应用程序将通过 `POST` 请求向你的应用程序申请接入令牌，此次请求需要携带用户通过授权时产生的授权码。在下面的例子中，我们使用 Guzzle HTTP 库来实现这次 `POST` 请求：

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

接口 `/oauth/token` 的 JSON 相应中会包含 `access_token` 、`refresh_token` 和 `expires_in` 属性。`expires_in` 的值即当前接入令牌的有效期（单位：秒）。

> {tip} 如上 `/oauth/authorize` 路由，`/oauth/token` 已经在 `Passport::routes` 方法中定义，所以无需再次定义。

<a name="refreshing-tokens"></a>
### 刷新令牌

如果你的应用程序发放了短期接入令牌，用户需要刷新接入令牌时，需要提供与接入令牌同时发放的刷新令牌。在下面的例子中，我们使用 Guzzle HTTP 库来刷新令牌：

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

接口 `/oauth/token` 会返回一个 JSON 响应，会包含 `access_token` 、`refresh_token` 和 `expires_in` 属性。`expires_in` 属性值即当前接入令牌的有效时间（单位：秒）。

<a name="password-grant-tokens"></a>
## Password Grant 令牌

OAuth2 Password Grant 可以让自有应用基于邮箱地址（用户名）和密码获取接入令牌，自有应用比如你的手机客户端。这样就允许自由应用无需跳转步骤即可通过整个 OAuth2 的授权过程。

<a name="creating-a-password-grant-client"></a>
### 创建 Password Grant 客户端

如果想要通过 Password Grant 来授予令牌，首先你需要创建一个 Password Grant 客户端。你可以使用带有 `--password` 参数的 `passport:client` 命令。如果你已经运行了 xx 命令，那无需再单独运行此命令：

    php artisan passport:client --password

<a name="requesting-password-grant-tokens"></a>
### 请求接入令牌

当你创建 Password Grant 客户端后，你可以向 `/oauth/token` 接口发起 `POST` 请求来获取接入令牌，请求时需要带有用户的邮箱地址和密码信息。注意，该接口已经在 `Passport::routes` 方法中定义，所以无需再次手动定义。请求成功后，服务端返回的 JSON 响应数据中会带有 `access_token` 和 `refresh_token` 属性：

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

> {tip} 注意：接入令牌默认是长期有效的。但是如果需要你可以 [配置你应用程序的接入令牌有效时间](#configuration)。

<a name="requesting-all-scopes"></a>
### 请求所有权限范围

使用 Password Grant 时，你可以通过请求权限范围 `*` 让你的令牌获取应用程序中定义的所有权限范围。如果你请求了所有权限范围，使用此令牌发起的请求处理中，`can` 函数会始终返回 `true` ，这种权限范围的授权最好只应用在使用 `password` 授权时发放的令牌中：

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '*',
        ],
    ]);

<a name="personal-access-tokens"></a>
## 私人接入令牌

有时候，你的用户可能想发布一个接入令牌自己使用，又不想经历典型的授权跳转流程，这时候如果用户能够在你的应用程序中通过界面来操作，可能会是一个更好的解决方案。

> {note} 私人接入令牌总是长期有效的，`tokensExpireIn` 和 `refreshTokensExpireIn` 方法不会影响他的有效期。

<a name="creating-a-personal-access-client"></a>
### 创建使用私人接入令牌的客户端

发布私人接入令牌之前，你需要先创建对应的客户端。你可以使用带 `--personal` 参数的 `passport:client` 命令来创建，如果你已经运行了 `passport:install` 命令，那无需再单独运行此命令：

    php artisan passport:client --personal

<a name="managing-personal-access-tokens"></a>
### 管理私人接入令牌

创建私人接入客户端后，你可以使用 xx 模型实例上的 xx 方法来为给定用户发布令牌，xx 方法的第一个参数为令牌名称，第二个参数（可选）是 [权限范围](#token-scopes) 的列表：

    $user = App\User::find(1);

    // Creating a token without scopes...
    $token = $user->createToken('Token Name')->accessToken;

    // Creating a token with scopes...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

#### JSON API

Passport also includes a JSON API for managing personal access tokens. You may pair this with your own frontend to offer your users a dashboard for managing personal access tokens. Below, we'll review all of the API endpoints for managing personal access tokens. For convenience, we'll use [Vue](https://vuejs.org) to demonstrate making HTTP requests to the endpoints.

> {tip} If you don't want to implement the personal access token frontend yourself, you can use the [frontend quickstart](#frontend-quickstart) to have a fully functional frontend in a matter of minutes.

#### `GET /oauth/scopes`

This route returns all of the [scopes](#scopes) defined for your application. You may use this route to list the scopes a user may assign to a personal access token:

    this.$http.get('/oauth/scopes')
        .then(response => {
            console.log(response.data);
        });

#### `GET /oauth/personal-access-tokens`

This route returns all of the personal access tokens that the authenticated user has created. This is primarily useful for listing all of the user's token so that they may edit or delete them:

    this.$http.get('/oauth/personal-access-tokens')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/personal-access-tokens`

This route creates new personal access tokens. It requires two pieces of data: the token's `name` and the `scopes` that should be assigned to the token:

    const data = {
        name: 'Token Name',
        scopes: []
    };

    this.$http.post('/oauth/personal-access-tokens', data)
        .then(response => {
            console.log(response.data.accessToken);
        })
        .catch (response => {
            // List errors on response...
        });

#### `DELETE /oauth/personal-access-tokens/{token-id}`

This route may be used to delete personal access tokens:

    this.$http.delete('/oauth/personal-access-tokens/' + tokenId);

<a name="protecting-routes"></a>
## Protecting Routes

<a name="via-middleware"></a>
### Via Middleware

Passport includes an [authentication guard](/docs/{{version}}/authentication#adding-custom-guards) that will validate access tokens on incoming requests. Once you have configured the `api` guard to use the `passport` driver, you only need to specify the `auth:api` middleware on any routes that require a valid access token:

    Route::get('/user', function () {
        //
    })->middleware('auth:api');

<a name="passing-the-access-token"></a>
### Passing The Access Token

When calling routes that are protected by Passport, your application's API consumers should specify their access token as a `Bearer` token in the `Authorization` header of their request. For example, when using the Guzzle HTTP library:

    $response = $client->request('GET', '/api/user', [
        'headers' => [
            'Accept' => 'application/json',
            'Authorization' => 'Bearer '.$accessToken,
        ],
    ]);

<a name="token-scopes"></a>
## Token Scopes


<a name="defining-scopes"></a>
### Defining Scopes

Scopes allow your API clients to request a specific set of permissions when requesting authorization to access an account. For example, if you are building an e-commerce application, not all API consumers will need the ability to place orders. Instead, you may allow the consumers to only request authorization to access order shipment statuses. In other words, scopes allow your application's users to limit the actions a third-party application can perform on their behalf.

You may define your API's scopes using the `Passport::tokensCan` method in the `boot` method of your `AuthServiceProvider`. The `tokensCan` method accepts an array of scope names and scope descriptions. The scope description may be anything you wish and will be displayed to users on the authorization approval screen:

    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

<a name="assigning-scopes-to-tokens"></a>
### Assigning Scopes To Tokens

#### When Requesting Authorization Codes

When requesting an access token using the authorization code grant, consumers should specify their desired scopes as the `scope` query string parameter. The `scope` parameter should be a space-delimited list of scopes:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

#### When Issuing Personal Access Tokens

If you are issuing personal access tokens using the `User` model's `createToken` method, you may pass the array of desired scopes as the second argument to the method:

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="checking-scopes"></a>
### Checking Scopes

Passport includes two middleware that may be used to verify that an incoming request is authenticated with a token that has been granted a given scope. To get started, add the following middleware to the `$routeMiddleware` property of your `app/Http/Kernel.php` file:

    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,

#### Check For All Scopes

The `scopes` middleware may be assigned to a route to verify that the incoming request's access token has *all* of the listed scopes:

    Route::get('/orders', function () {
        // Access token has both "check-status" and "place-orders" scopes...
    })->middleware('scopes:check-status,place-orders');

#### Check For Any Scopes

The `scope` middleware may be assigned to a route to verify that the incoming request's access token has *at least one* of the listed scopes:

    Route::get('/orders', function () {
        // Access token has either "check-status" or "place-orders" scope...
    })->middleware('scope:check-status,place-orders');

#### Checking Scopes On A Token Instance

Once an access token authenticated request has entered your application, you may still check if the token has a given scope using the `tokenCan` method on the authenticated `User` instance:

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });

<a name="consuming-your-api-with-javascript"></a>
## Consuming Your API With JavaScript

When building an API, it can be extremely useful to be able to consume your own API from your JavaScript application. This approach to API development allows your own application to consume the same API that you are sharing with the world. The same API may be consumed by your web application, mobile applications, third-party applications, and any SDKs that you may publish on various package managers.

Typically, if you want to consume your API from your JavaScript application, you would need to manually send an access token to the application and pass it with each request to your application. However, Passport includes a middleware that can handle this for you. All you need to do is add the `CreateFreshApiToken` middleware to your `web` middleware group:

    'web' => [
        // Other middleware...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],

This Passport middleware will attach a `laravel_token` cookie to your outgoing responses. This cookie contains an encrypted JWT that Passport will use to authenticate API requests from your JavaScript application. Now, you may make requests to your application's API without explicitly passing an access token:

    this.$http.get('/user')
        .then(response => {
            console.log(response.data);
        });

When using this method of authentication, you will need to send the CSRF token with every request via the `X-CSRF-TOKEN` header. Laravel will automatically send this header if you are using the default [Vue](https://vuejs.org) configuration that is included with the framework:

    Vue.http.interceptors.push((request, next) => {
        request.headers['X-CSRF-TOKEN'] = Laravel.csrfToken;

        next();
    });

> {note} If you are using a different JavaScript framework, you should make sure it is configured to send this header with every outgoing request.
