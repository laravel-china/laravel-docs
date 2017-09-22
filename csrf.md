# Laravel 下的伪造跨站请求保护 CSRF

- [简介](#csrf-introduction)
- [CSRF 白名单](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## 简介

Laravel 可以轻松地保护应用程序免受 [跨站请求伪造](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) 的攻击。跨站请求伪造是一种恶意的攻击，它凭借已通过身份验证的用户身份来运行未经过授权的命令。

Laravel 会自动为每个活跃用户的会话生成一个 CSRF「令牌」。该令牌用于验证经过身份验证的用户是否是向应用程序发出请求的用户。

任何情况下当你在应用程序中定义 HTML 表单时，都应该在表单中包含一个隐藏的 CSRF 令牌字段，以便 CSRF 保护中间件可以验证该请求。可以使用辅助函数 `csrf_field` 来生成令牌字段：

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

包含在 `web` 中间件组里的 `VerifyCsrfToken` [中间件](/docs/{{version}}/middleware)会自动验证请求里的令牌是否与存储在会话中令牌匹配。

#### CSRF 令牌 & JavaScript

构建由 Javascript 驱动的应用时，可以很方便地让 Javascript HTTP 函数库在发起每一个请求时自动附上 CSRF 令牌。默认情况下， `resources/assets/js/bootstrap.js` 文件会用 Axios HTTP 函数库注册的 `csrf-token` meta 标签中的值。如果你不使用这个函数库，你需要手动为你的应用配置此行为。

<a name="csrf-excluding-uris"></a>
## CSRF 白名单

有时候你可能希望设置一组并不需要 CSRF 保护的 URI。例如，如果你正在使用 [Stripe](https://stripe.com) 处理付款并使用了他们的 webhook 系统，你会需要从 CSRF 的保护中排除 Stripe Webhook 处理程序路由，因为 Stripe 并不会给你的路由发送 CSRF 令牌。

你可以把这类路由放到 `routes/web.php` 外，因为 `RouteServiceProvider` 的 `web` 中间件适用于该文件中的所有路由。不过，你也可以通过将这类 URI 添加到 `VerifyCsrfToken` 中间件中的 `$except` 属性来排除对这类路由的 CSRF 保护：

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

    class VerifyCsrfToken extends BaseVerifier
    {
        /**
         * 这些 URI 将免受 CSRF 验证
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
        ];
    }

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

除了检查 POST 参数中的 CSRF 令牌外，`VerifyCsrfToken` 中间件还会检查 `X-CSRF-TOKEN` 请求头。你可以将令牌保存在 HTML `meta` 标签中：

    <meta name="csrf-token" content="{{ csrf_token() }}">

然后你就可以使用类似 jQuery 的库自动将令牌添加到所有请求的头信息中。这可以为基于 AJAX 的应用提供简单、方便的 CSRF 保护：

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });

> {tip} 默认情况下， `resources/assets/js/bootstrap.js` 文件会用 Axios HTTP 函数库注册 `csrf-token` meta 标签中的值。如果你不使用这个函数库，则需要为你的应用手动配置此行为。

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel 将当前的 CSRF 令牌存储在由框架生成的每个响应中包含的一个 `XSRF-TOKEN` cookie 中。为方便起见，你可以使用 cookie 值来设置 X-XSRF-TOKEN 请求头，而一些 JavaScript 框架和库（如 Angular 和 Axios）会自动将这个值添加到 `X-XSRF-TOKEN` 头中。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@王凯波](http://weibo.com/wangkaibo)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/1924_1487053084.jpeg?imageView2/1/w/100/h/100">  |  翻译  | 面向工资编程  [@wangkaibo](https://github.com/wangkaibo/)  |
| [@Lichmaker](https://laravel-china.org/users/16370)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/16370_1499995124.jpg?imageView2/1/w/100/h/100">  |  翻译 | Happy Coding! :) 我的微博：[神经考拉君](http://weibo.com/1779555595/) |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  Review  | Stay Hungry. Stay Foolish. |


---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
