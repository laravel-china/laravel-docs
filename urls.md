# Laravel 的 URL 生成

- [简介](#introduction)
- [基础](#the-basics)
    - [生成基础 URL](#generating-basic-urls)
    - [访问当前 URL](#accessing-the-current-url)
- [命名路由的 URL](#urls-for-named-routes)
- [控制器行为的 URL](#urls-for-controller-actions)
- [默认值](#default-values)

<a name="introduction"></a>
## 简介

Laravel 提供了几个辅助函数来为应用程序生成 URL。主要用于在模板和 API 响应中构建 URL 或者在应用程序的其他部分生成重定向响应。

<a name="the-basics"></a>
## 基础

<a name="generating-basic-urls"></a>
### 生成基础 URL

辅助函数 `url` 可以用于应用的任何一个 URL。生成的 URL 将自动使用当前请求中的方案（ HTTP 或 HTTPS ）和主机：

    $post = App\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>
### 访问当前 URL

如果没有给辅助函数 `url` 提供路径，则会返回一个 `Illuminate\Routing\UrlGenerator` 实例，来允许你访问有关当前 URL 的信息：

    // 获取没有查询字符串的当前的 URL ...
    echo url()->current();

    // 获取包含查询字符串的当前的 URL ...
    echo url()->full();

    // 获取上一个请求的完整的 URL...
    echo url()->previous();

上面的这些方法都可以通过 `URL` [facade](/docs/{{version}}/facades) 访问:

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
##  命名路由的 URL

辅助函数 `route` 可以用于为指定路由生成 URL。命名路由生成的 URL 不与路由上定义的 URL 相耦合。因此，就算路由的 URL 有任何更改，都不需要对 `route` 函数调用进行任何更改。例如，假设你的应用程序包含以下路由：

    Route::get('/post/{post}', function () {
        //
    })->name('post.show');

要生成此路由的 URL，可以像这样使用辅助函数 `route`：

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1
将 [Eloquent 模型](/docs/{{version}}/eloquent) 作为参数值传给 `route` 方法，它会自动提取模型的主键来生成 URL。

    echo route('post.show', ['post' => $post]);

<a name="urls-for-controller-actions"></a>
## 控制器行为的 URL

`action` 功能可以为给定的控制器行为生成 URL。这个功能不需要你传递控制器的完整命名空间，但你需要传递相对于命名空间 `App\Http\Controllers` 的控制器类名：

    $url = action('HomeController@index');
如果控制器方法需要路由参数，那就将它们作为第二个参数传递给 `action` 函数：

    $url = action('UserController@profile', ['id' => 1]);

<a name="default-values"></a>
## 默认值

For some applications, you may wish to specify request-wide default values for certain URL parameters. For example, imagine many of your routes define a `{locale}` parameter:

对于某些应用程序，你可能希望为某些 URL 参数的请求范围指定默认值。例如，假设有些路由定义了 `{locale}` 参数：

    Route::get('/{locale}/posts', function () {
        //
    })->name('post.index');

每次都通过 `locale` 来调用辅助函数 `route` 也是一件很麻烦的事情。因此，使用 `URL::defaults` 方法定义这个参数的默认值，可以让该参数始终存在当前请求中。然后就能从 [路由中间件](/docs/{{version}}/middleware#assigning-middleware-to-routes) 调用此方法来访问当前请求：

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Support\Facades\URL;

    class SetDefaultLocaleForUrls
    {
        public function handle($request, Closure $next)
        {
            URL::defaults(['locale' => $request->user()->locale]);

            return $next($request);
        }
    }

一旦设置了 `locale` 参数的默认值，就不需要在使用辅助函数 `route` 生成 URL 时传递这个值。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
| --- | --- | --- | --- |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | 翻译 | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
