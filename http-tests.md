# Laravel 测试之：HTTP 测试

- [简介](#introduction)
- [Session / 认证](#session-and-authentication)
- [测试 JSON APIs](#testing-json-apis)
- [测试文件上传](#testing-file-uploads)
- [可用的断言方法](#available-assertions)

<a name="introduction"></a>
## 简介

Laravel 为 HTTP 请求的生成和输出的检查都提供了非常流畅的 API。例如，你可以查看下面的这个测试用例:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        /**
         * 一个基础的测试用例。
         *
         * @return void
         */
        public function testBasicTest()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

`get` 方法会创建一个 `GET` 请求来请求你的应用，而 `assertStatus` 方法断言返回的响应是指定的 HTTP 状态码。除了这个简单的断言之外，Laravel 也包含检查响应标头、内容、JSON 结构等各种断言。

<a name="session-and-authentication"></a>
## Session / 认证

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

    use App\User;

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $user = factory(User::class)->create();

            $response = $this->actingAs($user)
                             ->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

你也可以通过传递 guard 名称作为 `actingAs` 的第二参数以指定用户通过哪种 guard 来认证:

    $this->actingAs($user, 'api')

<a name="testing-json-apis"></a>
## Testing JSON APIs

Laravel 也提供了几个辅助函数来测试 JSON APIs 及其响应。例如，`json`，`get`，`post`，`put`，`patch` 和 `delete` 方法可以用于发出各种 HTTP 动作的请求。你也可以轻松的传入数据或标头到这些方法上。首先，让我们来编写一个测试，将一个 `POST` 请求发送至 `/user` ，并断言其会返回预期数据：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 一个基础的功能测试用例。
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
### 验证完全匹配

如果你想验证传入的数组是否与应用返回的 JSON **完全** 匹配，你可以使用 `assertExactJson` 方法：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 一个基础的功能测试用例。
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

<a name="testing-file-uploads"></a>
## 测试文件上传

`Illuminate\Http\UploadedFile` 类提供了一个 `fake` 方法，可用其生成用于测试的模拟文件或图像。将其与 `Storage` facade 的 `fake` 方法结合使用，可极大地简化文件上传的测试。例如，你可以结合这两个功能轻松测试头像上传表单：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        public function testAvatarUpload()
        {
            Storage::fake('avatars');

            $response = $this->json('POST', '/avatar', [
                'avatar' => UploadedFile::fake()->image('avatar.jpg')
            ]);

            // 断言文件已存储...
            Storage::disk('avatars')->assertExists('avatar.jpg');

            // 断言文件不存在...
            Storage::disk('avatars')->assertMissing('missing.jpg');
        }
    }

#### 自定义模拟文件

当使用 `fake` 方法创建文件时，你可以指定图片的宽度、高度和大小，以便更好地测试你的验证规则：

    UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);

除了创建图片，你还可以使用 `create` 方法创建任何其他类型的文件：

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);

<a name="available-assertions"></a>
## 可用的断言方法

Laravel 为你的 [PHPUnit](https://phpunit.de/) 测试提供了各种各样的自定义断言方法。`json`，`get`，`post`，`put` 和 `delete` 这些测试方法返回的响应都可以使用这些断言方法：

Method  | Description
------------- | -------------
`$response->assertSuccessful();`  |  断言该响应具有成功的状态码。
`$response->assertStatus($code);`  |  断言该响应具有指定的状态码。
`$response->assertRedirect($uri);`  |  断言该响应被重定向至指定的 URI。
`$response->assertHeader($headerName, $value = null);`  |  断言该响应存在指定的标头。
`$response->assertCookie($cookieName, $value = null);`  |  断言该响应包含了指定的 Cookie。
`$response->assertPlainCookie($cookieName, $value = null);`  |  断言该响应包含了指定的 Cookie（未加密）。
`$response->assertSessionHas($key, $value = null);`  |  断言该 Session 包含指定的数据。
`$response->assertSessionHasErrors(array $keys, $errorBag = 'default');`  |  断言该 Session 包含指定的字段的错误信息。
`$response->assertSessionMissing($key);`  |  断言该 Session 不包含指定的键。
`$response->assertJson(array $data);`  |  断言该响应包含指定的 JSON 数据。
`$response->assertJsonFragment(array $data);`  |  断言该响应包含指定的 JSON 片段。
`$response->assertJsonMissing(array $data);`  |  断言该响应不包含指定的 JSON 片段。
`$response->assertExactJson(array $data);`  |  断言该响应包含完全匹配指定的 JSON 数据。
`$response->assertJsonStructure(array $structure);`  |  断言该响应存在指定的 JSON 结构。
`$response->assertViewIs($value);`  |  断言该视图响应的视图名称为指定的值。
`$response->assertViewHas($key, $value = null);`  |  断言该视图响应存在指定的数据。

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@springjk](https://laravel-china.org/users/4550)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/4550_1464580958.png?imageView2/1/w/100/h/100">  |  翻译  | 再怎么说我也是我西北一匹狼 |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org