# HTTP 控制器

- [简介](#introduction)
- [基础控制器](#basic-controllers)
- [控制器中间件](#controller-middleware)
- [RESTful 资源控制器](#restful-resource-controllers)
    - [部分资源路由](#restful-partial-resource-routes)
    - [命名资源路由](#restful-naming-resource-routes)
    - [嵌套资源](#restful-nested-resources)
    - [附加资源控制器](#restful-supplementing-resource-controllers)
- [隐式控制器](#implicit-controllers)
- [依赖注入与控制器](#dependency-injection-and-controllers)
- [路由缓存](#route-caching)

<a name="introduction"></a>
## 简介

除了可以在单个的 `routes.php` 文件中定义所有的请求处理逻辑外，你可能还希望可以使用控制器类来组织此行为。控制器可将相关的 HTTP 请求处理逻辑组成一个类。控制器一般存放在 `app/Http/Controllers` 目录下。

> **[Summer](http://github.com/summerblue)：** 请不要在 `routes.php` 文件里面写逻辑代码，逻辑处理代码请在 Controller 里书写。
1. 因为这是最佳实践，一开始做对了，后面节省你重构代码的时间；
2. 路由缓存并不会作用在基于闭包的路由。

<a name="basic-controllers"></a>
## 基础控制器

这是一个基础控制器类的例子。所有 Laravel 控制器都应继承基础控制器类，它包含在 Laravel 的默认安装中：

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
        public function showProfile($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

我们可以通过路由来指定控制器行为，就像这样：

    Route::get('user/{id}', 'UserController@showProfile');

现在，当请求和此特定路由的 URI 相匹配时，`UserController` 类的 `showProfile` 方法就会被运行。当然，路由的参数也会被传递至该方法。

#### 控制器和命名空间

有一点非常重要，那就是我们在定义控制器路由时，不需要指定完整的控制器命名空间。我们只需要定义「根」命名空间 `App\Http\Controllers` 之后的部分类名称即可。默认 `RouteServiceProvider` 会将 `routes.php` 文件里的路由规则包在根控制器命名空间的路由群组下。

若你选择在 `App\Http\Controllers` 目录内层使用 PHP 命名空间嵌套或组织控制器，只要使用相对于 `App\Http\Controllers` 根命名空间的特定类名称即可。因此，若你的控制器类全名为 `App\Http\Controllers\Photos\AdminController`，你可以像这样注册一个路由：

    Route::get('foo', 'Photos\AdminController@method');

#### 命名控制器路由

就像闭包路由，你可以指定控制器路由的名称：

    Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

#### 控制器行为的 URLs

你也可以使用 `route` 辅助方法，生成命名控制器路由的 URL：

    $url = route('name');

一旦你指定了控制器路由的名称，则可以很容易地生成能实现该行为的 URL。你也可以使用 `action` 辅助方法生成指向控制器行为的 URL。同样地，我们只需指定基类命名空间 `App\Http\Controllers` 之后的部分控制器类名称就可以了：

    $url = action('FooController@method');

你可以使用 `Route` facade 的 `currentRouteAction` 方法访问正在运行的控制器行为名称：

	$action = Route::currentRouteAction();

<a name="controller-middleware"></a>
## 控制器中间件

可将[中间件](/docs/{{version}}/middleware)指定给控制器路由，例如：

    Route::get('profile', [
        'middleware' => 'auth',
        'uses' => 'UserController@showProfile'
    ]);

不过，在控制器构造器中指定中间件会更为方便。在控制器构造器中使用 `middleware` 方法，你可以很容易地将中间件指定给控制器。你甚至可以对中间件作出限制，仅将它提供给控制器类中的某些方法。

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

            $this->middleware('log', ['only' => ['fooAction', 'barAction']]);

            $this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
        }
    }

<a name="restful-resource-controllers"></a>
## RESTful 资源控制器

资源控制器让你可以轻松地创建与资源相关的 RESTful 控制器。例如，你可能想要创建一个用来处理应用程序保存「相片」时发送 HTTP 请求的控制器。使用 `make:controller` Artisan 命令，我们可以快速地创建一个像这样的控制器：

    php artisan make:controller PhotoController

此 Artisan 命令会生成 `app/Http/Controllers/PhotoController.php` 控制器文件。此控制器会包含用来操作可获取到的各种资源的方法。

接下来，你可以在控制器中注册资源化路由：

    Route::resource('photo', 'PhotoController');

这一条路由声明会创建多个路由，用来处理各式各样和相片资源相关的的 RESTful 行为。同样地，生成的控制器有着各种和这些行为绑定的方法，包含要处理的 URI 及动词的记录通知。

#### 由资源控制器处理的行为

| 动词      | 路径                  | 行为（方法） | 路由名称      |
|:----------|:----------------------|:-------------|:--------------|
| GET       | `/photo`              | index        | photo.index   |
| GET       | `/photo/create`       | create       | photo.create  |
| POST      | `/photo`              | store        | photo.store   |
| GET       | `/photo/{photo}`      | show         | photo.show    |
| GET       | `/photo/{photo}/edit` | edit         | photo.edit    |
| PUT/PATCH | `/photo/{photo}`      | update       | photo.update  |
| DELETE    | `/photo/{photo}`      | destroy      | photo.destroy |

<a name="restful-partial-resource-routes"></a>
#### 部分资源路由

声明资源路由时，你可以指定让此路由仅处理一部分的行为：

    Route::resource('photo', 'PhotoController',
                    ['only' => ['index', 'show']]);

    Route::resource('photo', 'PhotoController',
                    ['except' => ['create', 'store', 'update', 'destroy']]);

<a name="restful-naming-resource-routes"></a>
#### 命名资源路由

所有的资源控制器行为默认都有一路由名称；不过你可以在选项中传递一个 `names` 数组来重写这些名称：

    Route::resource('photo', 'PhotoController',
                    ['names' => ['create' => 'photo.build']]);

<a name="restful-nested-resources"></a>
#### 嵌套资源

有时你可能会需要对「嵌套」资源定义路由。例如，相片资源可能会附带多个「评论」。要「嵌套」此资源控制器，可在路由声明中使用「点」记号：

    Route::resource('photos.comments', 'PhotoCommentController');

此路由会注册一个「嵌套」资源，可通过类似这样的 URL 来访问它：`photos/{photos}/comments/{comments}`。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;

    class PhotoCommentController extends Controller
    {
        /**
         * 显示指定相片的评论。
         *
         * @param  int  $photoId
         * @param  int  $commentId
         * @return Response
         */
        public function show($photoId, $commentId)
        {
            //
        }
    }

<a name="restful-supplementing-resource-controllers"></a>
#### 附加资源控制器

如果想在资源控制器中默认的资源路由之外加入其它额外路由，则应该在调用 `Route::resource` **之前** 定义这些路由。否则，由 `resource` 方法定义的路由可能会不小心覆盖你附加的路由：

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

<a name="implicit-controllers"></a>
## 隐式控制器

Laravel 让你能够轻易地通过定义单个路由来处理控制器类中的各种行为。首先，使用 `Route::controller` 方法来定义路由。`controller` 方法接受两个参数。第一个参数是控制器所处理的基本 URI，第二个是控制器的类名称：

    Route::controller('users', 'UserController');

接下来，只要在控制器中加入方法。方法的名称应由它们所响应的 HTTP 动词作为开头，紧跟着首字母大写的 URI 所组成：

    <?php

    namespace App\Http\Controllers;

    class UserController extends Controller
    {
        /**
         * 响应对 GET /users 的请求
         */
        public function getIndex()
        {
            //
        }

        /**
         * 响应对 GET /users/show/1 的请求
         */
        public function getShow($id)
        {
            //
        }

        /**
         * 响应对 GET /users/admin-profile 的请求
         */
        public function getAdminProfile()
        {
            //
        }

        /**
         * 响应对 POST /users/profile 的请求
         */
        public function postProfile()
        {
            //
        }
    }

正如你在上述例子中所看到的，`index` 方法会响应控制器所处理的根 URI，在这个例子中是 `users`。

#### 分派路由名称

如果你想要[命名](/docs/{{version}}/routing#named-routes)控制器中的某些路由，你可以在 `controller` 方法中传入一个名称数组作为第三个参数：

    Route::controller('users', 'UserController', [
        'getShow' => 'user.show',
    ]);

<a name="dependency-injection-and-controllers"></a>
## 依赖注入与控制器

#### 构造器注入

Laravel [服务容器](/docs/{{version}}/container)用于解析所有的 Laravel 控制器。因此，在此构造器中，你可以对控制器需要的任何依赖使用类型提示。依赖会自动被解析并注入控制器实例之中。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;
    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * 用户保存库实例。
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

当然，你也可以对任何的 [Laravel contract](/docs/{{version}}/contracts) 使用类型提示。若容器能够解析它，你就可以使用类型提示。

#### 方法注入

除了构造器注入之外，你也可以对控制器行为方法的依赖使用类型提示。例如，让我们对 `Illuminate\Http\Request` 实例的其中一个方法使用类型提示：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

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
            $name = $request->input('name');

            //
        }
    }

若你想要控制器方法能预期从路由参数获得输入值，只要在你其它的依赖之后列出路由参数即可。例如，如果你的路由被定义成这个样子：

    Route::put('user/{id}', 'UserController@update');

你依然可以做 `Illuminate\Http\Request` 类型提示并通过类似下面例子这样来定义你的控制器方法，访问你的路由参数 `id`：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class UserController extends Controller
    {
        /**
         * 更新指定的用户。
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

<a name="route-caching"></a>
## 路由缓存

> **注意：** 路由缓存并不会作用在基于闭包的路由。要使用路由缓存，你必须将所有闭包路由转换为控制器类。

若你的应用程序完全通过控制器使用路由，你可以利用 Laravel 的路由缓存。使用路由缓存可以大幅降低注册你应用程序全部的路由所需的时间。在某些情况下，你的路由注册甚至可以快上一百倍！要生成路由缓存，只要运行 `route:cache` 此 Artisan 命令：

    php artisan route:cache

这就可以了！现在你的缓存路由文件将被用来代替 `app/Http/routes.php` 这一文件。请记得，若你添加了任何新的路由，就必须生成新的路由缓存。因此你可能希望只在你的项目部署时才运行 `route:cache` 这一命令。

要移除缓存路由文件而不生成新的缓存，请使用 `route:clear` 命令：

    php artisan route:clear
