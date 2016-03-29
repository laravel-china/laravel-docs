# 贡献导引

- [错误回报](#bug-reports)
- [核心开发讨论](#core-development-discussion)
- [选择分支](#which-branch)
- [安全漏洞](#security-vulnerabilities)
- [代码风格](#coding-style)
    - [代码风格修复器](#code-style-fixer)

<a name="bug-reports"></a>
## 错误回报

为了鼓励积极协作，Laravel 强烈地鼓励 pull request，而不只是回报错误。「错误回报」也可以用包含一个失败测试的 pull request 形式发送。

然而，如果你创建错误回报，你的问题应该包含标题和清楚的问题描述。你也应该尽可能地提供相关的信息和示范问题的代码例子。错误回报的目的是让自己和其他人可以简单地重现错误并开发修复程序。

请记住，我们希望创建错误回报可以让其他也有相同问题的人可以与你协作解决问题。不要期望错误回报后会自动地看到任何动静或其他人会马上修复它。创建错误回报用于帮助自己和其他人开始修复问题。

Laravel 原代码托管在 Github 上面，并且每个 Laravel 的项目都有自己的保存库：

- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Website](https://github.com/laravel/laravel.com)
- [Laravel Art](https://github.com/laravel/art)

<a name="core-development-discussion"></a>
## 核心开发讨论

有关错误、新功能和现有功能的实现讨论会在 [LaraChat](http://larachat.co) Slack 团队的 `#internals` 频道中进行。Laravel 的维护者 Taylor Otwell 在工作日的 8am 到 5pm（ UTC-06:00 或 America/Chicago ）通常会出现在频道，其他时间则是零星地出现在频道。

<a name="which-branch"></a>
## 选择分支

**所有的**错误修复应该发送到最新的稳定分支。除非它们修复的功能只存在下一版的发布中，不然错误修复**永远不**应该发送到 `master` 分支。

**次要的**且与现行的 Laravel 发布版本**完全向下兼容**的功能可以发送到最新的稳定分支。

**主要的**新功能应该都发送到 `master` 分支，它包含下一版的 Laravel 发布内容。

如果不确定你的功能是主要的还是次要的，请在 [LaraChat](http://larachat.co) Slack 团队的 `#internals` 频道询问 Taylor Otwell。

<a name="security-vulnerabilities"></a>
## 安全漏洞

如果你发现 Laravel 的安全漏洞，请寄电子邮件到 <a href="mailto:taylor@laravel.com">taylor@laravel.com</a> 给 Taylor Otwell。所有的安全漏洞，将会及时予以处理。

<a name="coding-style"></a>
## 编码风格

Laravel 遵守 [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) 编码规范和 [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md) 自动加载规范。

### DocBlocks

`@param` tags should **not be aligned** and arguments should be separated by **2 spaces**.

Here's an example block:

    /**
     * Register a binding with the container.
     *
     * @param  string|array  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        //
    }

<a name="code-style-fixer"></a>
### Code Style Fixer

You may use the [PHP-CS-Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer) to fix your code style before committing.

To get started, [install the tool globally](https://github.com/FriendsOfPHP/PHP-CS-Fixer#globally-manual) and check the code style by issuing the following terminal command from your project's root directory:

```sh
php-cs-fixer fix
```
