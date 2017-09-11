# Laravel 的本地化功能

- [简介](#introduction)
- [定义翻译语句](#defining-translation-strings)
    - [使用短键](#using-short-keys)
    - [使用翻译语句作为键](#using-translation-strings-as-keys)
- [提取翻译语句](#retrieving-translation-strings)
    - [翻译语句中的参数替换](#replacing-parameters-in-translation-strings)
    - [复数](#pluralization)
- [重写扩展包的语言包](#overriding-package-language-files)

<a name="introduction"></a>
## 简介

Laravel 的本地化功能提供方便的方法来获取多语言的字符串，让你的网站可以简单的支持多语言。

语言包存放在 `resources/lang` 目录下的文件里。在此目录中应用支持的每一种语言都应该有一个单独的子目录：

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

语言包简单地返回键值对数组，例如：

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

你也可以设置 「备用语言」 ，它将会在现有语言没有指定语句时被使用。就像默认语言那样，备用语言也可以在 `config/app.php` 配置文件设置：

    'fallback_locale' => 'en',

#### 指定当前语言

你可以使用 `App` facade 的 `getLocale` 及 `isLocale` 方法指定当前的语言环境或者检验当前语言是否是给定的值：

    $locale = App::getLocale();

    if (App::isLocale('en')) {
        //
    }

<a name="defining-translation-strings"></a>
## 定义翻译语句

<a name="using-short-keys"></a>
### 使用短键

通常，语言包存放在 `resources/lang` 目录下的文件里。在此目录中应用支持的每一种语言都应该有一个单独的子目录：

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

语言包简单地返回键值对数组，例如：

    <?php

    // resources/lang/en/messages.php

    return [
        'welcome' => 'Welcome to our application'
    ];

<a name="using-translation-strings-as-keys"></a>
### 使用翻译语句作为键

对于有大量翻译需求的应用， 如果每一条翻译语句都使用 「短键」 来定义，那么当你在视图中尝试去引用这些 「短键」 的时候，很容易变得混乱，分不清哪个对应哪个。因此，Laravel 也提供支持使用 「默认语言」 的翻译语句作为键，来定义其他语言的翻译语句。

使用翻译语句作为键的语言包需要在 `resources/lang` 目录下保存为 JSON 文件。例如，如果你的应用中有西班牙语的语言包，你应该新建一个 `resources/lang/es.json` 文件：

    {
        "I love programming.": "Me encanta programar."
    }

<a name="retrieving-translation-strings"></a>
## 提取翻译语句

你可以使用 `__` 辅助函数来获取翻译语句，`__` 方法接受文件名和键值作为其第一个参数。例如，让我们提取 `resources/lang/messages.php` 中的 `welcome` ：

    echo __('messages.welcome');

    echo __('I love programming.');

当然，如果你使用 [Blade 模板引擎](/docs/{{version}}/blade), 那么你可以在视图文件中使用 `{{ }}` 语法或者使用 `@lang` 指令来输出语句：

    {{ __('messages.welcome') }}

    @lang('messages.welcome')

如果指定的语句不存在，`__` 方法则会简单的返回这个键名。所以，如果上述示例中的键不存在，那么 `__` 方法则会返回 `messages.welcome` 。

<a name="replacing-parameters-in-translation-strings"></a>
### 翻译语句中的参数替换

如果需要，你也可以在翻译语句中定义占位符。所有的占位符都使用的 `:` 开头。例如，你可以自定义一则欢迎消息的占位符：

    'welcome' => 'Welcome, :name',

你可以在 `__` 方法中传递一个数组作为第二个参数，它会将数组的值替换到语言内容的占位符中：

    echo __('messages.welcome', ['name' => 'dayle']);

如果你的占位符中包含了首字母大写或者全体大写，翻译过来的内容也会做相应的处理：

    'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
    'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle


<a name="pluralization"></a>
### 复数

复数是个复杂的问题，不同语言对于复数有不同的规则。使用管道符 `|` ，可以区分单复数字符串格式：

    'apples' => 'There is one apple|There are many apples',

你甚至可以创建使用更复杂的复数规则，例如根据数量的范围不同来指定不同翻译语句：

    'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',

当你定义完复数语句条件的时候，你可以使用 `trans_choice` 方法来设置「总数」以获取符合对应条件的复数翻译语句。例如，在这个例子中，设置「总数」为 10 ，符合数量范围 1 至 19，所以会得到 `There are some` 这条复数语句：

    echo trans_choice('messages.apples', 10);

<a name="overriding-package-language-files"></a>
## 重写扩展包的语言包

部分扩展包带有自己的语言包，你可以通过在 `resources/lang/vendor/{package}/{locale}` 放置文件来重写它们，而不是直接修改扩展包的核心文件。

例如，你需要重写 `skyrim/hearthfire` 扩展包的英文语言包 `messages.php` ，则需要把文件放置在 `resources/lang/vendor/hearthfire/en/messages.php` 。在这个文件中定义你想要重写的翻译语句，所有没有重写的语句将会加载扩展包的语言包中原来的语句。


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org