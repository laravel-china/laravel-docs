# Laravel 的 社会化登录功能

- [简介](#introduction)
- [授权](#License)
- [官方文档](#official-documentation)
- [配置](#configuration)
- [基本用法](#basic-usage)
    - [无状态身份验证](#stateless-authentication)
    - [检索用户详细信息](#retrieving-user-details)
    - [从令牌检索用户详细信息](#retrieving-user-details-from-token)


## 简介

Laravel 社会化登录通过 Facebook ， Twitter ，Google ，LinkedIn ，GitHub 和 Bitbucket 提供了一个富有表现力的，流畅的 OAuth 身份验证界面。它几乎能处理所有你害怕处理的各种样板社会认证代码。

**我们不接受新的适配器。**

社区驱动的社会化登录提供商网站上可以找到为其他平台提供的适配器列表。

## 授权

Laravel 社会化登录是根据  [MIT 授权](http://opensource.org/licenses/MIT) 许可的开源软件

## 官方文档

除了常规的基于表单的身份验证之外， Laravel 也提供了一种简单，方便的办法来使用 [Laravel 社会化登录](https://github.com/laravel/socialite) 向 OAuth 提供程序进行身份验证。社公化登录目前支持 `Facebook` ， `Twitter` ， `LinkedIn` ，`Google` ，`GitHub` 和 `Bitbucket` 的身份验证。

要开始社会化登录，使用 `composer` 将相应包加入到你项目的依赖项中。

    composer require laravel/socialite

### 配置

安装完社会化登录库之后，在你的 `config/app.php` 文件中注册 `Laravel\Socialite\SocialiteServiceProvider` 。

```php
'providers' => [
    // Other service providers...

    Laravel\Socialite\SocialiteServiceProvider::class,
],
```

同时，在你的 `app` 配置文件中，把 `Socialite` facade 加入到 `aliases` 数组中。

```php
'Socialite' => Laravel\Socialite\Facades\Socialite::class,
```

你还需要为你应用使用的 OAuth 服务加入凭据。这些凭据应该放在你的 `config/services.php` 文件中，并且使用 `facebook` ， `twitter` ， `linkedin` ， `google` ， `github` 或 `bitbucket` 作为键名，具体取决于在你的应用中由哪个程序来提供验证服务，比如：

```php
'github' => [
    'client_id' => 'your-github-app-id',
    'client_secret' => 'your-github-app-secret',
    'redirect' => 'http://your-callback-url',
],
```
### 基本用法

接下来，你已经准备好了验证用户了！你需要两个路由：一个重定向用户到 OAuth 提供商，另一个在提供商验证之后接收回调。我们用 `Socialite` facade 来访问：

```php
<?php

namespace App\Http\Controllers\Auth;

use Socialite;

class LoginController extends Controller
{
    /**
     * Redirect the user to the GitHub authentication page.
     *
     * @return Response
     */
    public function redirectToProvider()
    {
        return Socialite::driver('github')->redirect();
    }

    /**
     * Obtain the user information from GitHub.
     *
     * @return Response
     */
    public function handleProviderCallback()
    {
        $user = Socialite::driver('github')->user();

        // $user->token;
    }
}
```

 `redirect` 方法负责发送用户到 OAuth 提供商，而 `user` 方法将读取传入的请求并从提供商处检索用户信息。在重定向用户之前，你还可以使用 `scope` 方法来设置请求的「scope」。这个方法会覆盖所有现有的范围。
 
```php
return Socialite::driver('github')
            ->scopes(['scope1', 'scope2'])->redirect();
```

当然，你需要定义通往你的控制器方法的路由。

```php
Route::get('login/github', 'Auth\LoginController@redirectToProvider');
Route::get('login/github/callback', 'Auth\LoginController@handleProviderCallback');
```

一部分 OAuth 提供商在重定向请求中支持携带可选参数。要在请求中包含任何可选参数，调用 `with` 方法时传入可选的数组即可。

```php
return Socialite::driver('google')
            ->with(['hd' => 'example.com'])->redirect();
```

当使用 `with` 方法时，注意不要传递保留关键字，比如 `state` 或 `response_type` 。

#### 无状态身份验证

 `stateless` 方法可以用于禁用 session 状态的验证，这个方法在向 API 添加社会化身份验证时非常有用。

```php
return Socialite::driver('google')->stateless()->user();
```


#### 检索用户详细信息

一旦你有了一个用户实例，你可以获取这个用户的更多详细信息：

```php
$user = Socialite::driver('github')->user();

// OAuth Two Providers
$token = $user->token;
$refreshToken = $user->refreshToken; // not always provided
$expiresIn = $user->expiresIn;

// OAuth One Providers
$token = $user->token;
$tokenSecret = $user->tokenSecret;

// All Providers
$user->getId();
$user->getNickname();
$user->getName();
$user->getEmail();
$user->getAvatar();
```

#### 从令牌检索用户详细信息

如果你已经有了一个用户的有效访问令牌，你可以使用 `userFromToken` 方法检索用户的详细信息。

```php
$user = Socialite::driver('github')->userFromToken($token);
```