# 测试

- [简介](#introduction)
- [测试环境](#environment)
- [定义并运行测试](#creating-and-running-tests)

<a name="introduction"></a>
## 简介

Laravel 天生就具有测试的基因。事实上，Laravel 默认就支持用 PHPUnit 来做测试，并为你的应用程序配置好了 `phpunit.xml` 文件。框架还提供了一些便利的辅助函数，让你可以更直观的测试应用程序。

在 `tests` 目录中有提供一个 `ExampleTest.php` 的示例文件。安装新的 Laravel 应用程序之后，只需在命令行上运行 `phpunit` 就可以进行测试。

<a name="environment"></a>
## 测试环境

在运行测试时，Laravel 会自动将环境变量设置为 `testing`，并将 Session 及缓存以 `array` 的形式存储，也就是说在测试时不会持久化任何 Session 或缓存数据。

你可以随意创建其它必要的测试环境配置。`testing` 环境的变量可以在 `phpunit.xml` 文件中被修改，但是在运行测试之前，请确保使用 `config:clear` Artisan 命令来清除配置信息的缓存。

<a name="creating-and-running-tests"></a>
## 定义并运行测试

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

> {note} 如果要在你的类自定义自己的 `setUp` 方法，请确保调用了 `parent::setUp`。
