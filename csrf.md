# CSRF 保护

- [介绍](#csrf-introduction)
- [CSRF 白名单](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## 介绍

Laravel 提供简单的方法保护你的应用不受到 [跨站请求伪造](http://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) 攻击。跨站请求伪造是一种恶意的攻击，它利用已通过身份验证的用户身份来运行未经授权的命令。

Laravel 会为每个活跃用户自动生成一个 CSRF "token" 。该 token 用来核实应用接收到的请求是通过身份验证的用户出于本意发送的。

无论何时，当你需要定义一个 HTML 表单，你都应该在里面包含一个隐藏的 CSRF token ，只有这样，CSRF 保护中间件才会验证请求。你可以使用辅助函数 `csrf_field` 来生成 token 隐藏字段：

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

`VerifyCsrfToken` [中间件](/docs/{{version}}/middleware)，是包含在 `web`中间件组中的，它会自动验证请求中的 token 是否与 session 中的相匹配。

<a name="csrf-excluding-uris"></a>
## CSRF 白名单

有时候你可能会希望一组 URIs 不要被 CSRF 保护。你如果使用 [Stripe](https://stripe.com) 处理付款，并且利用他们的 webhook 系统，你需要从 CSRF 保护中排除 webhook 的处理路由，因为 Stripe 不会知道传递什么 CSRF token 给你的路由。

一般的，你不应该把这种类型的路由写在  `routes/web.php` 文件中，因为此文件的所有路由在 `RouteServiceProvider` 中被绑定到 `web` 中间件组中（`web` 中间件组下的所有路由默认会进行 `VerifyCsrfToken` 过滤）。不过如果一定要这么做，你也可以通过在 `VerifyCsrfToken` 中间件中增加 `$except` 属性来排除这种路由：

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

    class VerifyCsrfToken extends BaseVerifier
    {
        /**
         * The URIs that should be excluded from CSRF verification.
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
        ];
    }

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

除了检查被作为 POST 参数传递的 CSRF token 之外, `VerifyCsrfToken` 中间件也会检查请求标头中的 `X-CSRF-TOKEN`。例如，你可以将其保存在 `meta` 标签中：

    <meta name="csrf-token" content="{{ csrf_token() }}">

一旦你创建了 `meta` 标签，你就可以使用 jQuery 之类的函数库将 token 自动地添加到所有的请求头中。这简单、方便的为你的应用的 AJAX 提供了 CSRF 保护：

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel 会通过响应把当前的 CSRF token 保存在 `XSRF-TOKEN` cookie 中。你可以使用该 cookie 的值来设置 `X-XSRF-TOKEN` 请求标头。

这个 cookie 通常会以更便捷的方式传递。因为一些 JavaScript 框架会自动将它的值设置到 `X-XSRF-TOKEN` 请求标头中，如 Angular。


## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@Macken](https://phphub.org/users/1289)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/1289_1473143048.jpg?imageView2/1/w/200/h/200">  |  翻译  | 专注Web开发，[麦肯先生](https://macken.me) My Blog  |