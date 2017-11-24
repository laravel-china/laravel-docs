# Laravel 测试: 入门指南

- [简介](#introduction)
- [测试环境](#environment)
- [定义并运行测试](#creating-and-running-tests)

<a name="introduction"></a>
## 简介

Laravel 天生就具有测试的基因。事实上，Laravel 默认就支持用 PHPUnit 来做测试，并为你的应用程序配置好了 `phpunit.xml` 文件。框架还提供了一些便利的辅助函数，让你可以更直观的测试应用程序。

默认在你应用的 `tests` 目录下包含了两个子目录： `Feature` 和 `Unit`。单元测试是针对你代码中相对独立而且非常少的一部分代码来进行测试。实际上，大多数单元测试可能都是针对某一个方法来进行的。功能测试是针对你代码中大部分的代码来进行测试，包括几个对象的相互作用，甚至是一个完整的 HTTP 请求 JSON 实例。

在 `Feature` 和 `Unit` 目录中都有提供一个 `ExampleTest.php` 的示例文件。安装新的 Laravel 应用程序之后，只需在命令行上运行 `phpunit` 就可以进行测试。

<a name="environment"></a>
## 测试环境

在运行测试时，Laravel 会根据 `phpunit.xml` 文件中设定好的环境变量自动将环境变量设置为 `testing`，并将 Session 及缓存以 `array` 的形式存储，也就是说在测试时不会持久化任何 Session 或缓存数据。

你可以随意创建其它必要的测试环境配置。`testing` 环境的变量可以在 `phpunit.xml` 文件中被修改，但是在运行测试之前，请确保使用 `config:clear` Artisan 命令来清除配置信息的缓存。

<a name="creating-and-running-tests"></a>
## 定义并运行测试

可以使用 `make:test` Artisan 命令，创建一个测试用例：

    // 在 Feature 目录下创建一个测试类...
    php artisan make:test UserTest

    // 在 Unit 目录下创建一个测试类...
    php artisan make:test UserTest --unit

测试类生成之后，你就可以像平常使用 PHPUnit 一样来定义测试方法。要运行测试只需要在终端上运行 `phpunit` 命令即可：

    <?php

    namespace Tests\Unit;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        /**
         * 基本的测试用例。
         *
         * @return void
         */
        public function testBasicTest()
        {
            $this->assertTrue(true);
        }
    }

> {note} 如果要在你的测试类自定义自己的 `setUp` 方法，请确保调用了 `parent::setUp()` 方法。


--- 

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@wqer1019](https://laravel-china.org/users/5435)  | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/9254545?v=4&s=100">  |  翻译  | laravel是世界上最优雅的框架，[@wqer1019](https://github.com/wqer1019) at Github  |

--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org