# Laravel 的请求返回 Response

- [创建响应](#creating-responses)
    - [附加头信息到响应](#attaching-headers-to-responses)
    - [附加 Cookie 到响应](#attaching-cookies-to-responses)
    - [Cookies 和 加密](#cookies-and-encryption)
- [重定向](#redirects)
    - [重定向至命名路由](#redirecting-named-routes)
    - [重定向至控制器行为](#redirecting-controller-actions)
    - [重定向并使用 Session 闪存数据](#redirecting-with-flashed-session-data)
- [其他响应类型](#other-response-types)
    - [视图响应](#view-responses)
    - [JSON 响应](#json-responses)
    - [文件下载](#file-downloads)
    - [文件响应](#file-responses)
- [响应宏](#response-macros)

<a name="creating-responses"></a>
## 创建响应

#### 字符串 & 数组

所有路由和控制器都会返回一个响应并返回给用户的浏览器。Laravel 提供了几种不同的方式来返回响应。最基本的响应就是从路由或控制器返回一串字符串。框架会自动将字符串转换为一个完整的 HTTP 响应：

    Route::get('/', function () {
        return 'Hello World';
    });

从路由和控制器不仅能返回字符串，也可以返回数组。框架也会自动地将数组转为 JSON 响应:

    Route::get('/', function () {
        return [1, 2, 3];
    });

> {tip} 你知道吗？ [Eloquent 集合](/docs/{{version}}/eloquent-collections) 也可以从路由和控制器中直接返回，它们会自动转为 JSON 响应。试试吧！

#### 响应对象

一般来说，你不需要从路由方法返回简单的字符串或数组。而是需要返回整个 `Illuminate\Http\Response` 实例或 [视图](/docs/{{version}}/views)。

当返回整个 `Response` 实例时，Laravel 允许自定义响应的 HTTP 状态码和响应头信息。`Response` 实例继承自 `Symfony\Component\HttpFoundation\Response` 类，该类提供了丰富的构建 HTTP 响应的方法：

    Route::get('home', function () {
        return response('Hello World', 200)
                      ->header('Content-Type', 'text/plain');
    });

<a name="attaching-headers-to-responses"></a>
#### 附加头信息至响应

大部分的响应方法都是可链式调用的，使得你可以流畅地创建响应实例。例如，你可以在响应返回给用户前使用 `header` 方法向响应实例中附加一系列的头信息：

    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');

或者，你可以使用 `withHeaders` 方法来指定一个包含头信息的数组：

    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-One' => 'Header Value',
                    'X-Header-Two' => 'Header Value',
                ]);

<a name="attaching-cookies-to-responses"></a>
#### 附加 Cookie 到响应

你可以使用响应上的 `cookie` 方法轻松的附加 Cookies 到响应，例如你可以使用 `cookie` 方法生成一个 Cookie 并轻松的附加到响应上，看例子：

    return response($content)
                    ->header('Content-Type', $type)
                    ->cookie('name', 'value', $minutes);

`cookie` 方法也接受另外几个参数，它们的使用频率较低。通常，这些参数和给予原生 PHP 的 [setcookie](https://secure.php.net/manual/en/function.setcookie.php) 方法的参数有着相同的目的和含义：

    ->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)

<a name="cookies-and-encryption"></a>
#### Cookies 和 加密

默认情况下，Laravel 生成的所有 Cookie 都是加密并通过签名验证的，用此不能被客户端修改或读取。 如果您想要为应用程序生成的部分 Cookie 禁用加密，那么可以使用 `App\Http\Middleware\EncryptCookies` 中间件的 `$except` 属性，文件都在 `app/Http/Middleware` 目录中：

    /**
     * 不需要加密的 Cookie 名称。
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];

<a name="redirects"></a>
## 重定向

重定向响应是 `Illuminate\Http\RedirectResponse` 类的实例，并且包含用户需要重定向至另一个 URL 所需的头信息。Laravel 提供了许多方法用于生成 `RedirectResponse` 实例。最简单的方法是使用全局的 `redirect` 辅助函数：

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

有时候你可能希望用户重定向到上一级页面，比如提交的表单无效是。这时你可以使用 全局辅助函数 `back` 完成。由于这个功能利用了 [Session](/docs/{{version}}/session)，请确保调用 `back` 函数的路由是使用 `web` 中间件组或应用了所有的 Session 中间件：

    Route::post('user/profile', function () {
        // 验证请求...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
### 重定向至命名路由

当你调用不带参数的 `redirect` 辅助函数时，会返回 `Illuminate\Routing\Redirector` 实例，这个实例允许你调用 `Redirector` 上的任何方法。例如生产一个 `RedirectResponse` 重定向到一个 命命路由 时，可以使用 `route` 方法:

    return redirect()->route('login');

如果你的路由有参数，你可以把它们放在 `route` 方法的第二个参数来传递：

    // 对于该 URI 的路由: profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### 通过 Eloquent 模型填充参数

如果你要重定向到一个使用了 Eloquent 模型并需要传递 ID 参数的路由，则可以简单的传递模型本身，ID 会被自动提取：

    // 对于此路由: profile/{id}

    return redirect()->route('profile', [$user]);

如果您想自定义路由参数中的健，那么应该重写你 Eloquent 模型里面的 `getRouteKey` 方法：

    /**
     * 获取模型的路由键.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### 重定向至控制器行为

你可能会生成重定向到 [控制器行为](/docs/{{version}}/controllers) 的响应。要实现这个功能，可以把控制器和行为名称传递给 `action` 方法来实现。记住，您不一定要为控制器指定完整的命名空间，因为 Laravel 的 `RouteServiceProvider` 会自动设置基本的控制器命名空间：

    return redirect()->action('HomeController@index');

如果你的控制器路由需要参数，你可以把它们放在 `action` 方法的第二个参数来传递

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );

<a name="redirecting-with-flashed-session-data"></a>
### 重定向并使用 Session 闪存数据

重定向到一个新的 URL 的同时通常会 [附加 Session 闪存数据](/docs/{{version}}/session#flash-data)。通常在控制器行为执行成功后会将成功的消息传在 Session 闪存中。为了方便起见，您可以用链式的方式创建一个 `RedirectResponse` 的实例并闪存在 Session 中：

    Route::post('user/profile', function () {
        // 更新用户的信息...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

用户重定向后，您可以从 [session](/docs/{{version}}/session) 中显示闪存的消息。例如，使用 [Blade 语法](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

<a name="other-response-types"></a>
## 其他响应类型

使用 `response` 辅助函数可以用来生成其他响应实例。当 `response` 不带参数调用时，则会返回 `Illuminate\Contracts\Routing\ResponseFactory` [Contract](/docs/{{version}}/contracts) 的实例。 Contract 提供了几种辅助生成响应的方法。

<a name="view-responses"></a>
### 视图响应

如果您的响应内容不但需要控制响应状态和响应头还需要返回一个 [视图](/docs/{{version}}/views) 最为响应内容，则应该使用 `view` 方法:

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);

当然，如果您不需要自定义 HTTP 状态码或者头信息，则应该使用全局的 `view` 辅助函数。

<a name="json-responses"></a>
### JSON 响应

`json` 方法会自动把 `Content-Type` 响应头信息设置为 `application/json`，并使用 PHP `json_encode` 函数将数组转换为 JSON：

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA'
    ]);

如果你想创建一个 JSONP 响应，你可以使用 `json` 方法并与 `withCallback` 方法配合使用：

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));

<a name="file-downloads"></a>
### 文件下载

`download` 方法可以用来生成强制用户浏览器下载指定路径文件的响应。`download` 方法的第二个参数接受一个文件名称，它将作为用户下载的时候看见的文件名称。最后，你可以传递一个 HTTP 响应头数组最为该方法的第三个参数：

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);
    
    return response()->download($pathToFile)->deleteFileAfterSend(true);


> {note} 管理文件下载的扩展包 Symfony HttpFoundation，要求下载文件名必须是 ASCII 编码的。

<a name="file-responses"></a>
### 文件响应

`file` 方法可以直接在用户浏览器显示文件，例如图像或者 PDF，而不是下载。此方法的第一个参数时文件路径，第二个参数为响应头信息的数组：

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

<a name="response-macros"></a>
## 响应宏

如果你想要自定义可以在很多路由和控制器重复使用的响应，可以使用 `Response` Facade 实现的 `macro` 方法。例如，从 [服务提供者](/docs/{{version}}/providers) 的 `boot` 方法：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Response;

    class ResponseMacroServiceProvider extends ServiceProvider
    {
        /**
         * Register the application's response macros.
         *
         * @return void
         */
        public function boot()
        {
            Response::macro('caps', function ($value) {
                return Response::make(strtoupper($value));
            });
        }
    }

`macro` 函数第一个参数为宏名称，第二个参数为闭包函数。宏的闭包函数会在 `ResponseFactory` 的实现或者辅助函数 `response` 调用宏名称的时候运行：

    return response()->caps('foo');

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@Seven Du](https://github.com/medz)  | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/5564821?s=300">  |  翻译  | 基于 Laravel 的社交开源系统 [ThinkSNS+](https://github.com/slimkit/thinksns-plus) 欢迎 Star。  |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： http://d.laravel-china.org
