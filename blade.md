# Laravel 的 Blade 模板引擎

- [简介](#introduction)
- [模板继承](#template-inheritance)
    - [定义页面布局](#defining-a-layout)
    - [继承页面布局](#extending-a-layout)
- [组件 & Slots](#components-and-slots)
- [显示数据](#displaying-data)
    - [Blade & JavaScript 框架](#blade-and-javascript-frameworks)
- [控制结构](#control-structures)
    - [If 语句](#if-statements)
    - [Switch 语句](#switch-statements)
    - [循环](#loops)
    - [循环变量](#the-loop-variable)
    - [注释](#comments)
    - [PHP](#php)
- [引入子视图](#including-sub-views)
    - [为集合渲染视图](#rendering-views-for-collections)
- [堆栈](#stacks)
- [服务注入](#service-injection)
- [扩充 Blade](#extending-blade)
    - [自定义 If 语句](#custom-if-statements)

<a name="introduction"></a>
## 简介

Blade 是 Laravel 提供的一个既简单又强大的模板引擎。和其他流行的 PHP 模板引擎不一样，Blade 并不限制你在视图中使用原生 PHP 代码。所有 Blade 视图文件都将被编译成原生的 PHP 代码并缓存起来，除非它被修改，否则不会重新编译，这就意味着 Blade 基本上不会给你的应用增加任何额外负担。Blade 视图文件使用 `.blade.php` 扩展名，一般被存放在 `resources/views` 目录。

<a name="template-inheritance"></a>
## 模板继承

<a name="defining-a-layout"></a>
### 定义页面布局

Blade 的两个主要优点是 _模板继承_ 和 _区块_ 。

为方便开始，让我们先通过一个简单的例子来上手。首先，我们需要确认一个 "master" 的页面布局。因为大多数 web 应用是在不同的页面中使用相同的布局方式，我们可以很方便的定义这个 Blade 布局视图：

    <!-- 文件保存于 resources/views/layouts/app.blade.php -->

    <html>
        <head>
            <title>应用程序名称 - @yield('title')</title>
        </head>
        <body>
            @section('sidebar')
                这是 master 的侧边栏。
            @show

            <div class="container">
                @yield('content')
            </div>
        </body>
    </html>

如你所见，该文件包含了典型的 HTML 语法。不过，请注意 `@section` 和 `@yield` 命令。 `@section` 命令正如其名字所暗示的一样是用来定义一个视图区块的，而  `@yield` 指令是用来显示指定区块的内容的。

现在，我们已经定义好了这个应用程序的布局，让我们接着来定义一个继承此布局的子页面。

<a name="extending-a-layout"></a>
### 继承页面布局

当定义子页面时，你可以使用 Blade 提供的 `@extends` 命令来为子页面指定其所 「继承」 的页面布局。 当子页面继承布局之后，即可使用 `@section` 命令将内容注入于布局的 `@section` 区块中。切记，在上面的例子里，布局中使用 `@yield` 的地方将会显示这些区块中的内容：

    <!-- 文件保存于 resources/views/child.blade.php -->

    @extends('layouts.app')

    @section('title', 'Page Title')

    @section('sidebar')
        @@parent

        <p>这将被添加到主侧边栏。</p>
    @endsection

    @section('content')
        <p>This is my body content.</p>
    @endsection

在上面的例子里，`sidebar` 区块利用了 `@@parent` 命令追加布局中的 sidebar 区块中的内容，如果不使用则会覆盖掉布局中的这部分内容。 `@@parent` 命令会在视图被渲染时替换为布局中的内容。

当然，可以通过在路由中使用全局辅助函数 `view` 来返回 Blade 视图：

    Route::get('blade', function () {
        return view('child');
    });

<a name="components-and-slots"></a>
## 组件 & Slots

组件和 slots 能提供类似于区块和布局的好处；不过，一些人可能发现组件和 slots 更容易理解。首先，让我们假设一个会在我们应用中重复使用的「警告」组件:

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        {{ $slot }}
    </div>

`{{ $slot }}` 变量将包含我们希望注入到组件的内容。现在，我们可以使用 `@component` 指令来构造这个组件：

    @component('alert')
        <strong>哇！</strong> 出现了一些问题！
    @endcomponent

有些时候它对于定义组件的多个 slots 是非常有帮助的。让我们修改我们的警告组件，让它支持注入一个「标题」。 已命名的 slots 将显示「相对应」名称的变量的值:


    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        <div class="alert-title">{{ $title }}</div>

        {{ $slot }}
    </div>

现在，我们可以使用 `@slot` 指令注入内容到已命名的 slot 中，任何没有被 `@slot` 指令包裹住的内容将传递给组件中的 `$slot` 变量:

    @component('alert')
        @slot('title')
            Forbidden
        @endslot

        你没有权限访问这个资源！
    @endcomponent

#### 传递额外的数据给组件

有时候你可能需要传递额外的数据给组件。为了解决这个问题，你可以传递一个数组作为第二个参数传递给 `@component` 指令。所有的数据都将以变量的形式传递给组件模版:

    @component('alert', ['foo' => 'bar'])
        ...
    @endcomponent

<a name="displaying-data"></a>
## 显示数据

你可以使用 「中括号」 包住变量以显示传递至 Blade 视图的数据。如下面的路由设置：

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

你可以像这样显示 `name` 变量的内容：

    Hello, {{ $name }}.

当然也不是说一定只能显示传递至视图的变量内容。你也可以显示 PHP 函数的结果。事实上，你可以在 Blade 中显示任意的 PHP 代码：

    The current UNIX timestamp is {{ time() }}.

> {note} Blade `{{ }}` 语法会自动调用 PHP `htmlspecialchars` 函数来避免 XSS 攻击。

#### 显示未被转义的数据

在默认情况下，Blade 模板中的 `{{ }}` 表达式将会自动调用 PHP `htmlspecialchars` 函数来转义数据以避免 XSS 的攻击。如果你不想你的数据被转义，你可以使用下面的语法：

    Hello, {!! $name !!}.

> {note} 要非常小心处理用户输入的数据时，你应该总是使用 `{{  }}` 语法来转义内容中的任何的 HTML 元素，以避免 XSS 攻击。

<a name="blade-and-javascript-frameworks"></a>
### Blade & JavaScript 框架

由于很多 JavaScript 框架都使用花括号来表明所提供的表达式，所以你可以使用 `@` 符号来告知 Blade 渲染引擎你需要保留这个表达式原始形态，例如：

    <h1>Laravel</h1>

    Hello, @{{ name }}.

在这个例子里，`@` 符号最终会被 Blade 引擎剔除，并且 `{{ name }}` 表达式会被原样的保留下来，这样就允许你的 JavaScript 框架来使用它了。

#### `@verbatim` 指令

如果你需要在页面中大片区块中展示 JavaScript 变量，你可以使用 `@verbatim` 指令来包裹 HTML 内容，这样你就不需要为每个需要解析的变量增加 `@` 符号前缀了：


    @verbatim
        <div class="container">
            Hello, {{ name }}.
        </div>
    @endverbatim

<a name="control-structures"></a>
## 控制结构

除了模板继承与数据显示的功能以外，Blade 也给一般的 PHP 结构控制语句提供了方便的缩写，比如条件表达式和循环语句。这些缩写提供了更为清晰简明的方式来使用 PHP 的控制结构，而且还保持与 PHP 语句的相似性。

<a name="if-statements"></a>
### If 语句

你可以通过 `@if`, `@elseif`, `@else` 及  `@endif` 指令构建 `if` 表达式。这些命令的功能等同于在 PHP 中的语法：

    @if (count($records) === 1)
        我有一条记录！
    @elseif (count($records) > 1)
        我有多条记录！
    @else
        我没有任何记录！
    @endif

为了方便，Blade 也提供了一个 `@unless` 命令：

    @unless (Auth::check())
        你尚未登录。
    @endunless

除了已经讨论的条件指令之外，`@isset` 和 `@empty`指令也可以用作各自PHP功能的方便的快捷方式：

    @isset($records)
        // $records is defined and is not null...
    @endisset

    @empty($records)
        // $records is "empty"...
    @endempty

#### 身份验证快捷方式

The `@auth` and `@guest` directives may be used to quickly determine if the current user is authenticated or is a guest:
`@auth` 和 `@guest` 指令可以用来快速确定当前用户是通过身份认证证的用户还是游客：

    @auth
        // 用户已经通过身份认证...
    @endauth

    @guest
        // 用户没有通过身份认证...
    @endguest

<a name="switch-statements"></a>
### Switch 语句

可以使用 `@switch`，`@case`，`@break`，`@default` 和 `@endswitch` 指令来构建 Switch 语句：

    @switch($i)
        @case(1)
            First case...
            @break

        @case(2)
            Second case...
            @break

        @default
            Default case...
    @endswitch

<a name="loops"></a>
### 循环

除了条件表达式外，Blade 也支持 PHP 的循环结构，这些命令的功能等同于在 PHP 中的语法：

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
        <p>死循环了。</p>
    @endwhile

> {tip} 当循环时，你可以使用 [循环变量](#the-loop-variable) 来获取循环中有价值的信息，比如循环中的首次或最后的迭代。

当使用循环时，你可能也需要一些结束循环或者跳出当前循环的命令：

    @foreach ($users as $user)
        @if ($user->type == 1)
            @continue
        @endif

        <li>{{ $user->name }}</li>

        @if ($user->number == 5)
            @break
        @endif
    @endforeach

你也可以使用命令声明包含条件的方式在一条语句中达到中断:

    @foreach ($users as $user)
        @continue($user->type == 1)

        <li>{{ $user->name }}</li>

        @break($user->number == 5)
    @endforeach

<a name="the-loop-variable"></a>
### 循环变量

当循环时，你可以在循环内访问 `$loop` 变量。这个变量可以提供一些有用的信息，比如当前循环的索引，当前循环是不是首次迭代，又或者当前循环是不是最后一次迭代：

    @foreach ($users as $user)
        @if ($loop->first)
            这是第一个迭代。
        @endif

        @if ($loop->last)
            这是最后一个迭代。
        @endif

        <p>This is user {{ $user->id }}</p>
    @endforeach

如果你是在一个嵌套的循环中，你可以通过使用 `$loop` 变量的 `parent` 属性来获取父循环中的 `$loop` 变量：

    @foreach ($users as $user)
        @foreach ($user->posts as $post)
            @if ($loop->parent->first)
                This is first iteration of the parent loop.
            @endif
        @endforeach
    @endforeach

`$loop` 变量也包含了其它各种有用的属性：

Property  | Description
------------- | -------------
`$loop->index`  |  当前循环所迭代的索引，起始为 0。
`$loop->iteration`  |  当前迭代数，起始为 1。
`$loop->remaining`  |  循环中迭代剩余的数量。
`$loop->count`  |  被迭代项的总数量。
`$loop->first`  |  当前迭代是否是循环中的首次迭代。
`$loop->last`  |  当前迭代是否是循环中的最后一次迭代。
`$loop->depth`  |  当前循环的嵌套深度。
`$loop->parent`  |  当在嵌套的循环内时，可以访问到父循环中的 $loop 变量。

<a name="comments"></a>
### 注释

Blade 也允许在页面中定义注释，然而，跟 HTML 的注释不同的是，Blade 注释不会被包含在应用程序返回的 HTML 内：

    {{-- 此注释将不会出现在渲染后的 HTML --}}

<a name="php"></a>
### PHP

在某些情况下，它对于你在视图文件中嵌入 php 代码是非常有帮助的。你可以在你的模版中使用 Blade 提供的 `@php` 指令来执行一段纯 PHP 代码：

    @php
        //
    @endphp

> {tip} 虽然 Blade 提供了这个功能，但频繁地使用也同时意味着你在你的模版中嵌入了太多的逻辑了。

<a name="including-sub-views"></a>
## 引入子视图

你可以使用 Blade 的 `@include` 命令来引入一个已存在的视图，所有在父视图的可用变量在被引入的视图中都是可用的。

    <div>
        @include('shared.errors')

        <form>
            <!-- Form Contents -->
        </form>
    </div>

尽管被引入的视图会继承父视图中的所有数据，你也可以通过传递额外的数组数据至被引入的页面：

    @include('view.name', ['some' => 'data'])

当然，如果你尝试使用 `@include` 去引用一个不存在的视图，Laravel 会抛出错误。如果你想引入一个视图，而你又无法确认这个视图存在与否，你可以使用 `@includeIf` 指令:

    @includeIf('view.name', ['some' => 'data'])

如果你想根据给定的布尔条件 `@include` 一个视图，你可以使用 `@includeWhen` 指令：

    @includeWhen($boolean, 'view.name', ['some' => 'data'])

> {note} 请避免在 Blade 视图中使用 `__DIR__` 及 `__FILE__` 常量，因为他们会引用视图被缓存的位置。

<a name="rendering-views-for-collections"></a>
### 为集合渲染视图

你可以使用 Blade 的 `@each` 命令将循环及引入结合成一行代码：

    @each('view.name', $jobs, 'job')

第一个参数为每个元素要渲染的子视图，第二个参数是你要迭代的数组或集合，而第三个参数为迭代时被分配至子视图中的变量名称。举个例子，如果你需要迭代一个 `jobs` 数组，通常子视图会使用 `job` 作为变量来访问 job 信息。子视图使用 `key` 变量作为当前迭代的键名。

你也可以传递第四个参数到 `@each` 命令。当需要迭代的数组为空时，将会使用这个参数提供的视图来渲染。

    @each('view.name', $jobs, 'job', 'view.empty')

> {note} 通过 `@each` 呈现的视图不会从父视图继承变量。 如果子视图需要这些变量，则应该使用 `@foreach` 和 `@include`。

<a name="stacks"></a>
## 堆栈

Blade 也允许你在其它视图或布局中为已经命名的堆栈中压入数据，这在子视图中引入必备的 JavaScript 类库时尤其有用：

    @push('scripts')
        <script src="/example.js"></script>
    @endpush

你可以根据需要多次压入堆栈，通过 `@stack` 命令中键入堆栈的名字来渲染整个堆栈：

    <head>
        <!-- Head Contents -->

        @stack('scripts')
    </head>

<a name="service-injection"></a>
## 服务注入

你可以使用 `@inject` 命令来从 Larvel [service container](/docs/{{version}}/container) 中取出服务。传递给 `@inject` 的第一个参数为置放该服务的变量名称，而第二个参数为你想要解析的服务的类或是接口的名称：

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
    </div>

<a name="extending-blade"></a>
## 拓展 Blade

Blade 甚至允许你使用 `directive` 方法来注册自己的命令。当 Blade 编译器遇到该命令时，它将会带参数调用提供的回调函数。

以下例子会创建一个把指定的 `$var` 格式化的 `@datetime($var)` 命令：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
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
            Blade::directive('datetime', function ($expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
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

如你所见，我们可以使用链式调用 `format` 方法的表述方式传递到指令。所以，在这个例子里，最终该指令生成了的 PHP 代码如下：

    <?php echo ($var)->format('m/d/Y H:i'); ?>

> {note} 在更新 Blade 指令的逻辑后，你将需要删除所有已缓存的 Blade 视图，使用 `view:clear` Artisan 命令来清除被缓存的视图。

<a name="custom-if-statements"></a>
### 自定义 If 语句

有时候设计自定义指令会比定义简单的自定义条件语句复杂很多，但是它又非常必要。因此，Blade提供了一个 `Blade::if` 方法，它允许你使用闭包快速定义自定义条件指令。 例如，让我们定义一个自定义条件来检查当前的应用程序环境。可以在 `AppServiceProvider` 的 `boot` 方法中这样做：

    use Illuminate\Support\Facades\Blade;

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

一旦定义了自定义条件，就可以很轻松地在模板中使用：

    @env('local')
        // The application is in the local environment...
    @else
        // The application is not in the local environment...
    @endenv

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [Wayne John](https://github.com/boxshadow) | <img class="avatar-66 rm-style" src="https://avatars1.githubusercontent.com/u/8577474?s=100"> | 翻译 | 基于 Laravel 的社交开源系统 [ThinkSNS+](https://github.com/slimkit/thinksns-plus) 欢迎 Star。  |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： http://d.laravel-china.org