# Laravel 源代码贡献指南

- [错误反馈](#bug-reports)
- [核心开发讨论](#core-development-discussion)
- [选择分支？](#which-branch)
- [安全漏洞](#security-vulnerabilities)
- [编码风格](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)

<a name="bug-reports"></a>
## 错误反馈

为了鼓励积极协作，Laravel 强烈地鼓励使用 Pull Request 指出修改的内容，而不仅仅只是反馈错误。「错误反馈」也可以用 PR 来提交失败测试。

如果你要提交错误反馈，你的问题应该包含标题和明确的问题描述，并尽可能多的提供相关的信息和演示该问题的代码示例。错误反馈的目的是让你和其他人可以轻松地重现并修复错误。

请记住，错误反馈的初衷是让其它有相同问题的人能够和你协作解决问题。不要指望反馈错误后会很快有人修复它。创建错误反馈是能帮助你和其他人开始着手修复问题的途径。

Laravel 源代码托管在 GitHub 上面，并且每个 Laravel 的项目都有自己的代码仓库：

<div class="content-list" markdown="1">
- [Laravel 应用](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel 文档](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Braintree 版 Laravel Cashier](https://github.com/laravel/cashier-braintree)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel 框架](https://github.com/laravel/framework)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead 构建脚本](https://github.com/laravel/settler)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel 搜索系统](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel 网站](https://github.com/laravel/laravel.com)
  </div>

<a name="core-development-discussion"></a>
## 核心开发讨论

如果你想提出现有的 Laravel 的功能建议或者改进，请到 Laravel Internals 的 [反馈栏](https://github.com/laravel/internals/issues) 讨论。如果你提出新功能，如果愿意，我们希望能请你至少实现一些完成该功能所需的代码。

有关错误、新功能和现有功能的实现的非正式讨论会在 [LaraChat](http://larachat.co) Slack 团队的 `#internals` 频道中进行。Laravel 的维护者 Taylor Otwell 通常都会在工作日的早上 8 点 到下午 5点（ UTC-06:00 或 America/Chicago ）出现在频道上，其它时间偶尔也会出现。

<a name="which-branch"></a>
## 选择分支？

**所有**错误修复都应该发送到最新的稳定分支或当前的 LTS 分支（5.5）上。错误修复**不**应该发送到 `master` 支，除非它们修复仅在即将发布的版本中存在的功能。

与当前 Laravel 版本**完全向后兼容**的**次要**功能可能会发送到最新的稳定分支。

**主要的** 新功能都应该发送到 `master` 分支，它包含即将发布的 Laravel 版本。

如果不确定你的功能是主要的还是次要的，请咨询 [LaraChat](http://larachat.co) Slack 团队的 `#internals` 频道上的 Taylor Otwell。

<a name="security-vulnerabilities"></a>
## 安全漏洞

如果你发现 Laravel 存在安全漏洞，请发送电子邮件给Taylor Otwell： <a href="mailto:taylor@laravel.com">taylor@laravel.com。他会及时解决所有安全漏洞。</a>

<a name="coding-style"></a>
## 编码风格

Laravel 遵循 [PSR-2](https://phphub.org/topics/2079) 编码规范和 [PSR-4](https://phphub.org/topics/2081) 自动加载规范。

<a name="phpdoc"></a>
### PHPDoc

以下是正确的 Laravel 注释的示例。请注意，`@param` 属性后跟两个空格、参数类型、两个空格，最后是变量名称：

    /**
     * 注册一个绑定到容器。
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

别担心你的编码风格不够漂亮！在合并 PR 后 [StyleCI](https://styleci.io) 会自动修正样式后再合并到 Laravel 仓库中。这样使得我们可以专注于贡献内容本身而不是编码风格。

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [Seven Du](https://github.com/medz) | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/5564821?s=300"> | 翻译 | 基于 Laravel 的社交开源系统 [ThinkSNS+](https://github.com/slimkit/thinksns-plus) 欢迎 Star。  |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  Review  | Stay Hungry. Stay Foolish. |


---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
