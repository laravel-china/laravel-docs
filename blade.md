# 模板Blade

- [简介](#introduction)
- [模板继承](#template-inheritance)
    - [定义布局](#defining-a-layout)
    - [扩展布局](#extending-a-layout)
- [显示数据](#displaying-data)
- [控制器](#control-structures)
- [服务依赖](#service-injection)
- [Blade扩展](#extending-blade)

<a name="introduction"></a>
## 简介

Laravel提供了一个简单强大的模板引擎. 与其他的PHP模板引擎不一样的是，Blade在Views中允许使用PHP代码. Blade视图发生变化时，就会被解析成PHP代码，并且缓存起来，所以Blade不会对你的应用程序产生任何负担。 Blade视图文件以‘blade.php’结尾，所有文件都放在'resources/views'文件夹中。

<a name="template-inheritance"></a>
## 模板继承

<a name="defining-a-layout"></a>
### 定义布局

Blade两个主要的优点就是_模板继承_和_模块化_.在这之前，先看一个简单的例子。首先，我们先看一个“master”布局页面。由于大多数的web应用都使用相同的布局，所以我们只需要定义一种Blade视图。

    <!-- 文件位置 resources/views/layouts/master.blade.php -->

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

这个文件是非常经典的HTTML标记。需要注意的是，这里面有`@section`和`@yield`命令。`@section`和字面上的意思一样，它定义了内容的一部分，而`@yield`命令用来表示给出的section的内容。

现在我们已经定义了应用的布局，接下来我们对这个布局继承一个子页面。

<a name="extending-a-layout"></a>
### 布局扩展

当定义子页面时，我们就可以用Blade的`@extends`命令来说明这个子页面'inherit'（继承）的是哪个。`@extends`一个Blade布局的视图可以将内容注入到用`@section`命令的布局中的某一个部分。记住，在上面的这个例子中，我们是用`@yield`命令来展示布局中的这些部分内容。

    <!-- 代码路径 resources/views/layouts/child.blade.php -->

    @extends('layouts.master')

    @section('title', 'Page Title')

    @section('sidebar')
        @@parent

        <p>This is appended to the master sidebar.</p>
    @endsection

    @section('content')
        <p>This is my body content.</p>
    @endsection

在这个例子中，'sidebar'部分就是利用`@@parent`命令来将内容追加到布局中的sidebar中。这个视图被渲染时，`@@parent`命令就会被布局中的内容所替换。

当然，这些简单的PHP视图，路由会调用`view`帮助方法将这些Blade视图返回。

    Route::get('blade', function () {
        return view('child');
    });

<a name="displaying-data"></a>
## 展示数据

在Blade视图中展示数据时，需要将变量放在大括号中括起来。请看下面的路由的例子：

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

如果想展示`name`的内容，可以用下面的方法：

    Hello, {{ $name }}.

当然，你不会被局限于使用视图中的变量，你也可以用PHP方法输出结果。事实上，你在Blade中可以任意使用PHP代码来输出语句：

    The current UNIX timestamp is {{ time() }}.

> **Note:** Blade 中的`{{}}`语句会自动调用PHP的`htmlentities`方法来抵御XSS攻击。

#### Blade & JavaScript 框架

由于JavaScript框架在浏览器中大都使用大括号来表示表达式，你可以使用`@`符号来告诉Blade引擎这个表达式不需要解析。例如：

    <h1>Laravel</h1>

    Hello, @{{ name }}.

这个例子中，这个`@`符号在Blade中会被删掉；然而，`{{name}}`表达式在Blade引擎中会保持不变，这样JavaScript框架就会对它进行处理。

#### 三元运算

有的时候你想输入一个变量时，但是你可能不确定这个变量是否被定义了。在PHP代码中，我们就需要这么写，但是这样有点啰嗦：

    {{ isset($name) ? $name : 'Default' }}

高兴的是，Blade提供了一个简单快捷的三元运算表示方法。

    {{ $name or 'Default' }}

这个例子中，`$name`如果存在，就会被展示。如果不存在，就会输出一个默认的`Default`。

#### 展示非转义的数据

默认情况下，Blade 中的`{{ }}`语句会自动调用PHP中的`htmlentities`方法转义来抵御XSS攻击。如果你不想让数据被转义，可以用下面的语法：

    Hello, {!! $name !!}.

> **Note:** 在应用程序中输出用户的输入的数据要非常小心。我们经常会用双花括号表示我们需要转义成HTML实体。

<a name="control-structures"></a>
## 控制结构

除了模板集成和展示动态数据，Blade也提供了简单的PHP结构控制结构，比如条件语句和循环语句。这些短标签和PHP控制语句一起使用，非常简洁优雅，对PHP非常友好。

#### 条件语句

条件语句中可以使用`@if`,`@elseif`,`@endif`命令，这些命令跟PHP的使用方式一样：

    @if (count($records) === 1)
        I have one record!
    @elseif (count($records) > 1)
        I have multiple records!
    @else
        I don't have any records!
    @endif

为了更方便，Blade还提供了一个`@unless`命令：

    @unless (Auth::check())
        You are not signed in.
    @endunless

#### 循环

除了条件语句，Blade还提供了一个简单的命令来对应PHP中的循环结构。他们跟PHP的用法是一样的：

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

#### 包含子视图

Blade中的`@include`命令，让你在一个已有的视图里面包含一个Blade视图变得更加容易。被包含的视图可以使用所有的父级视图中的变量：

    <div>
        @include('shared.errors')

        <form>
            <!-- Form Contents -->
        </form>
    </div>

尽管包含的视图可以继承父级视图的所有数据变量，你也仍然可以在包含的视图中对数据进行扩展：

    @include('view.name', ['some' => 'data'])

#### 注释

Blade 也允许你在视图中注释。但是你的应用程序只能包含HTML注释：

    {{-- This comment will not be present in the rendered HTML --}}

<a name="service-injection"></a>
## 服务注入

`@inject` 可以将Laravel中的服务[服务容器](/docs/{{version}}/container)注入进来.`@inject`的第一个参数将会被定义成service的名称，第二个参数是用来被处理的class/interface:

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
    </div>

<a name="extending-blade"></a>
## Blade 扩展

Blade甚至允许你定义你的习惯用法。你可以使用`directive`方式来注册一个命令。当Blade编译器遇到这个命令时，他就会调用回调方法来处理这些参数。

下面的例子就是创建了一个`@datetime($var)`的命令来格式化给定的`$var`：

    <?php

    namespace App\Providers;

    use Blade;
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
                return "<?php echo with{$expression}->format('m/d/Y H:i'); ?>";
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

你可以看到，Laravel中的`with`帮助方法在下面的语句中会被使用。这个`with`方法用链式方式简单的返回一个object/value，最终的PHP语句是这样的：

    <?php echo with($var)->format('m/d/Y H:i'); ?>


