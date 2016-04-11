# 测试

- [简介](#introduction)
- [测试应用程序](#application-testing)
    - [与你的应用程序进行交互](#interacting-with-your-application)
    - [测试 JSON APIs](#testing-json-apis)
    - [Sessions 和认证](#sessions-and-authentication)
    - [停用中间件](#disabling-middleware)
    - [自定义 HTTP 请求](#custom-http-requests)
    - [PHPUnit 断言](#phpunit-assertions)
- [使用数据库](#working-with-databases)
    - [每次测试结束后重置数据库](#resetting-the-database-after-each-test)
    - [模型工厂](#model-factories)
- [仿真](#mocking)
    - [仿真事件](#mocking-events)
    - [仿真任务](#mocking-jobs)
    - [仿真 Facades](#mocking-facades)

<a name="introduction"></a>
## 简介

Laravel 在创建时就已考虑到测试的部分。事实上，默认就支持以 PHPUnit 做测试，而且已经为你的应用程序创建了 `phpunit.xml` 文件。框架还提供了一些便利的辅助方法，让你更直观的测试你的应用程序。

在 `tests` 目录中有提供一个 `ExampleTest.php` 的例子文件。安装新的 Laravel 应用程序之后，只要在命令行上运行 `phpunit`，就可以进行测试。

### 测试环境

在运行测试时，Laravel 会自动将环境变量设置为 `testing`，并将 Session 及缓存存入`数组`形式，也就是说在测试时不会有任何 Session 或缓存数据被保存。

你可以自由创建其它必要的测试环境配置。`testing` 的环境变量可以在 `phpunit.xml` 文件中做修改。

### 定义并运行测试

要创建一个测试案例，使用 `make:test` Artisan 命令：

    php artisan make:test UserTest

此命令会放置一个新的 `UserTest` 类至你的 `tests` 目录。接着就可以像平常使用 PHPUnit 一样定义测试方法。要运行测试只需要在命令行上运行 `phpunit` 命令：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class UserTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testExample()
        {
            $this->assertTrue(true);
        }
    }

> **注意：** 如果你要在你的类定义自己的 `setUp` 方法，请确定有调用 `parent::setUp`。

<a name="application-testing"></a>
## 测试应用程序

Laravel 对于产生 HTTP 请求并送至应用程序，检查输出，甚至填写表单，都提供了非常流利的 API。 举例来说，你可以看看 `tests` 目录中的 `ExampleTest.php` 文件：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
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

`visit` 方法会制造 `GET` 请求至应用程序，`see` 方法则断言在应用程序返回的响应中，有指定的文本。The `dontSee` method asserts that the given text is not returned in the application response.这是 Laravel 所提供最基本的应用程序测试。

<a name="interacting-with-your-application"></a>
### 与你的应用程序进行交互

当然，除了断言文本是否存在于一个指定的响应，你可以做更多的交互。让我们看看点击链接及填写表单的例子：

#### 点击链接

在这个测试中，我们会产生一个请求送到应用程序，并「点击」返回响应中的链接，接着断言我们会停留在指定的 URI。举个例子，假设在响应中有个链接，并写着「About Us」：

    <a href="/about-us">About Us</a>

现在，我们编写一个测试，点击链接并断言用户会停留在正确的页面：

    public function testBasicExample()
    {
        $this->visit('/')
             ->click('About Us')
             ->seePageIs('/about-us');
    }

#### 使用表单

Laravel 还提供了几种用于测试表单的方法。通过 `type`、`select`、`check`、`attach` 及 `press` 方法让你与表单所有的输入栏进行交互。举个例子，让我们想像一下有个表单在应用程序的注册页面：

    <form action="/register" method="POST">
        {!! csrf_field() !!}

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

当然，如果你的表单包含像是单选栏或下拉式菜单的其它输入栏，你也可以很轻松的填入这些类型的区域。以下是每个表单操作方法的列表：

方法  | 说明
------------- | -------------
`$this->type($text, $elementName)`  |  「输入（type）」文本在一个指定的区域
`$this->select($value, $elementName)`  |  「选择（select）」一个单选栏或下拉式菜单的区域
`$this->check($elementName)`  |  「勾选（Check）」一个复选栏的区域
`$this->attach($pathToFile, $elementName)`  |  「附上（Attach）」一个文件至表单
`$this->press($buttonTextOrElementName)`  |  「按下（Press）」一个指定文本或名称的按钮

#### 使用附件

如果你的表单包含 `file` 的输入栏类型，可以使用 `attach` 方法附上文件：

    public function testPhotoCanBeUploaded()
    {
        $this->visit('/upload')
             ->name('File Name', 'name')
             ->attach($absolutePathToFile, 'photo')
             ->press('Upload')
             ->see('Upload Successful!');
    }

<a name="testing-json-apis"></a>
### 测试 JSON APIs

Laravel 也提供了几个辅助方法测试 JSON APIs 及其响应。举例来说，`get`、`post`、`put`、`patch` 及 `delete` 方法可以用于发出各种 HTTP 动词的请求。你也可以很轻松的传入数据或是标头到这些方法。首先，我们编写一个测试，发出 `POST` 请求至 `/user` ，并断言会返回JSON 格式的指定数组：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->post('/user', ['name' => 'Sally'])
                 ->seeJson([
                     'created' => true,
                 ]);
        }
    }

`seeJson` 方法会将传入的数组转换成 JSON，并验证该 JSON 片段是否存在于应用程序返回的 JSON 响应中的**任何位置**。也就是说，即使有其它的属性在 JSON 响应中，但是只要指定的片段存在，此测试仍然会通过。

#### 验证 JSON 完全匹配

如果你想验证传入的数组要与应用程序返回的 JSON **完全**匹配，可以使用 `seeJsonEquals` 方法：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->post('/user', ['name' => 'Sally'])
                 ->seeJsonEquals([
                     'created' => true,
                 ]);
        }
    }

<a name="sessions-and-authentication"></a>
### Sessions 和认证

Laravel 提供了几个辅助方法在测试时使用 Session。首先，你需要设置 Session 数据至指定的数组，并使用 `withSession` 方法。对于要测试送到应用程序的请求之前，先将数据加载 session 上相当有用：

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $this->withSession(['foo' => 'bar'])
                 ->visit('/');
        }
    }

当然，一般使用 Session 时都是用于保持用户的状态，像是认证用户。`actingAs`  辅助方法提供了简单的方式，让指定的用户认证为当前的用户。举个例子，我们可以使用[模型工厂](#model-factories)来产生并认证用户：

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

<a name="disabling-middleware"></a>
### 停用中间件

测试应用程序时，你会发现，在某些测试中停用[中间件](/docs/{{version}}/middleware)是很方便的。让你可以隔离任何中间件的影响，来测试路由及控制器。Laravel 包含一个简洁的 `WithoutMiddleware` trait，你能够使用它自动在测试类中停用所有的中间件：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        use WithoutMiddleware;

        //
    }

如果你只想要在某几个测试方法中停用中间件，你可以在测试方法中调用 `withoutMiddleware` 方法：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
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

如果你想要创建一个自定义的 HTTP 请求至你的应用程序，并获取完整的 `Illuminate\Http\Response` 对象，你可以使用 `call` 方法：

    public function testApplication()
    {
        $response = $this->call('GET', '/');

        $this->assertEquals(200, $response->status());
    }

如果你创建的是 `POST`、`PUT`、或是 `PATCH` 请求，你可以在请求时传入一个数组作为输入数据。当然，你可在你的路由及控制器中通过[请求实例](/docs/{{version}}/requests)取用这些数据：

       $response = $this->call('POST', '/user', ['name' => 'Taylor']);

<a name="phpunit-assertions"></a>
### PHPUnit 断言

Laravel 为 [PHPUnit](https://phpunit.de/) 测试提供了一些额外的断言方法：

方法  | 描述
------------- | -------------
`->assertResponseOk();`  |  断言客户端的响应拥有 OK 状态码。
`->assertResponseStatus($code);`  |  断言客户端的响应拥有指定的状态码。
`->assertViewHas($key, $value = null);`  |  断言响应视图拥有指定的部分绑定数据。
`->assertViewHasAll(array $bindings);`  |  断言响应视图拥有指定的绑定数据列表。
`->assertViewMissing($key);`  |  断言响应视图不包含指定的部分绑定数据。
`->assertRedirectedTo($uri, $with = []);`  |  断言客户端是否被重定向至指定的 URI。
`->assertRedirectedToRoute($name, $parameters = [], $with = []);`  | 断言客户端是否被重定向至指定的路由
`->assertRedirectedToAction($name, $parameters = [], $with = []);`  |  断言客户端是否被重定向至指定的行为。
`->assertSessionHas($key, $value = null);`  |  断言 session 中有指定的值。
`->assertSessionHasAll(array $bindings);`  |  断言 session 中有指定的列表值。
`->assertSessionHasErrors($bindings = [], $format = null);`  |  断言 session 有错误的绑定。
`->assertHasOldInput();`  |  断言 session 有旧输入数据。

<a name="working-with-databases"></a>
## 使用数据库

Laravel 也提供了多种有用的工具，让你更容易测试使用数据库的应用程序。首先，你可以使用 `seeInDatabase` 辅助方法，来断言数据库中是否存在与指定条件互相匹配的数据。举例来说，如果我们想验证 `users` 数据表中是否存在 `email` 值为 `sally@example.com` 的数据，我们可以按照以下的方式：

    public function testDatabase()
    {
        // 创建调用至应用程序...

        $this->seeInDatabase('users', ['email' => 'sally@example.com']);
    }

当然，使用 `seeInDatabase` 方法及其它的辅助方法只是基于方便。你可以自由使用任何 PHPUnit 内置的断言方法来扩充你的测试。

<a name="resetting-the-database-after-each-test"></a>
### 每次测试结束后重置数据库

在每次测试结束后都重置你的数据是相当有用的，这样前次的测试数据就不会干扰之后的测试。

#### 使用迁移

其中一种方式就是在每次测试后都还原数据库，并在下次测试前进行迁移。Laravel 提供了简洁的 `DatabaseMigrations` trait，它会自动帮你处理这些操作。你只需要在测试类中使用此 trait：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        use DatabaseMigrations;

        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->visit('/')
                 ->see('Laravel 5');
        }
    }

#### 使用交易

另一个方式，就是将每个测试案例包装在数据库交易中。当然，Laravel 提供了一个简洁的 `DatabaseTransactions` trait，它会自动帮你处理这些操作：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        use DatabaseTransactions;

        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->visit('/')
                 ->see('Laravel 5');
        }
    }

> **注意：**此 trait 的交易只会包含默认的数据库连接。

<a name="model-factories"></a>
### 模型工厂

测试时，常常需要在运行测试之前写入一些数据到数据库中。创建测试数据时，除了手动设置每个字段的值，Laravel 让你可以使用 [Eloquent 模型](/docs/{{version}}/eloquent)的「工厂」设置每个属性的默认值。开始之前，你可以查看应用程序的 `database/factories/ModelFactory.php` 文件。此文件包含一个现成的工厂定义：

    $factory->define(App\User::class, function (Faker\Generator $faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->email,
            'password' => bcrypt(str_random(10)),
            'remember_token' => str_random(10),
        ];
    });

闭包内为工厂的定义，你可以返回模型所有属性的默认测试值。该闭包内会接收到 [Faker](https://github.com/fzaninotto/Faker) PHP 函数库的实例，它可以让你方便的产生各种随机的数据以进行测试。

当然，你可以自由的增加自己额外的工厂至 `ModelFactory.php` 文件。

#### 多个工厂类型

有时你可能希望针对同一个 Eloquent 模型类，能创建多个工厂。例如，除了一般用户的工厂之外，还有「管理员」的工厂。你可以使用 `defineAs` 方法定义这个工厂：

    $factory->defineAs(App\User::class, 'admin', function ($faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->email,
            'password' => str_random(10),
            'remember_token' => str_random(10),
            'admin' => true,
        ];
    });

除了从一般用户工厂复制所有基底属性，你也可以使用 `raw` 方法来获取基底属性。一旦你获取这些属性，就可以轻松的增加额外任何你需要的值：

    $factory->defineAs(App\User::class, 'admin', function ($faker) use ($factory) {
        $user = $factory->raw(App\User::class);

        return array_merge($user, ['admin' => true]);
    });

#### 在测试中使用工厂

定义工厂后，就可以在测试或是数据库填充文件中，通过全局 `factory` 函数产生模型实例。那么让我们来看看几个创建模型的例子。首先我们会使用 `make` 方法创建模型，但是不将它们保存至数据库：

    public function testDatabase()
    {
        $user = factory(App\User::class)->make();

        // 在测试中使用模型...
    }

如果你想重写模型中的某些默认值，你可以传递一个包含数值的数组至 `make` 方法。只有指定的数值会被替换，其它剩余的数值则会按照工厂指定的默认值设置：

    $user = factory(App\User::class)->make([
        'name' => 'Abigail',
       ]);

你也可以创建一个含有多个模型的集合，或创建一个指定类型的模型：

    // 创建三个 App\User 实例...
    $users = factory(App\User::class, 3)->make();

    // 创建一个 App\User「管理员」实例...
    $user = factory(App\User::class, 'admin')->make();

    // 创建三个 App\User「管理员」实例...
    $users = factory(App\User::class, 'admin', 3)->make();

#### 保存工厂模型

`create` 方法不仅创建模型实例，也可以使用 Eloquent 的 `save` 方法将它们保存至数据库：

    public function testDatabase()
    {
        $user = factory(App\User::class)->create();

        // 在测试中使用模型...
    }

同样的，你可以在传递数组至 `create` 方法时重写模型的属性：

    $user = factory(App\User::class)->create([
        'name' => 'Abigail',
       ]);

#### 增加关联至模型

你甚至能保存多个模型至数据库。在本例中，我们还会增加关联至我们创建的模型。当使用 `create` 方法创建多个模型时，它会返回一个 Eloquent [集合实例](/docs/{{version}}/eloquent-collections)，让你能够使用集合所提供的方便函数，像是 `each`：

    $users = factory(App\User::class, 3)
               ->create()
               ->each(function($u) {
                    $u->posts()->save(factory(App\Post::class)->make());
                });


<a name="mocking"></a>
## 仿真

<a name="mocking-events"></a>
### 仿真事件

如果你正在频繁地使用 Laravel 的事件系统，你可能希望在测试时停止或是仿真某些事件。举例来说，如果你正在测试用户注册功能，你可能不希望所有 `UserRegistered` 事件的处理进程被运行，因为它们会发送「欢迎」的电子邮件等等。

Laravel 提供了简洁的 `expectsEvents` 方法，验证预期的事件有被运行，但是防止该事件的任何处理进程被运行：

    <?php

    class ExampleTest extends TestCase
    {
        public function testUserRegistration()
        {
            $this->expectsEvents(App\Events\UserRegistered::class);

            // 测试用户注册的代码...
        }
    }

如果你希望防止所有事件的处理进程被运行，你可以使用 `withoutEvents` 方法：

    <?php

    class ExampleTest extends TestCase
    {
        public function testUserRegistration()
        {
            $this->withoutEvents();

            // 测试用户注册的代码...
        }
    }

<a name="mocking-jobs"></a>
### 仿真任务

有时你可能希望当发出请求至你的应用程序时，简单的测试由你的控制器所派送的任务。这么做能够使你隔离并测试你的路由或控制器－设置除了任务以外的逻辑。当然，你之后可以在一个单独的测试案例测试该任务。

Laravel 提供了一个简洁的 `expectsJobs` 方法，验证预期的任务有被派送，但是任务本身不会被运行：

    <?php

    class ExampleTest extends TestCase
    {
        public function testPurchasePodcast()
        {
            $this->expectsJobs(App\Jobs\PurchasePodcast::class);

            // 测试购买 podcast 的代码...
        }
    }

> **注意：** 此方法只检测被 `DispatchesJobs` trait 的派送方法所派送的任务。它并不会检测直接发送到 `Queue::push` 的任务。

<a name="mocking-facades"></a>
### 仿真 Facades

测试时，你可能时常需要仿真调用一个 Laravel [facade](/docs/{{version}}/facades)。举个例子，参考下方的控制器行为：

    <?php

    namespace App\Http\Controllers;

    use Cache;
    use Illuminate\Routing\Controller;

    class UserController extends Controller
    {
        /**
         * 显示应用程序所有用户的列表。
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

我们可以通过 `shouldReceive` 方法仿真调用 `Cache` facade，它会返回一个 [Mockery](https://github.com/padraic/mockery) 仿真的实例。因为 facades 实际上已经被 Laravel [服务容器](/docs/{{version}}/container)解决及管理，它们比起一般的静态类更有可测试性。举个例子，让我们仿真我们调用 `Cache` facade：

    <?php

    class FooTest extends TestCase
    {
        public function testGetIndex()
        {
            Cache::shouldReceive('get')
                        ->once()
                        ->with('key')
                        ->andReturn('value');

            $this->visit('/users')->see('value');
        }
    }

> **注意：** 你不应该仿真 `Request` facade。应该在测试时使用像是 `call` 及 `post` 的 HTTP 辅助方法来传递你想要的数据。
