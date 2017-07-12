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
- [模拟](#mocking)
    - [模拟事件](#mocking-events)
    - [模拟任务](#mocking-jobs)
    - [模拟 Facades](#mocking-facades)


<a name="introduction"></a>
## 简介

Laravel 天生就具有测试的基因。事实上，Laravel 默认就支持用 PHPUnit 来做测试，并为你的应用程序配置好了 `phpunit.xml` 文件。框架还提供了一些便利的辅助函数，让你可以更直观的测试应用程序。

在 `tests` 目录中有提供一个 `ExampleTest.php` 的示例文件。安装新的 Laravel 应用程序之后，只需在命令行上运行 `phpunit` 就可以进行测试。

### 测试环境

在运行测试时，Laravel 会自动将环境变量设置为 `testing`，并将 Session 及缓存以 `array` 的形式存储，也就是说在测试时不会持久化任何 Session 或缓存数据。

你可以随意创建其它必要的测试环境配置。`testing` 环境的变量可以在 `phpunit.xml` 文件中被修改，但是在运行测试之前，请确保使用 `config:clear` Artisan 命令来清除配置信息的缓存。

### 定义并运行测试

可以使用 `make:test` Artisan 命令，创建一个测试案例：

    php artisan make:test UserTest

此命令会放置一个新的 `UserTest` 类至你的 `tests` 目录。接着就可以像平常使用 PHPUnit 一样来定义测试方法。要运行测试只需要在终端上运行 `phpunit` 命令即可：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class UserTest extends TestCase
    {
        /**
         * 基本的测试样例。
         *
         * @return void
         */
        public function testExample()
        {
            $this->assertTrue(true);
        }
    }

> **注意：** 如果要在你的类自定义自己的 `setUp` 方法，请确保调用了 `parent::setUp`。

<a name="application-testing"></a>
## 测试应用程序

Laravel 为 HTTP 请求的生成和发送操作、输出的检查，甚至表单的填写都提供了非常流利的 API。 举例来说，你可以看看 `tests` 目录中的 `ExampleTest.php` 文件：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        /**
         * 一个基本的功能测试样例。
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

<a name="interacting-with-your-application"></a>
### 与你的应用程序进行交互

当然，除了断言文本是否存在于一个指定的响应中，你还可以做更多的交互。让我们来看看点击链接及填写表单的例子：

#### 点击链接

在这个测试中，我们会生成一个请求并「点击」返回响应中的链接，接着断言我们会停留在指定的 URI 上。举个例子，假设在响应中有个链接，并写着「About Us」：

    <a href="/about-us">About Us</a>

现在，我们编写一个测试，点击链接并断言用户会停留在正确的页面：

    public function testBasicExample()
    {
        $this->visit('/')
             ->click('About Us')
             ->seePageIs('/about-us');
    }

#### 使用表单

Laravel 还提供了几种用于测试表单的方法。通过 `type`、`select`、`check`、`attach` 及 `press` 方法让你与表单的所有输入框进行交互。举个例子，让我们想像一下有个在应用程序注册页面的表单：

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

当然，如果你的表单中包含了类似单选框或下拉式菜单的其它输入框，也可以很轻松的填入这些类型的区域。以下是每个表单的方法操作列表：

方法  | 说明
------------- | -------------
`$this->type($text, $elementName)`  |  「输入（type）」文本在一个指定的区域
`$this->select($value, $elementName)`  |  「选择（select）」一个单选框或下拉式菜单的区域
`$this->check($elementName)`  | 「勾选（check）」一个复选框的区域
`$this->uncheck($elementName)`  | 「取消勾选（uncheck）」一个复选框的区域
`$this->attach($pathToFile, $elementName)`  | 「附加（attach）」一个文件至表单
`$this->press($buttonTextOrElementName)`  |  「按下（press）」一个指定文本或名称的按钮

#### 使用附件

如果你的表单包含 `file` 的输入框类型，则可以使用 `attach` 方法来附加文件：

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

Laravel 也提供了几个辅助函数来测试 JSON API 及其响应。举例来说，`get`、`post`、`put`、`patch` 及 `delete` 方法可以用于发出各种 HTTP 动作的请求。你也可以轻松的传入数据或标头到这些方法上。首先，让我们来编写一个测试，将 `POST` 请求发送至 `/user` ，并断言其会返回 JSON 格式的指定数组：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 一个基本的功能测试示例。
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

`seeJson` 方法会将传入的数组转换成 JSON，并验证该 JSON 片段是否存在于应用程序返回的 JSON 响应中的 **任何位置**。也就是说，即使有其它的属性存在于该 JSON 响应中，但是只要指定的片段存在，此测试便会通过。

#### 验证完全匹配的 JSON

如果你想验证传入的数组是否与应用程序返回的 JSON **完全** 匹配，则可以使用 `seeJsonEquals` 方法：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 一个基本的功能测试示例。
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

<a name="verify-structural-json-match"></a>
#### 验证 JSON 的结构一致

你可以使用 `seeJsonStructure` 来验证 JSON 结构是否一致：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 一个基本的功能测试示例。
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

需要注意的是如果验证数据里存在多余的字段，`seeJsonStructure` 不会报错。

你可以使用 `*` 来假设所有的列表内容都包含至少数组里面的内容：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 一个基本的功能测试示例。
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

你也可以灵活的使用 `*` 做嵌套：

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

    $this->actingAs($user, 'backend')

<a name="disabling-middleware"></a>
### 停用中间件

测试应用程序时，你会发现，在某些测试中停用 [中间件](/docs/{{version}}/middleware) 是很方便的。这让你可以隔离任何中间件的所有影响，以便更好的测试路由及控制器。Laravel 包含了一个简洁的 `WithoutMiddleware` trait，你能在测试类中使用它来自动停用所有的中间件：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
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
         * 一个基本的功能测试示例。
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

Laravel 为 [PHPUnit](https://phpunit.de/) 测试提供了一些额外的断言方法：

方法  | 描述
------------- | -------------
`->assertResponseOk();`  |  断言客户端的响应拥有 OK 状态码。
`->assertResponseStatus($code);`  | 断言客户端的响应拥有指定的状态码。
`->assertViewHas($key, $value = null);`  |  断言响应视图拥有指定的部分绑定数据。
`->assertViewHasAll(array $bindings);`  |  断言响应视图里存在传参数组里的所有数据。
`->assertViewMissing($key);`  |  断言响应视图拥有指定的绑定数据列表。
`->assertRedirectedTo($uri, $with = []);`  |  断言客户端是否被重定向至指定的 URI。
`->assertRedirectedToRoute($name, $parameters = [], $with = []);`  |  断言客户端是否被重定向到指定的路由。
`->assertRedirectedToAction($name, $parameters = [], $with = []);`  |  断言客户端是否被重定向到指定的行为。
`->assertSessionHas($key, $value = null);`  |  断言 session 中有指定的值。
`->assertSessionHasAll(array $bindings);`  |  断言 session 中有指定的列表值。
`->assertSessionHasErrors($bindings = [], $format = null);`  |  断言 session 有错误的绑定。
`->assertHasOldInput();`  | 断言 session 有旧的输入数据。
`->assertSessionMissing($key);`  | 断言 session 中没有指定的值。

<a name="working-with-databases"></a>
## 使用数据库

Laravel 也提供了多种有用的工具来让你更容易的测试使用数据库的应用程序。首先，你可以使用 `seeInDatabase` 辅助函数，来断言数据库中是否存在与指定条件互相匹配的数据。举例来说，如果我们想验证 `users` 数据表中是否存在 `email` 值为 `sally@example.com` 的数据，我们可以按照以下的方式来做测试：

    public function testDatabase()
    {
        // 创建调用至应用程序...

        $this->seeInDatabase('users', ['email' => 'sally@example.com']);
    }

当然，使用 `seeInDatabase` 方法及其它的辅助函数只是为了方便。你也可以随意使用 PHPUnit 内置的所有断言方法来扩充测试。

<a name="resetting-the-database-after-each-test"></a>
### 每次测试结束后重置数据库

在每次测试结束后都需要对数据进行重置，这样前面的测试数据就不会干扰到后面的测试。

#### 使用迁移

其中有一种方式就是在每次测试后都还原数据库，并在下次测试前运行迁移。Laravel 提供了简洁的 `DatabaseMigrations` trait，它会自动帮你处理好这些操作。你只需在测试类中使用此 trait 即可：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        use DatabaseMigrations;

        /**
         * 一个基本的功能测试示例。
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->visit('/')
                 ->see('Laravel 5');
        }
    }

#### 使用事务

另一个方式，就是将每个测试案例都包含在数据库事务中。Laravel 提供了一个简洁的 `DatabaseTransactions` trait 来自动帮你处理好这些操作：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        use DatabaseTransactions;

        /**
         * 一个基本的功能测试示例。
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->visit('/')
                 ->see('Laravel 5');
        }
    }

> **注意：**此 trait 的事务只包含默认的数据库连接。

<a name="model-factories"></a>
### 模型工厂

测试时，常常需要在运行测试之前写入一些数据到数据库中。创建测试数据时，除了手动的来设置每个字段的值，还可以使用 [Eloquent 模型](/docs/{{version}}/eloquent) 的「工厂」来设置每个属性的默认值。在开始之前，你可以先查看下应用程序的 `database/factories/ModelFactory.php` 文件。此文件包含一个现成的工厂定义：

    $factory->define(App\User::class, function (Faker\Generator $faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->email,
            'password' => bcrypt(str_random(10)),
            'remember_token' => str_random(10),
        ];
    });

闭包内为工厂的定义，你可以返回模型中所有属性的默认测试值。在该闭包内会接收到 [Faker](https://github.com/fzaninotto/Faker) PHP 函数库的实例，它可以让你很方便的生成各种随机数据以进行测试。

当然，你也可以随意将自己额外的工厂增加至 `ModelFactory.php` 文件。你也可以在 `database/factories` 里为每一个数据模型创建对应的工厂模型类，如 `UserFactory.php` 和 `CommentFactory.php`。

#### 多个工厂类型

有时你可能希望针对同一个 Eloquent 模型类来创建多个工厂。例如，除了一般用户的工厂之外，还有「管理员」工厂。你可以使用 `defineAs` 方法来定义这个工厂：

    $factory->defineAs(App\User::class, 'admin', function ($faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->email,
            'password' => str_random(10),
            'remember_token' => str_random(10),
            'admin' => true,
        ];
    });

除了从一般用户工厂复制所有基本属性，你也可以使用 `raw` 方法来获取所有基本属性。一旦你获取到这些属性，就可以轻松的为其增加任何额外值：

    $factory->defineAs(App\User::class, 'admin', function ($faker) use ($factory) {
        $user = $factory->raw(App\User::class);

        return array_merge($user, ['admin' => true]);
    });

#### 在测试中使用工厂

在工厂定义后，就可以在测试或是数据库的填充文件中，通过全局的 `factory` 函数来生成模型实例。接着让我们先来看看几个创建模型的例子。首先我们会使用 `make` 方法创建模型，但不将它们保存至数据库：

    public function testDatabase()
    {
        $user = factory(App\User::class)->make();

        // 在测试中使用模型...
    }

如果你想重写模型中的某些默认值，则可以传递一个包含数值的数组至 `make` 方法。只有指定的数值会被替换，其它剩余的数值则会按照工厂指定的默认值来设置：

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

`create` 方法不仅会创建模型实例, 同时也会将模型实例(通过调用 Eloquent 的 `save` 方法)保存至数据库. 

    public function testDatabase()
    {
        $user = factory(App\User::class)->create();

        // 在测试中使用模型...
    }

同样的，你可以在数组传递至 `create` 方法时重写模型的属性：

    $user = factory(App\User::class)->create([
        'name' => 'Abigail',
       ]);

#### 增加关联至模型

你甚至可以保存多个模型到数据库上。在本例中，我们还会增加关联至我们所创建的模型。当使用 `create` 方法创建多个模型时，它会返回一个 Eloquent [集合实例](/docs/{{version}}/eloquent-collections)，让你能够使用集合所提供的便利函数，像是 `each`：

    $users = factory(App\User::class, 3)
               ->create()
               ->each(function($u) {
                    $u->posts()->save(factory(App\Post::class)->make());
                });

#### 关联和属性闭包

你可以使用闭包参数来创建模型关联。例如如果你想在创建一个 `Post` 的顺便创建一个 `User` 实例：

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            }
        ];
    });

这些闭包也可以获取到生成的工厂包含的属性数组：

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            },
            'user_type' => function (array $post) {
                return App\User::find($post['user_id'])->type;
            }
        ];
    });

<a name="mocking"></a>
## 模拟

<a name="mocking-events"></a>
### 模拟事件

如果你正在频繁地使用 Laravel 的事件系统，你可能希望在测试时停止或是模拟某些事件。举例来说，如果你正在测试用户注册功能，你可能不希望所有 `UserRegistered` 事件的处理进程都被运行，因为它们会触发「欢迎」邮件的发送。

Laravel 提供了简洁的 `expectsEvents` 方法，以验证预期的事件有被运行，可防止该事件的任何处理进程被运行：

    <?php

    class ExampleTest extends TestCase
    {
        public function testUserRegistration()
        {
            $this->expectsEvents(App\Events\UserRegistered::class);

            // Test user registration...
        }
    }

你可以使用 `doesntExpectEvents` 来验证某个事件 **没被** 触发：

    <?php

    class ExampleTest extends TestCase
    {
        public function testPodcastPurchase()
        {
            $this->expectsEvents(App\Events\PodcastWasPurchased::class);

            $this->doesntExpectEvents(App\Events\PaymentWasDeclined::class);

            // 测试购买博客
        }
    }

如果你希望防止所有事件的处理进程被运行，则可以使用 `withoutEvents` 方法：

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
### 模拟任务

有时你可能希望当请求发送至应用程序时，简单的对控制器所派送的任务进行测试。这么做能够让你隔离测试路由或控制器，设置除了任务以外的逻辑。当然，在此之后你也可以在一个单独的测试案例中来测试该任务。

Laravel 提供了一个简洁的 `expectsJobs` 方法，以验证预期的任务有被派送，但任务本身不会被运行：

    <?php

    class ExampleTest extends TestCase
    {
        public function testPurchasePodcast()
        {
            $this->expectsJobs(App\Jobs\PurchasePodcast::class);

            // 测试购买 podcast 的代码...
        }
    }

> **注意：** 此方法只检测 `DispatchesJobs` trait 的派送方法所派送出的任务。它并不会检测直接发送到 `Queue::push` 的任务。

<a name="mocking-facades"></a>
### 模拟 Facades

测试时，你可能时常需要模拟调用一个 Laravel [facade](/docs/{{version}}/facades)。可参考下方的控制器行为：

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

我们可以通过 `shouldReceive` 方法模拟调用 `Cache` facade，它会返回一个 [Mockery](https://github.com/padraic/mockery) 模拟的实例。因为 facades 实际上已经被 Laravel 的 [服务容器](/docs/{{version}}/container) 解决并管理着，它们比起一般的静态类更有可测性。举个例子，让我们来模拟调用 `Cache` facade：

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

> **注意：** 你不应该模拟 `Request` facade。应该在测试时使用如 `call` 及 `post` 这样的 HTTP 辅助函数来传递你想要的数据。





--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译。
> 
> 文档永久地址： http://d.laravel-china.org