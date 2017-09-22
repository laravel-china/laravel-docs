# Laravel 的本地化功能

- [简介](#introduction)
- [定义翻译字符串](#defining-translation-strings)
    - [使用短键](#using-short-keys)
    - [使用翻译字符串作为键](#using-translation-strings-as-keys)
- [检索翻译字符串](#retrieving-translation-strings)
    - [翻译语句中的参数替换](#replacing-parameters-in-translation-strings)
    - [复数](#pluralization)
- [重写扩展包的语言文件](#overriding-package-language-files)

<a name="introduction"></a>
## 简介

Laravel 的本地化功能为在应用程序中支持多种语言提供方便的方法来检索各种语言的字符串。语言字符串存储在 `resources/lang` 目录下的文件里。在此目录中，但凡应用支持的语言都应该有一个对应的子目录：

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

所有语言文件只返回键值对数组，例如：

    <?php

    return [
        'welcome' => 'Welcome to our application'
    ];

### 区域设置

应用的默认语言保存在 `config/app.php` 配置文件中。你可以根据需要修改当前设置。还可以使用 `App` Facade 的 `setLocale` 方法动态地更改当前语言：

    Route::get('welcome/{locale}', function ($locale) {
        App::setLocale($locale);

        //
    });

你也可以设置 「备用语言」 ，它将会在当前语言不包含给定的翻译字符串时被使用。像默认语言一样，备用语言也可以在 `config/app.php` 配置文件设置：

    'fallback_locale' => 'en',

#### 确定当前语言环境

你可以使用 `App` Facade 的 `getLocale` 及 `isLocale` 方法确定当前的区域设置或者检查语言环境是否为给定值：

    $locale = App::getLocale();

    if (App::isLocale('en')) {
        //
    }

<a name="defining-translation-strings"></a>
## 定义翻译字符串

<a name="using-short-keys"></a>
### 使用短键

通常，翻译字符串存放在 `resources/lang` 目录下的文件里。在此目录中但凡应用支持的语言都应该有一个对应的子目录：

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

所有语言文件只返回键值对数组，例如：

    <?php

    // resources/lang/en/messages.php

    return [
        'welcome' => 'Welcome to our application'
    ];

<a name="using-translation-strings-as-keys"></a>
### 使用翻译字符串作为键

对于有大量翻译需求的应用， 如果每一条翻译语句都使用 「短键」 来定义，那么当你在视图中尝试去引用这些 「短键」 的时候，很容易变得混乱。因此，Laravel 也提供支持使用字符串的「默认」翻译作为关键字定义翻译字符串。

使用翻译字符串作为键的翻译文件作为 JSON 文件存储在 `resources/lang` 目录中。例如，如果你的应用中有西班牙语翻译，你应该新建一个 `resources/lang/es.json` 文件：

    {
        "I love programming.": "Me encanta programar."
    }

<a name="retrieving-translation-strings"></a>
## 检索翻译字符串

你可以使用辅助函数 `__` 从语言文件中检索，`__` 方法接受翻译字符串的文件名和键值作为其第一个参数。例如，让我们检索 `resources/lang/messages.php` 语言文件中的 `welcome` 翻译字符串：

    echo __('messages.welcome');

    echo __('I love programming.');

如果使用 [Blade 模板引擎](/docs/{{version}}/blade)，可以在视图文件中使用 `{{ }}` 语法或者使用 `@lang` 指令来打印翻译字符串：

    {{ __('messages.welcome') }}

    @lang('messages.welcome')

如果指定的翻译字符串不存在，`__` 方法则会简单地返回指定的翻译字符串键名。所以，如果上述示例中的翻译字符串键不存在，那么 `__` 方法则会返回 `messages.welcome` 。

<a name="replacing-parameters-in-translation-strings"></a>
### 翻译语句中的参数替换

如果需要也可以在翻译字符串中定义占位符。所有的占位符都有一个前缀 `:`。例如，你可以使用持有人名称定义欢迎消息：

    'welcome' => 'Welcome, :name',

你可以在 `__` 方法中传递一个数组作为第二个参数，它会将数组的值替换到翻译字符串的占位符中：

    echo __('messages.welcome', ['name' => 'dayle']);

如果你的占位符中包含了首字母大写或者全体大写，翻译过来的内容也会做相应的处理：

    'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
    'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle


<a name="pluralization"></a>
### 复数

复数是个复杂的问题，不同语言对于复数有不同的规则。使用管道符 `|` ，可以区分字符串的单复数形式：

    'apples' => 'There is one apple|There are many apples',

你甚至可以创建更复杂的复数规则，为多个数字范围指定翻译字符串：

    'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',

在定义具有复数选项的翻译字符串之后，你可以使用 `trans_choice` 方法来检索给定「数量」的内容。在这个例子中，设置「总数」为 10 ，符合数量范围 1 至 19，所以会得到 `There are some` 这条复数语句：

    echo trans_choice('messages.apples', 10);

<a name="overriding-package-language-files"></a>
## 重写扩展包的语言文件

部分扩展包可能会附带自己的语言文件。你可以通过在 `resources/lang/vendor/{package}/{locale}` 放置文件来重写它们，而不是直接修改扩展包的核心文件。

例如，你需要重写 `skyrim/hearthfire` 扩展包的英文语言文件 `messages.php` ，则需要把文件放置在 `resources/lang/vendor/hearthfire/en/messages.php` 。在这个文件中只应定义要覆盖的翻译字符串。任何没有被覆盖的翻译字符串仍将从扩展包的原始语言文件中加载。

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | Review | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
