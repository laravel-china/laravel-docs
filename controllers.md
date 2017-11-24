# Laravel 的 HTTP 控制器

- [简介](#introduction)
- [基础控制器](#basic-controllers)
    - [定义控制器](#defining-controllers)
    - [控制器与命名空间](#controllers-and-namespaces)
    - [单个行为控制器](#single-action-controllers)
- [控制器中间件](#controller-middleware)
- [资源控制器](#resource-controllers)
    - [部分资源路由](#restful-partial-resource-routes)
    - [命名资源路由](#restful-naming-resource-routes)
    - [命名资源路由参数](#restful-naming-resource-route-parameters)
    - [本地化资源 URI](#restful-localizing-resource-uris)
    - [补充资源控制器](#restful-supplementing-resource-controllers)
- [依赖注入 & 控制器](#dependency-injection-and-controllers)
- [路由缓存](#route-caching)

<a name="introduction"></a>
## 简介

除了在路由文件中以闭包的形式定义所有的请求处理逻辑外，还可以使用控制器类来组织此类行为。控制器能够将相关的请求处理逻辑组成一个单独的类。控制器被存放在 `app/Http/Controllers` 目录下。

<a name="basic-controllers"></a>
## 基础控制器

<a name="defining-controllers"></a>
### 定义控制器

下面是一个基础控制器类的例子。需要注意的是，该控制器继承了 Laravel 内置的基础控制器类。该基础控制器类提供了一些便捷的方法，比如 `middleware` 方法，该方法可以用来给控制器行为添加中间件：

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

你可以这样定义一个指向该控制器行为的路由：

    Route::get('user/{id}', 'UserController@show');

现在，当一个请求与此指定路由的 URI 匹配时， `UserController` 类的 `show` 方法就会被执行。当然，路由参数也会被传递至该方法。

> {tip} 控制器并不是一定要继承基础类。但是，如果控制器没有继承基础类，你将无法使用一些便捷的功能，比如 `middleware`、`validate` 和 `dispatch` 方法。

<a name="controllers-and-namespaces"></a>
### 控制器与命名空间

需要注意的是，在定义控制器路由时，我们不需要指定完整的控制器命名空间。因为 `RouteServiceProvider` 会在一个包含命名空间的路由器组中加载路由文件，所以我们只需要指定类名中 `App\Http\Controllers` 命名空间之后的部分就可以了。

如果你选择将控制器存放在 `App\Http\Controllers` 目录下的某一目录，只需要简单地使用相对于 `App\Http\Controllers` 根命名空间的特定类名。也就是说，如果完整的控制器类是 `App\Http\Controllers\Photos\AdminController` ，那你应该用以下这种方式向控制器注册路由：

    Route::get('foo', 'Photos\AdminController@method');

<a name="single-action-controllers"></a>
### 单个行为控制器

如果你想定义一个只处理单个行为的控制器，你可以在控制器中放置一个 `__invoke` 方法：

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

注册单个行为控制器的路由时，不需要指定方法：

    Route::get('user/{id}', 'ShowProfile');

<a name="controller-middleware"></a>
## 控制器中间件

[中间件](/docs/{{version}}/middleware) 可以在路由文件中被分配给控制器路由：

    Route::get('profile', 'UserController@show')->middleware('auth');

但是，在控制器的构造方法中指定中间件会更方便。使用控制器构造函数中的 `middleware` 方法，你可以很容易地将中间件分配给控制器的行为。你甚至可以约束中间件只对控制器类中的某些特定方法生效：

    class UserController extends Controller
    {
        /**
         * 实例化一个新的控制器实例。
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

还能使用闭包来为控制器注册中间件。闭包的方便之处在于，你无需特地创建一个中间件类来为某一个特殊的控制器注册中间件：

    $this->middleware(function ($request, $next) {
        // ...

        return $next($request);
    });

> {tip} 你可以将中间件分配给控制器的部分行为上，然而这样可能意味着你的控制器正在变得很大。这里建议你将控制器分成多个更小的控制器。

<a name="resource-controllers"></a>
## 资源控制器

Laravel 资源路由将典型的「CRUD」路由分配给具有单行代码的控制器。比如，创建一个控制器来处理应用保存的「照片」的所有 HTTP 请求。使用 Artisan 命令 `make:controller` 来快速创建控制器：

    php artisan make:controller PhotoController --resource

这个命令会生成一个控制器 `app/Http/Controllers/PhotoController.php`。其中包含了每个可用资源的操作方法。

接下来，你可以给控制器注册一个资源路由：

    Route::resource('photos', 'PhotoController');

这个路由声明创建多个路由来处理资源上的各种行为。生成的控制器为每个行为保留了方法，同时还包括了 处理 HTTP 动作和 URI 的声明注释。

#### 资源控制器操作处理

| 动作 | URI | 行为 | 路由名称 |
| --- | --- | --- | --- |
| GET       | `/photos`              | index   | photos.index   |
| GET       | `/photos/create`       | create  | photos.create  |
| POST      | `/photos`              | store   | photos.store   |
| GET       | `/photos/{photo}`      | show    | photos.show    |
| GET       | `/photos/{photo}/edit` | edit    | photos.edit    |
| PUT/PATCH | `/photos/{photo}`      | update  | photos.update  |
| DELETE    | `/photos/{photo}`      | destroy | photos.destroy |

#### 指定资源模型

如果你使用了路由模型绑定，并且想在资源控制器的方法中使用类型提示，你可以在生成控制器的时候使用 `--model` 选项：

    php artisan make:controller PhotoController --resource --model=Photo

#### 伪造表单方法

因为 HTML 表单不能生成 `PUT`、 `PATCH` 或者 `DELETE` 请求，所以你需要添加一个隐藏的 `_method` 输入字段来伪造这些 HTTP 动作。辅助函数 `method_field` 可以帮你创建这个字段：

    {{ method_field('PUT') }}

<a name="restful-partial-resource-routes"></a>
### 部分资源路由

声明资源路由时，你可以指定控制器处理的部分行为，而不是所有默认的行为：

    Route::resource('photo', 'PhotoController', ['only' => [
        'index', 'show'
    ]]);

    Route::resource('photo', 'PhotoController', ['except' => [
        'create', 'store', 'update', 'destroy'
    ]]);

#### API资源路由

当声明用于 APIs 的资源路由时，通常需要排除显示 HTML 模板的路由（如 `create` 和 `edit` ）。为了方便起见，你可以使用 `apiResource` 方法自动排除这两个路由：

    Route::apiResource('photo', 'PhotoController');
    
你可以传递一个数组给 `apiResources` 方法来注册多个API资源控制器：

    Route::apiResources([
        'photos' => 'PhotoController',
        'posts' => 'PostController'
    ]);

<a name="restful-naming-resource-routes"></a>
### 命名资源路由

默认情况下，所有的资源控制器行为都有一个路由名称。你可以传入 `names` 数组来覆盖这些名称：

    Route::resource('photo', 'PhotoController', ['names' => [
        'create' => 'photo.build'
    ]]);

<a name="restful-naming-resource-route-parameters"></a>
### 命名资源路由参数

默认情况下，`Route::resource` 会根据资源名称的「单数」形式创建资源路由的路由参数。你可以在选项数组中传入 `parameters` 参数来轻松地覆盖每个资源。`parameters` 数组应该是资源名称和参数名称的关联数组：

    Route::resource('user', 'AdminUserController', ['parameters' => [
        'user' => 'admin_user'
    ]]);

上例将会为资源的 `show` 路由生成如下的 URI ：

    /user/{admin_user}

<a name="restful-localizing-resource-uris"></a>
### 本地化资源 URI

默认情况下，`Route::resource` 将会用英文动词创建资源 URI。如果需要本地化 `create` 和 `edit` 行为动作名，可以在 `AppServiceProvider` 的 `boot` 中使用 `Route::resourceVerbs` 方法实现：

    use Illuminate\Support\Facades\Route;

    /**
     * 引导任何应用服务。
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

动作被自定义后，像 `Route::resource('fotos', 'PhotoController')` 这样注册的资源路由将会产生如下的 URI：

    /fotos/crear

    /fotos/{foto}/editar

<a name="restful-supplementing-resource-controllers"></a>
### 补充资源控制器

如果你想在默认的资源路由中增加额外的路由，你应该在 `Route::resource` 之前定义这些路由。否则由 `resource` 方法定义的路由可能会无意中优先于你补充的路由：

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

> {tip} 记住保持控制器的专一性。如果你需要典型的资源操作之外的方法，可以考虑将你的控制器分成两个更小的控制器。

<a name="dependency-injection-and-controllers"></a>
## 依赖注入 & 控制器

#### 构造函数注入

Laravel 使用 [服务容器](/docs/{{version}}/container) 来解析所有的控制器。因此，你可以在控制器的构造函数中使用类型提示需要的依赖项，而声明的依赖项会自动解析并注入控制器实例中：

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * 用户 repository 实例.
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

当然，你也可以类型提示 [Laravel 契约](/docs/{{version}}/contracts)，只要它能被解析。根据你的应用，将你的依赖项注入控制器能提供更好的可测试性。

#### 方法注入

除了构造函数注入之外，你还可以在控制器方法中类型提示依赖项。最常见的用法就是将 `Illuminate\Http\Request` 实例注入到控制器方法中：

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

如果控制器方法需要从路由参数中获取输入内容，只需要在其他依赖项后列出路由参数即可。比如，如果你的路由是这样定义的：

    Route::put('user/{id}', 'UserController@update');
你仍然可以类型提示 `Illuminate\Http\Request` 并通过定义控制器方法获取 `id` 参数，如下所示：

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

> {note} 基于闭包的路由不能被缓存。如果要使用路由缓存，你必须将所有的闭包路由转换成控制器类路由。

如果你的应用只使用了基于控制器的路由，那么你应该充分利用 Laravel 的路由缓存。使用路由缓存将极大地减少注册所有应用路由所需的时间。某些情况下，路由注册的速度甚至可以快一百倍。要生成路由缓存，只需执行 Artisan 命令 `route:cache`：

    php artisan route:cache

运行这个命令之后，每一次请求的时候都将会加载缓存的路由文件。如果你添加了新的路由，你需要生成 一个新的路由缓存。因此，你应该只在生产环境运行 `route:cache` 命令：

你可以使用 `route:clear` 命令清除路由缓存：

    php artisan route:clear

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@easyFroce](https://github.com/easyForce)  | <img class="avatar-66 rm-style" src="https://s.gravatar.com/avatar/6c3b9c5876f09ef9603c6d64c503ca19?s=80">  |  翻译  | LOL |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  Review  | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
