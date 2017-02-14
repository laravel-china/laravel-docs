# Laravel 测试之：HTTP 测试

- [基础介绍](#introduction)
- [Session / 认证](#session-and-authentication)
- [测试 JSON APIs](#testing-json-apis)
- [可用的断言方法](#available-assertions)

<a name="introduction"></a>
## 基础介绍

Laravel 为 HTTP 请求的生成和发送操作、输出的检查都提供了非常流利的 API。例如，你可以看看下面的这个测试用例：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testBasicTest()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

`get` 方法会创建一个 `GET` 请求来请求你的应用，而 `assertStatus` 方法断言返回的响应是给定的 HTTP  状态码。除了这个简单的断言之外，Laravel 也包含检查响应标头、内容、 JSON 结构等等的各种断言。

<a name="session-and-authentication"></a>
### Session / 认证

Laravel 提供了几个可在测试时使用 Session 的辅助函数。首先，你需要传递一个数组给 `withSession` 方法来设置 Seesion 数据。这让你在应用程序的测试请求发送之前，先给数据加载 Session 变得简单：

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $response = $this->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

当然，一般使用 Session 时都是用于维持用户的状态，如认证用户。`actingAs` 辅助函数提供了简单的方式来让指定的用户认证为当前的用户。例如，我们可以使用 [模型工厂](/docs/{{version}}/database-testing#writing-factories) 来生成并认证用户：

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $user = factory(App\User::class)->create();

            $response = $this->actingAs($user)
                             ->withSession(['foo' => 'bar'])
                             ->get('/')
        }
    }

你也可以通过传递 guard 名称作为 `actingAs` 的第二参数以指定用户通过哪种 guard 来认证:

    $this->actingAs($user, 'api')

<a name="testing-json-apis"></a>
### 测试 JSON APIs

Laravel 也提供了几个辅助函数来测试 JSON APIs 及其响应。举例来说，`json`， `get`， `post`， `put`， `patch` 和 `delete` 方法可以用于发出各种 HTTP 动作的请求。你也可以轻松的传入数据或标头到这些方法上。首先，让我们来编写一个测试，将 POST 请求发送至 /user ，并断言其会返回预期数据：

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
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(200)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

> {tip} `assertJson` 方法会将响应转换为数组并且利用 `PHPUnit::assertArraySubset` 方法来验证传入的数组是否在应用返回的 JSON 中。也就是说，即使有其它的属性存在于该 JSON 响应中，但是只要指定的片段存在，此测试仍然会通过。

<a name="verifying-exact-match"></a>
### 验证完全匹配的 JSON

如果你想验证传入的数组是否与应用程序返回的 JSON **完全** 匹配，你可以使用 `assertExactJson` 方法：

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
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(200)
                ->assertExactJson([
                    'created' => true,
                ]);
        }
    }

<a name="available-assertions"></a>
### 可用的断言方法

Laravel 为你的 [PHPUnit](https://phpunit.de/) 测试提供了各种各样的自定义断言方法。`json`， `get`， `post`， `put`  和 `delete` 这些测试方法返回的响应都可以使用这些断言方法：

Method  | Description
------------- | -------------
`$response->assertStatus($code);`  |  断言该响应具有给定的状态码。
`$response->assertRedirect($uri);`  |  断言该响应被重定向至指定的 URI。
`$response->assertHeader($headerName, $value = null);`  |  断言该响应存在指定的标头。
`$response->assertCookie($cookieName, $value = null);`  |  断言该响应包含了指定的 cookie。
`$response->assertPlainCookie($cookieName, $value = null);`  |  断言该响应包含了指定的 cookie （未加密）。
`$response->assertSessionHas($key, $value = null);`  |  断言该 session 包含指定数据。
`$response->assertSessionMissing($key);`  |  断言该 session 不包含指定键。
`$response->assertJson(array $data);`  |  断言该响应包含指定 JSON 数据。
`$response->assertJsonFragment(array $data);`  |  断言该响应包含指定 JSON 片段。
`$response->assertExactJson(array $data);`  |  断言该响应包含完全匹配的 JSON 数据。
`$response->assertJsonStructure(array $structure);`  |  断言该响应存在指定 JSON 结构。
`$response->assertViewHas($key, $value = null);`  |  断言该响应视图存在指定数据。
