# 本土化

- [简介](#introduction)
- [提取语句](#retrieving-language-lines)
    - [语句中的参数替换](#replacing-parameters-in-language-lines)
    - [复数](#pluralization)
- [重写扩展包的语言包](#overriding-package-language-files)

<a name="introduction"></a>
## 简介

Laravel 的本地化功能提供方便的方法来获取多语言的字符串，让你的网站可以简单的支持多语言。

语言包存放在 `resources/lang` 目录下的文件里。在此目录中应该有应用对应支持的语言并将其对应到每一个子目录：

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

语言包简单地返回键值和字符串数组，例如：

    <?php

    return [
        'welcome' => 'Welcome to our application'
    ];

### 切换语言

应用的默认语言保存在 `config/app.php` 配置文件中。当然，你可以根据需求自由的修改当前设置，可以使用 `App` facade 的 `setLocale` 方法动态地更改现有语言：

    Route::get('welcome/{locale}', function ($locale) {
        App::setLocale($locale);

        //
    });

你也可以设置 「备用语言」 ，它将会在当现有语言没有指定语句时被使用。就像默认语言那样，备用语言也可以在 `config/app.php` 配置文件设置：

    'fallback_locale' => 'en',

#### 指定当前语言

你可以使用 `App` facade 的 `getLocale` 及 `isLocale` 方法指定当前的语言环境或者检验当前语言是否是给定的值：

    $locale = App::getLocale();

    if (App::isLocale('en')) {
        //
    }

<a name="retrieving-language-lines"></a>
## 提取语句

你可以使用 `trans` 辅助函数来获取语言字符串，`trans` 方法接受文件名和键值作为其第一个参数。例如，让我们提取 `resources/lang/messages.php` 中的 `welcome` ：

    echo trans('messages.welcome');

当然，如果你使用 [Blade 模板引擎](/docs/{{version}}/blade), 那么你可以在视图文件中使用 `{{ }}` 语法或者使用 `@lang` 命令来输出语句：

    {{ trans('messages.welcome') }}

    @lang('messages.welcome')

如果指定的语句不存在，`trans` 方法则会简单的返回这个键名。所以，如果上述示例中的键不存在，那么 `trans` 方法则会返回 `messages.welcome` 。

<a name="replacing-parameters-in-language-lines"></a>
### 语句中的参数替换

如果需要，你也可以在语句中定义占位符。所有的占位符都使用的 `:` 开头。例如，你可以自定义一则欢迎消息的占位符：

    'welcome' => 'Welcome, :name',

你可以在 `trans` 方法中传递一个数组作为第二个参数，它会将数组的值替换到语言内容的占位符中：

    echo trans('messages.welcome', ['name' => 'dayle']);

如果你的占位符中包含了首字母大写或者全体大写，翻译过来的内容也会相应的做相应的处理：

    'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
    'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle


<a name="pluralization"></a>
### 复数

复数是个复杂的问题，不同语言对于复数有不同的规则。使用管道符 `|` ，可以区分单复数字符串格式：

    'apples' => 'There is one apple|There are many apples',

接着，你可以使用 `trans_choice` 方法来设置总数。例如，当总数大于一时将会获取复数语句：

    echo trans_choice('messages.apples', 10);

因为 laravel 的翻译器是基于 Symfony 翻译扩展包的，因此你甚至可以使用更复杂的复数规则：

    'apples' => '{0} There are none|[1,19] There are some|[20,Inf] There are many',

<a name="overriding-package-language-files"></a>
## 重写扩展包的语言包

部分扩展包带有自己的语言包，你可以通过在 `resources/lang/vendor/{package}/{locale}` 放置文件来重写它们，而不是直接修改扩展包的核心文件。

例如，你需要重写 `skyrim/hearthfire` 扩展包的英文语言包 `messages.php` ，则需要把文件放置在 `resources/lang/vendor/hearthfire/en/messages.php` 。所有没有重写的语句仍将会从扩展包的语言包中被加载。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@江边望海](http://blog.jiangbianwanghai.com)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5306_1470714129.jpeg?imageView2/1/w/100/h/100">  |  翻译  | 郑州悉知资深技术经理、讲师，10多年软件产品研发、测试、咨询及管理工作经验。Follow me [@jiangbianwanghai](https://github.com/jiangbianwanghai/) at Github |
| [@summerblue](https://github.com/summerblue)  | <img class="avatar-66 rm-style" src="https://avatars2.githubusercontent.com/u/324764?v=3&s=100">  |  Review  | A man seeking for Wisdom. |
