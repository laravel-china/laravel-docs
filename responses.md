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

The `cookie` method also accepts a few more arguments which are used less frequently. Generally, these arguments have the same purpose and meaning as the arguments that would be given to PHP's native [setcookie](https://secure.php.net/manual/en/function.setcookie.php) method:

`cookie` 方法也接受另外几个参数，它们的使用频率较低。通常，这些参数和给予原生 PHP 方法的参数有着相同的目的和含义：

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

重定向响应是 `Illuminate\Http\RedirectResponse` 类的实例，并且包含用户需要重定向至另一个 URL 所需的头信息。Laravel 提供了许多方法用于生成 `RedirectResponse` 实例。最简单的方法是使用全局辅助函数 `redirect`：

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

有些情况下你可能希望用户重定向至上级一页面，比如，当提交表单失败时。这时可以使用 全局辅助函数 `back`。由于此功能利用了 [session](/docs/{{version}}/session)，请确保调用 `back` 函数的路由是使用 `web` 中间件组或应用了所有的 session 中间件：

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

If you are redirecting to a route with an "ID" parameter that is being populated from an Eloquent model, you may simply pass the model itself. The ID will be extracted automatically:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', [$user]);

If you would like to customize the value that is placed in the route parameter, you should override the `getRouteKey` method on your Eloquent model:

    /**
     * Get the value of the model's route key.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### Redirecting To Controller Actions

You may also generate redirects to [controller actions](/docs/{{version}}/controllers). To do so, pass the controller and action name to the `action` method. Remember, you do not need to specify the full namespace to the controller since Laravel's `RouteServiceProvider` will automatically set the base controller namespace:

    return redirect()->action('HomeController@index');

If your controller route requires parameters, you may pass them as the second argument to the `action` method:

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );

<a name="redirecting-with-flashed-session-data"></a>
### Redirecting With Flashed Session Data

Redirecting to a new URL and [flashing data to the session](/docs/{{version}}/session#flash-data) are usually done at the same time. Typically, this is done after successfully performing an action when you flash a success message to the session. For convenience, you may create a `RedirectResponse` instance and flash data to the session in a single, fluent method chain:

    Route::post('user/profile', function () {
        // Update the user's profile...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

After the user is redirected, you may display the flashed message from the [session](/docs/{{version}}/session). For example, using [Blade syntax](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

<a name="other-response-types"></a>
## Other Response Types

The `response` helper may be used to generate other types of response instances. When the `response` helper is called without arguments, an implementation of the `Illuminate\Contracts\Routing\ResponseFactory` [contract](/docs/{{version}}/contracts) is returned. This contract provides several helpful methods for generating responses.

<a name="view-responses"></a>
### View Responses

If you need control over the response's status and headers but also need to return a [view](/docs/{{version}}/views) as the response's content, you should use the `view` method:

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);

Of course, if you do not need to pass a custom HTTP status code or custom headers, you should use the global `view` helper function.

<a name="json-responses"></a>
### JSON Responses

The `json` method will automatically set the `Content-Type` header to `application/json`, as well as convert the given array to JSON using the `json_encode` PHP function:

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA'
    ]);

If you would like to create a JSONP response, you may use the `json` method in combination with the `withCallback` method:

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));

<a name="file-downloads"></a>
### File Downloads

The `download` method may be used to generate a response that forces the user's browser to download the file at the given path. The `download` method accepts a file name as the second argument to the method, which will determine the file name that is seen by the user downloading the file. Finally, you may pass an array of HTTP headers as the third argument to the method:

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);

> {note} Symfony HttpFoundation, which manages file downloads, requires the file being downloaded to have an ASCII file name.

<a name="file-responses"></a>
### File Responses

The `file` method may be used to display a file, such as an image or PDF, directly in the user's browser instead of initiating a download. This method accepts the path to the file as its first argument and an array of headers as its second argument:

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

<a name="response-macros"></a>
## Response Macros

If you would like to define a custom response that you can re-use in a variety of your routes and controllers, you may use the `macro` method on the `Response` facade. For example, from a [service provider's](/docs/{{version}}/providers) `boot` method:

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

The `macro` function accepts a name as its first argument, and a Closure as its second. The macro's Closure will be executed when calling the macro name from a `ResponseFactory` implementation or the `response` helper:

    return response()->caps('foo');
