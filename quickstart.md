# 基本任务清单

- [简介](#introduction)
- [安装](#installation)
- [准备数据库](#prepping-the-database)
	- [数据库迁移](#database-migrations)
	- [Eloquent 模型](#eloquent-models)
- [路由](#routing)
	- [构建路由](#stubbing-the-routes)
	- [显示视图](#displaying-a-view)
- [构建布局与视图](#building-layouts-and-views)
	- [定义布局](#defining-the-layout)
	- [定义子视图](#defining-the-child-view)
- [增加任务](#adding-tasks)
	- [验证](#validation)
	- [创建任务](#creating-the-task)
	- [显示已有的任务](#displaying-existing-tasks)
- [删除任务](#deleting-tasks)
	- [增加删除按钮](#adding-the-delete-button)
	- [删除该任务](#deleting-the-task)

<a name="introduction"></a>
## 简介

此快速入门指南主要为 Laravel 框架提供基本的介绍，其中内容包括数据库迁移、Eloquent ORM、路由、验证、视图，及 Blade 模版。如果你是第一次使用 Laravel 框架或 PHP，那么这会是个很好的开头。如果你已经在使用 Laravel 或者其它的框架，不仿参考我们高级的快速入门指南。

为了在 Laravel 功能中给样本做基本的选择，我们将会构建一个简单的任务清单，可以使用它追踪所有想完成的任务（典型的「代办事项清单」例子）。此项目完整的源代码[在 GitHub 上](http://github.com/laravel/quickstart-basic)。

<a name="installation"></a>
## 安装

首先你需要安装一个全新的 Laravel 框架。你可以选择使用 [Homestead 虚拟机](/docs/{{version}}/homestead)或是其它本机 PHP 环境来运行框架。只要你准备好了本机环境，就可以使用 Composer 安装 Laravel 框架：

	composer create-project laravel/laravel quickstart --prefer-dist

你可以随意阅读快速入门指南的剩余部分；不过，如果你想下载这个快速入门指南的源代码并在你的本机机器运行，那么你需要克隆它的 Git 代码仓库并安装依赖：

	git clone https://github.com/laravel/quickstart-basic quickstart
	cd quickstart
	composer install
	php artisan migrate

欲了解更多关于构建本机 Laravel 开发环境的文档，请查阅完整的 [Homestead](/docs/{{version}}/homestead) 及[安装](/docs/{{version}}/installation)文档。

<a name="prepping-the-database"></a>
## 准备数据库

<a name="database-migrations"></a>
### 数据库迁移

首先，让我们使用迁移来定义数据表以容纳我们所有的任务。Laravel 的数据库迁移提供了一个简单的方式，使用流畅、一目了然的 PHP 代码来定义数据表的结构与修改。你无需再告诉团队成员要手动增加字段至他们本机的数据库副本中，你的队友就可以简单运行你推送到版本控制的迁移。

让我们来构建一张将容纳所有任务的数据表。[Artisan 命令行接口](/docs/{{version}}/artisan)可以被用于生成各种类，为你构建 Laravel 项目时节省大量手动输入的时间。在此例中，让我们使用 `make:migration` 命令为 `tasks` 数据表生成新的数据库迁移：

	php artisan make:migration create_tasks_table --create=tasks

此迁移会被放置于你项目的 `database/migrations` 目录中。你可能已经注意到，`make:migration` 命令已经增加了自动递增的 ID 及时间戳至迁移文件。让我们编辑这个文件并为任务的名称增加额外的 `string` 字段：

	<?php

	use Illuminate\Database\Schema\Blueprint;
	use Illuminate\Database\Migrations\Migration;

	class CreateTasksTable extends Migration
	{
	    /**
	     * 运行迁移。
	     *
	     * @return void
	     */
	    public function up()
	    {
	        Schema::create('tasks', function (Blueprint $table) {
	            $table->increments('id');
	            $table->string('name');
	            $table->timestamps();
	        });
	    }

	    /**
	     * 还原迁移。
	     *
	     * @return void
	     */
	    public function down()
	    {
	        Schema::drop('tasks');
	    }
	}

我们可以使用 `migrate` Artisan 命令运行迁移。如果你使用了 Homestead，则必须在虚拟机中运行这个命令，因为你的主机无法直接访问数据库：

	php artisan migrate

这个命令会创建我们所有的数据表。如果你使用了数据库客户端来检查数据表，那么你应该会看到新的 `tasks` 数据表，其中包含了我们迁移中所定义的字段。接着，我们已经准备好为我们的任务定义一个 Eloquent ORM 模型了！

<a name="eloquent-models"></a>
### Eloquent 模型

[Eloquent](/docs/{{version}}/eloquent) 是 Laravel 默认的 ORM（对象关联映射）。Eloqunet 通过明确的定义「模型」，让你可以很轻松的在数据库获取及保存数据。一般情况下，每个 Eloqunet 模型会直接对应一张数据表。

所以，让我们定义一个对应至 `tasks` 数据表的 `Task` 模型。同样的，我们可以使用 Artisan 命令来生成此模型。在此例中，我们会使用 `make:model` 命令：

	php artisan make:model Task

这个模型会放置在你应用程序的 `app` 目录中。默认情况下此模型类将是空的。我们不必明确告知 Eloquent 模型要对应哪张数据表，因为它会假设数据表是模型名称的复数型态。所以，在此例中，`Task` 模型会假设对应至 `tasks` 数据表。所以我们的空模型看起来应该如下：

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Task extends Model
	{
		//
	}

在为我们的应用程序增加路由时，我们会学习更多关于如何使用 Eloquent 模型的知识。当然，你也可以随意参考[完整的 Eloquent 文档](/docs/{{version}}/eloquent)来获取更多信息。

<a name="routing"></a>
## 路由

<a name="stubbing-the-routes"></a>
### 构建路由

接着，我们已经在应用程序中准备好增加一些路由。这些路由会将 URLs 指向控制器或是匿名函数上，当用户进入特定页面时即会被运行。默认情况下，Laravel 所有的路由都会被定义在 `app/Http/routes.php` 文件中，每个新的 Laravel 项目都会包含此文件。

对于本应用程序，我们知道最后会用到三个路由：一个路由用于显示我们所有任务的清单、一个路由用于添加任务、一个路由用于删除已有的任务。所以，让我们在 `app/Http/routes.php` 中构建这所有路由：

	<?php

	use App\Task;
	use Illuminate\Http\Request;

	/**
	 * 显示所有任务
	 */
	Route::get('/', function () {
		//
	});

	/**
	 * 增加新的任务
	 */
	Route::post('/task', function (Request $request) {
		//
	});

	/**
	 * 删除一个已有的任务
	 */
	Route::delete('/task/{id}', function ($id) {
		//
	});

<a name="displaying-a-view"></a>
### 显示视图

接着，让我们填写我们的 `/` 路由。在此路由中，我们想要渲染一个包含添加任务的表单，及目前所有任务清单的 HTML 模版。

在 Laravel 里，所有的 HTML 模版都保存在 `resources/views` 目录，且我们可以在路由中使用 `view` 辅助方法来返回这些模版的其中一个：

	Route::get('/', function () {
		return view('tasks');
	});

当然，我们必须明确定义这些视图，所以现在开始动手做吧！

<a name="building-layouts-and-views"></a>
## 构建布局与视图

这个应用程序只会有一张视图，包含添加任务的表单，及目前所有任务的清单。为了帮助你想像此视图的画面，下方是完成后应用程序的截屏，采用了基本的 Bootstrap CSS 样式：

![应用程序图片](https://laravel.tw/assets/img/quickstart/basic-overview.png)

<a name="defining-the-layout"></a>
### 定义布局

几乎所有的网页应用程序都会在不同页面中共用相同的布局。举个例子，应用程序通常在每个页面（如果我们有一个以上页面）的顶部都拥有导航栏。Laravel 使用了 Blade **布局**来让不同页面共用这些相同的功能。

如同我们前面讨论的那样，Laravel 所有的视图都被保存在 `resources/views`。所以，让我们来定义一个新的布局视图至 `resources/views/layouts/app.blade.php` 中。`.blade.php` 扩展名会告知框架使用 [Blade 模板引擎](/docs/{{version}}/blade)渲染此视图。当然，你可以在 Laravel 使用纯 PHP 的模版。不过，Blade 提供了更方便的捷径来编写干净、简洁的模板。

我们的 `app.blade.php` 视图看起来应该如下面这样：

    // resources/views/layouts/app.blade.php

	<!DOCTYPE html>
	<html lang="en">
		<head>
			<title>Laravel 快速入门 - 基本</title>

			<!-- CSS 及 JavaScript -->
		</head>

		<body>
			<div class="container">
				<nav class="navbar navbar-default">
					<!-- Navbar 内容 -->
				</nav>
			</div>

			@yield('content')
		</body>
	</html>

注意布局中的 `@yield('content')` 部分。这是特殊的 Blade 命令，让子页面可以在此处注入自己的内容以扩展布局。接着，让我们定义将会使用此布局并提供主要内容的子视图。

<a name="defining-the-child-view"></a>
### 定义子视图

很好，我们的应用程序布局已经完成。接下来，我们需要定义包含创建任务的表单及列出已有任务数据库表的视图。让我们将此视图定义在 `resources/views/tasks.blade.php`。

我们会跳过一些 Bootstrap CSS 模版，只专注在重要的事物上。你可以在 [GitHub](https://github.com/laravel/quickstart-basic) 下载到应用程序的完整源代码：

    // resources/views/tasks.blade.php

	@extends('layouts.app')

	@section('content')

        <!-- Bootstrap 模版... -->

		<div class="panel-body">
            <!-- 显示验证错误 -->
			@include('common.errors')

			<!-- 新任务的表单 -->
			<form action="/task" method="POST" class="form-horizontal">
				{{ csrf_field() }}

                <!-- 任务名称 -->
				<div class="form-group">
					<label for="task" class="col-sm-3 control-label">Task</label>

					<div class="col-sm-6">
						<input type="text" name="name" id="task-name" class="form-control">
					</div>
				</div>

                <!-- 增加任务按钮-->
				<div class="form-group">
					<div class="col-sm-offset-3 col-sm-6">
						<button type="submit" class="btn btn-default">
							<i class="fa fa-plus"></i> 增加任务
						</button>
					</div>
				</div>
			</form>
		</div>

		<!-- 代办：目前任务 -->
	@endsection

#### 一些注意事项的说明

在继续开始之前，让我们先来谈谈有关模板的一些注意事项。首先 `@extends` 命令会告知 Blade，我们使用了定义于 `resources/views/layouts/app.blade.php` 的布局。所有在 `@section('content')` 及 `@endsection` 之间的内容都会被注入到 `app.blade.php` 布局中的 `@yield('content')` 位置里。

现在我们已经为我们的应用程序定义了基本的布局及视图。请记住我们在 `/` 路由中返回了此视图，就像这样：

	Route::get('/', function () {
		return view('tasks');
	});

接着，我们已经准备好增加代码至我们的 `POST /task` 路由，以处理接收到的表单输入并增加新的任务至数据库中。

> **注意：**`@include('common.errors')` 命令会加载位于 `resources/views/common/errors.blade.php` 的模板。我们尚未定义此模板，但是我们将会在后面定义它！

<a name="adding-tasks"></a>
## 增加任务

<a name="validation"></a>
### 验证

现在我们的视图中已经有一个表单，我们需要增加代码至我们的 `POST /task` 路由来验证接收到的表单输入并创建新的任务。首先，让我们先来验证表单输入。

对此表单来说，我们要让 `name` 字段为必填，且它必须少于 `255` 字符。如果验证失败，我们会将用户重定向回 `/` URL，并将旧的输入及错误消息闪存到 [session](/docs/{{version}}/session) 中：

	Route::post('/task', function (Request $request) {
		$validator = Validator::make($request->all(), [
			'name' => 'required|max:255',
		]);

		if ($validator->fails()) {
			return redirect('/')
				->withInput()
				->withErrors($validator);
		}

		// 创建该任务...
	});

#### `$errors` 变量

让我们休息一下说说例子中 `->withErrors($validator)` 的部分。`->withErrors($validator)` 的调用会通过指定的验证器实例将错误消息闪存至 session 中，所以我们可以在视图中通过 `$errors` 变量访问它们。

我们在视图中使用了 `@include('common.errors')` 命令来渲染表单的错误验证消息。`common.errors` 让我们可以简单的在所有的页面都显示相同格式的错误验证消息。现在让我们定义此视图的内容：

    // resources/views/common/errors.blade.php

    @if (count($errors) > 0)
        <!-- 表单错误清单 -->
        <div class="alert alert-danger">
            <strong>哎呀！出了些问题！</strong>

            <br><br>

            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif


> **注意：**`errors` 变量可用于**每个** Laravel 的视图中。如果没有错误验证消息存在，那么它就会是一个空的 `ViewErrorBag` 实例。

<a name="creating-the-task"></a>
### 创建任务

现在输入已经被验证处理完毕。让我们继续填写我们的路由来创建一条新的任务。一旦新的任务被创建，我们将会把用户重定向回 `/` URL。要创建该任务，我们可以在为新的 Eloquent 模型创建及设置属性后使用 `save` 方法：

	Route::post('/task', function (Request $request) {
		$validator = Validator::make($request->all(), [
			'name' => 'required|max:255',
		]);

		if ($validator->fails()) {
			return redirect('/')
				->withInput()
				->withErrors($validator);
		}

		$task = new Task;
		$task->name = $request->name;
		$task->save();

		return redirect('/');
	});

好极了！我们现在已经可以成功的创建任务了。接着，让我们继续构建已有的任务清单，并增加至我们的视图中。

<a name="displaying-existing-tasks"></a>
### 显示已有的任务

首先，我们需要编辑我们的 `/` 路由，以传递所有已有的任务至视图。`view` 函数接收一个能在视图中被取用的数据数组作为第二个参数，数组中的每个键都会在视图中作为变量：

	Route::get('/', function () {
		$tasks = Task::orderBy('created_at', 'asc')->get();

		return view('tasks', [
			'tasks' => $tasks
		]);
	});

一旦数据被传递，我们便可以在 `tasks.blade.php` 视图中将任务切分并将它们显示至数据库表中。`@foreach` 命令结构让我们可以编写简洁的循环的语句，并编译成快速的纯 PHP 代码：

	@extends('layouts.app')

	@section('content')
        <!-- 创建任务表单... -->

        <!-- 目前任务 -->
        @if (count($tasks) > 0)
            <div class="panel panel-default">
                <div class="panel-heading">
                    目前任务
                </div>

                <div class="panel-body">
                    <table class="table table-striped task-table">

                        <!-- 表头 -->
                        <thead>
                            <th>Task</th>
                            <th>&nbsp;</th>
                        </thead>

                        <!-- 表身 -->
                        <tbody>
                            @foreach ($tasks as $task)
                                <tr>
                                    <!-- 任务名称 -->
                                    <td class="table-text">
                                        <div>{{ $task->name }}</div>
                                    </td>

                                    <td>
                                        <!-- 待办：删除按钮 -->
                                    </td>
                                </tr>
                            @endforeach
                        </tbody>
                    </table>
                </div>
            </div>
        @endif
	@endsection

我们任务应用程序大部分都完成了。但是，当我们完成已有的任务后，还没有任何方式可以删除它们。接着让我们增加此功能！

<a name="deleting-tasks"></a>
## 删除任务

<a name="adding-the-delete-button"></a>
### 增加删除按钮

我们在我们的代码中应该放删除按钮的地方放了「待办」的注释。所以，让我们在 `tasks.blade.php` 视图中列出任务的每一行并增加一个删除按钮。我们会为列表中的每个任务创建一个只有单个按钮的小表单。当按钮被按下时，一个 `DELETE /task` 的请求将会被发送到应用程序：

    <tr>
        <!-- 任务名称 -->
        <td class="table-text">
            <div>{{ $task->name }}</div>
        </td>

        <!-- 删除按钮 -->
        <td>
            <form action="/task/{{ $task->id }}" method="POST">
                {{ csrf_field() }}
                {{ method_field('DELETE') }}

                <button>删除任务</button>
            </form>
        </td>
    </tr>

<a name="a-note-on-method-spoofing"></a>
#### 方法欺骗的注记

注意，删除按钮的表单 `method` 被列为 `POST`，即使我们响应的请求使用了 `Route::delete` 路由。HTML 表单只允许 `GET` 及 `POST` HTTP 动词，所以我们需要有个方式在表单中假冒一个 `DELETE` 请求。

我们可以在表单中通过 `method_field('DELETE')` 函数输出的结果假冒一个 `DELETE` 请求。此函数会生成一个隐藏的表单输入，Laravel 会辨识并覆盖掉实际使用的 HTTP 请求方法。生成的字段看起来如下：

	<input type="hidden" name="_method" value="DELETE">

<a name="deleting-the-task"></a>
### 删除该任务

最后，让我们增加实际的删除逻辑。我们可以使用 Eloquent 的 `findOrFail` 方法通过 ID 获取模型，当该模型不存在时则会抛出 404 异常。一旦我们成功获取到模型，我们就可以使用 `delete` 方法来删除该条记录。只要该记录被删除，我们便会把用户重定向回 `/` URL：

	Route::delete('/task/{id}', function ($id) {
		Task::findOrFail($id)->delete();

		return redirect('/');
	});
