# Blade 模板

- [简介](#introduction)
- [模板继承](#template-inheritance)
    - [定义布局](#defining-a-layout)
    - [继承布局](#extending-a-layout)
- [展示数据](#displaying-data)
    - [Blade & JavaScript 框架](#blade-and-javascript-frameworks)
- [控制结构](#control-structures)
    - [If 语句](#if-statements)
    - [循环](#loops)
    - [循环变量](#the-loop-variable)
    - [注释](#comments)
- [融入子试图](#including-sub-views)
    - [为集合渲染试图](#rendering-views-for-collections)
- [堆栈](#stacks)
- [服务注入](#service-injection)
- [扩展 Blade](#extending-blade)

<a name="introduction"></a>
## 简介

Blade 是 Laravel 提供的一个既简单又强大的模板引擎。和其他流行的 PHP 模板引擎不一样，Blade 并不限制你在视图中使用原生 PHP 代码。 所有 Blade 视图文件都将被编译成原生 PHP 代码并缓存起来，除非它被修改，否则不会重新编译，这就意味着 Blade 基本上不会给你的应用增加任何额外负担。Blade 视图文件使用 `.blade.php` 扩展名，一般被存放在 `resources/views` 目录。

<a name="template-inheritance"></a>
## 模板继承

<a name="defining-a-layout"></a>
### 定义布局

Blade 的两个主要好处是 _模板继承_ 和 _区块_ 。 方便开始，我们来看一个简单的例子。首先，我们看一个命名为 "master" 的页面布局。因为多数 web 应用是在不同的页面中使用相同的总体布局，我们可以很方便的定义这个布局为单独的 Blade 视图：

    <!-- Stored in resources/views/layouts/app.blade.php -->

    <html>
        <head>
            <title>App Name - @yield('title')</title>
        </head>
        <body>
            @section('sidebar')
                This is the master sidebar.
            @show

            <div class="container">
                @yield('content')
            </div>
        </body>
    </html>

如你所见，该文件包含了典型的 HTML 标记。不过，请注意 `@section` 和 `@yield` 指令。 `@section` 指令正像其名字所暗示的一样是用来定义一个视图片断的，而  `@yield` 指令是用来显示所提供的片段区块的内容的。

现在为我们的应用定义好了一个布局，接下来定义一个继承此布局的子页面。

<a name="extending-a-layout"></a>
### 扩展布局

定义子页面时，使用 Blade 提供的 `@extends` 指令来为子页面指定其所“继承”的页面布局。 视图扩展了 Blade 布局后使用 `@section` 指令将内容注入到布局的区块中。 切记，在上面的例子里，布局中使用 `@yield` 的地方将会显示这些区块中的内容：

    <!-- Stored in resources/views/child.blade.php -->

    @extends('layouts.app')

    @section('title', 'Page Title')

    @section('sidebar')
        @@parent

        <p>This is appended to the master sidebar.</p>
    @endsection

    @section('content')
        <p>This is my body content.</p>
    @endsection

在上面的例子中，`sidebar` 区块利用 `@parent` 指令追加布局中的 sidebar 区块中的内容，如果不使用则会覆盖掉布局中的这部分内容。 `@parent` 指令会在视图被渲染时替换为布局中的内容。

Blade 视图可以通过在路由中使用全局辅助函数 `view` 方法来返回：

    Route::get('blade', function () {
        return view('child');
    });

<a name="displaying-data"></a>
## 展示数据

你可以使用中括号给变量视图中的变量复制。例如， 如下面的路由设置：

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

你可以在视图中这样来输出 `name` 变量的内容：

    Hello, {{ $name }}.

当然， 你并没有被限制只允许显示传递到视图中的变量的内容。你也可以从原生 PHP 方法中返回内容。事实上，你可以在 Blade 中 echo 任意的 PHP 代码：

    The current UNIX timestamp is {{ time() }}.

> {note} Blade `{{ }}` 声明中的内容是自动通过 PHP 的 `htmlentities` 方法过滤的，用以避免 XSS 的攻击。

#### 输出数据假如它存在

有时候你可能希望输出一个变量，但是你并不确定这个变量是否已经被定义，你可以使用 PHP 的这个表达式：

    {{ isset($name) ? $name : 'Default' }}

事实上，Blade 提供了更便捷的方式来代替这种三元运算：

    {{ $name or 'Default' }}

在这个例子中，如果变量 `$name` 存在， 它的值将被输出显示。 但是，如果它不存在，将会显示 `Default` 。

#### 显示未转义的数据

默认的，Blade `{{ }}` 声明会自动的使用 PHP 的 `htmlentities` f方法来转义数据避免 XSS 的攻击。如果你不想你的数据被转义，你可以使用下面的语法：

    Hello, {!! $name !!}.

> {note} B当你在应用中输出用户输入的数据时应该非常的谨慎，你应该总是使用 `{{  }}` 语法来转义内容中的任何的 HTML 实体。

<a name="blade-and-javascript-frameworks"></a>
### Blade & JavaScript 框架

由于很多 JavaScript 框架都使用花括号来表明所提供的表达式应该被显示在浏览器中，所以你可以使用 `@` s符号来通知 Blade 渲染引擎你需要这个表达式原样保留，例如：

    <h1>Laravel</h1>

    Hello, @{{ name }}.

在这个例子里，`@` 符号最终会被 Blade 引擎剔除，并且 `{{ name }}` 表达式会被原样的保留下来，这样就允许你的 JavaScript 框架来渲染它了。

#### `@verbatim` 指令

如果你需要在目标中的大片区域中展示 JavaScript 变量，你可以使用 `@verbatim` 指令来包裹 HTML 内容，这样你就不需要为每个需要解析的变量增加 `@` 符号前缀了：

    @verbatim
        <div class="container">
            Hello, {{ name }}.
        </div>
    @endverbatim

<a name="control-structures"></a>
## 控制结构

除了模板继承和数据展示，Blade 也为通用的 PHP 控制结构提供了便利的简写方式，比如条件声明和循环。这些简写提供了一个干净简洁的方式来处理 PHP 的控制结构，而且还保持与 PHP 语句的相似性。

<a name="if-statements"></a>
### If 语句

你可以通过 `@if`, `@elseif`, `@else` 和  `@endif` 指令来使用 `if` 控制结构，这些指令和 PHP 方法保持一致：

    @if (count($records) === 1)
        I have one record!
    @elseif (count($records) > 1)
        I have multiple records!
    @else
        I don't have any records!
    @endif

为了方便，Blade 也提供了一个 `@unless` 指令：

    @unless (Auth::check())
        You are not signed in.
    @endunless

<a name="loops"></a>
### 循环

除了条件声明，Blade 也提供了简单的指令来支持 PHP 的循环结构，这些指令方法和 PHP 语法相同：

    @for ($i = 0; $i < 10; $i++)
        The current value is {{ $i }}
    @endfor

    @foreach ($users as $user)
        <p>This is user {{ $user->id }}</p>
    @endforeach

    @forelse ($users as $user)
        <li>{{ $user->name }}</li>
    @empty
        <p>No users</p>
    @endforelse

    @while (true)
        <p>I'm looping forever.</p>
    @endwhile

> {tip} 当循环时，你可以使用 [循环变量](#the-loop-variable) 来获取循环中有价值的信息，比如循环中的首次或最后的迭代。

当使用循环时，你可能也需要一些结束循环或者跳出当前循环的指令：

    @foreach ($users as $user)
        @if ($user->type == 1)
            @continue
        @endif

        <li>{{ $user->name }}</li>

        @if ($user->number == 5)
            @break
        @endif
    @endforeach

你也可以使用指令声明包含条件的方式在一条语句中达到中断:

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
            This is the first iteration.
        @endif

        @if ($loop->last)
            This is the last iteration.
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

属性  | 描述
------------- | -------------
`$loop->index`  |  当前循环所迭代的索引，起始为 0。
`$loop->iteration`  |  当前迭代数，起始为 1。
`$loop->remaining`  |  循环中迭代剩余的数量。
`$loop->count`  |  被迭代项的总数量。
`$loop->first`  |  当前迭代是否循环中的首次迭代。
`$loop->last`  |  当前迭代是否循环中的最后一次迭代。
`$loop->depth`  |  当前循环的嵌套深度。
`$loop->parent`  |  当在嵌套的循环内时，可以访问到付循环中的 $loop 变量。

<a name="comments"></a>
### 注释

Blade 也允许你在视图中定义注释，但是它不会再渲染时生成 HTML 注释：

    {{-- This comment will not be present in the rendered HTML --}}

<a name="including-sub-views"></a>
## 包含子视图

你可以使用 Blade 的 `@include` 指令来包含一个已存在的视图的内容，当前视图中的变量也会共享给子视图：

    <div>
        @include('shared.errors')

        <form>
            <!-- Form Contents -->
        </form>
    </div>

尽管子视图会自动的继承父视图中的所有数据变量，但是你也可以传递一个数组变量来添扩充试图的数据：

    @include('view.name', ['some' => 'data'])

> {note} 你应该在 Blade 视图中避免使用 `__DIR__` 和 `__FILE__` 常量，因为它们会被解析为视图缓存所在的位置。

<a name="rendering-views-for-collections"></a>
### 为集合渲染视图

你可以使用 Blade 的 `@each` 指令在一行中合并循环引入多个视图：

    @each('view.name', $jobs, 'job')

第一个参数是数组或集合中每个元素需要被渲染的视图名称。第二个参数是一数组或集合，被用来提供迭代。而第三个参数是要分配给当前视图的变量名。举个例子，如果你需要遍历一个数组 `job` 。通常你会想要在局部渲染视图中使用 `job` 作为变量来访问 job 信息。在你的试图部分中 `key` 变量将是当前迭代的关键。

你也可以传递第四个参数到 `@each` 指令。如果所提供的数组是空数组的话，该参数所提供的视图将会被引入。

    @each('view.name', $jobs, 'job', 'view.empty')

<a name="stacks"></a>
## 堆栈

Blade 也允许你在其它视图或布局中已经命名的堆栈中压入数据，这在子视图中引入必备的 JavaScript 类库时尤其有用：

    @push('scripts')
        <script src="/example.js"></script>
    @endpush

你可以根据需要多次压入堆栈，通过键入堆栈的名字 `@stack` 指令来渲染整个堆栈：

    <head>
        <!-- Head Contents -->

        @stack('scripts')
    </head>

<a name="service-injection"></a>
## 服务注入

你可以使用 `@inject` 指令来从 Larvel [service container](/docs/{{version}}/container) 中取回服务。该指令的第一个参数将作为所取回服务存放的变量名，而第二个参数是你想要在服务容器中取回的类或接口名称：

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
    </div>

<a name="extending-blade"></a>
## 扩展 Blade

Blade 允许你使用 `directive` 方法来自定义自己的指令。当 Blade 编译器遇到该指令时，它会自动的调用该指令注册时所提供的回调函数并传递它相应的参数。

下面的例子中创建了一个 `@datetime($var)` 指令来格式化 `$var` 并实例化了 `DateTime` ：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::directive('datetime', function($expression) {
                return "<?php echo $expression->format('m/d/Y H:i'); ?>";
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

正如你所看到的，我们可以将使用链式调用 `format` 方法的表述传递到指令。所以，在这个例子里，最终该指令生成了的 PHP 代码如下：

    <?php echo $var->format('m/d/Y H:i'); ?>

> {note} 在更新 Blade 指令的逻辑后，你将需要删除所有已缓存的 Blade 视图，使用 `view:clear` Artisan 命令来视图缓存将被清除。
