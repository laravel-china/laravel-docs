# Laravel 的请求返回 Response

- [创建响应](#creating-responses)
    - [附加头信息至响应](#attaching-headers-to-responses)
    - [附加 Cookie 至响应](#attaching-cookies-to-responses)
    - [Cookies 加密](#cookies-and-encryption)
- [重定向](#redirects)
    - [重定向至命名路由](#redirecting-named-routes)
    - [重定向至控制器行为](#redirecting-controller-actions)
    - [重定向并附加 Session 闪存数据](#redirecting-with-flashed-session-data)
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

从路由和控制器不仅能返回字符串，也可以返回数组。框架也会自动地将数组转为 JSON 响应：

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

大部分的响应方法都是可链式调用的，以使你更酣畅淋漓的创建响应实例。例如，你可以在响应返回给用户前使用 `header` 方法向响应实例中附加一系列的头信息：

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
#### 附加 Cookie 至响应

通过响应对象的 cookie 方法可以让你轻松的附加 cookies 至响应。你可以使用 `cookie` 方法来生成 cookie 并附加至响应实例，像这样：

    return response($content)
                    ->header('Content-Type', $type)
                    ->cookie('name', 'value', $minutes);

`cookie` 方法也接受另外几个参数，它们的使用频率较低。通常，这些参数和给予原生 PHP 的 [setcookie](https://secure.php.net/manual/en/function.setcookie.php) 方法的参数有着相同的目的和含义：

    ->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)

<a name="cookies-and-encryption"></a>
#### Cookies 加密

默认情况下，Laravel 生成的所有 cookie 都是加密并通过签名验证的，因此他们并不能够在客户端被修改和读取。如果你想对你的应用程序生成的部分 cookie 禁用加密，可以使用 `App\Http\Middleware\EncryptCookies` 中间件的 `$except` 属性，该文件存储在 `app/Http/Middleware`：

    /**
     * 无需被加密的 cookie 名
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

有些情况下你可能希望用户重定向至上级一页面，比如，当提交表单失败时。这时可以使用 全局辅助函数 `back`。由于此功能利用了 [Session](/docs/{{version}}/session)，请确保调用 `back` 函数的路由是使用 `web` 中间件组或应用了所有的 Session 中间件：

    Route::post('user/profile', function () {
        // 验证请求...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
### 重定向至命名路由

当调用不带参数的辅助函数 `redirect` 时，会返回一个 `Illuminate\Routing\Redirector` 实例，该实例允许你调用 `Redirector` 实例的任何方法。例如，生成一个 `RedirectResponse` 重定向至一个被命名的路由时，您可以使用 `route` 方法：

    return redirect()->route('login');

如果你的路由有参数，它们可以作为 `route` 方法的第二个参数来传递：

    // 对于该 URI 的路由: profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### 通过 Eloquent 模型填充参数

如果要重定向到一个使用了 Eloquent 模型并需要传递 ID 参数的路由上，你只需传递模型本身即可，ID 会自动提取。

    // 对于此路由: profile/{id}

    return redirect()->route('profile', [$user]);

如果想要更改自动提取的路由参数的键值，你应该重写 Eloquent 模型里的 `getRouteKey` 方法：

    /**
     * 获取模型的路由键值.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### 重定向至控制器行为

你可能也会用到生成重定向至 [控制器行为](/docs/{{version}}/controllers)的响应。要实现此功能，可以向 `action` 方法传递控制器和行为名称作为参数来实现。请记住，这里并不需要指定完整的命名空间，因为 Laravel 的 `RouteServiceProvider` 会自动设置基本的控制器命名空间：
 
    return redirect()->action('HomeController@index');

如果控制器路由包含参数则需要把他们作为 `action` 函数的第二个参数传递：

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );

<a name="redirecting-with-flashed-session-data"></a>
### 重定向并附加 Session 闪存数据

重定向至一个新的 URL 的同时通常会 [附加 Session 闪存数据](/docs/{{version}}/session#flash-data)。一般来说，在控制器行为成功地执行之后才会向 Session 中闪存成功的消息。为了方便，你可以利用链式调用的方式创建一个 RedirectResponse 的实例并闪存数据至 Session：

    Route::post('user/profile', function () {
        // 更新用户的信息

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

用户重定向至指定页面后，你可以从 Session 中获取并展示闪存数据。例如，使用 [Blade 语法](/docs/{{version}}/blade)：

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

<a name="other-response-types"></a>
## 其他响应类型

使用全局辅助函数 `response` 可以轻松的生成其他类型的响应实例。当不带任何参数调用 `response` 时，将会返回 Illuminate\Contracts\Routing\ResponseFactory [Contract](/docs/{{version}}/contracts) 的实现。Contract 包含许多有用的用来辅助生成响应的方法。

<a name="view-responses"></a>
### 视图响应

如果你的响应内容不但需要控制响应状态码和响应头信息而且还需要返回一个 [视图](/docs/{{version}}/views)，这时你应该使用 `view` 方法：

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);

当然，如果不需要自定义 HTTP 状态码和响应头信息，则可使用全局的 `view` 辅助函数。

<a name="json-responses"></a>
### JSON 响应

`json` 方法会自动将 `Content-Type` 响应头信息设置为 `application/json`，并使用 PHP 的 `json_encode` 函数将数组转换为 JSON 字符串。 

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA'
    ]);

如果想要创建一个 JSONP 响应，则可以使用 `json` 方法并结合 setCallback 函数：

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));

<a name="file-downloads"></a>
### 文件下载

`download` 方法可以用于生成强制让用户的浏览器下载指定路径文件的响应。`download` 方法接受文件名称作为方法的第二个参数，此名称为用户下载文件时看见的文件名称。最后，你可以传递一个包含 HTTP 头信息的数组作为第三个参数传入该方法：

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);

> {note} 管理文件下载的扩展包 Symfony HttpFoundation，要求下载文件名必须是 ASCII 编码。

<a name="file-responses"></a>
### 文件响应

`file` 方法可以用来显示一个文件，例如图片或者 PDF，直接在用户的浏览器中显示，而不是开始下载。这个方法的第一个参数是文件的路径，第二个参数是包含头信息的数组：

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

<a name="response-macros"></a>
## 响应宏

如果你想要自定义可以在很多路由和控制器重复使用的响应，可以使用 `Response` Facade 实现的 `macro` 方法。举个例子，来自 [服务提供者](/docs/{{version}}/providers)的 `boot` 方法：

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

`macro` 函数第一个参数为宏名称，第二个参数为闭包函数。宏的闭包函数会在 `ResponseFactory` 的实现或者辅助函数 `response` 调用宏名称的时候运行：

    return response()->caps('foo');
