# 测试应用程序

- [简介](#introduction)
- [与你的应用程序进行交互](#interacting-with-your-application)
    - [与链接交互](#interacting-with-links)
    - [与表单交互](#interacting-with-forms)
- [测试 JSON APIs](#testing-json-apis)
    - [验证完全匹配的 JSON](#verifying-exact-match)
    - [验证 JSON 的结构匹配](#verifying-structural-match)
- [Sessions 和认证](#sessions-and-authentication)
- [停用中间件](#disabling-middleware)
- [自定义 HTTP 请求](#custom-http-requests)
- [PHPUnit 断言](#phpunit-assertions)

<a name="introduction"></a>
## 简介

Laravel 为 HTTP 请求的生成和发送操作、输出的检查，甚至表单的填写都提供了非常流利的 API。举例来说，你可以看看下面的这个测试用例：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        /**
         * 基本的测试用例。
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->visit('/')
                 ->see('Laravel 5')
                 ->dontSee('Rails');
        }
    }

`visit` 方法会创建一个 `GET` 请求，`see` 方法则断言在返回的响应中会有指定的文本，`dontSee` 方法断言在返回的响应中不会有指定的文本。这是 Laravel 所提供的最基本的应用程序测试。

你也可以使用 `visitRoute` 方法来创建一个 `GET` 请求访问命名路由。

    $this->visitRoute('profile');

    $this->visitRoute('profile', ['user' => 1]);

<a name="interacting-with-your-application"></a>
## 与你的应用程序进行交互

当然，除了断言文本是否存在于一个指定的响应中，你还可以做更多的交互。让我们来看看点击链接及填写表单的例子：

<a name="interacting-with-links"></a>
### 与链接交互

在这个测试中，我们会生成一个请求并「点击」返回响应中的链接，接着断言我们会停留在指定的 URI 上。举个例子，假设在响应中有个链接，并写着「About Us」：

    <a href="/about-us">About Us</a>

我们可以编写一个测试来填写此表单，并检查结果：

    public function testBasicExample()
    {
        $this->visit('/')
             ->click('About Us')
             ->seePageIs('/about-us');
    }

你也可以使用 `seeRouteIs` 方法来检查用户是否跳转到了正确的路由：

    ->seeRouteIs('profile', ['user' => 1]);

<a name="interacting-with-forms"></a>
### 与表单交互

Laravel 还提供了几种用于测试表单的方法。通过 `type`、`select`、`check`、`attach` 及 `press` 方法让你与表单的所有输入框进行交互。举个例子，设想有一个在应用程序注册页面的表单：

    <form action="/register" method="POST">
        {{ csrf_field() }}

        <div>
            Name: <input type="text" name="name">
        </div>

        <div>
            <input type="checkbox" value="yes" name="terms"> Accept Terms
        </div>

        <div>
            <input type="submit" value="Register">
        </div>
    </form>

我们可以编写一个测试来填写此表单，并检查结果：

    public function testNewUserRegistration()
    {
        $this->visit('/register')
             ->type('Taylor', 'name')
             ->check('terms')
             ->press('Register')
             ->seePageIs('/dashboard');
    }

当然，如果你的表单中包含了类似单选框或下拉式菜单的其它输入框，也可以很轻松的填入这些类型的区域。以下是每个表单的方法操作列表：

方法  | 说明
------------- | -------------
`$this->type($text, $elementName)`  |  「输入（type）」文本在一个指定的区域
`$this->select($value, $elementName)`  |  「选择（select）」一个单选框或下拉式菜单的区域
`$this->check($elementName)`  |  「勾选（check）」一个复选框的区域
`$this->uncheck($elementName)`  |  「取消勾选（uncheck）」一个复选框的区域
`$this->attach($pathToFile, $elementName)`  |  「附加（attach）」一个文件至表单
`$this->press($buttonTextOrElementName)`  |  「按下（press）」一个指定文本或名称的按钮 text or name.

<a name="file-inputs"></a>
#### 文件上传

如果你的表单包含 `上传文件` 的输入框类型，则可以使用 `attach` 方法来附加文件：

    public function testPhotoCanBeUploaded()
    {
        $this->visit('/upload')
             ->attach($pathToFile, 'photo')
             ->press('Upload')
             ->see('Upload Successful!');
    }

<a name="testing-json-apis"></a>
### 测试 JSON APIs

Laravel 也提供了几个辅助函数来测试 JSON API 及其响应。举例来说，`get`、`post`、`put`、`patch` 及 `delete` 方法可以用于发出各种 HTTP 动作的请求。你也可以轻松的传入数据或标头到这些方法上。首先，让我们来编写一个测试，将 `POST` 请求发送至 `/user` ，并断言其会返回 JSON 格式的指定数组：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 基本的功能测试示例。
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->json('POST', '/user', ['name' => 'Sally'])
                 ->seeJson([
                     'created' => true,
                 ]);
        }
    }

> {tip} `seeJson` 方法会将传入的数组转换成 JSON，并验证该 JSON 片段是否存在于应用程序返回的 JSON 响应中的 **任何位置**。也就是说，即使有其它的属性存在于该 JSON 响应中，但是只要指定的片段存在，此测试便会通过。

<a name="verifying-exact-match"></a>
### 验证完全匹配的 JSON

如果你想验证传入的数组是否与应用程序返回的 JSON **完全** 匹配，则可以使用 `seeJsonEquals` 方法：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 基本的功能测试示例。
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->json('POST', '/user', ['name' => 'Sally'])
                 ->seeJsonEquals([
                     'created' => true,
                 ]);
        }
    }

<a name="verifying-structural-match"></a>
### 验证 JSON 的结构匹配

你可以使用 `seeJsonStructure` 来验证 JSON 结构是否符合你的预期：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 基本的功能测试示例。
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->get('/user/1')
                 ->seeJsonStructure([
                     'name',
                     'pet' => [
                         'name', 'age'
                     ]
                 ]);
        }
    }

上面这个例子展示接受到的数据会有一个 `name` 属性和一个拥有 `name` 和 `age` 属性的嵌套 `pet` 对象。`seeJsonStructure` 并不会因为响应结果中包含额外的属性而失败。比如，即使响应中 `pet` 对象含有一个 `weight` 属性，测试依旧会通过。

你可以使用 `*` 来假设所有的列表内容都包含至少数组里面的内容：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 基本的功能测试示例。
         *
         * @return void
         */
        public function testBasicExample()
        {
            // 每一个列表里至少拥有 id, name 和 email 属性
            $this->get('/users')
                 ->seeJsonStructure([
                     '*' => [
                         'id', 'name', 'email'
                     ]
                 ]);
        }
    }

你也可以灵活的使用 `*` 做嵌套。在下面这个实例中，我们可以断言在响应数据中每个 user 都包含指定的属性，而且用户的每个 pet 对象也包含指定的属性集：

    $this->get('/users')
         ->seeJsonStructure([
             '*' => [
                 'id', 'name', 'email', 'pets' => [
                     '*' => [
                         'name', 'age'
                     ]
                 ]
             ]
         ]);

<a name="sessions-and-authentication"></a>
### Sessions 和认证

Laravel 提供了几个可在测试时使用 Session 的辅助函数。首先，你需要设置 Session 数据至指定的数组中，并使用 `withSession` 方法。在应用程序的测试请求发送之前，可先给数据加载 session：

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $this->withSession(['foo' => 'bar'])
                 ->visit('/');
        }
    }

当然，一般使用 Session 时都是用于维持用户的状态，如认证用户。`actingAs` 辅助函数提供了简单的方式来让指定的用户认证为当前的用户。举个例子，我们可以使用 [模型工厂](#model-factories) 来生成并认证用户：

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $user = factory(App\User::class)->create();

            $this->actingAs($user)
                 ->withSession(['foo' => 'bar'])
                 ->visit('/')
                 ->see('Hello, '.$user->name);
        }
    }

你也可以使用 `actingAs` 来指定用户使用哪个 guard：

    $this->actingAs($user, 'api')

<a name="disabling-middleware"></a>
### 停用中间件

测试应用程序时，你会发现，在某些测试中停用 [中间件](/docs/{{version}}/middleware) 是很方便的。这让你可以隔离任何中间件的所有影响，以便更好的测试路由及控制器。Laravel 包含了一个简洁的 `WithoutMiddleware` trait，你能在测试类中使用它来自动停用所有的中间件：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        use WithoutMiddleware;

        //
    }

如果你只想要在某几个测试方法中停用中间件，则可以在测试方法中调用 `withoutMiddleware` 方法：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 基本的功能测试示例。
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->withoutMiddleware();

            $this->visit('/')
                 ->see('Laravel 5');
        }
    }

<a name="custom-http-requests"></a>
### 自定义 HTTP 请求

如果你想要创建一个自定义的 HTTP 请求到应用程序上，并获取完整的 `Illuminate\Http\Response` 对象，则可以使用 `call` 方法：

    public function testApplication()
    {
        $response = $this->call('GET', '/');

        $this->assertEquals(200, $response->status());
    }

如果你创建的是 `POST`、`PUT`、或是 `PATCH` 请求，则可以在请求时传入一个数组作为输入数据。当然，你也可以在路由及控制器中通过 [请求实例](/docs/{{version}}/requests) 取用这些数据：

       $response = $this->call('POST', '/user', ['name' => 'Taylor']);

<a name="phpunit-assertions"></a>
### PHPUnit 断言

方法  | 描述
------------- | -------------
`->assertResponseOk();`  |  断言客户端的响应拥有 OK 状态码。
`->assertResponseStatus($code);`  | 断言客户端的响应拥有指定的状态码。
`->assertViewHas($key, $value = null);`  |  断言响应视图拥有指定的部分绑定数据。
`->assertViewHasAll(array $bindings);`  |  断言响应视图里存在传参数组里的所有数据
`->assertViewMissing($key);`  |  断言响应视图拥有指定的绑定数据列表。
`->assertRedirectedTo($uri, $with = []);`  |  断言客户端是否被重定向至指定的 URI。
`->assertRedirectedToRoute($name, $parameters = [], $with = []);`  |  断言客户端是否被重定向到指定的路由。
`->assertRedirectedToAction($name, $parameters = [], $with = []);`  |  断言客户端是否被重定向到指定的行为。
`->assertSessionHas($key, $value = null);`  |  断言 session 中有指定的值。
`->assertSessionHasAll(array $bindings);`  |  断言 session 中有指定的列表值。
`->assertSessionHasErrors($bindings = [], $format = null);`  |  断言 session 有错误的绑定。
`->assertHasOldInput();`  | 断言 session 有旧的输入数据。
`->assertSessionMissing($key);`  | 断言 session 中没有指定的值。


## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@JobsLong](https://phphub.org/users/56)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/56_1427370654.jpeg?imageView2/1/w/100/h/100">  |  翻译  | 我的个人主页：[http://jobslong.com](http://jobslong.com)  |