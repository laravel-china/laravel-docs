# HTTP 请求

- [获取请求](#accessing-the-request)
    - [基本请求信息](#basic-request-information)
    - [PSR-7 请求](#psr7-requests)
- [获取输入数据](#retrieving-input)
    - [旧输入数据](#old-input)
    - [Cookies](#cookies)
    - [上传文件](#files)

<a name="accessing-the-request"></a>
## 获取请求

要通过依赖注入的方式获取 HTTP 请求的实例，就必须在控制器的构造器或方法中，使用 `Illuminate\Http\Request` 类型提示。当前的请求实例便会自动由 [服务容器](/docs/{{version}}/container)注入：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class UserController extends Controller
    {
        /**
         * 保存新的用户。
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

如果控制器方法也有输入数据是从路由参数传入的，只需将路由参数置于其它依赖之后。举例来说，如果你的路由是这样定义的：

    Route::put('user/{id}', 'UserController@update');

只要像下方一样定义控制器方法，就可以使用 `Illuminate\Http\Request` 类型提示，同时获取到你的路由参数 `id`：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class UserController extends Controller
    {
        /**
         * 更新指定的用户。
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

<a name="basic-request-information"></a>
### 基本请求信息

`Illuminate\Http\Request` 的实例提供了多种方法来用于检查应用程序的 HTTP 请求。Larevel 的 `Illuminate\Http\Request` 继承了 `Symfony\Component\HttpFoundation\Request` 类。下方是该类的几个有用的方法：

#### 获取请求的 URI

`path` 方法会返回请求的 URI。所以，如果接收到的请求目标是 `http://domain.com/foo/bar`，那么 `path` 方法就会返回 `foo/bar`：

    $uri = $request->path();

`is` 方法可以验证接收到的请求 URI 与指定的规则是否相匹配。使用此方法时你可以将 `*` 符号作为通配符：

    if ($request->is('admin/*')) {
        //
    }

使用 `url` 方法，可以获取完整的网址：

    $url = $request->url();

#### 获取请求的方法

`method` 方法会返回此次请求的 HTTP 动作。也可以通过 `isMethod` 方法来验证 HTTP 动作和指定的字符串是否相匹配：

    $method = $request->method();

    if ($request->isMethod('post')) {
        //
    }

<a name="psr7-requests"></a>
### PSR-7 请求

PSR-7 标准制定的 HTTP 消息接口包含了请求及响应。如果你想获得一个 PSR-7 的请求实例，你就必须先安装几个函数库。Laravel 使用 Symfony 的 HTTP 消息桥接组件，将原 Laravel 的请求及响应转换至 PSR-7 所支持的实现：

    composer require symfony/psr-http-message-bridge

    composer require zendframework/zend-diactoros

安装完这些函数库后，就可以在路由或控制器中，简单的对请求类型使用类型提示来获取 PSR-7 的请求：

    use Psr\Http\Message\ServerRequestInterface;

    Route::get('/', function (ServerRequestInterface $request) {
        //
    });

如果你从路由或控制器返回一个 PSR-7 的响应实例，那么它会被框架自动转换回 Laravel 的响应实例并显示。

<a name="retrieving-input"></a>
## 获取输入数据

#### 获取特定输入值

你可以通过 `Illuminate\Http\Request` 的实例，借助几个简洁的方法获取所有的用户输入数据。而不需要去担心发出请求时使用的 HTTP 动作，因为它们获取输入数据的方式都是相同的。

    $name = $request->input('name');

此外，可以使用 `Illuminate\Http\Request` 的属性访问用户输入。例如，如果你应用程序的表单含有一个 `name` 字段，你可以从传递的字段访问它的值，像这样：

    $name = $request->name;

你可以在 `input` 方法的第二个参数中传入一个默认值。当请求的输入数据不存在于此次请求时，就会返回默认值：

    $name = $request->input('name', 'Sally');

如果是「数组」形式的输入数据，则可以使用「点」语法来获取数组：

    $input = $request->input('products.0.name');

#### 确认是否有输入值

要判断数据是否存在于此次请求，可以使用 `has` 方法。当该数据存在 **并且** 字符串不为空时，`has` 方法就会传回 `true`：

    if ($request->has('name')) {
        //
    }

#### 获取所有输入数据

你也可以使用 `all` 方法以 `数组` 形式获取到所有输入数据：

    $input = $request->all();

#### 获取部分输入数据

如果你想获取输入数据的子集，则可以使用 `only` 及 `except` 方法。这两个方法都接受单个`数组`或是动态列表作为参数：

    $input = $request->only(['username', 'password']);

    $input = $request->only('username', 'password');

    $input = $request->except(['credit_card']);

    $input = $request->except('credit_card');

<a name="old-input"></a>
### 旧输入数据

Laravel 可以让你将本次的输入数据保留到下一次请求发送前。对于在表单验证失败后重新填入表单值相当有用。当然，如果你使用 Laravel 的 [验证服务](/docs/{{version}}/validation)，你就不需要再手动使用这些方法，因为 Laravel 的一些内置验证功能会自动调用它们。

#### 将输入数据闪存至 Session

`Illuminate\Http\Request` 实例的 `flash` 方法会将当前的输入数据存进 [Session](/docs/{{version}}/session) 中，所以下次用户发出请求至应用程序时就可以使用它们：

    $request->flash();

你也可以使用 `flashOnly` 及 `flashExcept` 方法将请求数据的子集保存至 Session：

    $request->flashOnly('username', 'email');

    $request->flashExcept('password');

#### 闪存输入数据至 Session 后重定向

你可能需要将输入数据闪存并重定向至前一页，这时只要在重定向方法后加上 `withInput` 即可：

    return redirect('form')->withInput();

    return redirect('form')->withInput($request->except('password'));

#### 获取旧输入数据

若要获取上一次请求后所闪存的输入数据，则可以使用 `Request` 实例中的 `old` 方法。`old` 方法提供一个简便的方式从 [Session](/docs/{{version}}/session) 取出被闪存的输入数据：

    $username = $request->old('username');

Laravel 也提供了全局辅助函数 `old`。如果你要在 [Blade 模板](/docs/{{version}}/blade) 中显示旧输入数据，可以使用更加方便的 `old` 辅助函数：

    {{ old('username') }}

<a name="cookies"></a>
### Cookies

#### 从请求取出 Cookie 值

Laravel 框架创建的每个 cookie 都会被加密并且加上认证标识，这代表着用户擅自更改的 cookie 都会失效。若要从此次请求获取 cookie 值，你可以使用 `Illuminate\Http\Request` 实例中的 `cookie` 方法：

    $value = $request->cookie('name');

#### 将新的 Cookie 附加到响应

Laravel 提供了全局辅助函数 `cookie`，可通过简易的工厂生成新的 `Symfony\Component\HttpFoundation\Cookie` 实例。可以在 `Illuminate\Http\Response` 实例之后加上 `withCookie` 方法来把 cookie 附加至响应：

    $response = new Illuminate\Http\Response('Hello World');

    $response->withCookie(cookie('name', 'value', $minutes));

    return $response;

如果要创建一个可长期存在，为期五年的 cookie，可以先调用 `cookie` 辅助函数且不带入任何参数，再使用 cookie 工厂的 `forever` 方法，接着将 `forever` 方法拼接在返回的 cookie 工厂之后：

    $response->withCookie(cookie()->forever('name', 'value'));

> 译者注： 关于 Cookie，需要注意一点，默认 Laravel 创建的所有 Cookie 都是加密过的，创建未加密的 Cookie 的方法请见 [【小技巧分享】在 Laravel 中设置没有加密的 cookie](https://phphub.org/topics/1758)

<a name="files"></a>
### 上传文件

#### 获取上传文件

你可以使用 `Illuminate\Http\Request` 实例中的 `file` 方法获取上传的文件。file 方法返回的对象是 `Symfony\Component\HttpFoundation\File\UploadedFile` 类的实例，该类继承了 PHP 的 `SplFileInfo` 类，并提供了许多和文件交互的方法：

    $file = $request->file('photo');

#### 确认文件是否有上传

你可以使用请求的 `hasFile` 方法确认上传的文件是否存在：

    if ($request->hasFile('photo')) {
        //
    }

#### 确认上传的文件是否有效

除了检查上传的文件是否存在外，你也可以通过 `isValid` 方法验证上传的文件是否有效：

    if ($request->file('photo')->isValid()) {
        //
    }

#### 移动上传的文件

若要移动上传的文件至新位置，则必须使用 `move` 方法。该方法会将文件从缓存位置（由你的 PHP 配置决定）移动至你指定的永久保存位置：

    $request->file('photo')->move($destinationPath);

    $request->file('photo')->move($destinationPath, $fileName);

#### 其它上传文件的方法

`UploadedFile` 的实例还有许多可用的方法，可以到 [该对象的 API 文档](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/File/UploadedFile.html) 了解这些方法的详细信息。





--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译。
> 
> 文档永久地址： http://d.laravel-china.org