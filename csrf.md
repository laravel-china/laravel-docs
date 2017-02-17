# Laravel 下的伪造跨站请求保护 CSRF

- [简介](#csrf-introduction)
- [CSRF 白名单](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## 简介

Laravel 提供了简单的方法使你的应用免受 [跨站请求伪造](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) 的袭击。跨站请求伪造是一种恶意的攻击，它凭借已通过身份验证的用户身份来运行未经过授权的命令。

Laravel 为每个活跃用户的 Session 自动生成一个 CSRF 「token」。该 token 用来核实应用接收到的请求是通过身份验证的用户出于本意发送的。

任何情况下在你的应用程序中定义 HTML 表单时都应该包含 CSRF token 隐藏域，这样 CSRF 保护中间件才可以验证请求。辅助函数 `csrf_field` 可以用来生成 token 字段：

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

包含在 `web` 中间件组里的 `VerifyCsrfToken` [中间件](/docs/{{version}}/middleware)会自动验证请求里的 token 与 Session 中存储的 token 是否匹配。

<a name="csrf-excluding-uris"></a>
## CSRF 白名单

有时候你可能希望设置一组并不需要 CSRF 保护的 URI。例如，如果你正在使用 [Stripe](https://stripe.com) 处理付款并使用了他们的 webhook 系统，你会需要将 Stripe webhook 处理的路由排除在 CSRF 保护外，因为 Stripe 并不知道发送给你路由的 CSRF token 是什么。

一般地，你可以把这类路由放到 `web` 中间件外，因为 `RouteServiceProvider` 适用于 `routes/web.php` 中的所有路由。不过如果一定要这么做，你也可以将这类 URI 添加到 `VerifyCsrfToken` 中间件中的 `$except` 属性来排除对这类路由的 CSRF 保护：

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

    class VerifyCsrfToken extends BaseVerifier
    {
        /**
         * 这些 URI 会被免除 CSRF 验证
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
        ];
    }

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

除了检查 POST 参数中的 CSRF token 外，`VerifyCsrfToken` 中间件还会检查 `X-CSRF-TOKEN` 请求头。你可以将 token 保存在 HTML `meta` 标签中：

    <meta name="csrf-token" content="{{ csrf_token() }}">

一旦创建了 `meta` 标签，你就可以使用类似 jQuery 的库将 token 自动添加到所有请求的头信息中。这可以为您基于 AJAX 的应用提供简单、方便的 CSRF 保护。

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel 将当前的 CSRF token 存储在由框架生成的每个响应中包含的一个`XSRF-TOKEN` cookie 中。你可以使用该 cookie 的值来设置 X-XSRF-TOKEN 请求头信息。

这个 cookie 作为头信息发送主要是为了方便，因为一些 JacaScript 框架，如 Angular，会自动将其值添加到 `X-XSRF-TOKEN` 头中.


## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@王凯波](http://weibo.com/wangkaibo)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/1924_1487053084.jpeg?imageView2/1/w/100/h/100">  |  翻译  | 面向工资编程 😆 [@wangkaibo](https://github.com/wangkaibo/)  |
