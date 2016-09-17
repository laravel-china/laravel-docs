# HTTP 响应

- [创建响应](#creating-responses)
    - [附加标头至响应](#attaching-headers-to-responses)
    - [附加 Cookies 至响应](#attaching-cookies-to-responses)
    - [加密 Cookie](#cookies-and-encryption)
- [重定向](#redirects)
    - [重定向至命名路由](#redirecting-named-routes)
    - [重定向至控制器行为](#redirecting-controller-actions)
    - [重定向并加上 Session 闪存数据](#redirecting-with-flashed-session-data)
- [其它响应类型](#other-response-types)
    - [视图响应](#view-responses)
    - [JSON 响应](#json-responses)
    - [文件下载](#file-downloads)
    - [文件响应](#file-responses)
- [响应宏](#response-macros)

<a name="creating-responses"></a>
## 创建响应

#### 字符串 & 数组

所有的路由及控制器必须返回某个类型的响应，并发送回用户的浏览器。Laravel 提供了几种不同的方法来返回响应。最基本的响应就是从路由或控制器简单的返回一个字符串。指定的字符串会被框架自动转换成 HTTP 响应：

    Route::get('/', function () {
        return 'Hello World';
    });

从路由和控制器不仅可以返回字符串，也可以直接返回数组，这个数组将会被自动转换为 JSON 响应：

    Route::get('/', function () {
        return [1, 2, 3];
    });
    
> {tip} 在路由或控制器中也可以返回 [Eloquent 集合](/docs/{{version}}/eloquent-collections)，集合将被自动转换为 JSON 数据，试一下吧。

#### 响应对象

通常情况下，不需要从路由方法返回字符串或数组，需要的是返回完整的 `Illuminate\Http\Response` 实例或是一个 [视图](/docs/{{version}}/views)。

返回一个完整的 `Response` 实例时，就能够自定义响应的 HTTP 状态码以及标头。`Response` 实例继承了 `Symfony\Component\HttpFoundation\Response` 类，其提供了很多创建 HTTP 响应的方法：

    Route::get('home', function () {
        return response('Hello World', 200)
                      ->header('Content-Type', 'text/plain');
    });

> **注意：**有关 `Response` 方法的完整列表可以参照 [API 文档](http://laravel-china.org/api/master/Illuminate/Http/Response.html) 以及 [Symfony API 文档](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/Response.html)。

<a name="attaching-headers-to-responses"></a>
#### 附加标头至响应

大部份的响应方法是可链式调用的，这让你可以顺畅的创建响应。举例来说，你可以在响应发送给用户之前，使用 `header` 方法增加一系列的标头至响应：

    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');

或者你可以使用 `withHeaders` 来设置数组标头：

    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-One' => 'Header Value',
                    'X-Header-Two' => 'Header Value',
                ]);

<a name="attaching-cookies-to-responses"></a>
#### 附加 Cookies 至响应

通过响应实例的 `cookie` 辅助方法可以让你轻松的附加 cookies 至响应。举个例子，你可以使用 `cookie` 方法来生成 cookie 并附加至响应实例：

    return response($content)
                    ->header('Content-Type', $type)
                    ->cookie('name', 'value', $minutes);

`cookie` 方法可以接受额外的可选参数，传递给 PHP 原生 [设置 cookie](http://php.net/manual/en/function.setcookie.php) 方法：

    ->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)

<a name="cookies-and-encryption"></a>
#### Cookie 与加密

默认情况下，所有 Laravel 生成的 cookies 都会被加密并加上认证标识，因此无法被用户读取及修改。如果你想停止对某个 cookies 的加密，则可以利用 `App\Http\Middleware\EncryptCookies` 中间件的 `$except` 属性：
    /**
     * 无需被加密的 cookies 名称。
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];

<a name="redirects"></a>
## 重定向

重定向响应是类 `Illuminate\Http\RedirectResponse` 的实例，并且包含用户要重定向至另一个 URL 所需的标头。有几种方法可以生成 `RedirectResponse` 的实例。最简单的方式就是通过全局的 `redirect` 辅助函数：

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

有时你可能希望将用户重定向至前一个位置，例如当提交一个无效的表单之后。这时可以使用全局的 `back` 辅助函数来达成这个目的。保证路由调用的 `back` 函数使用了 `web` 中间件组或在所有的 session 中间件添加：

    Route::post('user/profile', function () {
        // 验证请求。。。

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
### 重定向至命名路由

当你调用 `redirect` 辅助函数且不带任何参数时，将会返回 `Illuminate\Routing\Redirector` 的实例，你可以对该 `Redirector` 的实例调用任何方法。举个例子，要生成一个 `RedirectResponse` 到一个命名路由，你可以使用 `route` 方法：

    return redirect()->route('login');

如果你的路由有参数，则可以将参数放进 `route` 方法的第二个参数：

    // 重定向到以下 URI: profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### 通过 Eloquent 模型填充参数

如果重定向到一个使用了 Eloquent 模型传递 ID 的路由上，你可以通过 ID 自动提取并传递这个模型：

    // 通过下面的 URI 参数定向到路由: profile/{id}

    return redirect()->route('profile', [$user]);

如果要自定义路由参数，则在模型类中重写 `getRouteKey` 方法：

    /**
     * 获取模型的路由键值。
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### 重定向至控制器行为

你可能也会希望生成重定向至 [控制器的行为](/docs/{{version}}/controllers)。要做到这一点，只需传递控制器及行为名称至 `action` 方法。请记得，你不需要指定完整的命名空间，因为 Laravel 的 `RouteServiceProvider` 会自动设置默认的控制器命名空间：

    return redirect()->action('HomeController@index');

当然，如果你的控制器路由需要参数的话，你可以传递它们至 `action` 方法的第二个参数：

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );

<a name="redirecting-with-flashed-session-data"></a>
### 重定向并加上 Session 闪存数据

通常重定向至新的 URL 时会一并 [写入闪存数据至 session](/docs/{{version}}/session#flash-data)。所以为了方便，你可以利用链式调用的方式创建一个 `RedirectResponse` 的实例 **并** 闪存数据至 Session。这对于在一个动作之后保存状态消息相当方便：

    Route::post('user/profile', function () {
        // 更新用户的配置文件。。。

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

当然，在用户重定向至新的页面后，你可以获取并显示 [session](/docs/{{version}}/session) 的闪存数据。举个例子，使用 [Blade 的语法](/docs/{{version}}/blade)：

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

<a name="other-response-types"></a>
## 其它响应类型

使用辅助函数 `response` 可以轻松的生成其它类型的响应实例。当你调用辅助函数 `response` 且不带任何参数时，将会返回 `Illuminate\Contracts\Routing\ResponseFactory` [contract](/docs/{{version}}/contracts) 的实现。此 Contract 提供了一些有用的方法来生成响应。

<a name="view-responses"></a>
### 视图响应

如果你想要控制响应状态码及标头，同时也想要返回一个 [视图](/docs/{{version}}/views) 作为返回的内容时，则可以使用 `view` 方法：

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);

当然，如果你没有自定义 HTTP 状态码及标头的需求，则可以简单的使用全局的 `view` 辅助函数。

<a name="json-responses"></a>
### JSON 响应

`json` 方法会自动将标头的 `Content-Type` 设置为 `application/json`，并通过 PHP 的 `json_encode` 函数将指定的数组转换为 JSON：

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA'
    ]);

如果你想创建一个 JSONP 响应，则可以使用 `json` 方法并加上 `setCallback`：

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));

<a name="file-downloads"></a>
### 文件下载

`download` 方法可以用于生成强制让用户的浏览器下载指定路径文件的响应。`download` 方法接受文件名称作为方法的第二个参数，此名称为用户下载文件时看见的文件名称。最后，你可以传递一个 HTTP 标头的数组作为第三个参数传入该方法：

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);

> **注意：**管理文件下载的扩展包 Symfony HttpFoundation，要求下载文件必须是 ASCII 文件名。

<a name="file-responses"></a>
### 文件响应

`file` 方法可以被用来显示一个文件，例如图片或者 PDF，直接在用户的浏览器中显示，而不是下载。这个方法的第一个参数是文件的路径，第二个参数是表头数组：

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

<a name="response-macros"></a>
## 响应宏

如果你想要自定义可以在很多路由和控制器重复使用的响应，可以使用 `Illuminate\Contracts\Routing\ResponseFactory` 实现的方法 `macro`。举个例子，来自 [服务提供者的](/docs/{{version}}/providers) `boot` 方法：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Response;

    class ResponseMacroServiceProvider extends ServiceProvider
    {
        /**
         * 注册应用的响应宏
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

`macro` 函数第一个参数为宏名称，第二个参数为闭包函数。宏的闭包函数会在 `ResponseFactory` 的实现或者辅助函数 `response` 调用宏名称的时候被运行：

    return response()->caps('foo');
