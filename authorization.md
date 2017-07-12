# 用户授权

- [简介](#introduction)
- [Gates](#gates)
    - [编写 Gates](#writing-gates)
    - [通过 Gates 授权动作](#authorizing-actions-via-gates)
- [创建策略](#creating-policies)
    - [生成策略](#generating-policies)
    - [注册策略](#registering-policies)
- [编写策略](#writing-policies)
    - [策略方法](#policy-methods)
    - [不使用模型方法](#methods-without-models)
    - [策略过滤器](#policy-filters)
- [使用策略授权动作](#authorizing-actions-using-policies)
    - [通过用户模型](#via-the-user-model)
    - [通过中间件](#via-middleware)
    - [通过控制器辅助函数](#via-controller-helpers)
    - [通过 Blade 模板](#via-blade-templates)

<a name="introduction"></a>
## 简介

除了内置提供的 [用户认证](/docs/{{version}}/authentication) 服务外，Laravel 还提供一种更简单的方式来处理用户授权动作。类似用户认证，Laravel 有 2 种主要方式来实现用户授权：gates 和策略。

可以把 gates 和策略类比于路由和控制器。Gates 提供了一个简单、基于闭包的方式来授权认证。策略则和控制器类似，在特定的模型或者资源中通过分组来实现授权认证的逻辑。我们先来看看 gates，然后再看策略。

在你的应用中，不要将 gates 和策略当作相互排斥的方式。大部分应用很可能同时包含 gates 和策略，并且能很好的工作。Gates 大部分应用在模型和资源无关的地方，比如查看管理员的面板。与此相比，策略应该用在特定的模型或者资源中。

<a name="gates"></a>
## Gates

<a name="writing-gates"></a>
### 编写 Gates

Gates 是用来决定用户是否授权访问给定的动作的闭包函数，并且典型的做法是在 `App\Providers\AuthServiceProvider` 类中使用 `Gate` facade 定义。Gates 接受一个用户实例作为第一个参数，接受可选参数，比如相关的 Eloquent 模型：

    /**
     * 注册任意用户认证 、用户授权服务。
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', function ($user, $post) {
            return $user->id == $post->user_id;
        });
    }

<a name="authorizing-actions-via-gates"></a>
### 使用 gates 授权动作

使用 gates 来授权动作时，应当使用 `allows` 方法。注意你并不需要传递当前认证通过的用户给 `allows` 方法。Laravel 会自动处理好传入的用户，然后传递给 gate 闭包函数：

    if (Gate::allows('update-post', $post)) {
        // 当前用户可以更新 post
    });

如果需要指定一个特定的用户可以访问某个动作，可以使用 `Gate` facade 中的 `forUser` 方法：

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // 指定用户可以更新博客...
    }

<a name="creating-policies"></a>
## 创建策略

<a name="generating-policies"></a>
### 生成策略

策略是在特定模型或者资源中组织授权逻辑的类。例如，如果应用是一个博客，会有一个 `Post` 模型和一个相应的 `PostPolicy` 来授权用户动作，比如创建或者更新博客。

可以使用 `make:policy` [artisan 命令](/docs/{{version}}/artisan) 来生成策略。生成的策略将放置在 `app/Policies` 目录。如果在你的应用中不存在这个目录，那么 Laravel 会自动创建：

    php artisan make:policy PostPolicy

`make:policy` 会生成空的策略类。如果希望生成的类包含基本的「CRUD」策略方法， 可以在使用命令时指定 `--model` 选项：

    php artisan make:policy PostPolicy --model=Post

> {tip} 所有授权策略会通过 Laravel [服务容器](/docs/{{version}}/container) 解析，意指你可以在授权策略的构造器对任何需要的依赖使用类型提示，它们将会被自动注入。

<a name="registering-policies"></a>
### 注册策略

一旦该授权策略存在，需要将它进行注册。`AuthServiceProvider` 包含了一个 `policies` 属性，可将各种模型对应至管理它们的授权策略。注册一个策略将引导 Laravel 在授权动作访问给定模型时使用何种策略：

    <?php

    namespace App\Providers;

    use App\Post;
    use App\Policies\PostPolicy;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * 应用的策略映射。
         *
         * @var array
         */
        protected $policies = [
            Post::class => PostPolicy::class,
        ];

        /**
         * 注册任意用户认证、用户授权服务。
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            //
        }
    }

<a name="writing-policies"></a>
## 编写策略

<a name="policy-methods"></a>
### 策略方法

一旦授权策略被生成且注册，我们就可以为每个权限的授权增加方法。例如，让我们在 `PostPolicy` 中定义一个 `update` 方法，它会判断指定的 `User` 是否可以「更新」一条 `Post`。

`update` 方法接受 `User` 和 `Post` 实例作为参数，并且应当返回 `true` 或 `false` 来指明用户是否授权更新给定的 `Post`。因此，这个例子中，我们判断用户的 id 是否和 post 中的 `user_id` 匹配：

    <?php

    namespace App\Policies;

    use App\User;
    use App\Post;

    class PostPolicy
    {
        /**
         * 判断给定博客能否被用户更新
         *
         * @param  \App\User  $user
         * @param  \App\Post  $post
         * @return bool
         */
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }

你可以接着在此授权策略定义额外的方法，作为各种权限需要的授权。例如，你可以定义 `view` 或 `delete` 方法来授权 `Post` 的多种行为。可以为自定义策略方法使用自己喜欢的名字。

> {tip} 如果在 Artisan 控制台生成策略使用 `--model` 选项，会自动包含
 `view`、`create`、`update` 和 `delete` 动作。

<a name="methods-without-models"></a>
### 不包含模型方法

一些策略方法只接受当前认证通过的用户作为参数，而不用传入授权相关的模型实例。最普遍的应用场景就是授权 `create` 动作。例如，如果正在创建一篇博客，你可能希望检查一下当前用户是否授权创建博客。

当定义一个不需要传入模型实例的策略方法时，比如 `create` 方法，你需要定义这个方法只接受已授权的用户作为参数：

    /**
     * 判断给定用户是否可以创建博客。
     *
     * @param  \App\User  $user
     * @return bool
     */
    public function create(User $user)
    {
        //
    }

> {tip} 如果在 Artisan 控制台生成策略使用 `--model` 选项，所有相关的「CRUD」策略方法已经定义好了。

<a name="policy-filters"></a>
### 策略过滤器

对特定用户，你可能希望通过指定的策略授权所有动作。要达到这个目的，可以在策略中定义一个 `before` 方法。`before` 方法会在策略中其他所有方法之前执行，这样提供了一种方式来授权动作而不是指定的策略方法来执行判断。这个功能最常见的场景是授权应用的管理员可以访问所有动作：

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

<a name="authorizing-actions-using-policies"></a>
## 使用策略授权动作

<a name="via-the-user-model"></a>
### 通过用户模型

Laravel 应用内置的 `User` 模型包含 2 个有用的方法来授权动作：`can` 和 `cant`。`can` 方法指定需要授权的动作和相关的模型。例如，判定一个用户是否授权更新给定的 `Post` 模型：

    if ($user->can('update', $post)) {
        //
    }

如果给定模型的 [策略已被注册](#registering-policies)，`can` 方法会自动调用核实的策略方法并且返回 boolean 值。如果没有策略注册到这个模型，`can` 方法会尝试调用和动作名相匹配的基于闭包的 Gate。

#### 不需要指定模型的动作

一些动作，比如 `create`，并不需要指定模型实例。在这种情况下，可传递一个类名给 `can` 方法。当授权动作时，这个类名将被用来判断使用哪个策略：

    use App\Post;

    if ($user->can('create', Post::class)) {
        // 执行相关策略中的「create」方法...
    }

<a name="via-middleware"></a>
### 通过中间件

Laravel 包含一个可以在请求到达路由或控制器之前就进行动作授权的中间件。默认，`Illuminate\Auth\Middleware\Authorize` 中间件被指定到 `App\Http\Kernel` 类中 `can` 键上。我们用一个授权用户更新博客的例子来讲解 `can` 中间件的使用：

    use App\Post;

    Route::put('/post/{post}', function (Post $post) {
        // 当前用户可以更新博客...
    })->middleware('can:update,post');

在这个例子中，我们传递给 `can` 中间件 2 个参数。第一个是需要授权的动作的名称，第二个是我们希望传递给策略方法的路由参数。这里因为使用了 [隐式模型绑定](/docs/{{version}}/routing#implicit-binding)，一个 `Post` 会被传递给策略方法。如果用户不被授权访问指定的动作，这个中间件会生成带有 `403` 状态码的 HTTP 响应。

#### 不需要指定模型的动作

同样的，一些动作，比如 `create`，并不需要指定模型实例。在这种情况下，可传递一个类名给中间件。当授权动作时，这个类名将被用来判断使用哪个策略：

    Route::post('/post', function () {
        // 当前用户可以创建博客...
    })->middleware('can:create,App\Post');

<a name="via-controller-helpers"></a>
### 通过控制器辅助函数

除了在 `User` 模型中提供辅助方法外，Laravel 也为所有继承了 `App\Http\Controllers\Controller` 基类的控制器提供了一个有用的 `authorize` 方法。和 `can` 方法类似，这个方法接收需要授权的动作和相关的模型作为参数。如果动作不被授权，`authorize` 方法会抛出 `Illuminate\Auth\Access\AuthorizationException` 异常，然后被 Laravel 默认的异常处理器转化为带有 `403` 状态码的 HTTP 响应：

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * 更新给定博客
         *
         * @param  Request  $request
         * @param  Post  $post
         * @return Response
         */
        public function update(Request $request, Post $post)
        {
            $this->authorize('update', $post);

            // 当前用户可以更新博客...
        }
    }

#### 不需要指定模型的动作

和之前讨论的一样，一些动作，比如 `create`，并不需要指定模型实例。在这种情况下，可传递一个类名给 `authorize` 方法。当授权动作时，这个类名将被用来判断使用哪个策略：

    /**
     * 新建博客
     *
     * @param  Request  $request
     * @return Response
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);

        // 当前用户可以新建博客...
    }

<a name="via-blade-templates"></a>
### 通过 Blade 模板

当编写 Blade 模板时，你可能希望页面的指定部分只展示给允许授权访问给定动作的用户。例如，你可能希望只展示更新表单给有权更新博客的用户。这种情况下，你可以直接使用 `@can` 和 `@cannot` 指令。

    @can('update', $post)
        <!-- 当前用户可以更新博客 -->
    @endcan

    @cannot('update', $post)
        <!-- 当前用户不可以更新博客 -->
    @endcannot

这些指令在编写 `@if` 和 `@unless` 时提供了方便的缩写。`@can` 和 `@cannot` 各自转化为如下声明：

    @if (Auth::user()->can('update', $post))
        <!-- 当前用户可以更新博客 -->
    @endif

    @unless (Auth::user()->can('update', $post))
        <!-- 当前用户不可以更新博客 -->
    @endunless

#### 不需要指定模型的动作

和大部分其他的授权方法类似，当动作不需要模型实例时，你可以传递一个类名给 `@can` 和 `@cannot` 指令：

    @can('create', Post::class)
        <!-- 当前用户可以新建博客 -->
    @endcan

    @cannot('create', Post::class)
        <!-- 当前用户不可以新建博客 -->
    @endcannot



--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/2752/laravel-53-document-translation-completed)。
> 
> 文档永久地址： http://d.laravel-china.org