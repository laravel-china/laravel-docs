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

为了提倡积极协作，Laravel 强烈地鼓励使用 Pull Request，而不仅仅只是反馈错误。「错误反馈」可以以一个包含失败测试的 Pull Request 的形式发送。

假如你提交了错误反馈，那么反馈中应该包含着标题和详尽的问题描述，并尽可能多的提供相关的信息和错误问题的代码示例。错误反馈的主要目的是让自己和其他人可以简单地重现并修复错误。

请记住，错误反馈的初衷是让其它有相同问题的人可以协作解决问题。不要期望反馈错误后会很快有人会马上修复它。创建错误反馈主要是为了能帮助你自己和其他人开始着手修复问题。

Laravel 源代码托管在 GitHub 上面，并且每个 Laravel 的项目都有自己的代码仓库：

<div class="content-list" markdown="1">
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Cashier for Braintree](https://github.com/laravel/cashier-braintree)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Website](https://github.com/laravel/laravel.com)
</div>

<a name="core-development-discussion"></a>
## 核心开发讨论

如果你想提出功能建议，或者改进现有的 Laravel 的行为，请到 Laravel Internals 项目的 [反馈栏](https://github.com/laravel/internals/issues) 讨论。如果你想提出功能建议，我们希望你愿意为此功能贡献一些代码。

有关错误、新功能和现有功能的实现讨论会在 Slack 的 [LaraChat](http://larachat.co) 群组上的 `#internals` 频道中进行。Laravel 的维护者 Taylor Otwell 在工作日的 8am 到 5pm（ UTC-06:00 或 America/Chicago ）通常都会出现在频道上，其它时间偶尔也会出现。

<a name="which-branch"></a>
## 选择分支？

**所有的** 错误修复都应该发送到最新的稳定分支或当前的 LTS 分支（5.5）上。否则错误修复 **永远不** 应该发送到 `master` 分支。

**次要的** 且与现有的 Laravel 发布版本 **完全向下兼容** 的功能可以发送到最新的稳定分支。

**主要的** 新功能应该都发送到 `master` 分支，它包含下一版的 Laravel 发布内容。

如果不确定你的功能是主要的还是次要的，请在 Slack 的 [LaraChat](http://larachat.co) 群组上的 `#internals` 频道询问 Taylor Otwell。

<a name="security-vulnerabilities"></a>
## 安全漏洞

如果你发现 Laravel 存在安全漏洞，请发送电子邮件到 <a href="mailto:taylor@laravel.com">taylor@laravel.com</a> 给 Taylor Otwell。所有的安全漏洞将会及时予以处理。

<a name="coding-style"></a>
## 编码风格

Laravel 遵循 [PSR-2](https://phphub.org/topics/2079) 编码规范和 [PSR-4](https://phphub.org/topics/2081) 自动加载规范。

<a name="phpdoc"></a>
### PHPDoc

`@param` 标签应该分行显示，并且每一个参数中间需相隔 **两个空格**，举个例子:

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

别担心你的编码风格不够漂亮！所有 Pull Request 合并后 [StyleCI](https://styleci.io) 会自动将任何样式修正并合并到 Laravel 仓库中。这样使得我们可以专注于贡献内容本身而不是编码风格。

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [Seven Du](https://github.com/medz) | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/5564821?s=300"> | 翻译 | 基于 Laravel 的社交开源系统 [ThinkSNS+](https://github.com/slimkit/thinksns-plus) 欢迎 Star。  |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： http://d.laravel-china.org
