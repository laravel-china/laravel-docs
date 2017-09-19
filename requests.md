# Laravel 的 HTTP 请求

- [获取请求](#accessing-the-request)
    - [请求路径 & 方法](#request-path-and-method)
    - [PSR-7 请求](#psr7-requests)
- [输入预处理 & 规范化](#input-trimming-and-normalization)
- [获取输入](#retrieving-input)
    - [旧输入](#old-input)
    - [Cookies](#cookies)
- [文件](#files)
    - [获取上传文件](#retrieving-uploaded-files)
    - [储存上传文件](#storing-uploaded-files)
- [配置可信代理](#configuring-trusted-proxies)

<a name="accessing-the-request"></a>
## 获取请求

要通过依赖注入的方式来获取当前 HTTP 请求的实例，你应该在控制器方法中类型提示 `Illuminate\Http\Request`。传入的请求的实例将通过 [服务容器](/docs/{{version}}/container) 自动注入：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 储存一个新用户。
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->input('name');

            //
        }
    }

#### 依赖注入 & 路由参数

如果控制器方法要从路由参数中获取数据，则应在其他依赖项之后列出路由参数。例如，如果你的路由是这样定义的：

    Route::put('user/{id}', 'UserController@update');

如下所示使用类型提示 `Illuminate\Http\Request`，就可以通过定义控制器方法获取路由参数 `id`：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 更新指定的用户。
         *
         * @param  Request  $request
         * @param  string  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

#### 通过路由闭包获取请求

你也可以在路由闭包中类型提示 `Illuminate\Http\Request` 类。服务容器在执行时会自动将当前请求注入到闭包中：

    use Illuminate\Http\Request;

    Route::get('/', function (Request $request) {
        //
    });

<a name="request-path-and-method"></a>
### 请求路径 & 方法

`Illuminate\Http\Request` 实例提供了多种方法来检查应用程序的 HTTP 请求，并继承了 `Symfony\Component\HttpFoundation\Request` 类。下面是该类几个有用的方法：

####  获取请求路径

`path` 方法返回请求的路径信息。也就是说，如果传入的请求的目标地址是 `http://domain.com/foo/bar`，那么 `path` 将会返回 `foo/bar`：

    $uri = $request->path();

`is` 方法可以验证传入的请求路径和指定规则是否匹配。使用这个方法的时，你也可以传递一个 `*` 字符作为通配符：

    if ($request->is('admin/*')) {
        //
    }

#### 获取请求的 URL

你可以使用 `url` 或 `fullUrl` 方法去获取传入请求的完整 URL。`url` 方法返回不带有查询字符串的 URL，而 `fullUrl ` 方法的返回值包含查询字符串：

    // Without Query String...
    $url = $request->url();

    // With Query String...
    $url = $request->fullUrl();

#### 获取请求方法

对于传入的请求 `method` 方法将返回 HTTP 的请求方式。你也可以使用 `isMethod` 方法去验证 HTTP 的请求方式与指定规则是否相配：

    $method = $request->method();

    if ($request->isMethod('post')) {
        //
    }

<a name="psr7-requests"></a>
### PSR-7 请求

[PSR-7 标准](http://www.php-fig.org/psr/psr-7/) 规定的 HTTP 消息接口包含了请求和响应。如果你想使用一个 PSR-7 请求来代替一个 Laravel 请求实例，那么你首先要安装几个函数库。Laravel 使用 Symfony 的 HTTP 消息桥接组件将典型的 Laravel 请求和响应转换为 PSR-7 兼容实现：

    composer require symfony/psr-http-message-bridge
    composer require zendframework/zend-diactoros

安装完这些库后， 就可以在路由闭包或控制器中类型提示请求的接口来获取 PSR-7 请求：

    use Psr\Http\Message\ServerRequestInterface;

    Route::get('/', function (ServerRequestInterface $request) {
        //
    });

> {tip} 如果从路由或者控制器返回 PSR-7 响应实例，它会自动转换回 Laravel 响应实例，并由框架显示。

<a name="input-trimming-and-normalization"></a>
## 输入预处理 & 规范化

默认情况下，Laravel 在应用程序的全局中间件堆栈中包含了 `TrimStrings` 和 `ConvertEmptyStringsToNull` 两个中间件。这些中间件由 `App\Http\Kernel` 类列在堆栈中。它们会自动处理请求上所有传入的字符串字段，并将空的字符串字段转变成 `null` 值。这样你就不用再担心路由和控制器中数据规范化的问题。

如果你想停用这些功能，可以从应用程序的中间件堆栈中删除这两个中间件，只需在 `App\Http\Kernel` 类的 `$middleware` 属性中移除它们。

<a name="retrieving-input"></a>
## 获取输入

#### 获取所有输入数据

你可以使用 `all` 方法以 `数组` 形式获取到所有输入数据:

    $input = $request->all();

#### 获取指定输入值

使用几种简单的方法（不需要特别指定哪个 HTTP 动作），就可以访问 `Illuminate\Http\Request` 实例中所有的用户输入。也就是说无论是什么样的 HTTP 动作，`input` 方法都可以被用来获取用户输入数据：

    $name = $request->input('name');

你可以给 `input` 方法的第二个参数传入一个默认值。如果请求的输入值不存在请求上，就返回默认值：

    $name = $request->input('name', 'Sally');

如果传输表单数据中包含「数组」形式的数据，那么可以使用「点」语法来获取数组：

    $name = $request->input('products.0.name');

    $names = $request->input('products.*.name');

#### 从查询字符串获取输入

使用 `input` 方法可以从整个请求中获取输入数据（包括查询字符串），而 `query` 方法可以只从查询字符串中获取输入数据：

    $name = $request->query('name');

如果请求的查询字符串数据不存在，则将返回这个方法的第二个参数：

    $name = $request->query('name', 'Helen');

你可以不提供参数调用 `query` 方法来以关联数组的形式检索所有查询字符串值：

    $query = $request->query();

#### 通过动态属性获取输入

你也可以通过 `Illuminate\Http\Request` 实例的动态属性来获取用户输入。例如，如果你应用的表单中包含 `name` 字段，那么可以像这样访问该字段的值：

    $name = $request->name;

Laravel 在处理动态属性的优先级是，先在请求的数据中查找，如果没有，再到路由参数中查找。

#### 获取 JSON 输入信息

如果发送到应用程序的请求数据是 JSON，只要请求的 `Content-Type` 标头正确设置为 `application/json`，就可以通过 `Input` 方法访问 JSON 数据。你甚至可以使用 「点」语法来读取 JSON 数组：

    $name = $request->input('user.name');

#### 获取部分输入数据

如果你需要获取输入数据的子集，则可以用 `only` 和 `except` 方法。这两个方法都接收 `数组` 或动态列表作为参数：

    $input = $request->only(['username', 'password']);

    $input = $request->only('username', 'password');

    $input = $request->except(['credit_card']);

    $input = $request->except('credit_card');

> {tip} `only` 方法会返回所有你指定的键值对，但不会返回请求中不存在的键值对。

#### 确定是否存在输入值

要判断请求是否存在某个值，可以使用 `has` 方法。如果请求中存在该值，`has` 方法就会返回 `true`：

    if ($request->has('name')) {
        //
    }

当提供一个数组作为参数时，`has` 方法将确定是否存在数组中所有给定的值：

    if ($request->has(['name', 'email'])) {
        //
    }

如果你想确定请求中是否存在值并且不为空，可以使用 `filled` 方法：

    if ($request->filled('name')) {
        //
    }

<a name="old-input"></a>
### 旧输入

Laravel 允许你将本次请求的数据保留到下一次请求发送前。如果第一次请求的表单不能通过验证，就可以使用这个功能重新填充表单。但是，如果你使用了 Laravel 的 [验证功能](/docs/{{version}}/validation)，你就不需要在手动实现这些方法，因为 Laravel 内置的验证工具会自动调用他们。

#### 将输入闪存至 Session

`Illuminate\Http\Request` 的 `flash` 方法会将当前输入的数据存进 [session](/docs/{{version}}/session) 中，以便在用户下次发送请求到应用程序之前可以使用它们：

    $request->flash();

你也可以使用 `flashOnly` 和 `flashExcept` 方法将请求数据的一部分闪存到 session。这些方法对敏感信息（例如密码）的保护非常有用：

    $request->flashOnly(['username', 'email']);

    $request->flashExcept('password');

#### 闪存输入后重定向


你可能需要把输入闪存到 session 然后重定向到上一页，这时只需要在重定向方法后加上 `withInput` 即可：

    return redirect('form')->withInput();

    return redirect('form')->withInput(
        $request->except('password')
    );

#### 获取旧输入

若要获取上一次请求中闪存的输入，则可以使用 `Request` 实例中的 `old` 方法。`old` 方法会从 [Session](/docs/{{version}}/session) 取出之前被闪存的输入数据：

    $username = $request->old('username');
Laravel 也提供了全局辅助函数 `old`。如果你要在 [Blade 模板](/docs/{{version}}/blade) 中显示旧的输入，使用 `old` 会更加方便。如果给定字段没有旧的输入，则返回 `null`：

    <input type="text" name="username" value="{{ old('username') }}">

<a name="cookies"></a>
### Cookies

#### 从请求中获取 Cookie

Laravel 框架创建的每个 cookie 都会被加密并使用验证码进行签名，这意味着如果客户端更改了它们，便视为无效。若要从请求中获取 cookie 值，你可以在 `Illuminate\Http\Request` 实例上使用 `cookie` 方法：

    $value = $request->cookie('name');

#### 将 Cookies 附加到响应

你可以使用 `cookie` 方法将 cookie 附加到传出的 `Illuminate\Http\Response` 实例。你需要传递 Cookie 名称、值、以及有效期（分钟）到这个方法：

    return response('Hello World')->cookie(
        'name', 'value', $minutes
    );

`cookie` 方法还接受一些不太频繁使用的参数。通常，这些参数与 PHP 原生 [setcookie](http://php.net/manual/en/function.setcookie.php) 方法的参数具有相同的目的和意义：

    return response('Hello World')->cookie(
        'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
    );

#### 生成 Cookie 实例

如果你想要在一段时间以后生成一个可以给定 `Symfony\Component\HttpFoundation\Cookie` 的响应实例，你可以使用全局辅助函数 `cookie`。除非此 cookie 被附加到响应实例，否则不会发送回客户端：

    $cookie = cookie('name', 'value', $minutes);

    return response('Hello World')->cookie($cookie);
<a name="files"></a>
## 文件

<a name="retrieving-uploaded-files"></a>
### 获取上传文件

你可以使用 `file` 方法或使用动态属性从 `Illuminate\Http\Request` 实例中访问上传的文件。该 `file` 方法返回一个 `Illuminate\Http\UploadedFile` 类的实例，该类继承了PHP 的 `SplFileInfo` 类的同时也提供了各种与文件交互的方法：

    $file = $request->file('photo');

    $file = $request->photo;

你可以使用 `hasFile` 方法确认请求中是否存在文件：

    if ($request->hasFile('photo')) {
        //
    }

#### 验证成功上传

除了检查上传的文件是否存在外，你也可以通过 `isValid` 方法验证上传的文件是否有效：

    if ($request->file('photo')->isValid()) {
        //
    }

#### 文件路径 & 扩展名

`UploadedFile` 类还包含访问文件的完整路径及其扩展名方法。`extension` 方法会根据文件内容判断文件的扩展名。该扩展名可能会和客户端提供的扩展名不同：

    $path = $request->photo->path();

    $extension = $request->photo->extension();

#### 其它文件方法

`UploadedFile` 实例上还有许多可用的方法，可以查看该类的 [API 文档](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html) 了解这些方法的详细信息。

<a name="storing-uploaded-files"></a>
### 存储上传文件


要存储上传的文件，先配置好 [文件系统](/docs/{{version}}/filesystem)。你可以使用 `UploadedFile` 的 `store` 方法把上传文件移动到你的某个磁盘上，该文件可能是本地文件系统中的一个位置，甚至像 Amazon S3 这样的云存储位置。

`store` 方法接受相对于文件系统配置的存储文件根目录的路径。这个路径不能包含文件名，因为系统会自动生成唯一的 ID 作为文件名。

`store` 方法还接受可选的第二个参数，用于存储文件的磁盘名称。这个方法会返回相对于磁盘根目录的文件路径：

    $path = $request->photo->store('images');

    $path = $request->photo->store('images', 's3');

如果你不想自动生成文件名，那么可以使用 `storeAs` 方法，它接受路径、文件名和磁盘名作为其参数：

    $path = $request->photo->storeAs('images', 'filename.jpg');

    $path = $request->photo->storeAs('images', 'filename.jpg', 's3');

<a name="configuring-trusted-proxies"></a>
## 配置可信代理

如果你的应用程序运行在失效的 TLS / SSL 证书的负载均衡器后，你可能会注意到你的应用程序有时不能生成 HTTPS 链接。通常这是因为你的应用程序正在从端口 80 上的负载平衡器转发流量，却不知道是否应该生成安全链接。

解决这个问题需要在 Laravel 应用程序中包含 `App\Http\Middleware\TrustProxies` 中间件，这使得你可以快速自定义应用程序信任的负载均衡器或代理。你信任的代理应该保存在这个中间件的 `$proxies` 数组中。除了配置可信代理之外，你还可以配置代理发送包含原始请求信息的请求头：

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Http\Request;
    use Fideloper\Proxy\TrustProxies as Middleware;

    class TrustProxies extends Middleware
    {
        /**
         * 这个应用程序的可信代理列表
         *
         * @var array
         */
        protected $proxies = [
            '192.168.1.1',
            '192.168.1.2',
        ];

        /**
         * 当前代理响应头映射关系
         *
         * @var array
         */
        protected $headers = [
            Request::HEADER_FORWARDED => 'FORWARDED',
            Request::HEADER_X_FORWARDED_FOR => 'X_FORWARDED_FOR',
            Request::HEADER_X_FORWARDED_HOST => 'X_FORWARDED_HOST',
            Request::HEADER_X_FORWARDED_PORT => 'X_FORWARDED_PORT',
            Request::HEADER_X_FORWARDED_PROTO => 'X_FORWARDED_PROTO',
        ];
    }

#### 信任所有代理

如果你使用 Amazon AWS 或其他的「云」负载均衡器提供程序，你可能不知道负载均衡器的实际 IP 地址。在这种情况下，你可以使用 `**` 来信任所有代理：

    /**
     * 这个应用程序的可信代理列表
     *
     * @var array
     */
    protected $proxies = '**';

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  Review  | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
