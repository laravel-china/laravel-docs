# Laravel 的 HTTP 请求 Request

- [获取请求](#accessing-the-request)
    - [请求路径 & 方法](#request-path-and-method)
    - [PSR-7 请求](#psr7-requests)
- [输入数据的预处理和规范化](#input-trimming-and-normaliation)
- [获取输入数据](#retrieving-input)
    - [旧输入数据](#old-input)
    - [Cookies](#cookies)
- [文件资源](#files)
    - [获取上传文件](#retrieving-uploaded-files)
    - [储存上传文件](#storing-uploaded-files)

<a name="accessing-the-request"></a>
## 获取请求

要通过依赖注入的方式来获取当前 HTTP 请求的实例，你应该在控制器方法中使用 `Illuminate\Http\Request` 类型提示。当前的请求实例将通过 [服务容器](/docs/{{version}}/container) 自动注入：

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

如果控制器方法也有输入数据是从路由参数中传入的，只需将路由参数置于其他依赖之后。
例如，你的路由是这样定义的： 

    Route::put('user/{id}', 'UserController@update');

只要像下方一样定义控制器方法，你就可以使用 `Illuminate\Http\Request` 类型提示了，同时获取到路由参数 `id`：

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

你也可以在一个路由闭包中使用 `Illuminate\Http\Request` 类型提示。当它执行时，服务容器会自动注入当前请求到闭包中：

    use Illuminate\Http\Request;

    Route::get('/', function (Request $request) {
        //
    });

<a name="request-path-and-method"></a>
### 请求路径 & 方法

`Illuminate\Http\Request` 的实例提供了多种方法去检查应用程序的 HTTP 请求，Laravel 的 `Illuminate\Http\Request` 继承了 `Symfony\Component\HttpFoundation\Request` 类。下面是该类几个有用的方法：

####  获取请求路径

`path` 方法返回请求路径信息。所以，如果收到的请求目标地址是 `http://domain.com/foo/bar`，那么 `path` 将会返回 `foo/bar`：

	$uri = $request->path();
	
`is` 方法可以验证收到的请求路径和指定规则是否匹配。使用这个方法的时候你也可以传递一个 `*` 字符作为通配符：

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

[PSR-7 标准](http://www.php-fig.org/psr/psr-7/)制定的 HTTP 消息接口包含了请求和响应。如果你想使用一个 PSR-7 请求来代替一个 Laravel 请求，那么你首先要安装几个函数库。Laravel 使用了 Symfony 的 HTTP 消息桥接组件，将原来的 Laravel 请求和响应转换到 PSR-7 所兼容的实现：

    composer require symfony/psr-http-message-bridge
    composer require zendframework/zend-diactoros

安装完这些库后， 就可以在路由闭包或控制器中，简单的对请求类型使用类型提示来获取 PSR-7 的请求： 

    use Psr\Http\Message\ServerRequestInterface;

    Route::get('/', function (ServerRequestInterface $request) {
        //
    });

> {tip} 如果你从路由或者控制器返回了一个 PSR-7 响应实例，那么这个实例将被自动转换回一个 Laravel 响应实例，同时由框架显示。

<a name="input-trimming-and-normaliation"></a>
## 输入数据的预处理和规范化

在 Laravel 的全局中间件中默认包含了 `TrimStrings` 和 `ConvertEmptyStringsToNull` 两个中间件。这些中间件被列在 `App\Http\Kernel` 类中。它们会自动处理所有请求中传入的字符串字段，比如将空的字符串字段转变成 `null` 值。你再也不用担心路由和控制器中数据规范化的问题。

如果你想停用这些功能，你可以在 `App\Http\Kernel` 类的 `$middleware` 属性中移除这些中间件。

<a name="retrieving-input"></a>
## 获取输入数据

#### 获取所有输入数据

你可以使用 `all` 方法以 `数组` 形式获取到所有输入数据:

    $input = $request->all();

#### 获取指定输入值

你可以通过 `Illuminate\Http\Request` 的实例，借助几个简单的方法，就可以获取到用户输入的所有数据。而不需要担心发起请求时使用了哪一种请求方式，`input` 方法通常被用来获取用户输入数据：

    $name = $request->input('name');

你可以给 `input` 方法的第二个参数传入一个默认值。当请求的输入数据不存在于此请求时，返回该默认值：

    $name = $request->input('name', 'Sally');

如果传输表单数据中包含「数组」形式的数据，那么可以使用「点」语法来获取数组：

    $name = $request->input('products.0.name');

    $names = $request->input('products.*.name');

#### 通过动态属性获取输入数据

你也可以通过 `Illuminate\Http\Request` 实例的动态属性来获取用户输入的数据。例如，如果你应用程序表单中包含 `name` 字段，那么可以像这样访问提交的值：

    $name = $request->name;

Laravel 在处理动态属性的优先级是，先从请求的数据中查找，没有的话再到路由参数中找。

#### 获取 JSON 输入信息

当你发送 JSON 请求到应用时，只要请求表头中设置了 `Content-Type` 为　`application/json`，你就可以直接从 `Input` 方法中获取 JSON 数据。你也可以通过 「点」语法来读取 JSON 数组：

    $name = $request->input('user.name');

#### 获取部分输入数据

如果你需要获取输入数据的子集，则可以用 `only` 和 `except` 方法。这两个方法都接收单个 `数组` 或动态列表作为参数：

    $input = $request->only(['username', 'password']);

    $input = $request->only('username', 'password');

    $input = $request->except(['credit_card']);

    $input = $request->except('credit_card');

`only` 方法会所有你指定的键值对，即使这个键在输入数据中并不存在。如果一个键在输入数据中并不存在时，它对应的值是 `null` 。当你想要获取请求中实际存在的输入数据时，你可以使用 `intersect` 方法。

    $input = $request->intersect(['username', 'password']);

#### 确定是否有输入值

要判断请求是否存在该数据，可以使用 `has` 方法。当数据存在　**并且**　字符串不为空时，`has` 方法就会返回 `true`：

    if ($request->has('name')) {
        //
    }

<a name="old-input"></a>
### 旧输入数据

Laravel 允许你将本次的输入数据保留到下一次请求发送前。这个特性在表单验证错误后重新填写表单相当有用。但是，如果你使用了 Laravel 的 [验证特性](/docs/{{version}}/validation)，你就不需要在手动实现这些方法，因为 Laravel 内置的验证工具会自动调用他们。

#### 将输入数据闪存至 Session

`Illuminate\Http\Request` 的 `flash` 方法会将当前输入的数据存进 [session](/docs/{{version}}/session) 中，因此下次用户发送请求到应用程序时就可以使用它们：

    $request->flash();

你也可以使用 `flashOnly` 和 `flashExcept` 方法将当前请求数据的子集保存到 session。这些方法对敏感信息的保护非常有用：

    $request->flashOnly(['username', 'email']);

    $request->flashExcept('password');

#### 闪存输入数据到 Session 后重定向


你可能需要把输入数据闪存到 session 并重定向到前一个页面，这时只需要在重定向方法后加上 `withInput` 即可：

    return redirect('form')->withInput();

    return redirect('form')->withInput(
        $request->except('password')
    );

#### 获取旧输入数据

若要获取上一次请求后所闪存的输入数据，则可以使用 `Request` 实例中的 `old` 方法。`old` 方法提供一个简便的方式从 [Session](/docs/{{version}}/session) 取出被闪存的输入数据：

    $username = $request->old('username');

Laravel 也提供了全局辅助函数 `old`。如果你要在 [Blade 模板](/docs/{{version}}/blade) 中显示旧输入数据，可以使用更加方便的 `old` 辅助函数。如果旧数据不存在，则返回 `null`：

    <input type="text" name="username" value="{{ old('username') }}">

<a name="cookies"></a>
### Cookies

#### 从请求中获取 Cookie 值

Laravel 框架创建的每个 cookie 都会被加密并且加上认证标识，这代表着用户擅自更改的 cookie 都会失效。若要从此次请求获取 cookie 值，你可以使用 `Illuminate\Http\Request` 实例中的 `cookie` 方法：

    $value = $request->cookie('name');

#### 将 Cookies 附加到响应

你可以使用 `cookie` 方法附加一个 cookie 到 `Illuminate\Http\Response` 实例。有效 cookie 应该传递字段名称，字段值和过期时间给这个方法：

    return response('Hello World')->cookie(
        'name', 'value', $minutes
    );

`cookie` 方法还可以接受更多参数，只是使用频率较低。通常作用是传递参数给 PHP 原生  [设置 cookie](http://php.net/manual/en/function.setcookie.php) 方法：

    return response('Hello World')->cookie(
        'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
    );

#### 生成 Cookie 实例

如果你想要在一段时间以后生成一个可以给定 `Symfony\Component\HttpFoundation\Cookie` 的响应实例，你可以先生成 $cookie 实例，然后再指定给 response 实例，否则这个 cookie 不会被发送回客户端：

    $cookie = cookie('name', 'value', $minutes);

    return response('Hello World')->cookie($cookie);
    
> 译者注： 关于 Cookie，需要注意一点，默认 Laravel 创建的所有 Cookie 都是加密过的，创建未加密的 Cookie 的方法请见 [【小技巧分享】在 Laravel 中设置没有加密的 cookie](https://phphub.org/topics/1758)

<a name="files"></a>
## 文件资源

<a name="retrieving-uploaded-files"></a>
### 获取上传文件

你可以使用 `Illuminate\Http\Request` 实例中的 `file` 方法获取上传的文件。file 方法返回的对象是 `Symfony\Component\HttpFoundation\File\UploadedFile` 类的实例，该类继承了 PHP 的 `SplFileInfo` 类，并提供了许多和文件交互的方法：

    $file = $request->file('photo');

    $file = $request->photo;

你可以使用请求的 `hasFile` 方法确认上传的文件是否存在：

    if ($request->hasFile('photo')) {
        //
    }

#### 确认上传的文件是否有效

除了检查上传的文件是否存在外，你也可以通过 `isValid` 方法验证上传的文件是否有效：

    if ($request->file('photo')->isValid()) {
        //
    }

#### 文件路径 & 扩展

`UploadedFile` 这个类也包含了访问文件完整路径和扩展的方法。`extension` 方法会尝试根据文件内容猜测文件的扩展名。猜测结果可能不同于客户端原始的扩展名：　

    $path = $request->photo->path();

    $extension = $request->photo->extension();

#### 其它上传文件的方法

`UploadedFile` 的实例还有许多可用的方法，可以到 [该对象的 API 文档](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html) 了解这些方法的详细信息。

<a name="storing-uploaded-files"></a>
### 储存上传文件


在设置好 [文件系统](/docs/{{version}}/filesystem) 的配置信息后，你可以使用 `UploadedFile` 的 `store` 方法把上传文件储存到本地磁盘，或者是亚马逊 S3 云存储上。

`store` 方法允许存储文件到相对于文件系统根目录配置的路径。这个路径不能包含文件名，名称将使用 MD5 散列文件内容自动生成。

`store` 方法还接受一个可选的第二个参数，用于文件存储到磁盘的名称。这个方法会返回文件相对于磁盘根目录的路径：

    $path = $request->photo->store('images');

    $path = $request->photo->store('images', 's3');

如果你不想自动生成文件名，那么可以使用 `storeAs` 方法去设置路径，文件名和磁盘名作为方法参数：

    $path = $request->photo->storeAs('images', 'filename.jpg');

    $path = $request->photo->storeAs('images', 'filename.jpg', 's3');

