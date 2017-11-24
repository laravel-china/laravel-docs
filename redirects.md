# Laravel 的 HTTP 重定向 Redirect

- [创建重定向](#creating-redirects)
- [重定向到命名路由](#redirecting-named-routes)
- [重定向到控制器动作](#redirecting-controller-actions)
- [闪存 Session 数据重定向](#redirecting-with-flashed-session-data)

<a name="creating-redirects"></a>
## 创建重定向

重定向响应是类 `Illuminate\Http\RedirectResponse` 的实例, 包含了重定向用户到其他 URL 所需要的合适头信息。有很多方式生成 `RedirectResponse` 实例。最简单的方法是使用全局的 `redirect` 辅助函数：

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

有时候你希望将用户重定向到他们的上一个访问位置，例如当提交的表单不合法时，你就可以通过全局的 `back` 辅助函数来这样做。 因为该特性使用了 [session](/docs/{{version}}/session)，请确保路由调用 `back` 函数时使用了 `web` 中间件组或者应用了全部的 session 中间件：

    Route::post('user/profile', function () {
        // Validate the request...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
## 重定向到命名路由

当你不带参数调用 `redirect` 辅助函数时，会返回一个 `Illuminate\Routing\Redirector` 实例，它允许你调用 `Redirector` 实例上的任何方法。例如，你可以这样使用 `route` 方法为命名路由生成一个 `RedirectResponse` ：

    return redirect()->route('login');

如果你的路由包含参数，你可以把它们当做第二参数传给 `route` 方法：

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### 通过 Eloquent 模型填充参数

如果你要携带一个从 Eloquent 模型中填充过来的 "ID" 参数进行重定向，你可以简单的把该模型本身传进去。 ID 会被自动解析：

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', [$user]);

如果你想自定义路由参数的值，你应该在 Eloquent 模型中重写 `getRouteKey` 方法：

    /**
     * Get the value of the model's route key.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
## 重定向到控制器动作

你也可以生成重定向到 [控制器动作](/docs/{{version}}/controllers)。要达到这个目的，传递控制器名和动作名到 `action` 方法即可。记住，你不需要指定完整的控制器命名空间，因为 Laravel 的 `RouteServiceProvider` 会自动设置基础的控制器命名空间：

    return redirect()->action('HomeController@index');

如果你的控制器路由需要参数，你可以把它们当做第二个参数传递给 `action` 方法：

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );

<a name="redirecting-with-flashed-session-data"></a>
## 闪存 Session 数据重定向

重定向到新的 URL 并且 [闪存数据到 session](/docs/{{version}}/session#flash-data) 常常在同时完成。 通常的，这会在你成功的执行一个动作、闪存消息到 session 后完成。方便起见，你可以创建一个 `RedirectResponse` 实例并在单个的、流畅的方法链上闪存数据到 session ：

    Route::post('user/profile', function () {
        // Update the user's profile...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

用户被重定向后，你可以从 [session](/docs/{{version}}/session) 中显示闪存消息。例如，使用 [Blade 语法](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@limxx](https://github.com/limxx)  | <img class="avatar-66 rm-style" src="https://avatars0.githubusercontent.com/u/16585030?v=4&s=400">  |  翻译  | Winter is coming. |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org