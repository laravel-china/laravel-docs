# Controllers

- [简介](#introduction)
- [基础控制器](#basic-controllers)
    - [定义控制器](#defining-controllers)
    - [控制器与命名空间](#controllers-and-namespaces)
    - [单一行为控制器](#single-action-controllers)
- [控制器中间件](#controller-middleware)
- [RESTful 资源控制器](#resource-controllers)
    - [部分资源路由](#restful-partial-resource-routes)
    - [命名资源路由](#restful-naming-resource-routes)
    - [重命名资源路由器参数](#restful-naming-resource-route-parameters)
    - [附加资源控制器](#restful-supplementing-resource-controllers)
- [依赖注入与控制器](#dependency-injection-and-controllers)
- [路由缓存](#route-caching)

<a name="introduction"></a>
## 简介

除了可以以闭包的形式在路由文件中定义所有的请求处理逻辑外，你可能还希望可以使用控制器类来组织此行为。控制器可将相关的 HTTP 请求处理逻辑组成一个类。控制器一般存放在 `app/Http/Controllers` 目录下。

> 译者注：请不要在路由文件里面写逻辑代码，逻辑处理代码请在 Controller 里书写。

<a name="basic-controllers"></a>
## 基础控制器

<a name="defining-controllers"></a>
### 定义控制器

下面是一个基础控制器类的例子。所有 Laravel 控制器都应继承基础控制器类，它包含在 Laravel 的默认安装中。该基础类提供了一些便捷的方法，例如 `middleware` 方法，该方法可以用来将中间件附加在控制器行为上：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 显示指定用户的个人数据。
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

你可以定义一个指向该控制器行为的路由，就像这样：

    Route::get('user/{id}', 'UserController@show');

现在，当请求和此特定路由的 URI 相匹配时，`UserController` 类的 `show` 方法就会被运行。当然，路由的参数也会被传递至该方法。

> {tip} 控制器不是 **必须** 要继承基础类。只不过，你将不能使用一些便捷的特性，如 `middleware`，`validate`，和 `dispatch` 等方法。

<a name="controllers-and-namespaces"></a>
### 控制器与命名空间

有一点非常重要，那就是我们在定义控制器路由时，不需要指定完整的控制器命名空间。我们只需要定义「根」命名空间 `App\Http\Controllers` 之后的部分类名称即可。默认 `RouteServiceProvider` 会使用路由群组，把 `routes.php` 文件里所有路由规则都配置了根控制器命名空间。

若你需要在 `App\Http\Controllers` 目录内层使用 PHP 命名空间嵌套或组织控制器，只要使用相对于 `App\Http\Controllers` 根命名空间的特定类名称即可。例如控制器类全名为 `App\Http\Controllers\Photos\AdminController`，你可以像这样注册一个路由：

    Route::get('foo', 'Photos\AdminController@method');

<a name="single-action-controllers"></a>
### 单一行为控制器

如果你想定义只有一个行为的控制器，只需在控制器中设置一个名为 `__invoke` 的方法即可：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class ShowProfile extends Controller
    {
        /**
         * 显示指定用户的个人数据。
         *
         * @param  int  $id
         * @return Response
         */
        public function __invoke($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

给单一行为控制器注册路由时，不需要指定方法：

    Route::get('user/{id}', 'ShowProfile');

<a name="controller-middleware"></a>
## 控制器中间件

[中间件](/docs/{{version}}/middleware) 可以指定到路由文件中的控制器路由上：

    Route::get('profile', 'UserController@show')->middleware('auth');

不过，在控制器构造器中指定中间件会更为灵活。在控制器构造器中使用 `middleware` 方法，你可以很容易地将中间件指定给控制器。你甚至可以对中间件作出限制，仅将它提供给控制器类中的某些方法。

    class UserController extends Controller
    {
        /**
         * 添加一个 UserController 实例。
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

> {tip} 你可以将某个中间件指定到一部分控制器行为上，然而，那可能意味着你的控制器过于臃肿。反而，你可以考虑将控制器分解成多个较小的控制器。

<a name="resource-controllers"></a>
## RESTful 资源控制器

Laravel 资源路由只需一行代码就可以将典型的 "CRUD" 路由指定到一个控制器上。例如，你可能想要创建一个用来处理应用程序保存「相片」时发送 HTTP 请求的控制器。使用 `make:controller` Artisan 命令，我们可以快速地创建一个像这样的控制器：

    php artisan make:controller PhotoController --resource

此命令会生成 `app/Http/Controllers/PhotoController.php` 控制器文件。该控制器会包含各种资源操作的方法。

接下来，你可以在控制器中注册资源化路由：

    Route::resource('photos', 'PhotoController');

这一条路由声明会创建多个路由，用来处理各式各样和相片资源相关的的 RESTful 行为。同样地，生成的控制器有着各种和这些行为绑定的方法，包含要处理的 URI 及方法对应的注释。

#### 由资源控制器处理的行为

| 动词       | 路径                   | 行为（方法）   | 路由名称       |
|:----------|:----------------------|:-------------|:--------------|
| GET       | `/photos`              | index        | photos.index   |
| GET       | `/photos/create`       | create       | photos.create  |
| POST      | `/photos`              | store        | photos.store   |
| GET       | `/photos/{photo}`         | show         | photos.show    |
| GET       | `/photos/{photo}/edit`    | edit         | photos.edit    |
| PUT/PATCH | `/photos/{photo}`         | update       | photos.update  |
| DELETE    | `/photos/{photo}`         | destroy      | photos.destroy |

#### 模拟表单方法

因为 HTML 表单不能发送 `PUT`，`PATCH`，或 `DELETE` 请求, 你需要使用隐藏的 `_method` 表单字段来模拟这些 HTTP 动词。 你可以使用辅助函数 `method_field` 生成该表单字段:

    {{ method_field('PUT') }}

<a name="restful-partial-resource-routes"></a>
### 部分资源路由

声明资源路由时，你可以指定让此路由仅处理一部分的行为：

    Route::resource('photo', 'PhotoController', ['only' => [
        'index', 'show'
    ]]);

    Route::resource('photo', 'PhotoController', ['except' => [
        'create', 'store', 'update', 'destroy'
    ]]);

<a name="restful-naming-resource-routes"></a>
### 命名资源路由

所有的资源控制器行为默认都有路由名称；不过你可以在`选项`参数数组中传递一个 `names` 数组来重写这些名称：

    Route::resource('photo', 'PhotoController', ['names' => [
        'create' => 'photo.build'
    ]]);

<a name="restful-naming-resource-route-parameters"></a>
### 重命名资源路由器参数

默认情况下，`Route::resource` 会基于资源英文名称的「单数」形式生成路由参数。如果你想重写此参数，可以在 `选项` 参数数组中传入 `parameters` 数组，在此数组中指定参数名称：

    Route::resource('user', 'AdminUserController', ['parameters' => [
        'user' => 'admin_user'
    ]]);

 上面的示例代码，会为资源的 `show` 路由生成以下的 URIs：

    /user/{admin_user}

<a name="restful-supplementing-resource-controllers"></a>
### 附加资源控制器

如果想在资源控制器中默认的资源路由之外加入其它额外路由，则应该在调用 `Route::resource` **之前** 定义这些路由。否则，由 `resource` 方法定义的路由可能会不小心覆盖你附加的路由：

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

> {tip} 记住要保持控制器的专注性。如果你发现控制器需要典型资源化行为之外的一些方法，就可以考虑将其分解为两个较小的控制器了。

<a name="dependency-injection-and-controllers"></a>
## 依赖注入与控制器

#### 构造器注入

Laravel 使用 [服务容器](/docs/{{version}}/container) 来解析所有的控制器。因此，你能够在控制器的构造方法中，对任何需要的依赖使用类型提示。这些声明的依赖将被自动解析并注入到控制器实例当中：

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * 用户 Repository 实例。
         */
        protected $users;

        /**
         * 创建新的控制器实例。
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }
    }

当然，你也可以对任何的 [Laravel contract](/docs/{{version}}/contracts) 使用类型提示。若容器能够解析它，你就可以使用类型提示。根据你的应用的情况，把依赖注入到控制器中会提供更好的可测试性。

#### 方法注入

除了构造器注入之外，你也可以对 `控制器行为方法的依赖` 使用类型提示。例如，让我们对 `Illuminate\Http\Request` 实例的其中一个方法使用类型提示：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 保存一个新的用户。
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

想要从控制器方法中获取路由参数的话，只要在其它的依赖之后列出路由参数即可。例如：

    Route::put('user/{id}', 'UserController@update');

你依然可以做 `Illuminate\Http\Request` 类型提示并通过类似下面例子这样来定义你的控制器方法，访问你的路由参数 `id`：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 更新指定用户。
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

> {note} 路由缓存并不会作用在基于闭包的路由。要使用路由缓存，你必须将所有闭包路由转换为控制器类。

若你的应用程序完全通过控制器使用路由，你应该充分利用 Laravel 路由缓存。使用路由缓存可以大幅降低注册全部路由所需的时间。在某些情况下，你的路由注册甚至可以快上一百倍！要生成路由缓存，只要运行 `route:cache` 此 Artisan 命令：

    php artisan route:cache

这就可以了！现在你的缓存路由文件将被用来代替 `app/Http/routes.php` 这一文件。请记得，若你添加了任何新的路由，就必须生成新的路由缓存。因此你可能希望只在你的项目部署时才运行 `route:cache` 这一命令。

要移除缓存路由文件而不生成新的缓存，请使用 `route:clear` 命令：

    php artisan route:clear

> 译者注： 想知道更多 Laravel 程序调优的技巧？请参阅：[Laravel 5 程序优化技巧](https://phphub.org/topics/2020)

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@Macken](https://phphub.org/users/1289)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/1289_1473143048.jpg?imageView2/1/w/200/h/200">  |  翻译  | 专注Web开发，[麦肯先生](https://macken.me) My Blog  |
