# 贡献指南

- [Bug 反馈](#bug-reports)
- [核心开发讨论](#core-development-discussion)
- [选择分支](#which-branch)
- [安全漏洞](#security-vulnerabilities)
- [编码风格](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)

<a name="bug-reports"></a>
## Bug 反馈

为了提倡积极协作，[Laravel](http://laravelacademy.org/tags/laravel) 强烈鼓励使用 [GitHub](http://laravelacademy.org/tags/github) 的 `pull requests`，而不是仅仅报告 Bug，「Bug 反馈」可以以一个包含失败测试的 Pull Request 的形式发送。

假如你提交了「Bug 反馈」，那么反馈中应该包含着标题和详尽的问题描述，并尽可能多的提供相关的信息和该 Bug 的代码示例。这样做的主要目的是让自己和其他人可以简单地重现并修复该 Bug。

请记住，「Bug 反馈」的初衷是让其它有相同问题的人可以协作解决问题。不要期望反馈 Bug 后，马上就会有人修复了它。创建「Bug 反馈」主要是为了能帮助你自己和其他人开始着手修复问题。

Laravel 源代码托管在 GitHub 上面，并且每一个 Laravel 项目都有其对应的代码仓库：

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

你可以在 Laravel 讨论组 [issue board](https://github.com/laravel/internals/issues) 提出新特性或者改进。如果你提出了新的特性，请至少附上完成新特性需要的相关代码。

有关错误、新功能和现有功能的实现讨论会在 Slack 的 [LaraChat](http://larachat.co) 群组上的 `#internals` 频道中进行。Laravel 的维护者 Taylor Otwell 在工作日的上午8点到下午5点（西六区或美国芝加哥时间）通常会出现在频道上，其它时间也可能偶尔在线。

<a name="which-branch"></a>
## 选择分支

**所有的** Bug 修复应该提交到 `latest stable` 稳定分支上。Bug 修复 **永远不要** 提交到 `master` 分支，除非它修复的功能只存在下一版的发布中。 

**次要的** 且与现有的 Laravel 发布版本 **完全向下兼容** 的功能可以发送到最新的稳定分支。

**主要的** 新功能应该都发送到 `master` 分支，它包含下一版的 Laravel 发布内容。

如果不确定你的功能是主要的还是次要的，请在 Slack 的 [LaraChat](http://larachat.co) 群组上的 `#internals` 频道询问 Taylor Otwell。

<a name="security-vulnerabilities"></a>
## 安全漏洞

如果你发现 Laravel 存在安全漏洞，请发送电子邮件到 <a href="mailto:taylor@laravel.com">taylor@laravel.com</a> 给 Taylor Otwell。所有的安全漏洞将会被及时解决。

<a name="coding-style"></a>
## 编码风格

Laravel 遵守 [PSR-2](https://phphub.org/topics/2079) 编码规范和 [PSR-4](https://phphub.org/topics/2081) 自动加载规范。

> 译者注：扩展阅读 - [所有 PSR 的标准规范](https://psr.phphub.org/)。

<a name="phpdoc"></a>
### PHPDoc

下面是一个示范文档单元。请注意 `@param` 属性后有是有两个空格，参数类型两个以上空格，最后是变量类型名称。

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

如果你的编码风格不完美，别着急！当你 `pull requests` 到 Laravel  版本库后， [StyleCI](https://styleci.io/) 可以自动合并修复任何风格。这使我们能够专注于内容的贡献，而不是编码风格 。