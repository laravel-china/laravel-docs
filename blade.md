# Blade 模板

- [简介](#introduction)
- [模板继承](#template-inheritance)
    - [定义页面布局](#defining-a-layout)
    - [继承页面布局](#extending-a-layout)
- [显示数据](#displaying-data)
- [控制结构](#control-structures)
- [服务注入](#service-injection)
- [扩充 Blade](#extending-blade)

<a name="introduction"></a>
## 简介

Blade 是 Laravel 所提供的简单且强大的模板引擎。相较于其它知名的 PHP 模板引擎，Blade 并不会限制你必须在视图中使用 PHP 代码。所有 Blade 视图会被编译成一般的 PHP 代码并缓存，直到它们被更动为止，这代表着基本上 Blade 不会对你的应用程序产生负担。Blade 视图文件使用 `.blade.php` 做为扩展名，且通常保存于 `resources/views` 文件夹。

<a name="template-inheritance"></a>
## 模板继承

<a name="defining-a-layout"></a>
### 定义页面布局

使用 Blade 模板的两个主要优点为 *模板继承* 与 *区块*。让我们先看一个简单的例子来上手。首先，我们确认一下「主要的」页面布局。由于大多数的网页应用程序在不同页面都保持着相同的布局方式，这便于定义这个布局为单一的 Blade 视图：

    <!-- 文件保存于 resources/views/layouts/master.blade.php -->

    <html>
        <head>
            <title>应用程序名称 - @yield('title')</title>
        </head>
        <body>
            @section('sidebar')
                这是主要的侧边栏。
            @show

            <div class="container">
                @yield('content')
            </div>
        </body>
    </html>

如你所见，这个文件包含了传统的 HTML 语法。不过，请注意 `@section` 与 `@yield` 命令。正如其名，`@section` 命令定义一个内容区块，而 `@yield` 命令被用来 “显示给定区块” 的内容。

现在，我们已经定义了这个应用程序的布局，让我们来定义一个继承此布局的子页面。

<a name="extending-a-layout"></a>
### 继承页面布局

当正在定义子页面时，你可以使用 Blade 的 `@extends` 命令指定子页面应该「继承」哪一个布局。当视图 `@extends` Blade 的布局之后，即可使用 `@section` 命令将内容注入于布局的区块中。切记，如上述例子所见，这些区块的内容都会使用 `@yield` 显示在布局中：

    <!-- 保存于 resources/views/child.blade.php -->

    @extends('layouts.master')

    @section('title', '页面标题')

    @section('sidebar')
        @@parent

        <p>这边会附加在主要的侧边栏。</p>
    @endsection

    @section('content')
        <p>这是我的主要内容。</p>
    @endsection

在这个例子中，`sidebar` 区块利用了 `@@parent` 命令增加（而不是覆盖）内容至布局的侧边栏。`@@parent` 命令会在视图输出时被置换成布局的内容。

当然，就像一般的 PHP 视图，可以在路由中使用 `view` 辅助函数返回 Blade 视图：

    Route::get('blade', function () {
        return view('child');
    });

<a name="displaying-data"></a>
## 显示数据

你可以使用「中」括号包住变量以显示传递至 Blade 视图的数据。举例而言，就像以下的路由设置：

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

你可以像这样显示 `name` 变量的内容：

    Hello, {{ $name }}.

当然，也不是一定只能显示传递至视图的变量内容。你也可以显示 PHP 函数的结果。实际上，你可以放置任何你需要的 PHP 代码到 Blade 显示语法里面：

    目前的 UNIX 时间戳记为 {{ time() }}。

> **注意：** Blade 的 `{{ }}` 语法会自动调用 PHP `htmlentites` 函数来防御 XSS 攻击。

#### Blade 与 JavaScript 框架

由于许多 JavaScript 框架也使用「大」括号在浏览器中显示给定的表达式，你可以使用 `@` 符号来告知 Blade 渲染引擎该表达式应该维持原样。举个例子：

    <h1>Laravel</h1>

    Hello, @{{ name }}.

在这个例子中，`@` 符号会被 Blade 移除。而且，Blade 引擎会保留 `{{ name }}` 表达式，如此一来便可让其它 JavaScript 框架所应用。

#### 当数据存在时输出

有时候你想要输出一个变量，但你并不确定这个变量是否已被设置。我们可以用像这样的冗长 PHP 代码表达：

    {{ isset($name) ? $name : 'Default' }}

不过，Blade 提供了较方便的缩写来替代写三元运算符表达式：

    {{ $name or 'Default' }}

在这个例子中，如果 `$name` 变量存在，它的值将会被显示出来。但是，如果这个变量不存在，便会显示 `Default`。

#### 显示未转译的数据

在默认情况下，Blade 模板中的 `{{ }}` 表达式将会自动调用 PHP 的 `htmlentities` 函数，以避免 XSS 攻击。如果你不希望你的数据被转义，可以使用下列的语法：

    Hello, {!! $name !!}.

> **注意：** 要非常小心处理用户提供的字符串，请总是使用双大括号语法来转义内容中的 HTML 元素，避免 XSS 攻击。

<a name="control-structures"></a>
## 控制结构

除了模板继承与显示数据功能以外，Blade 也提供了方便的缩写给一般的 PHP 结构控制语句，像是条件表达式和循环。这些缩写提供了干净、简洁的方式来使用 PHP 的控制结构。

#### If 表达式

你可以使用 `@if`、`@elseif`、`@else` 及 `@endif` 命令建构 `if` 表达式。这些命令的功能等同于在 PHP 中的语法：

    @if (count($records) === 1)
        我有一条记录！
    @elseif (count($records) > 1)
        我有多条记录！
    @else
        我没有任何记录！
    @endif

为了方便，Blade 也提供了 `@unless` 命令：

    @unless (Auth::check())
        你尚未登录。
    @endunless

#### 循环

除了条件表达式外，Blade 也支持 PHP 的循环结构：

    @for ($i = 0; $i < 10; $i++)
        目前的值为 {{ $i }}
    @endfor

    @foreach ($users as $user)
        <p>此用户为 {{ $user->id }}</p>
    @endforeach

    @forelse ($users as $user)
        <li>{{ $user->name }}</li>
    @empty
        <p>没有用户</p>
    @endforelse

    @while (true)
        <p>我永远都在跑循环。</p>
    @endwhile

#### 引入子视图

Blade 的 `@include` 命令用来引入已存在的视图，所有在父视图的可用变量在被引入的视图中都是可用的。

    <div>
        @include('shared.errors')

        <form>
            <!-- 表单内容 -->
        </form>
    </div>

尽管被引入的视图会继承父视图中的所有数据，你也可以传递额外数据的数组至被引入的页面：

    @include('view.name', ['some' => 'data'])

> **注意：** 你必须在 Blade 视图中避免使用 `__DIR__` 及 `__FILE__` 常数，因为他们会引用视图被缓存的位置。

#### 为集合渲染视图

你可以使用 Blade 的 `@each` 命令将循环及引入结合成一行：

    @each('view.name', $jobs, 'job')

第一个参数为每个元素渲染的局部视图，第二个参数为你要迭代的数组或集合，而第三个参数为迭代时被分配至视图中的变量名称。所以，举例来说，如果你迭代一个 `jobs` 数组，通常你会希望在局部视图中通过 `job` 变量访问每一个 job。

你也可以传递第四个参数至 `@each` 命令。此参数为当给定的数组为空时，将会被渲染的视图。

    @each('view.name', $jobs, 'job', 'view.empty')

#### 注解

Blade 也允许在页面中定义注解，然而，有异于 HTML 的注解，Blade 的注解并不会被包含在应用程序返回的 HTML 内：

    {{-- 此注解将不会出现在渲染后的 HTML --}}

<a name="service-injection"></a>
## 服务注入

`@inject` 命令可以取出 Laravel [服务容器](/docs/{{version}}/container)中的服务。传递给 `@inject` 的第一个参数为置放该服务的变量名称，而第二个参数为你想要解析的服务的类或是接口的名称：

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        每月收入：{{ $metrics->monthlyRevenue() }}。
    </div>

<a name="extending-blade"></a>
## 扩充 Blade

Blade 甚至允许你自定义命令，你可以使用 `directive` 方法注册命令。当 Blade 编译器遇到该命令时，它将会带参数调用提供的回调函数。

以下例子创建一个把给定的 `$var` 格式化的 `@datetime($var)` 命令：

    <?php

    namespace App\Providers;

    use Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 运行服务注册后的启动进程。
         *
         * @return void
         */
        public function boot()
        {
            Blade::directive('datetime', function($expression) {
                return "<?php echo with{$expression}->format('m/d/Y H:i'); ?>";
            });
        }

        /**
         * 在容器注册绑定。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

如你所见，Laravel 的 `with` 辅助函数被用在这个命令中。`with` 辅助函数会简单地返回给定的对象或值，并允许使用便利的方法链结。最后此命令生成的 PHP 会是：

    <?php echo with($var)->format('m/d/Y H:i'); ?>


