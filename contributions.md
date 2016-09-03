# Contribution Guide

- [错误反馈](#bug-reports)
- [核心开发讨论](#core-development-discussion)
- [选择分支](#which-branch)
- [安全漏洞](#security-vulnerabilities)
- [代码风格](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)

<a name="bug-reports"></a>
## Bug Reports

为了提倡积极协作，Laravel 强烈地鼓励使用 Pull Request，而不仅仅只是反馈错误。「错误反馈」可以以一个包含失败测试的 Pull Request 的形式发送。

假如你提交了错误反馈，那么反馈中应该包含着标题和详尽的问题描述，并尽可能多的提供相关的信息和错误问题的代码示例。错误反馈的主要目的是让自己和其他人可以简单地重现并修复错误。

请记住，错误反馈的初衷是让其它有相同问题的人可以协作解决问题。不要期望反馈错误后会很快有人会马上修复它。创建错误反馈主要是为了能帮助你自己和其他人开始着手修复问题。

Laravel 源代码托管在 GitHub 上面，并且每个 Laravel 的项目都有自己的代码仓库：

- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Cashier for Braintree](https://github.com/laravel/cashier-braintree)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Website](https://github.com/laravel/laravel.com)
- [Laravel Art](https://github.com/laravel/art)

<a name="core-development-discussion"></a>
## 核心开发讨论

如果你想提出功能建议，或者改进现有的 Laravel 的行为，请到 Laravel Internals 项目的 [反馈栏](https://github.com/laravel/internals/issues) 讨论。如果你想提出功能建议，我们希望你愿意为此功能贡献一些代码。

有关错误、新功能和现有功能的实现讨论会在 Slack 的 [LaraChat](http://larachat.co) 群组上的 `#internals` 频道中进行。Laravel 的维护者 Taylor Otwell 在工作日的 8am 到 5pm（ UTC-06:00 或 America/Chicago ）通常都会出现在频道上，其它时间偶尔也会出现。

<a name="which-branch"></a>
## 选择分支

**所有的** 错误修复都应该发送到最新的稳定分支上。除非它们修复的功能只存在下一版的发布中，不然错误修复 **永远不** 应该发送到 `master` 分支。

**次要的** 且与现有的 Laravel 发布版本 **完全向下兼容** 的功能可以发送到最新的稳定分支。

**主要的** 新功能应该都发送到 `master` 分支，它包含下一版的 Laravel 发布内容。

如果不确定你的功能是主要的还是次要的，请在 Slack 的 [LaraChat](http://larachat.co) 群组上的 `#internals` 频道询问 Taylor Otwell。

<a name="security-vulnerabilities"></a>
## 安全漏洞

如果你发现 Laravel 存在安全漏洞，请发送电子邮件到 <a href="mailto:taylor@laravel.com">taylor@laravel.com</a> 给 Taylor Otwell。所有的安全漏洞将会及时予以处理。

<a name="coding-style"></a>
## Coding Style

Laravel 遵守 [PSR-2](https://phphub.org/topics/2079) 编码规范和 [PSR-4](https://phphub.org/topics/2081) 自动加载规范。

> 译者注：扩展阅读 - [所有 PSR 的标准规范](https://psr.phphub.org/)。

<a name="phpdoc"></a>
### 注释区块

`@param` 标签应该分行显示，并且每一个参数中间需相隔 **两个空格**，举个例子:

    /**
     * 为服务容器注册一个绑定
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

<a name="styleci"></a>
### StyleCI

Laravel 使用 [StyleCI](https://styleci.io/) 来做代码矫正，所有 PR 都会在合并后被矫正代码样式，这样做允许我们把精力放到贡献的内容上，而不是代码风格。

