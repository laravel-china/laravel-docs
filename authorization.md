# 用户授权

- [简介](#introduction)
- [定义权限](#defining-abilities)
- [检查权限](#checking-abilities)
	- [通过 Gate Facade](#via-the-gate-facade)
	- [通过用户模型](#via-the-user-model)
	- [使用 Blade 模板](#within-blade-templates)
    - [使用表单请求](#within-form-requests)
- [授权策略](#policies)
	- [创建授权策略](#creating-policies)
	- [编写授权策略](#writing-policies)
	- [检查授权策略](#checking-policies)
- [控制器授权](#controller-authorization)

<a name="introduction"></a>
## 简介

除了内置提供的[用户认证](/docs/{{version}}/authentication)服务外，Laravel 还提供了用户授权和资源访问控制的方案。有很多种方法与辅助函数能帮你处理授权逻辑，在本文档中我们将会涵盖每一种方式。

> **注意：**授权在 Laravel 5.1.11 被加入，请在集成这些功能前参考[升级导引](/docs/{{version}}/upgrade)。

<a name="defining-abilities"></a>
## 定义权限

判断一个用户是否允许运行特定行为，最简单的方式就是使用 `Illuminate\Auth\Access\Gate` 类定义「权限」。可以在 `AuthServiceProvider` 文件中定义应用程序中的所有权限。举个例子，我们需要定义一个 `update-post` 的权限，需要判断目前的 `User` 及 `Post` [模型](/docs/{{version}}/eloquent) 是否有所属关系，我们会判断用户的 `id` 与文章的 `user_id` 是否相符：

	<?php

	namespace App\Providers;

	use Illuminate\Contracts\Auth\Access\Gate as GateContract;
	use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

	class AuthServiceProvider extends ServiceProvider
	{
	    /**
	     * 注册任何应用程序的认证或授权服务。
	     *
	     * @param  \Illuminate\Contracts\Auth\Access\Gate  $gate
	     * @return void
	     */
	    public function boot(GateContract $gate)
	    {
	        $this->registerPolicies($gate);

	        $gate->define('update-post', function ($user, $post) {
	        	return $user->id === $post->user_id;
	        });
	    }
	}

注意，我们并不会检查当给定的 `$user` 不是 `NULL`，未登录用户或是没有用 `forUser` 方法指定的用户，`Gate` 会自动为**所有权限**返回 `false`。

#### 基于类的权限

除了注册`闭包`作为授权的回调，你可以通过传递包含 `类名称` 及 `方法` 的字符串来注册类方法，该类会通过[服务容器](/docs/{{version}}/container)被解析：

    $gate->define('update-post', 'Class@method');

<a name="intercepting-all-checks"></a>
<a name="intercepting-authorization-checks"></a>
#### 拦截授权检查

有时你希望赋予所有权限给指定用户，如管理员拥有所有权限，可以使用 `before` 方法来定义所有授权检查前会被运行的回调：

    $gate->before(function ($user, $ability) {
        if ($user->isSuperAdmin()) {
            return true;
        }
    });

如果 `before` 的回调返回一个非 null 的结果，则该结果会被作为检查的结果。

你还可以使用 `after` 方法定义一个当所有授权检查后会被运行的回调。但是，你不应该修改 `after` 回调中授权检查的结果：

    $gate->after(function ($user, $ability, $result, $arguments) {
        //
    });

<a name="checking-abilities"></a>
## 检查权限

<a name="via-the-gate-facade"></a>
### 通过 Gate Facade

一旦权限被定义后，我们可以使用不同方式来做「权限检查」。首先，我们可以使用 `Gate` [facade](/docs/{{version}}/facades) 的 `check`、`allows` 或 `denies` 方法。所有的这些方法会取得权限的名称及参数，并会被传递至权限的回调中。你**不**需要传递目前的用户至该方法，因为 `Gate` 会自动载入当然登录用户，所以，当通过我们前面定义的 `update-post` 权限进行检查时，只需传递一个 `Post` 实例至 `denies` 方法即可：

    <?php

    namespace App\Http\Controllers;

    use Gate;
    use App\User;
    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * 更新给定的文章。
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
        	$post = Post::findOrFail($id);

        	if (Gate::denies('update-post', $post)) {
        		abort(403);
        	}

        	// 更新文章...
        }
    }

`allows` 方法只是简单的将 `denies` 方法给颠倒过来，当行为授权成功时候会返回 `true`。`check` 方法则是 `allows` 方法的别名。

#### 检查指定用户的权限

如果你想检查 **除了当前登录用户外的其他用户** 是否有给定的权限，你可以使用 `forUser` 方法来指定：

	if (Gate::forUser($user)->allows('update-post', $post)) {
		//
	}

#### 传递多个参数

当然，权限的回调可以传递多个参数：

	Gate::define('delete-comment', function ($user, $post, $comment) {
		//
	});

如果你的权限需要多个参数，只要简单的传递一个数组作为 `Gate` 方法的参数：

	if (Gate::allows('delete-comment', [$post, $comment])) {
		//
	}

<a name="via-the-user-model"></a>
### 通过用户模型

另外，你也可以通过 `User` 模型的实例检查权限。默认情况下，Laravel 的 `App\User` 模型使用了 `Authorizable` trait，它提供了两个方法：`can` 及 `cannot`。这些方法使用起来相似于 `Gate` facade 提供的 `allows` 与 `denies` 方法。所以，使用我们之前的例子，我们可以将代码改成如下：

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * 更新给定的文章。
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  int  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
        	$post = Post::findOrFail($id);

        	if ($request->user()->cannot('update-post', $post)) {
        		abort(403);
        	}

        	// 更新文章...
        }
    }

当然，`can` 方法只是简单的将 `cannot` 方法给颠倒过来：

	if ($request->user()->can('update-post', $post)) {
		// 更新文章...
	}

<a name="within-blade-templates"></a>
### 使用 Blade 模板

在 Blade 模板中，你还可以使用 `@can` 命令来快速检查当前登录用户是否有给定的权限。例如：

	<a href="/post/{{ $post->id }}">查看文章</a>

	@can('update-post', $post)
		<a href="/post/{{ $post->id }}/edit">编辑文章</a>
	@endcan

你也可以将 `@else` 命令结合 `@can` 命令：

	@can('update-post', $post)
		<!-- 目前的用户可以更新文章 -->
	@else
		<!-- 目前的用户不可以更新文章 -->
	@endcan

<a name="within-form-requests"></a>
### 使用表单请求

你也可以在 [表单请求](/docs/{{version}}/validation#form-request-validation) 的 `authorize` 方法中采用你的 `Gate` 定义的权限。举个例子：

    /**
     * 判断当用户已被授权并发送此请求。
     *
     * @return bool
     */
    public function authorize()
    {
        $postId = $this->route('post');

        return Gate::allows('update', Post::findOrFail($postId));
    }

<a name="policies"></a>
## 授权策略

<a name="creating-policies"></a>
### 创建授权策略

在大型应用程序中，把你所有的授权逻辑定义在 `AuthServiceProvider` 中可能成为累赘，你可以切分你的授权逻辑至「授权策略」类。授权策略是简单的 PHP 类，并基于授权的资源将授权逻辑进行分组。

首先，让我们产生一个授权策略来管理 `Post` 模型的授权。你可以通过 `make:policy` [artisan 命令](/docs/{{version}}/artisan)产生一个授权策略。产生的授权策略会被放置于 `app/Policies` 目录中：

	php artisan make:policy PostPolicy

#### 注册授权策略

一旦该授权策略存在，我们需要将它与 `Gate` 类进行注册。`AuthServiceProvider` 包含了一个 `policies` 属性，将各种模型对应至管理它们的授权策略。所以，我们需要指定 `Post` 模型的授权策略是 `PostPilicy` 类：

	<?php

	namespace App\Providers;

	use App\Post;
	use App\Policies\PostPolicy;
	use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

	class AuthServiceProvider extends ServiceProvider
	{
	    /**
	     * 应用程序的授权策略对应。
	     *
	     * @var array
	     */
	    protected $policies = [
	        Post::class => PostPolicy::class,
	    ];

	    /**
	     * 注册任何应用程序的认证或授权服务。
	     *
	     * @param  \Illuminate\Contracts\Auth\Access\Gate  $gate
	     * @return void
	     */
	    public function boot(GateContract $gate)
	    {
	        $this->registerPolicies($gate);
	    }
	}

<a name="writing-policies"></a>
### 编写授权策略

一旦授权策略被产生且注册，我们可以为每个权限的授权增加方法。例如，让我们在 `PostPolicy` 中定义一个 `update` 方法，它会判断给定的 `User` 是否可以「更新」一条 `Post`：

	<?php

	namespace App\Policies;

	use App\User;
	use App\Post;

	class PostPolicy
	{
		/**
		 * 判断给定的文章是否可以被该用户更新。
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

你可以接着在此授权策略定义额外的方法，作为各种权限需要的授权。例如，你可以定义 `show`、`destroy` 或 `addComment` 方法来授权 `Post` 的多种行为。

> **注意：**所有授权策略会通过 Laravel [服务容器](/docs/{{version}}/container)解析，意指你可以在授权策略的构造器对任何需要的依赖使用类型提示，它们将会被自动注入。

#### 拦截所有检查

有时，你可能希望在授权策略赋予所有权限给指定用户。对于这种情况，只要在授权策略中定义一个 `before` 方法。授权策略的此方法会在其他所有授权检查前被运行：

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

如果 `before` 的回调返回一个非 null 的结果，则该结果会被作为检查的结果。

<a name="checking-policies"></a>
### 检查授权策略

授权策略方法的调用方式和基于授权的回调`闭包`是完全相同的。你可以使用`Gate` facade、`User` 模型、`@can` Blade 命令或是 `policy` 辅助方法。

#### 通过 Gate Facade

`Gate` 会通过检查传递给该类方法的参数自动的判断该使用授权策略。所以，如果我们传递一个 `Post` 实例至 `denies` 方法，`Gate` 会采用对应的 `PostPolicy` 来授权行为：

    <?php

    namespace App\Http\Controllers;

    use Gate;
    use App\User;
    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * 更新给定的文章。
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
        	$post = Post::findOrFail($id);

        	if (Gate::denies('update', $post)) {
        		abort(403);
        	}

        	// 更新文章...
        }
    }

#### 通过用户模型

`User` 模型的 `can` 与 `cannot` 方法也会自动采用给定参数可用的授权策略。此方法提供一个简单的方式在应用程序中为任何取得到的 `User` 实例授权行为：

	if ($user->can('update', $post)) {
		//
	}

	if ($user->cannot('update', $post)) {
		//
	}

#### 使用 Blade 模板

同样的，`@can` Blade 命令会采用给定参数可用的授权。

	@can('update', $post)
		<!-- 目前的用户可以更新文章 -->
	@endcan

#### 通过授权策略辅助方法

全域的 `policy` 辅助函数可以被用于为给定的类实例取得 `Policy` 类。例如，我们可以传递一个 `Post` 实例至 `policy` 辅助方法，取得对应的 `PostPolicy` 类实例：

	if (policy($post)->update($user, $post)) {
		//
	}

<a name="controller-authorization"></a>
## 控制器授权

默认的，`App\Http\Controllers\Controller` 类包含了 Laravel 使用的 `AuthorizesRequests` trait。此 trait 提供了 `authorize` 方法，它可以被用于快速授权一个给定的行为，当无权限运行该行为时会抛出 `HttpException`。

`authorize` 方法与其他授权方法共用了同样的特征，像是 `Gate::allows` 与 `$user->can()`。所以，让我们使用 `authorize` 方法来快速授权一个请求以更新一条 `Post`：

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * 更新给定的文章。
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
        	$post = Post::findOrFail($id);

        	$this->authorize('update', $post);

        	// 更新文章...
        }
    }

如果该行为被授权了，控制器将会继续正常运行；但是，如果 `authorize` 方法判断没有权限运行该行为，那么将会自动产生一个带有 `403 Not Authorized` 状态码的 HTTP 回应并抛出异常。如你所见，`authorize` 方法是进行授权行为或抛出异常简单、快速的方法，只使用了一行的程序。

`AuthorizesRequests` trait 也提供了 `authorizeForUser` 方法来为非目前已认证的用户授权行为：

	$this->authorizeForUser($user, 'update', $post);

#### 自动判断授权策略方法

通常，一个授权策略的方法会对应至一个控制器方法。以下方的 `update` 方法为例，控制器方法及授权策略方法会共用相同的名称：`update`。

因此，Laravel 让你能够简单的传递实例参数至 `authorize` 方法，基于被调用的函数名称，自动判断出应该授权的权限。在本例中，因为 `authorize` 被控制器的 `update` 方法调用，所以也会调用 `PostPolicy` 中的 `update` 方法：

    /**
     * 更新给定的文章。
     *
     * @param  int  $id
     * @return Response
     */
    public function update($id)
    {
    	$post = Post::findOrFail($id);

    	$this->authorize($post);

    	// 更新文章...
    }
