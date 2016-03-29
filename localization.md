# 本地化

- [简介](#introduction)
- [基本用法](#basic-usage)
    - [复数](#pluralization)
- [覆写扩展包的语言档](#overriding-vendor-language-files)

<a name="introduction"></a>
## 简介

Laravel 的本地化功能提供方便的方法来取得多语系的字符串，让你的网站可以简单的支持多语系。

语系档存放在 `resources/lang` 文件夹的文件里。在此文件夹内，应该要有网站支持的语系并对应到每一个子目录。：

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

语系档简单地返回键值和字符串数组，例如：

    <?php

    return [
        'welcome' => 'Welcome to our application'
    ];

#### 切换语系

网站的默认语系保存在 `config/app.php` 配置文件。您可以在任何时后使用 `App` facade 的 `setLocale` 方法动态地变换现行语系：

    Route::get('welcome/{locale}', function ($locale) {
        App::setLocale($locale);

        //
    });

您也可以设置 "备用语系"，它将会在当现行语言没有给定的语句时被使用。就像默认语言，备用语言也可以在 `config/app.php` 配置文件设置：

    'fallback_locale' => 'en',

<a name="basic-usage"></a>
## 基本用法

您可以使用 `trans` 辅助函数来取得语系字符串，`trans` 函数的第一个参数接受文件名和键值名称，例如，从 `resources/lang/messages.php` 语言档取得名称为 `welcome` 的句子：

    echo trans('messages.welcome');

当然，若您使用 [Blade 样版引擎](/docs/{{version}}/blade), 您可以使用 `{{ }}` 来输出句子：

    {{ trans('messages.welcome') }}

如果句子不存在， `trans` 方法将会返回键值的名称，如上例子会返回 `messages.welcome` 。

#### 在句子中做替代

如果需要，你也可以在语系档中定义占位符，占位符使用 `:` 开头，例如，您可以定义一则欢迎消息的占位符：

    'welcome' => 'Welcome, :name',

接着，传入替代用的第二个参数给 `trans` 方法：

    echo trans('messages.welcome', ['name' => 'Dayle']);

<a name="pluralization"></a>
### 复数

复数是个复杂的问题，不同语言对于复数有不同的规则，使用 "管线" 字符，可以区分单复数字串格式：

    'apples' => 'There is one apple|There are many apples',

接着，可以使用 `trans_choice` 方法来设置总数，例如，当总数大于一将会取得复数句子：

    echo trans_choice('messages.apples', 10);

由于 Laravel 的翻译器是来自于 Symfony 翻译扩展包，您甚至可以使用更复杂的复数规则：

    'apples' => '{0} There are none|[1,19] There are some|[20,Inf] There are many',

<a name="overriding-vendor-language-files"></a>
## 覆写扩展包的语言档

部分扩展包带有自己的语系档，您可以借由放置文件在 `resources/lang/vendor/{package}/{locale}` 来复写它们，而不是直接修改扩展包的核心文件。

例如，您需要复写 `skyrim/hearthfire` 扩展包的英文语系档 `messages.php`，您需要把文件放置在 `resources/lang/vendor/hearthfire/en/messages.php`。这个文件内，只要去定义需要覆写的语句，任何没有覆写的语句将会仍从扩展包的语言档加载。
