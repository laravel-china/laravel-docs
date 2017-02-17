# Laravel 的 HTTP 控制器

- [Introduction](#introduction)
- [Basic Controllers](#basic-controllers)
    - [Defining Controllers](#defining-controllers)
    - [Controllers & Namespaces](#controllers-and-namespaces)
    - [Single Action Controllers](#single-action-controllers)
- [Controller Middleware](#controller-middleware)
- [Resource Controllers](#resource-controllers)
    - [Partial Resource Routes](#restful-partial-resource-routes)
    - [Naming Resource Routes](#restful-naming-resource-routes)
    - [Naming Resource Route Parameters](#restful-naming-resource-route-parameters)
    - [Localizing Resource URIs](#restful-localizing-resource-uris)
    - [Supplementing Resource Controllers](#restful-supplementing-resource-controllers)
- [Dependency Injection & Controllers](#dependency-injection-and-controllers)
- [Route Caching](#route-caching)

- [简介](#introduction)
- [基础控制器](#basic-controllers)
    - [定义控制器](#defining-controllers)
    - [控制器与命名空间](#controllers-and-namespaces)
    - [单一操作控制器](#single-action-controllers)
- [控制器中间件](#controller-middleware)
- [资源控制器](#resource-controllers)
    - [部分资源路由](#restful-partial-resource-routes)
    - [命名资源路由](#restful-naming-resource-routes)
    - [命名资源路由参数](#restful-naming-resource-route-parameters)
    - [本地化资源 URI](#restful-localizing-resource-uris)
    - [附加资源控制器](#restful-supplementing-resource-controllers)
- [依赖注入与控制器](#dependency-injection-and-controllers)
- [路由缓存](#route-caching)

<a name="introduction"></a>
## Introduction

## 简介

Instead of defining all of your request handling logic as Closures in route files, you may wish to organize this behavior using Controller classes. Controllers can group related request handling logic into a single class. Controllers are stored in the `app/Http/Controllers` directory.

除了在路由文件中以闭包的形式定义所有的请求处理逻辑外，你可能还想使用控制器类来组织此类操作。控制器能够将相关的请求处理逻辑组成一个单独的类。控制器被存放在 `app/Http/Controllers` 目录下。

<a name="basic-controllers"></a>
## Basic Controllers

## 基础控制器

<a name="defining-controllers"></a>
### Defining Controllers

### 定义控制器

Below is an example of a basic controller class. Note that the controller extends the base controller class included with Laravel. The base class provides a few convenience methods such as the `middleware` method, which may be used to attach middleware to controller actions:

以下是一个基础控制器类的例子。需要注意的是，该控制器继承了 Laravel 内置的基础控制器类。该基础类提供了一些便捷的方法，比如 `middleware` 方法，该方法可以用来给控制器操作添加中间件：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

You can define a route to this controller action like so:

你可以这样定义一个指向该控制器操作的路由：

    Route::get('user/{id}', 'UserController@show');

Now, when a request matches the specified route URI, the `show` method on the `UserController` class will be executed. Of course, the route parameters will also be passed to the method.

现在，当请求和此特定路由的 URI 匹配时，`UserController` 类的 `show` 方法就会被执行。当然，路由参数也会被传递至该方法。

> {tip} Controllers are not **required** to extend a base class. However, you will not have access to convenience features such as the `middleware`, `validate`, and `dispatch` methods.

> {tip} 控制器并不是**一定**要继承基础类。只是，你将无法使用一些便捷的功能，比如 `middleware`，`validate` 和 `dispatch` 方法。

<a name="controllers-and-namespaces"></a>
### Controllers & Namespaces

### 控制器与命名空间

It is very important to note that we did not need to specify the full controller namespace when defining the controller route. Since the `RouteServiceProvider` loads your route files within a route group that contains the namespace, we only specified the portion of the class name that comes after the `App\Http\Controllers` portion of the namespace.

这一点很重要，我们在定义控制器路由时，不必指定完整的控制器命名空间。`RouteServiceProvider` 会在一个包含命名空间的路由组中加载路由文件，因此我们只需要指定类名中 `App\Http\Controllers` 命名空间之后的部分就可以了。

If you choose to nest your controllers deeper into the `App\Http\Controllers` directory, simply use the specific class name relative to the `App\Http\Controllers` root namespace. So, if your full controller class is `App\Http\Controllers\Photos\AdminController`, you should register routes to the controller like so:

如果你选择将控制器存放在 `App\Http\Controllers` 目录下，只需简单地使用相对于根命名空间 `App\Http\Controllers` 的特定类名。因此，如果你的完整控制器类是 `App\Http\Controllers\Photos\AdminController`，你应该用这种方式注册指向该控制器的路由：

    Route::get('foo', 'Photos\AdminController@method');

<a name="single-action-controllers"></a>
### Single Action Controllers

### 单一操作控制器

If you would like to define a controller that only handles a single action, you may place a single `__invoke` method on the controller:

如果想定义一个只处理单个操作的控制器，你可以在控制器中只设置一个 `__invoke` 方法：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class ShowProfile extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function __invoke($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

When registering routes for single action controllers, you do not need to specify a method:

为单一操作控制器注册路由时，无需指定方法：

    Route::get('user/{id}', 'ShowProfile');

<a name="controller-middleware"></a>
## Controller Middleware

## 控制器中间件

[Middleware](/docs/{{version}}/middleware) may be assigned to the controller's routes in your route files:

[中间件](/docs/{{version}}/middleware)可以在路由文件中指定给控制器路由：

    Route::get('profile', 'UserController@show')->middleware('auth');

However, it is more convenient to specify middleware within your controller's constructor. Using the `middleware` method from your controller's constructor, you may easily assign middleware to the controller's action. You may even restrict the middleware to only certain methods on the controller class:

然而，在控制器的构造方法中指定中间件会更为便捷。在控制器构造方法中使用 `middleware` 方法，你可以很容易地将中间件指定给控制器操作。你甚至可以约束中间件只对控制器类中的某个特定方法生效：

    class UserController extends Controller
    {
        /**
         * Instantiate a new controller instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');

            $this->middleware('log')->only('index');

            $this->middleware('subscribed')->except('store');
        }
    }

Controllers also allow you to register middleware using a Closure. This provides a convenient way to define a middleware for a single controller without defining an entire middleware class:

控制器也允许你使用闭包的方式注册中间件。这提供了一种很便捷地为单个控制器定义中间件的方式，而不用定义一个完整的中间件类：

    $this->middleware(function ($request, $next) {
        // ...

        return $next($request);
    });

> {tip} You may assign middleware to a subset of controller actions; however, it may indicate your controller is growing too large. Instead, consider breaking your controller into multiple, smaller controllers.

> {tip} 你可能将中间件指定到控制器的部分操作上，然而，这会使你的控制器过于臃肿。换个角度，考虑将控制器分成多个更小的控制器。

<a name="resource-controllers"></a>
## Resource Controllers

## 资源控制器

Laravel resource routing assigns the typical "CRUD" routes to a controller with a single line of code. For example, you may wish to create a controller that handles all HTTP requests for "photos" stored by your application. Using the `make:controller` Artisan command, we can quickly create such a controller:

Laravel 资源路由可以将典型的“CURD”路由指定到一个控制器上，仅需一行代码就可以实现。比如，你可能希望创建一个控制器来处理所有应用保存的「相片」的 HTTP 请求。使用 `make:controller` Artisan 命令，就能快速创建这样一个控制器：

    php artisan make:controller PhotoController --resource

This command will generate a controller at `app/Http/Controllers/PhotoController.php`. The controller will contain a method for each of the available resource operations.

这个命令会在 `app/Http/Controllers/PhotoController.php` 中生成一个控制器，该控制器包含了各种可用的资源操作方法。

Next, you may register a resourceful route to the controller:

接下来，你可以给控制器注册一个资源路由：

    Route::resource('photos', 'PhotoController');

This single route declaration creates multiple routes to handle a variety of actions on the resource. The generated controller will already have methods stubbed for each of these actions, including notes informing you of the HTTP verbs and URIs they handle.

这个路由声明会创建多个路由来处理各种各样的资源操作。前面生成的控制器已经包含了这些操作的方法，还包括了 HTTP 动作和操作 URI 的注释。

#### Actions Handled By Resource Controller

#### 资源控制器操作处理

Verb      | URI                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | `/photos`              | index        | photos.index
GET       | `/photos/create`       | create       | photos.create
POST      | `/photos`              | store        | photos.store
GET       | `/photos/{photo}`      | show         | photos.show
GET       | `/photos/{photo}/edit` | edit         | photos.edit
PUT/PATCH | `/photos/{photo}`      | update       | photos.update
DELETE    | `/photos/{photo}`      | destroy      | photos.destroy

动作      | URI                  | 操作       | 路由名称
----------|-----------------------|--------------|---------------------
GET       | `/photos`              | index        | photos.index
GET       | `/photos/create`       | create       | photos.create
POST      | `/photos`              | store        | photos.store
GET       | `/photos/{photo}`      | show         | photos.show
GET       | `/photos/{photo}/edit` | edit         | photos.edit
PUT/PATCH | `/photos/{photo}`      | update       | photos.update
DELETE    | `/photos/{photo}`      | destroy      | photos.destroy

#### Specifying The Resource Model

#### 指定资源模型

If you are using route model binding and would like the resource controller's methods to type-hint a model instance, you may use the `--model` option when generating the controller:

如果你使用了路由模型绑定，并且想在资源控制器的方法中对某个模型实例做类型约束，你可以在生成控制器的时候使用 `--model` 选项：

    php artisan make:controller PhotoController --resource --model=Photo

#### Spoofing Form Methods

#### 伪造表单方法

Since HTML forms can't make `PUT`, `PATCH`, or `DELETE` requests, you will need to add a hidden `_method` field to spoof these HTTP verbs. The `method_field` helper can create this field for you:

因为 HTML 表单不能发送 `PUT`，`PATCH` 或者 `DELETE` 请求，你需要添加一个 `_method` 隐藏域字段来伪造 HTTP 动作。`method_field` 辅助函数可以为你创建这个字段：

    {{ method_field('PUT') }}

<a name="restful-partial-resource-routes"></a>
### Partial Resource Routes

### 部分资源路由

When declaring a resource route, you may specify a subset of actions the controller should handle instead of the full set of default actions:

声明资源路由的时候，你可以指定控制器处理部分操作，而不必使用全部默认的操作：

    Route::resource('photo', 'PhotoController', ['only' => [
        'index', 'show'
    ]]);

    Route::resource('photo', 'PhotoController', ['except' => [
        'create', 'store', 'update', 'destroy'
    ]]);

<a name="restful-naming-resource-routes"></a>
### Naming Resource Routes

### 命名资源路由

By default, all resource controller actions have a route name; however, you can override these names by passing a `names` array with your options:

默认地，所有的资源路由操作都有一个路由名称；不过你可以在参数选项中传入一个 `names` 数组来重写这些名称：

    Route::resource('photo', 'PhotoController', ['names' => [
        'create' => 'photo.build'
    ]]);

<a name="restful-naming-resource-route-parameters"></a>
### Naming Resource Route Parameters

### 命名资源路由参数

By default, `Route::resource` will create the route parameters for your resource routes based on the "singularized" version of the resource name. You can easily override this on a per resource basis by passing `parameters` in the options array. The `parameters` array should be an associative array of resource names and parameter names:

默认地，`Route::resource` 会基于资源名称的「单数」形式生成路由参数。你可以在选项数组中传入 `parameters` 参数，实现每个资源基础中参数名称的重写。`parameters` 应该是一个将资源名称和参数名称联系在一起的数组：

    Route::resource('user', 'AdminUserController', ['parameters' => [
        'user' => 'admin_user'
    ]]);

 The example above generates the following URIs for the resource's `show` route:

 上例将会为 `show` 方法的路由生成如下的 URI：

    /user/{admin_user}

<a name="restful-localizing-resource-uris"></a>
### Localizing Resource URIs

### 本地化资源 URI

By default, `Route::resource` will create resource URIs using English verbs. If you need to localize the `create` and `edit` action verbs, you may use the `Route::resourceVerbs` method. This may be done in the `boot` method of your `AppServiceProvider`:

默认地，`Route::resource` 将会用英文动词创建资源 URI。如果你想本地化 `create` 和 `edit` 的动作名，可以使用 `Route::resourceVerb` 方法，可以在 `AppServiceProvider` 的 `boot` 方法中实现：

    use Illuminate\Support\Facades\Route;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Route::resourceVerbs([
            'create' => 'crear',
            'edit' => 'editar',
        ]);
    }

Once the verbs have been customized, a resource route registration such as `Route::resource('fotos', 'PhotoController')` will produce the following URIs:

动作名被自定义后，像 `Route::resource('fotos', 'PhtotController')` 这样注册的资源路由将会产生如下的 URI:

    /fotos/crear

    /fotos/{foto}/editar

<a name="restful-supplementing-resource-controllers"></a>
### Supplementing Resource Controllers

### 附加资源控制器

If you need to add additional routes to a resource controller beyond the default set of resource routes, you should define those routes before your call to `Route::resource`; otherwise, the routes defined by the `resource` method may unintentionally take precedence over your supplemental routes:

如果你想在默认的资源路由之外增加资源控制器路由，你应该在调用 `Route::resource` 之前定义这些路由；否则，`resource` 方法定义的路由可能会不小心覆盖你的附加路由：

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

> {tip} Remember to keep your controllers focused. If you find yourself routinely needing methods outside of the typical set of resource actions, consider splitting your controller into two, smaller controllers.

> {tip} 记住保持控制器的专一性。如果你需要典型的资源操作之外的方法，考虑将你的控制器分成两个更小的控制器吧。

<a name="dependency-injection-and-controllers"></a>
## Dependency Injection & Controllers

## 依赖注入与控制器

#### Constructor Injection

#### 构造方法注入

The Laravel [service container](/docs/{{version}}/container) is used to resolve all Laravel controllers. As a result, you are able to type-hint any dependencies your controller may need in its constructor. The declared dependencies will automatically be resolved and injected into the controller instance:

Laravel 使用[服务容器](/docs/{{version}}/container)来解析所有的控制器。因此，你可以在控制器的构造方法中对任何依赖使用类型约束，声明的依赖会自动被解析并注入控制器实例中：

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }
    }

Of course, you may also type-hint any [Laravel contract](/docs/{{version}}/contracts). If the container can resolve it, you can type-hint it. Depending on your application, injecting your dependencies into your controller may provide better testability.

当然，你也可以对任何的 [Laravel contract](/docs/{{version}}/contracts) 使用类型约束。当容器解析 contract 的时候，就会使用类型约束。直接将依赖注入控制器可能会提供更好的可测试性，但这取决于你的项目的具体情况。

#### Method Injection

#### 方法注入

In addition to constructor injection, you may also type-hint dependencies on your controller's methods. A common use-case for method injection is injecting the `Illuminate\Http\Request` instance into your controller methods:

除了构造方法注入之外，你还可以在控制器方法中使用依赖类型约束。一个常见的用法就是将 `Illuminate\Http\Request` 实例注入控制器方法中：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Store a new user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->name;

            //
        }
    }

If your controller method is also expecting input from a route parameter, simply list your route arguments after your other dependencies. For example, if your route is defined like so:

如果控制器方法需要从路由参数中获取输入内容，只需在其他依赖后列出路由参数即可。比如，路由定义如下：

    Route::put('user/{id}', 'UserController@update');

You may still type-hint the `Illuminate\Http\Request` and access your `id` parameter by defining your controller method as follows:

通过以下方式定义控制器方法，可以让你在使用 `Illuminate\Http\Request` 类型约束的同时仍然可以获取参数 `id`：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Update the given user.
         *
         * @param  Request  $request
         * @param  string  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

<a name="route-caching"></a>
## Route Caching

## 路由缓存

> {note} Closure based routes cannot be cached. To use route caching, you must convert any Closure routes to controller classes.

> {note} 基于闭包的路由并不能被缓存。如果要使用路由缓存，你必须将所有的闭包路由转换成控制器类。

If your application is exclusively using controller based routes, you should take advantage of Laravel's route cache. Using the route cache will drastically decrease the amount of time it takes to register all of your application's routes. In some cases, your route registration may even be up to 100x faster. To generate a route cache, just execute the `route:cache` Artisan command:

如果你的应用只用到了基于控制器的路由，那么你应该充分利用 Laravel 的路由缓存。使用路由缓存将极大地减少注册全部应用路由的时间。某些情况下，路由注册甚至可以快一百倍。要生成路由缓存，只需在 Artisan 命令行中执行 `route:cache` 命令：

    php artisan route:cache

After running this command, your cached routes file will be loaded on every request. Remember, if you add any new routes you will need to generate a fresh route cache. Because of this, you should only run the `route:cache` command during your project's deployment.

运行这个命令之后，每一次请求的时候都将会加载缓存的路由文件。记住，如果添加了新的路由，你需要刷新路由缓存。因此，你应该只在项目部署时才运行 `route:cache` 命令：

You may use the `route:clear` command to clear the route cache:

你可以使用 `route:clear` 命令清除路由缓存：

    php artisan route:clear


## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@Romeo](https://github.com/Romeo0906)  | <img class="avatar-66 rm-style" src="https://avatars1.githubusercontent.com/u/22153498?v=3&s=460">  |  翻译  | No bug, no gain.  |
