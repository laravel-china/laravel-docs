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
- [个人访问令牌](#personal-access-tokens)
    - [创建个人访问令牌的客户端](#creating-a-personal-access-client)
    - [管理个人访问令牌](#managing-personal-access-tokens)
- [路由保护](#protecting-routes)
    - [通过中间件](#via-middleware)
    - [传递访问令牌](#passing-the-access-token)
- [令牌作用域](#token-scopes)
    - [定义作用域](#defining-scopes)
    - [给令牌分配作用域](#assigning-scopes-to-tokens)
    - [检查作用域](#checking-scopes)
- [使用 JavaScript 接入 API](#consuming-your-api-with-javascript)
- [事件](#events)
- [测试](#testing)

<a name="introduction"></a>
## 介绍

在 Laravel 中，实现基于传统表单的登陆和授权已经非常简单，但是如何满足 API 场景下的授权需求呢？在 API 场景里通常通过令牌来实现用户授权，而非维护请求之间的 Session 状态。在 Laravel 项目中使用 Passport 可以轻而易举地实现 API 授权认证，Passport 可以在几分钟之内为你的应用程序提供完整的 OAuth2 服务端实现。Passport 是基于由 [Alex Bilbie](https://github.com/alexbilbie) 维护的 [League OAuth2 server](https://github.com/thephpleague/oauth2-server) 建立的。

> {note} 本文档假定你已熟悉 OAuth2 。如果你并不了解 OAuth2 ，阅读之前请先熟悉下 OAuth2 的常用术语和功能。

<a name="installation"></a>
## 安装

使用 Composer 安装 Passport ：

    composer require laravel/passport

接下来，将 Passport 的服务提供者注册到配置文件 `config/app.php` 的 `providers` 数组中：

    Laravel\Passport\PassportServiceProvider::class,

Passport 服务提供器使用框架注册自己的数据库迁移目录，因此在注册提供器后，就应该运行 Passport 的迁移命令来自动创建存储客户端和令牌的数据表：

    php artisan migrate

> {note} 如果你不打算使用 Passport 的默认迁移，你应该在 `AppServiceProvider` 的 `register` 方法中调用 `Passport::ignoreMigrations` 方法。 你可以用这个命令 `php artisan vendor:publish --tag=passport-migrations` 导出默认迁移。

接下来，运行 `passport:install` 命令来创建生成安全访问令牌时所需的加密密钥，同时，这条命令也会创建用于生成访问令牌的「个人访问」客户端和「密码授权」客户端：

    php artisan passport:install

上面命令执行后，请将 `Laravel\Passport\HasApiTokens` Trait 添加到 `App\User` 模型中，这个 Trait 会给你的模型提供一些辅助函数，用于检查已认证用户的令牌和使用范围：

    <?php

    namespace App;

    use Laravel\Passport\HasApiTokens;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;
    }

接下来，在 `AuthServiceProvider` 的 `boot` 方法中调用 `Passport::routes` 函数。这个函数会注册发出访问令牌并撤销访问令牌、客户端和个人访问令牌所必需的路由：

    <?php

    namespace App\Providers;

    use Laravel\Passport\Passport;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * 应用程序的策略映射。
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

最后，将配置文件 `config/auth.php` 中授权看守器 `guards` 的 `api` 的 `driver` 选项改为 `passport`。此调整会让你的应用程序在在验证传入的 API 的请求时使用 Passport 的 `TokenGuard` 来处理：

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

Passport 配备了一些可以让你的用户自行创建客户端和个人访问令牌的 JSON API。然而，编写一些前端代码来与这些 API 进行交互很是耗时。因此 Passport 也引入了预先构建的 [Vue](https://vuejs.org) 组件，你可以直接使用，也可以基于这些代码实现自己的前端部分。

使用 Artisan 命令 `vendor:publish` 来发布 Passport 的 Vue 组件：

    php artisan vendor:publish --tag=passport-components

已发布的组件将被放置在 `resources/assets/js/components` 目录中，可以在 `resources/assets/js/app.js` 文件中注册它们：

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

这些组件注册后，运行 `npm install`安装vue所依赖的文件，运行`npm run dev` 命令以确保重新编译你的资源。重新编译资源后，你可以将这些组件放入应用程序的模板中，然后开始创建客户端和个人访问令牌：

    <passport-clients></passport-clients>
    <passport-authorized-clients></passport-authorized-clients>
    <passport-personal-access-tokens></passport-personal-access-tokens>

<a name="deploying-passport"></a>
### 部署 Passport

第一次将 Passport 部署到生产服务器时，需要运行 `passport:keys` 命令。该命令生成 Passport 所需要的用来产生访问令牌的加密密钥。生成的这些密钥不会保存在源码控制中：

    php artisan passport:keys

<a name="configuration"></a>
## 配置

<a name="token-lifetimes"></a>
### 令牌的有效期

默认情况下，Passport 发放的访问令牌是永久有效的，不需要刷新。但是如果你想自定义访问令牌的有效期，可以使用 `tokensExpireIn` 和 `refreshTokensExpireIn` 方法。上述两个方法同样需要在 `AuthServiceProvider` 的 `boot` 方法中调用：

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

熟悉 OAuth2 的开发者一定知道， OAuth2 中必不可少的部分就是授权码。当使用授权码时，客户端应用程序会将用户重定向到你的服务器，他们将批准或拒绝向客户端发出访问令牌的请求。

<a name="managing-clients"></a>
### 管理客户端

首先，构建需要与应用程序 API 交互的应用程序，开发人员将需要通过创建一个「客户端」来注册自己的应用程序。一般来说，这包括在用户批准其授权请求后，提供其应用程序的名称和应用程序可以重定向到的 URL。

#### `passport:client` 命令

创建客户端最简单的方式是使用 Artisan 命令 `passport:client`，你可以使用此命令创建自己的客户端，用于测试你的 OAuth2 的功能。在你执行 `client` 命令时，Passport 会提示你输入有关客户端的信息，最终会给你提供客户端的 ID 和 密钥：

    php artisan passport:client

#### JSON API

考虑到你的用户无法使用 `client` 命令，Passport 为此提供了可用于创建客户端的 JSON API。这样你就不用再花时间编写控制器来创建、更新和删除客户端。

然而，你仍旧需要基于 Passport 的 JSON API 开发一套前端界面，为你的用户提供管理客户端的仪表板。下面我们会列出所有用于管理客户端的 API，为了方便起见，我们使用 [Axios](https://github.com/mzabriskie/axios) 来演示对端口发出 HTTP 请求。

> {tip} 如果你不想自己实现整个客户端管理的前端界面，可以使用 [前端快速上手](#frontend-quickstart) 在几分钟内组建一套功能齐全的前端界面。

#### `GET /oauth/clients`

此路由会返回认证用户的所有客户端。主要用途是列出所有用户的客户端，以便他们可以编辑或删除它们：

    axios.get('/oauth/clients')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/clients`

此路由用于创建新客户端。它需要两部分数据：客户端的 `name` 和 `redirect` 的链接。在批准或拒绝授权请求后，用户会被重定向 `redirect` 到这个链接。

创建客户端时，会发出此客户端的 ID 和密钥。客户端可以使用这两个值从你的应用程序请求访问令牌。该路由会返回新的客户端实例：

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

此路由用于更新客户端信息。它需要两部分数据：客户端的 `name` 和 `redirect` 的链接。在批准或拒绝授权请求后，用户会被重定向 `redirect` 到这个链接。此路由会返回更新的客户端实例：

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

此路由用于删除客户端：

    axios.delete('/oauth/clients/' + clientId)
        .then(response => {
            //
        });

<a name="requesting-tokens"></a>
### 请求令牌

#### 授权时的重定向

客户端创建之后，开发者会使用此客户端的 ID 和密钥来请求授权代码，并从应用程序访问令牌。首先，接入应用的用户向你应用程序的 `/oauth/authorize` 路由发出重定向请求，示例如下：

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} 注意，路由 `/oauth/authorize` 已经在 `Passport::routes` 方法中定义。你不需要手动定义此路由。

#### 批准请求

接收到授权请求时，Passport 会自动向用户显示一个模版页面，允许用户批准或拒绝授权请求。如果用户批准请求，他们会被重定向回接入的应用程序指定的 `redirect_uri`。`redirect_uri` 必须和客户端创建时指定的 `redirect` 链接完全一致。

如果你想自定义授权确认页面，可以使用 Artisan 命令 `vendor:publish` 发布 Passport 的视图。发布后的视图文件存放在 `resources/views/vendor/passport`：

    php artisan vendor:publish --tag=passport-views

#### 将授权码转换为访问令牌

用户批准授权请求后，会被重定向回接入的应用程序。然后接入应用应该将通过 `POST` 请求向你的应用程序申请访问令牌。请求应该包括当用户批准授权请求时由应用程序发出的授权码。在下面的例子中，我们使用 Guzzle HTTP 库来实现这次 `POST` 请求：

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


路由 `/oauth/token` 返回的 JSON 响应中会包含 `access_token` 、`refresh_token` 和 `expires_in` 属性。`expires_in` 属性包含访问令牌的有效期（单位：秒）。

> {tip} 像 `/oauth/authorize` 路由一样，`/oauth/token` 路由在 `Passport::routes` 方法中定义了。

<a name="refreshing-tokens"></a>
### 刷新令牌

如果你的应用程序发放了短期的访问令牌，用户将需要通过在发出访问令牌时提供给他们的刷新令牌来刷新其访问令牌。在下面的例子中，我们使用 Guzzle HTTP 库来刷新令牌：

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

路由 `/oauth/token` 会返回一个 JSON 响应，其中包含 `access_token` 、`refresh_token` 和 `expires_in` 属性。`expires_in` 属性包含访问令牌的有效时间（单位：秒）。

<a name="password-grant-tokens"></a>
# 密码授权令牌

OAuth2 密码授权机制可以让你自己的客户端（如移动应用程序）邮箱地址或者用户名和密码获取访问令牌。如此一来你就可以安全地向自己的客户端发出访问令牌，而不需要遍历整个 OAuth2 授权代码重定向流程。

<a name="creating-a-password-grant-client"></a>
### 创建密码授权客户端

在应用程序通过密码授权机制来发布令牌之前，在 `passport:client` 命令后加上 `--password` 参数来创建密码授权的客户端。如果你已经运行了 `passport:install` 命令，则不需要再运行此命令：

    php artisan passport:client --password

<a name="requesting-password-grant-tokens"></a>
### 请求令牌

创建密码授权的客户端后，就可以通过向用户的电子邮件地址和密码向 `/oauth/token` 路由发出 `POST` 请求来获取访问令牌。而该路由已经由 `Passport::routes` 方法注册，因此不需要手动定义它。如果请求成功，会在服务端返回的 JSON 响应中收到一个 `access_token` 和 `refresh_token`：

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

> {tip} 默认情况下，访问令牌是永久有效的。你可以根据需要 [配置访问令牌的有效时间](#configuration)。

<a name="requesting-all-scopes"></a>
### 请求所有作用域

使用密码授权机制时，可以通过请求 scope 参数 `*` 来授权应用程序支持的所有范围的令牌。如果你的请求中包含 scope 为 `*` 的参数，令牌实例上的 `can` 方法会始终返回 `true`。这种作用域的授权只能分配给使用 `password` 授权时发出的令牌：

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
## 隐式授权令牌

隐式授权类似于授权码授权，但是它只将令牌返回给客户端而不交换授权码。这种授权最常用于无法安全存储客户端凭据的 JavaScript 或移动应用程序。通过调用 `AuthServiceProvider` 中的 `enableImplicitGrant` 方法来启用这种授权：

    /**
     * 注册任何身份验证/授权服务。
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::enableImplicitGrant();
    }

调用上面方法开启授权后，开发者可以使用他们的客户端 ID 从应用程序请求访问令牌。接入的应用程序应该向你的应用程序的 `/oauth/authorize` 路由发出重定向请求，如下所示：


    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'token',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });


> {tip} `/oauth/authorize` 路由在 `Passport::routes` 定义中，所以无需再次手动定义此路由。

<a name="client-credentials-grant-tokens"></a>

## 客户端凭据授权令牌


客户端凭据授权适用于机器到机器的认证。例如，你可以在通过 API 执行维护任务中使用此授权。要使用这种授权，你首先需要在 `app/Http/Kernel.php` 的 `$routeMiddleware` 变量中添加新的中间件：


    use Laravel\Passport\Http\Middleware\CheckClientCredentials::class;

    protected $routeMiddleware = [
        'client' => CheckClientCredentials::class,
    ];

然后在路由上追加这个中间件：


    Route::get('/user', function(Request $request) {
        ...
    })->middleware('client');


接下来通过向 `oauth/token` 接口发出请求来获取令牌:


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
## 个人访问令牌

如果用户要在不经过典型的授权码重定向流的情况下向自己发出访问令牌，可以允许用户通过应用程序的用户界面对自己发出令牌，用户可以因此顺便测试你的 API，或者也可以将其作为一种更简单的发布访问令牌的方式。


> {note} 个人访问令牌是永久有效的，就算使用了 `tokensExpireIn` 和 `refreshTokensExpireIn` 方法也不会修改它的生命周期。

<a name="creating-a-personal-access-client"></a>
### 创建个人访问客户端

在你的应用程序发布个人访问令牌之前，你需要在 `passport:client` 命令后带上 `--personal` 参数来创建对应的客户端。如果你已经运行了 `passport:install` 命令，则无需再运行此命令：

    php artisan passport:client --personal

<a name="managing-personal-access-tokens"></a>
### 管理个人访问令牌

创建个人访问客户端后，你可以使用 `User` 模型实例上的 `createToken` 方法来为给定用户发布令牌。`createToken` 方法接受令牌的名称作为其第一个参数和可选的  [作用域](#token-scopes) 数组作为其第二个参数：

    $user = App\User::find(1);

    // Creating a token without scopes...
    $token = $user->createToken('Token Name')->accessToken;

    // Creating a token with scopes...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

#### JSON API

Passport 中也有用来管理个人访问令牌的 JSON API，你可以将其与自己的前端配对，为用户提供管理个人访问令牌的仪表板。下面我们会介绍用于管理个人访问令牌的所有 API 接口。方便起见，我们使用 [Axios](https://github.com/mzabriskie/axios) 来演示对 API 的接口发出 HTTP 请求。

> {tip} 如果你不想实现自己的个人访问令牌管理的前端界面，可以根据 [前端快速上手](#frontend-quickstart) 在几分钟内组建功能齐全的前端界面。

#### `GET /oauth/scopes`

此路由会返回应用程序中定义的所有 [作用域](#scopes)。你可以使用此路由列出用户可能分配给个人访问令牌的范围：

    axios.get('/oauth/scopes')
        .then(response => {
            console.log(response.data);
        });

#### `GET /oauth/personal-access-tokens`

此路由返回认证用户创建的所有个人访问令牌。这主要用于列出所有用户的令牌，以便他们可以编辑或删除它们：


    axios.get('/oauth/personal-access-tokens')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/personal-access-tokens`

此路由创建新的个人访问令牌。它需要两个数据：令牌的名称和应该分配给令牌的作用范围：

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

此路由可用于删除个人访问令牌：

    axios.delete('/oauth/personal-access-tokens/' + tokenId);

<a name="protecting-routes"></a>
## 路由保护

<a name="via-middleware"></a>
### 通过中间件

Passport 包含一个 [验证保护机制](/docs/{{version}}/authentication#adding-custom-guards) 可以验证请求中传入的访问令牌。配置 `api` 的看守器使用 `passport` 驱动程序后，只需要在需要有效访问令牌的任何路由上指定 `auth:api` 中间件：

    Route::get('/user', function () {
        //
    })->middleware('auth:api');

<a name="passing-the-access-token"></a>
### 传递访问令牌

当调用 Passport 保护下的路由时，接入的 API 应用需要将访问令牌作为 `Bearer` 令牌放在请求头 `Authorization` 中。例如，使用 Guzzle HTTP 库时：

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

作用域可以让 API 客户端在请求账户授权时请求特定的权限。例如，如果你正在构建电子商务应用程序，并不是所有接入的 API 应用都需要下订单的功能。你可以让接入的 API 应用只被允许授权访问订单发货状态。换句话说，作用域允许应用程序的用户限制第三方应用程序执行的操作。

你可以在 `AuthServiceProvider` 的 `boot` 方法中使用 `Passport::tokensCan` 方法来定义 API 的作用域。`tokensCan` 方法接受一个作用域名称、描述的数组作为参数。作用域描述将会在授权确认页中直接展示给用户，你可以将其定义为任何你需要的内容：

    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

<a name="assigning-scopes-to-tokens"></a>
### 给令牌分配作用域

#### 请求授权码时

使用授权码授权请求访问令牌时，接入的应用应该将其所需的作用域指定为 `scope` 查询字符串参数。`scope` 包含多个作用域名称时，名称之间使用空格分隔：

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

#### 个人访问令牌

使用 `User` 模型的 `createToken` 方法发放个人访问令牌时，可以将所需作用域的数组作为第二个参数传给此方法：

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="checking-scopes"></a>
### 检查作用域

Passport 包含两个中间件，可用于验证传入的请求是否已被授予给定作用域的令牌进行身份验证。使用之前，需要将下面的中间件添加到 `app/Http/Kernel.php` 文件的 `$routeMiddleware` 属性中：

    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,

#### 检查所有作用域

路由可以使用 `scopes` 中间件来检查当前请求是否拥有指定的 *所有* 作用域：

    Route::get('/orders', function () {
        // 访问令牌具有 "check-status" and "place-orders" 的作用域...
    })->middleware('scopes:check-status,place-orders');

#### 检查任意作用域

路由可以使用 `scope` 中间件来检查当前请求是否拥有指定的 *任意* 作用域：

    Route::get('/orders', function () {
        // Access token has either "check-status" or "place-orders" scope...
    })->middleware('scope:check-status,place-orders');

#### 检查令牌实例上的作用域

就算访问令牌验证的请求已经通过应用程序的验证，你仍然可以使用当前授权 `User` 实例上的 `tokenCan` 方法来验证令牌是否拥有指定的作用域：

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });

<a name="consuming-your-api-with-javascript"></a>
## 使用 JavaScript 接入 API

在构建 API 时，如果能通过 JavaScript 应用接入自己的 API 将会给开发过程带来极大的便利。这种 API 开发方法允许你使用自己的应用程序的 API 和别人共享的 API。你的 Web 应用程序、移动应用程序、第三方应用程序以及可能在各种软件包管理器上发布的任何 SDK 都可能会使用相同的API。

通常，如果要从 JavaScript 应用程序中使用 API，则需要手动向应用程序发送访问令牌，并将其传递给应用程序。但是，Passport 有一个可以处理这个问题的中间件。将 `CreateFreshApiToken` 中间件添加到 `web` 中间件组就可以了：

    'web' => [
        // Other middleware...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],


Passport 的这个中间件将会在你所有的对外请求中添加一个 `laravel_token` cookie。该 cookie 将包含一个加密后的 [JWT](https://jwt.io/) ，Passport 将用来验证来自 JavaScript 应用程序的 API 请求。至此，你可以在不明确传递访问令牌的情况下向应用程序的 API 发出请求

    axios.get('/user')
        .then(response => {
            console.log(response.data);
        });

当使用上面的授权方法时，Axios 会自动带上 `X-CSRF-TOKEN` 请求头传递。另外，默认的 Laravel JavaScript 脚手架会让 Axios 发送 `X-Requested-With` 请求头:

    window.axios.defaults.headers.common = {
        'X-Requested-With': 'XMLHttpRequest',
    };

> {note} 如果你用了其他的 JavaScript 框架，应确保配置了每次对外请求都会带有 `X-CSRF-TOKEN` 和 `X-Requested-With` 请求头。


<a name="events"></a>
## 事件

Passport 在发出访问令牌和刷新令牌时触发事件。 在应用程序的 `EventServiceProvider` 中为这些事件追加监听器，可以通过触发这些事件来修改或删除数据库中的其他访问令牌：

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

Passport 的 `actingAs` 方法可以用于指定当前已认证的用户及其作用域。 `actingAs` 方法第一个参数是用户实例，第二个参数是应该授予用户令牌的作用范围的数组:

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
| [@Kirisky](https://github.com/kirisky)   | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/18491_1503379318.jpeg?imageView2/1/w/100/h/100"> | 翻译 | 部分关键字翻译参考 [@KevinDiamen](https://github.com/KevinDiamen) |
| Cloes | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/6187_1477389867.jpg?imageView2/1/w/100/h/100"> | 翻译 | 我的[github](https://github.com/cloes) |
| [@JokerLinly](https://laravel-china.org/users/5350) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg"> | Review | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
