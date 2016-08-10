# 中级任务清单

- [简介](#introduction)
- [安装](#installation)
- [准备数据库](#prepping-the-database)
	- [数据库迁移](#database-migrations)
	- [Eloquent 模型](#eloquent-models)
	- [Eloquent 关联](#eloquent-relationships)
- [路由](#routing)
	- [显示视图](#displaying-a-view)
	- [认证](#authentication-routing)
	- [任务控制器](#the-task-controller)
- [构建布局与视图](#building-layouts-and-views)
	- [定义布局](#defining-the-layout)
	- [定义子视图](#defining-the-child-view)
- [增加任务](#adding-tasks)
	- [验证](#validation)
	- [创建任务](#creating-the-task)
- [显示已有的任务](#displaying-existing-tasks)
	- [依赖注入](#dependency-injection)
	- [显示任务](#displaying-the-tasks)
- [删除任务](#deleting-tasks)
	- [增加删除按钮](#adding-the-delete-button)
	- [路由模型绑定](#route-model-binding)
	- [授权](#authorization)
	- [删除该任务](#deleting-the-task)

<a name="introduction"></a>
## 简介

此快速入门指南为 Laravel 框架提供了高级的介绍，其中内容包括数据库迁移、Eloquent ORM、路由、认证、授权、依赖注入、验证、视图，及 Blade 模版。如果你已经有 Laravel 框架或 PHP 的基础，那么这会是个很好的开头。

要在 Laravel 功能中为样本做基本的选择，我们会构建一个简单的任务清单，可以使用它追踪所有想完成的任务（典型的「代办事项清单」例子）。与「基本」快速入门教程不同，此教程会允许用户在应用程序创建帐号并认证。此项目完整的源代码 [在 GitHub 上](https://github.com/summerblue/laravel-quickstart-intermediate-5.1)。

<a name="installation"></a>
## 安装

首先你需要安装一个全新的 Laravel 框架。你可以选择使用 [Homestead 虚拟机](/docs/{{version}}/homestead) 或是其它本机 PHP 环境来运行框架。只要你准备好了本机环境，就可以使用 Composer 安装 Laravel 框架：

	composer create-project laravel/laravel quickstart --prefer-dist

你可以随意阅读快速入门指南的剩余部分；不过，如果你想下载这个快速入门指南的源代码并在你的本机机器运行，那么你需要克隆它的 Git 代码仓库并安装依赖：

	git clone https://github.com/laravel/quickstart-intermediate quickstart
	cd quickstart
	composer install
	php artisan migrate

欲了解更多关于构建本机 Laravel 开发环境的文档，请查阅完整的 [Homestead](/docs/{{version}}/homestead) 及 [安装](/docs/{{version}}/installation) 文档。

<a name="prepping-the-database"></a>
## 准备数据库

<a name="database-migrations"></a>
### 数据库迁移

首先，让我们使用迁移来定义数据表以容纳我们所有的任务。Laravel 的数据库迁移提供了一个简单的方式，使用流畅、一目了然的 PHP 代码来定义数据表的结构与修改。你无需再告诉团队成员要手动增加字段至他们本机的数据库副本中，你的队友就可以简单运行你推送到版本控制的迁移。

#### `users` 数据表

因为我们要让用户可以在应用程序中创建他们的帐号，所以我们需要一张数据表来保存我们的用户。值得庆幸的是，Laravel 已经附带了创建 `users` 数据表的迁移，所以我们没必要再手动生成一个。默认的 `users` 数据表迁移位于 `database/migrations` 目录中。

#### `tasks` 数据表

下一步，让我们构建一张将容纳所有任务的数据表。[Artisan 命令行接口](/docs/{{version}}/artisan) 可以被用于生成各种类，为你构建 Laravel 项目时节省大量手动输入的时间。在此例中，让我们使用 `make:migration` 命令为 `tasks` 数据表生成新的数据库迁移：

	php artisan make:migration create_tasks_table --create=tasks

此迁移会被放置在你项目的 `database/migrations` 目录中。你可能已经注意到，`make:migration` 命令已经增加了自动递增的 ID 及时间戳至迁移文件。让我们编辑这个文件并为任务的名称增加额外的 `string` 字段，也增加连接 `tasks` 与 `users` 数据表的 `user_id` 字段：

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
	            $table->integer('user_id')->index();
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

我们可以使用 `migrate` Artisan 命令来运行迁移。如果你使用了 Homestead，则必须在虚拟机中运行这个命令，因为你的主机无法直接访问数据库：

	php artisan migrate

这个命令会创建我们所有的数据表。如果你使用数据库客户端检查数据表，那么你应该看到新的 `tasks` 与 `users` 数据表，其中包含了我们迁移中所定义的字段。接着，我们已经准备好定义我们的 Eloquent ORM 模型！

<a name="eloquent-models"></a>
### Eloquent 模型

[Eloquent](/docs/{{version}}/eloquent) 是 Laravel 默认的 ORM（对象关联映射）。Eloqunet 通过明确的定义「模型」，让你可以轻松的在数据库获取及保存数据。一般情况下，每个 Eloqunet 模型会直接对应一张数据表。

#### `User` 模型

首先，我们需要对应 `users` 数据表的模型。不过，如果你看过项目的 `app` 目录，你会发现 Laravel 已经附带了一个 `User` 模型，所以没必要去手动生成。

#### `Task` 模型

让我们定义一个对应至 `tasks` 数据表的 `Task` 模型。同样的，我们可以使用 Artisan 命令来生成此模型。在此例中，我们会使用 `make:model` 命令：

	php artisan make:model Task

这个模型会放置到你应用程序的 `app` 目录中。默认情况下此模型类将是空的。我们不必明确告知 Eloquent 模型要对应哪张数据表，因为它会假设数据表是模型名称的复数型态。所以，在此例中，`Task` 模型会假设对应至 `tasks` 数据表。

让我们增加一些东西到模型上。首先，我们需要声明模型的 `name` 属性应该能被「批量赋值」：

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Task extends Model
	{
	    /**
	     * 这些属性能被批量赋值。
	     *
	     * @var array
	     */
	    protected $fillable = ['name'];
	}

在为我们的应用程序增加路由时，我们会学习更多关于如何使用 Eloquent 模型的知识。当然，你也可以随意参考 [完整的 Eloquent 文档](/docs/{{version}}/eloquent) 来获取更多信息。

<a name="eloquent-relationships"></a>
### Eloquent 关联

现在我们的模型已经定义好了，这时候需要将他们链接在一起。例如，我们的 `User` 可以拥有多个 `Task` 实例，而一条 `Task` 则只能被赋给一名 `User`。定义好关联可以让我们很轻松的整理它们之间的关系，就像这样：

	$user = App\User::find(1);

	foreach ($user->tasks as $task) {
		echo $task->name;
	}

#### `tasks` 关联

首先，让我们在 `User` 模型定义 `tasks` 的关联。Eloquent 关联被定义为模型中的方法。Eloquent 支持多种不同类型的关联，所以请务必查阅 [完整的 Eloquent 文档](/docs/{{version}}/eloquent-relationships) 以获取更多信息。在本例中，我们会在 `User` 模型中定义一个 `tasks` 函数，并调用 Eloquent 提供的 `hasMany` 方法：

	<?php

	namespace App;

	// 导入的命名空间...

	class User extends Model implements AuthenticatableContract,
	                                    AuthorizableContract,
	                                    CanResetPasswordContract
	{
	    use Authenticatable, Authorizable, CanResetPassword;

	    // 其它的 Eloquent 属性...

	    /**
	     * 获取该用户的所有任务。
	     */
	    public function tasks()
	    {
	        return $this->hasMany(Task::class);
	    }
	}

#### `user` 关联

接着，让我们在 `Task` 模型定义 `user` 关联。同样的，我们会将此关联定义为模型中的方法。在本例中，我们会使用 Eloquent 提供的 `belongsTo` 方法来定义关联：

	<?php

	namespace App;

	use App\User;
	use Illuminate\Database\Eloquent\Model;

	class Task extends Model
	{
	    /**
	     * 这些属性能被批量赋值。
	     *
	     * @var array
	     */
	    protected $fillable = ['name'];

	    /**
	     * 获取拥有此任务的用户。
	     */
	    public function user()
	    {
	        return $this->belongsTo(User::class);
	    }
	}

太棒了！现在我们的关联已经定义好了，可以开始构建我们的控制器啦！

<a name="routing"></a>
## 路由

在我们任务清单应用程序的 [基本版本](/docs/{{version}}/quickstart) 中，我们在 `routes.php` 中将所有逻辑都定义为闭包。对于大多数的应用程序来说，一般都会使用 [控制器](/docs/{{version}}/controllers) 来组织路由。控制器让我们将 HTTP 请求处理逻辑分散至多个文件以便进行更好的组织。

<a name="displaying-a-view"></a>
### 显示视图

我们只会有一个使用闭包的`/` 路由，它是给应用程序访客的一个简单起始页面。所以，让我们先填写 `/` 路由。对于此路由，我们想渲染一个包含「欢迎」页面的 HTML 模板：

在 Laravel 里，所有的 HTML 模版都保存在 `resources/views` 目录，且我们可以在路由中使用 `view` 辅助函数来返回这些模版的其中一个：

	Route::get('/', function () {
		return view('welcome');
	});

当然，我们必须明确定义好这些视图。这个将在稍后完成！

<a name="authentication-routing"></a>
### 认证

我们需要让用户创建并登录帐号至我们的应用程序。一般来说，为网页应用程序构建完整的认证是相当乏味的工作。不过，因为它是一个通用的需求，所以 Laravel 在试着让这个过程变得更轻松便捷。

首先，你会注意到在应用程序中已经包含一个 `app/Http/Controllers/Auth/AuthController`。这个控制器使用了特别的 `AuthenticatesAndRegistersUsers` trait，它包含了所有创建及认证用户的必要逻辑。

#### 认证路由

所以，还有哪些部分是留给我们做的？我们依然需要创建注册及登录模板，并定义指向认证控制器的路由。首先，让我们在 `app/Http/routes.php` 文件中增加需要的路由：

	// 认证路由...
	Route::get('auth/login', 'Auth\AuthController@getLogin');
	Route::post('auth/login', 'Auth\AuthController@postLogin');
	Route::get('auth/logout', 'Auth\AuthController@getLogout');

	// 注册路由...
	Route::get('auth/register', 'Auth\AuthController@getRegister');
	Route::post('auth/register', 'Auth\AuthController@postRegister');

#### 认证视图

认证需要在 `resources/views/auth` 目录中创建 `login.blade.php` 与 `register.blade.php`。当然，这些视图的设计及样式并不重要；不过，它们至少得包含一些基本的字段。

`register.blade.php` 文件必需有一个包含 `name`、`email`、`password` 与 `password_confirmation` 字段的表单，并创建一个到 `/auth/register` 路由上的 `POST` 请求。

`login.blade.php` 文件必需有一个包含 `email` 与 `password` 字段的表单，并创建一个到 `/auth/login` 上的 `POST` 请求。

> **注意：**如果你想查看这些视图的完整例子，可以在 [GitHub 上获取](https://github.com/laravel/quickstart-intermediate) 完整的源代码。

<a name="the-task-controller"></a>
### 任务控制器

因为我们已经知道任务需要可被获取及保存，所以让我们使用 Artisan 命令行接口创建一个 `TaskController`，这个新的控制器会放置在 `app/Http/Controllers` 目录中：

	php artisan make:controller TaskController --plain

现在这个控制器已经被生成，让我们继续在 `app/Http/routes.php` 文件中构建一些对应至此控制器的路由：

	Route::get('/tasks', 'TaskController@index');
	Route::post('/task', 'TaskController@store');
	Route::delete('/task/{task}', 'TaskController@destroy');

#### 认证所有的任务路由

对于此应用程序，我们希望所有的任务路由都需要一个认证的用户。换句话说，用户为了创建任务必须「登录至」应用程序中。所以，我们需要对任务路由进行限制让其仅限已认证的用户访问。Laravel 使用 [中间件](/docs/{{version}}/middleware) 让这件事变得相当容易。

要让所有控制器中的行为要求已认证的用户，我们可以在控制器的构造器中增加 `middleware` 方法的调用。所以可用的路由中间件都被定义在 `app/Http/Kernel.php` 文件中。在本例中，我们希望为所有控制器的动作指派 `auth` 中间件：

	<?php

	namespace App\Http\Controllers;

	use App\Http\Requests;
	use Illuminate\Http\Request;
	use App\Http\Controllers\Controller;

	class TaskController extends Controller
	{
	    /**
	     * 创建一个新的控制器实例。
	     *
	     * @return void
	     */
	    public function __construct()
	    {
	        $this->middleware('auth');
	    }
	}

<a name="building-layouts-and-views"></a>
## 构建布局与视图

这个应用程序只会有一张视图，包含添加任务的表单，及目前所有任务的清单。为了帮助你想像此视图的画面，下方是完成后应用程序的截屏，采用了基本的 Bootstrap CSS 样式：

![应用程序图片](https://laravel.tw/assets/img/quickstart/basic-overview.png)

<a name="defining-the-layout"></a>
### 定义布局

几乎所有的网页应用程序都会在不同页面中共用相同的布局。举个例子，应用程序通常在每个页面（如果我们有一个以上页面）的顶部都拥有导航栏。Laravel 使用了 Blade **布局** 来让不同页面共用这些相同的功能。

如同我们前面讨论的那样，Laravel 所有的视图都被保存在 `resources/views`。所以，让我们来定义一个新的布局视图至 `resources/views/layouts/app.blade.php` 中。`.blade.php` 扩展名会告知框架使用 [Blade 模板引擎](/docs/{{version}}/blade) 渲染此视图。当然，你可以在 Laravel 使用纯 PHP 的模版。不过，Blade 提供了更方便的捷径来编写干净、简洁的模板。

我们的 `app.blade.php` 视图看起来应该如下面这样：

    // resources/views/layouts/app.blade.php

	<!DOCTYPE html>
	<html lang="en">
		<head>
			<title>Laravel 快速入门 - 高级</title>

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

很好，我们的应用程序布局已经完成。接下来，我们需要定义包含创建任务的表单及列出已有任务数据库表的视图。让我们将此视图定义在 `resources/views/tasks/index.blade.php`，它会对应至我们 `TaskController` 的 `index` 方法。

我们会跳过一些 Bootstrap CSS 模版，只专注在重要的事物上。你可以在 [GitHub](https://github.com/laravel/quickstart-intermediate) 下载到应用程序的完整源代码：

    // resources/views/tasks/index.blade.php

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
					<label for="task-name" class="col-sm-3 control-label">任务</label>

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

		<!-- 待办：目前任务 -->
	@endsection

#### 一些注意事项的说明

在继续开始之前，让我们先来谈谈有关模板的一些注意事项。首先 `@extends` 命令会告知 Blade，我们使用了定义于 `resources/views/layouts/app.blade.php` 的布局。所有在 `@section('content')` 及 `@endsection` 之间的内容都会被注入到 `app.blade.php` 布局中的 `@yield('content')` 位置里。

现在我们已经为我们的应用程序定义了基本的布局及视图。接着让我们在 `TaskController` 的 `index` 方法返回此视图：

    /**
     * 显示用户所有任务的清单。
     *
     * @param  Request  $request
     * @return Response
     */
	public function index(Request $request)
	{
		return view('tasks.index');
	}

接着，我们已经准备好增加代码至我们的 `POST /task` 路由的控制器方法内，以处理接收到的表单输入并增加新的任务至数据库中。

> **注意：**`@include('common.errors')` 命令会加载位于 `resources/views/common/errors.blade.php` 的模板。我们尚未定义此模板，但是我们将会在后面定义它！

<a name="adding-tasks"></a>
## 增加任务

<a name="validation"></a>
### 验证

现在我们视图中已经有一个表单，我们需要增加代码至我们的 `TaskController@store` 方法来验证接收到的表单输入并创建新的任务。首先，让我们先来验证表单输入。

对此表单来说，我们要让 `name` 字段为必填，且它必须少于 `255` 字符。如果验证失败，我们会将用户重定向回 `/` URL，并将旧的输入及错误消息闪存至 [session](/docs/{{version}}/session) 中：

    /**
     * 创建新的任务。
     *
     * @param  Request  $request
     * @return Response
     */
	public function store(Request $request)
	{
		$this->validate($request, [
			'name' => 'required|max:255',
		]);

		// 创建该任务...
	}

如果你看过 [基本快速入门指南](/docs/{{version}}/quickstart)，你会注意到验证的代码相比起来有些不同！这是因为在控制器内可以使用 Laravel 基底控制器所包含的 `ValidatesRequests` trait。这个 trait 提供了一个简单的 `validate` 方法，此方法可接收一个请求和包含验证规则的数组。

当验证失败时我们无需再手动重定向。如果指定的规则验证失败，用户会自动被重定向回原本的位置，并自动将错误消息闪存至 session 中。

#### `$errors` 变量

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


> **注意：**`errors` 变量可用于 **每个** Laravel 的视图中。如果没有错误验证消息存在，那么它就会是一个空的 `ViewErrorBag` 实例。

<a name="creating-the-task"></a>
### 创建任务

现在输入已经被验证处理完毕。让我们继续填写我们的路由来实际的创建一条新的任务。一旦新的任务被创建，我们便会将用户重定向回 `/tasks` URL。我们会充分的利用 Eloquent 的关联功能来创建该任务。

Laravel 大部分的关联提供了一个 `create` 方法，它接收一个包含属性的数组，并会在保存至数据库前自动设置关联模型的外键值。在此例中，`create` 方法会自动将指定任务的 `user_id` 属性设置为目前已验证用户的 ID，因为我们通过 `$request->user()` 访问。

    /**
     * 创建新的任务。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $this->validate($request, [
            'name' => 'required|max:255',
        ]);

        $request->user()->tasks()->create([
            'name' => $request->name,
        ]);

        return redirect('/tasks');
    }

太棒了！现在可以成功的创建任务了。接着，让我们继续构建已有的任务清单，并增加至我们的视图中。

<a name="displaying-existing-tasks"></a>
### 显示已有的任务

首先，我们需要编辑我们的 `TaskController@index` 方法，以传递所有已有的任务到此视图上。`view` 函数接收一个能在视图中取用的数据数组作为第二个参数，数组中的每个键都会在视图中作为变量。就像这样:

    /**
     * 显示用户的所有任务清单。
     *
     * @param  Request  $request
     * @return Response
     */
    public function index(Request $request)
    {
    	$tasks = Task::where('user_id', $request->user()->id)->get();

        return view('tasks.index', [
            'tasks' => $tasks,
        ]);
    }

不过，让我们先来探讨一些 Laravel 的依赖注入功能，来将 `TaskRepository` 注入至我们的 `TaskController`，我们将会通过它来访问所有的数据。

<a name="dependency-injection"></a>
### 依赖注入

Laravel 的 [服务容器](/docs/{{version}}/container) 是整个框架中最强大的功能之一。在读完本快速上手之后，请务必阅读容器文档的全部内容。

#### 创建资源库

如前面所提，我们希望定义一个 `TaskRepository` 存放所有 `Task` 模型的数据访问逻辑。当应用程序扩增时，在整个应用程序中共用同样的 Eloquent 查找将会是相当有用的。

所以，让我们先来创建一个 `app/Repositories` 目录，并增加 `TaskRepository` 类。切记，Laravel 的 `app` 中所有的文件夹会自动加载并使用 PSR-4 自动加载标准，所以你可以随意创建许多额外目录。

	<?php

	namespace App\Repositories;

	use App\User;
	use App\Task;

	class TaskRepository
	{
	    /**
	     * 获取指定用户的所有任务。
	     *
	     * @param  User  $user
	     * @return Collection
	     */
	    public function forUser(User $user)
	    {
	        return Task::where('user_id', $user->id)
	                    ->orderBy('created_at', 'asc')
	                    ->get();
	    }
	}

#### 注入资源库

一旦我们的资源库定义完成，我们就可以在 `TaskController` 控制器的构造器中对它使用「类型提示」，并在我们的 `index` 路由中使用它。因为 Laravel 使用容器来解析所有的控制器，所以我们的依赖会自动被注入至控制器的实例中：

	<?php

	namespace App\Http\Controllers;

	use App\Task;
	use App\Http\Requests;
	use Illuminate\Http\Request;
	use App\Http\Controllers\Controller;
	use App\Repositories\TaskRepository;

	class TaskController extends Controller
	{
	    /**
	     * 任务资源库的实例。
	     *
	     * @var TaskRepository
	     */
	    protected $tasks;

	    /**
	     * 创建新的控制器实例。
	     *
	     * @param  TaskRepository  $tasks
	     * @return void
	     */
	    public function __construct(TaskRepository $tasks)
	    {
	        $this->middleware('auth');

	        $this->tasks = $tasks;
	    }

	    /**
	     * 获取指定用户的所有任务。
	     *
	     * @param  Request  $request
	     * @return Response
	     */
	    public function index(Request $request)
	    {
	        return view('tasks.index', [
	            'tasks' => $this->tasks->forUser($request->user()),
	        ]);
	    }
	}

<a name="displaying-the-tasks"></a>
### 显示任务

一旦数据被传递，我们将在 `tasks/index.blade.php` 视图中将任务切分并将它们显示至数据库表中。`@foreach` 命令结构让我们可以编写简洁的循环语句，并编译成快速的纯 PHP 代码：

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

我们的任务应用大部分都完成了。但是，当我们完成已有的任务后，还没有任何方式可以删除它们。接着让我们来增加此功能！

<a name="deleting-tasks"></a>
## 删除任务

<a name="adding-the-delete-button"></a>
### 增加删除按钮

我们在我们的代码中应该放删除按钮的地方放了「待办」的注释。所以，让我们在 `tasks/index.blade.php` 视图中列出任务的每一行增加一个删除按钮。我们会为列表中的每个任务创建一个只有单个按钮的小表单。当按钮被按下时，一个 `DELETE /task` 的请求将会被发送到应用程序，它会触发我们的 `TaskController@destroy` 方法：

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

注意，删除按钮的表单 `method` 被列为 `POST`，即使我们响应的请求使用了 `Route::delete` 路由。HTML 表单只允许 `GET` 及 `POST` HTTP 动词，所以我们需要有个方式在表单假冒一个 `DELETE` 请求。

我们可以在表单中通过 `method_field('DELETE')` 函数输出的结果假冒一个 `DELETE` 请求。此函数会生成一个隐藏的表单输入，Laravel 会辨识并覆盖掉实际使用的 HTTP 请求方法。生成的字段看起来如下：

	<input type="hidden" name="_method" value="DELETE">

<a name="route-model-binding"></a>
### 路由模型绑定

现在，我们大致上已经准备好定义我们 `TaskController` 的 `destroy` 方法。但是首先，让我们重新检查我们为它声明的路由：

	Route::delete('/task/{task}', 'TaskController@destroy');

无需添加任何额外的代码，Laravel 便会将指定的任务 ID 注入至 `TaskController@destroy` 方法中，如下：

    /**
     * Destroy the given task.
     *
     * @param  Request  $request
     * @param  string  $taskId
     * @return Response
     */
	public function destroy(Request $request, $taskId)
	{
		//
	}

但是，我们要在这个方法中做的第一件事，就是通过指定的 ID 从数据库中获取 `Task` 实例。所以，如果 Laravel 可以先注入与 ID 符合的 `Task` 实例，那岂不是很棒？让我们来做到这一点！

在你的 `app/Providers/RouteServiceProvider.php` 文件的 `boot` 方法中，增加下方这行代码：

	$router->model('task', 'App\Task');

这一小行的代码会告知 Laravel，若在路由声明中看见 `{task}`，就会获取与指定 ID 对应的 `Task` 模型。现在我们可以定义我们的 destroy 方法，如下：

    /**
     * 卸除指定的任务。
     *
     * @param  Request  $request
     * @param  Task  $task
     * @return Response
     */
    public function destroy(Request $request, Task $task)
    {
        //
    }

<a name="authorization"></a>
### 认证

现在，我们有一个注入至 `destroy` 方法的 `Task` 实例；然而，我们不能保证通过认证的用户实际上「拥有」指定的任务。举个例子，一个恶意的请求可能通过传递一个随机任务 ID 至 `/tasks/{task}` URL，企图尝试删除其他用户的任务。所以，我们需要使用 Laravel 的授权功能，以确保只有已认证的用户才能注入路由的 `Task` 实例。

#### 创建一个授权策略

Laravel 使用了「授权策略」将授权逻辑组织至简单，小型的类。一般来说，每个授权策略会对应至一个模型。所以，让我们使用 Artisan 命令行接口创建一个 `TaskPolicy`，生成的文件会被放置于 `app/Policies/TaskPolicy.php`：

	php artisan make:policy TaskPolicy

接着，让我们给授权策略增加一个 `destroy` 方法。此方法会获取一个 `User` 实例及一个 `Task` 实例。此方法会简单的检查当用户的 ID 是否符合任务的 `user_id`。实际上，所有的授权方法都必须返回 `true` 或 `false`：

	<?php

	namespace App\Policies;

	use App\User;
	use App\Task;
	use Illuminate\Auth\Access\HandlesAuthorization;

	class TaskPolicy
	{
	    use HandlesAuthorization;

	    /**
	     * 判断指定用户是否可以删除指定的任务。
	     *
	     * @param  User  $user
	     * @param  Task  $task
	     * @return bool
	     */
	    public function destroy(User $user, Task $task)
	    {
	        return $user->id === $task->user_id;
	    }
	}

最后，我们需要将 `Task` 模型与 `TaskPolicy` 进行连接。可以通过在 `app/Providers/AuthServiceProvider.php` 文件增加一行 `$policies` 属性做到这件事。这会告知 Laravel，当我们尝试授权 `Task` 实例的行为时该用哪个授权策略：

    /**
     * 应用程序的授权策略对应。
     *
     * @var array
     */
    protected $policies = [
        Task::class => TaskPolicy::class,
    ];


#### 授权行为

现在我们的授权策略已经编写完毕，让我们在 `destroy` 方法中使用它。Laravel 所有的控制器都可以调用一个 `authorize` 方法，它由 `AuthorizesRequest` trait 所提供：

    /**
     * 卸除指定的任务。
     *
     * @param  Request  $request
     * @param  Task  $task
     * @return Response
     */
    public function destroy(Request $request, Task $task)
    {
        $this->authorize('destroy', $task);

        // 删除该任务...
    }

让我们花点时间看看此方法的调用。传递至 `authorize` 的第一个参数是我们希望调用的授权策略方法名称。第二个参数是我们目前有关的模型实例。切记，我们已经告诉 Laravel 我们的 `Task` 模型会对应至我们的 `TaskPolicy`，所以框架会知道该触发哪个授权策略的 `destroy` 方法。当前用户会被自动发送至授权方法中，所以我们不必在此手动传递它。

如果该行为被授权了，我们的代码就会继续正常运行。但是，如果该行为不被授权（意指授权策略的 `destroy` 方法返回 `false`），就会自动被抛出一个 403 异常并将错误页面显示给用户。

> **注意：**Laravel 提供的授权服务还有很多种其它的方式可供交互。请务必浏览完整的 [授权文档](/docs/{{version}}/authorization)。

<a name="deleting-the-task"></a>
### 删除该任务

最后，让我们完成增加逻辑至我们的 `destroy` 方法来实际删除指定的任务。我们可以使用 Eloquent 的 `delete` 方法从数据库中删除指定的模型实例。一旦记录被删除，我们将会把用户重定向回 `tasks` URL：

    /**
     * 删除指定的任务。
     *
     * @param  Request  $request
     * @param  Task  $task
     * @return Response
     */
    public function destroy(Request $request, Task $task)
    {
        $this->authorize('destroy', $task);

        $task->delete();

        return redirect('/tasks');
    }


