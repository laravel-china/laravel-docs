# Laravel 的 HTTP 控制器

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
## 简介

除了在路由文件中以闭包的形式定义所有的请求处理逻辑外，你可能还想使用控制器类来组织此类操作。控制器能够将相关的请求处理逻辑组成一个单独的类。控制器被存放在 `app/Http/Controllers` 目录下。

<a name="basic-controllers"></a>
## 基础控制器

<a name="defining-controllers"></a>
### 定义控制器

以下是一个基础控制器类的例子。需要注意的是，该控制器继承了 Laravel 内置的基础控制器类。该基础类提供了一些便捷的方法，比如 `middleware` 方法，该方法可以用来给控制器操作添加中间件：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 展示给定用户的信息。
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

你可以这样定义一个指向该控制器操作的路由：

    Route::get('user/{id}', 'UserController@show');

现在，当请求和此特定路由的 URI 匹配时，`UserController` 类的 `show` 方法就会被执行。当然，路由参数也会被传递至该方法。

> {tip} 控制器并不是 **一定** 要继承基础类。只是，你将无法使用一些便捷的功能，比如 `middleware`，`validate` 和 `dispatch` 方法。

<a name="controllers-and-namespaces"></a>
### 控制器与命名空间

这一点很重要，我们在定义控制器路由时，不必指定完整的控制器命名空间。`RouteServiceProvider` 会在一个包含命名空间的路由组中加载路由文件，因此我们只需要指定类名中 `App\Http\Controllers` 命名空间之后的部分就可以了。

如果你选择将控制器存放在 `App\Http\Controllers` 目录下，只需简单地使用相对于根命名空间 `App\Http\Controllers` 的特定类名。因此，如果你的完整控制器类是 `App\Http\Controllers\Photos\AdminController`，你应该用这种方式注册指向该控制器的路由：

    Route::get('foo', 'Photos\AdminController@method');

<a name="single-action-controllers"></a>
### 单一操作控制器

如果想定义一个只处理单个操作的控制器，你可以在控制器中只设置一个 `__invoke` 方法：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class ShowProfile extends Controller
    {
        /**
         * 展示给定用户的信息。
         *
         * @param  int  $id
         * @return Response
         */
        public function __invoke($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

为单一操作控制器注册路由时，无需指定方法：

    Route::get('user/{id}', 'ShowProfile');

<a name="controller-middleware"></a>
## 控制器中间件

[中间件](/docs/{{version}}/middleware) 可以在路由文件中指定给控制器路由：

    Route::get('profile', 'UserController@show')->middleware('auth');

然而，在控制器的构造方法中指定中间件会更为便捷。在控制器构造方法中使用 `middleware` 方法，你可以很容易地将中间件指定给控制器操作。你甚至可以约束中间件只对控制器类中的某个特定方法生效：

    class UserController extends Controller
    {
        /**
         * 创建一个新的控制器实例。
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

控制器也允许你使用闭包的方式注册中间件。这提供了一种很便捷地为单个控制器定义中间件的方式，而不用定义一个完整的中间件类：

    $this->middleware(function ($request, $next) {
        // ...

        return $next($request);
    });

> {tip} 你可能将中间件指定到控制器的部分操作上，然而，这会使你的控制器过于臃肿。换个角度，考虑将控制器分成多个更小的控制器。

<a name="resource-controllers"></a>
## 资源控制器

Laravel 资源路由可以将典型的「CURD」路由指定到一个控制器上，仅需一行代码就可以实现。比如，你可能希望创建一个控制器来处理所有应用保存的「相片」的 HTTP 请求。使用 Artisan 命令 `make:controller`，就能快速创建这样一个控制器：

    php artisan make:controller PhotoController --resource

这个命令会在 `app/Http/Controllers/PhotoController.php` 中生成一个控制器，该控制器包含了各种可用的资源操作方法。

接下来，你可以给控制器注册一个资源路由：

    Route::resource('photos', 'PhotoController');

这个路由声明会创建多个路由来处理各种各样的资源操作。前面生成的控制器已经包含了这些操作的方法，还包括了 HTTP 动作和操作 URI 的注释。

#### 资源控制器操作处理

动作      | URI                  | 操作       | 路由名称
----------|-----------------------|--------------|---------------------
GET       | `/photos`              | index        | photos.index
GET       | `/photos/create`       | create       | photos.create
POST      | `/photos`              | store        | photos.store
GET       | `/photos/{photo}`      | show         | photos.show
GET       | `/photos/{photo}/edit` | edit         | photos.edit
PUT/PATCH | `/photos/{photo}`      | update       | photos.update
DELETE    | `/photos/{photo}`      | destroy      | photos.destroy

#### 指定资源模型

如果你使用了路由模型绑定，并且想在资源控制器的方法中对某个模型实例做类型约束，你可以在生成控制器的时候使用 `--model` 选项：

    php artisan make:controller PhotoController --resource --model=Photo

#### 伪造表单方法
送 `PUT`，`PATCH` 或者 `DELETE` 请求，你需要添加一个 `_method` 隐藏域字段来伪造 HTTP 动作。`method_field` 辅助函数可以为你创建这个字段：

    {{ method_field('PUT') }}

<a name="restful-partial-resource-routes"></a>
### 部分资源路由

声明资源路由的时候，你可以指定控制器处理部分操作，而不必使用全部默认的操作：

    Route::resource('photo', 'PhotoController', ['only' => [
        'index', 'show'
    ]]);

    Route::resource('photo', 'PhotoController', ['except' => [
        'create', 'store', 'update', 'destroy'
    ]]);

<a name="restful-naming-resource-routes"></a>
### 命名资源路由

默认地，所有的资源路由操作都有一个路由名称；不过你可以在参数选项中传入一个 `names` 数组来重写这些名称：

    Route::resource('photo', 'PhotoController', ['names' => [
        'create' => 'photo.build'
    ]]);

<a name="restful-naming-resource-route-parameters"></a>
### 命名资源路由参数

默认地，`Route::resource` 会基于资源名称的「单数」形式生成路由参数。你可以在选项数组中传入 `parameters` 参数，实现每个资源基础中参数名称的重写。`parameters` 应该是一个将资源名称和参数名称联系在一起的数组：

    Route::resource('user', 'AdminUserController', ['parameters' => [
        'user' => 'admin_user'
    ]]);

 上例将会为 `show` 方法的路由生成如下的 URI：

    /user/{admin_user}

<a name="restful-localizing-resource-uris"></a>
### 本地化资源 URI

默认地，`Route::resource` 将会用英文动词创建资源 URI。如果你想本地化 `create` 和 `edit` 的动作名，可以使用 `Route::resourceVerb` 方法，可以在 `AppServiceProvider` 的 `boot` 方法中实现：

    use Illuminate\Support\Facades\Route;

    /**
     * 自定义任何应用服务。
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

动作名被自定义后，像 `Route::resource('fotos', 'PhtotController')` 这样注册的资源路由将会产生如下的 URI:

    /fotos/crear

    /fotos/{foto}/editar

<a name="restful-supplementing-resource-controllers"></a>
### 附加资源控制器

如果你想在默认的资源路由之外增加资源控制器路由，你应该在调用 `Route::resource` 之前定义这些路由；否则，`resource` 方法定义的路由可能会不小心覆盖你的附加路由：

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

> {tip} 记住保持控制器的专一性。如果你需要典型的资源操作之外的方法，考虑将你的控制器分成两个更小的控制器吧。

<a name="dependency-injection-and-controllers"></a>
## 依赖注入与控制器

#### 构造方法注入

Laravel 使用 [服务容器](/docs/{{version}}/container) 来解析所有的控制器。因此，你可以在控制器的构造方法中对任何依赖使用类型约束，声明的依赖会自动被解析并注入控制器实例中：

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * 用户 repository 实例。
         */
        protected $users;

        /**
         * 创建一个新的控制器实例。
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }
    }

当然，你也可以对任何的 [Laravel contract](/docs/{{version}}/contracts) 使用类型约束。当容器解析 contract 的时候，就会使用类型约束。直接将依赖注入控制器可能会提供更好的可测试性，但这取决于你的项目的具体情况。

#### 方法注入

除了构造方法注入之外，你还可以在控制器方法中使用依赖类型约束。一个常见的用法就是将 `Illuminate\Http\Request` 实例注入控制器方法中：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 保存一个新用户。
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

如果控制器方法需要从路由参数中获取输入内容，只需在其他依赖后列出路由参数即可。比如，路由定义如下：

    Route::put('user/{id}', 'UserController@update');

通过以下方式定义控制器方法，可以让你在使用 `Illuminate\Http\Request` 类型约束的同时仍然可以获取参数 `id`：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 更新给定用户的信息。
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
## 路由缓存

> {note} 基于闭包的路由并不能被缓存。如果要使用路由缓存，你必须将所有的闭包路由转换成控制器类。

如果你的应用只用到了基于控制器的路由，那么你应该充分利用 Laravel 的路由缓存。使用路由缓存将极大地减少注册全部应用路由的时间。某些情况下，路由注册甚至可以快一百倍。要生成路由缓存，只需在 Artisan 命令行中执行 `route:cache` 命令：

    php artisan route:cache

运行这个命令之后，每一次请求的时候都将会加载缓存的路由文件。记住，如果添加了新的路由，你需要刷新路由缓存。因此，你应该只在项目部署时才运行 `route:cache` 命令：

你可以使用 `route:clear` 命令清除路由缓存：

    php artisan route:clear


## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@Romeo](https://github.com/Romeo0906)  | <img class="avatar-66 rm-style" src="https://avatars1.githubusercontent.com/u/22153498?v=3&s=460">  |  翻译  | No bug, no gain.  |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/3810/laravel-54-document-translation-come-and-join-the-translation)。
> 
> 文档永久地址： http://d.laravel-china.org