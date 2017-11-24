# Laravel 升级索引

- [从 5.4 升级至 5.5.0](#upgrade-5.5.0)

<a name="upgrade-5.5.0"></a>

## 从 5.4 升级到 5.5.0

#### 预计升级耗时：1小时

> {note}我们尽量记录每一个可能的破坏性变化。但因为其中一些不兼容变更只存在于框架很不起眼的地方，事实上只有一小部分可能会影响到你的应用程序。

### 更新依赖

在 `composer.json` 文件中将 `laravel/framework` 更新为 `5.5.*` 。 此外，你还应该将 `phpunit/phpunit` 依赖关系更新到 `~6.0`。

> {tip} 如果你是通过使用 `laravel new` 来安装 Laravel 程序，则应该使用命令 `composer global update` 来更新 Laravel 安装程序包。

#### Laravel Dusk

Laravel Dusk `2.0.0` 已经发布，该版本同时兼容 Laravel 5.5 和 Chrome 的 headless 模式测试。

#### Pusher

Pusher 事件广播驱动现在需要 `~3.0`  版本的 Pusher SDK。

### Artisan

#### `fire` 方法

该方法已重命名为 `handle` 方法。

#### `optimize` 命令

随着对 PHP 操作码缓存的最新改进，不再需要优化 Artisan 命令。你应该从部署脚本中删除对此命令的任何引用，因为它在未来的 Laravel 版本中会被删除。

### 用户授权

#### `authorizeResouce` 控制器方法

当将一个大驼峰命名的模型名称传递给 `authorizeResource` 方法时，为了和资源控制器的行为相匹配，生成的路由会用蛇形命名法来命名。

#### `before` 策略方法

如果类中不包含与给定名称相匹配的方法，则不会调用策略类的 `before` 方法。

### 缓存

#### 数据库驱动

如果你正在使用数据库缓存驱动，那在你第一次部署升级至 Laravel 5.5时，应该先运行 `php artisan cache:clear` 命令。

### Eloquent

#### `belongsToMany` 方法

如果你在 Eloquent 模型中重写了 `belongsToMany` 方法，应该更新方法签名来映射新增的参数：

    /**
     * 定义多对多关系。
     *
     * @param  string  $related
     * @param  string  $table
     * @param  string  $foreignPivotKey
     * @param  string  $relatedPivotKey
     * @param  string  $parentKey
     * @param  string  $relatedKey
     * @param  string  $relation
     * @return \Illuminate\Database\Eloquent\Relations\BelongsToMany
     */
    public function belongsToMany($related, $table = null, $foreignPivotKey = null,
                                  $relatedPivotKey = null,$parentKey = null,
                                  $relatedKey = null, $relation = null)
    {
        //
    }

#### 模型的 `is` 方法

如果你重写了 Eloquent 模型的 `is` 方法 ，则应该从该方法中删除 `Model` 的类型提示。这将允许 `is` 方法接受 `null` 作为参数。

    /**
     * 确定两个模型是否具有相同的ID并且属于同一个表。
     *
     * @param  \Illuminate\Database\Eloquent\Model|null  $model
     * @return bool
     */
    public function is($model)
    {
        //
    }

#### 模型 `$events` 属性

模型上的 `$events` 属性应该重命名为 `$dispatchesEvents`。由于大量用户需要定义事件关系，导致与旧属性名称的冲突，因此进行了更改。

#### 中间表 `$parent` 属性

`Illuminate\Database\Eloquent\Relations\Pivot` 类中受保护的 `$parent` 属性已被重命名为 `$pivotParent` 。

#### 关联 `create` 方法

`BelongsToMany`、`HasOneOrMany` 以及 `MorphOneOrMany` 类中的 `create` 方法已被修改为 `$attributes` 参数提供默认值。如果你重写了这些方法，你应该更新你的签名来匹配新的定义。

    public function create(array $attributes = [])
    {
        //
    }

#### 软删除模型

删除 「软删除」 模型时，模型上的 `exists` 属性将保持为 `true` 。

#### `withCount` 列格式化

当使用别名时，`withCount` 方法将不再自动将 `_count` 附加到生成的列名称上。例如，在 Laravel 5.4 中，以下查询会将 `bar_count` 列添加到查询中：

    $users = User::withCount('foo as bar')->get();

但是在 Laravel 5.5 中，别名将严格按照给定的方式使用。如果要将 `_count` 追加到结果列，则必须在定义别名时指定该后缀：

    $users = User::withCount('foo as bar_count')->get();

### 异常格式

在 Laravel 5.5 中，所有的异常（包括验证异常）都被异常处理程序转换成 HTTP 响应。另外，验证错误默认返回的 JSON 格式已经更改。新格式遵循以下规则：

    {
        "message": "The given data was invalid.",
        "errors": {
            "field-1": [
                "Error 1",
                "Error 2"
            ],
            "field-2": [
                "Error 1",
                "Error 2"
            ],
        }
    }

但是，如果你想沿用 Laravel 5.4 错误提示的  JSON 格式，则可以将以下方法添加到 `App\Exceptions\Handler` 类中：

    use Illuminate\Validation\ValidationException;

    /**
     * 将验证异常转换成 JSON 响应
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Validation\ValidationException  $exception
     * @return \Illuminate\Http\JsonResponse
     */
    protected function invalidJson($request, ValidationException $exception)
    {
        return response()->json($exception->errors(), $exception->status);
    }

#### JSON 身份验证尝试

此更改也会影响通过 JSON 进行的身份验证尝试的验证错误返回的格式。在 Laravel 5.5 中，身份验证失败的 JSON 将按照上述新的格式约定返回错误消息。

#### 表单请求注意事项

现在自定义了单个表单请求的响应格式，应重写该表单请求的 `failedValidation` 方法 ，并抛出一个包含自定义响应的 `HttpResponseException` 实例。

    use Illuminate\Http\Exceptions\HttpResponseException;

    /**
     * 处理失败的验证尝试。
     *
     * @param  \Illuminate\Contracts\Validation\Validator  $validator
     * @return void
     *
     * @throws \Illuminate\Validation\ValidationException
     */
    protected function failedValidation(Validator $validator)
    {
        throw new HttpResponseException(response()->json(..., 422));
    }

### 文件系统

#### `files` 方法

`files` 方法与 `allFiles` 方法类似，会返回一个 `SqlFileInfo` 对象的数组。之前的 `files` 方法会返回一个字符串路径名的数组。

### 邮件

#### 未使用的参数

未使用的 `$data` 和 `$callback` 参数已从 `Illuminate\Contracts\Mail\MailQueue` 契约的 `queue` 和 `later` 方法中删除

    /**
     * 在队列中发送新的电子邮件
     *
     * @param  string|array|MailableContract  $view
     * @param  string  $queue
     * @return mixed
     */
    public function queue($view, $queue = null);

    /**
     * n 秒后在队列中发送新的电子邮件
     *
     * @param  \DateTimeInterface|\DateInterval|int  $delay
     * @param  string|array|MailableContract  $view
     * @param  string  $queue
     * @return mixed
     */
    public function later($delay, $view, $queue = null);

### 请求

#### `has` 方法

`$request->has` 方法现在对于空字符串和 `null` 将返回 `true`。新添加的 `$request->filled` 方法提供了之前的 `has` 方法的行为。

#### `intersect` 方法

`intersect` 方法已被移除。你只需在调用 `$request->only` 时使用 `array_filter` 来代替该方法：

    return array_filter($request->only('foo'));

#### `only` 方法

`only` 方法现在只会返回请求中实际存在的属性。如果你想保留该方法的旧功能，可以使用 `all` 方法来替代。

    return $request->all('foo');

#### 辅助函数 `request()`

辅助函数 `request` 将不再检索嵌套的键。如果有需要，你可以使用 `request` 中的 `input` 方法来达成此目的：

    return request()->input('filters.date');

### 测试

#### 认证声明

一些认证断言被重命名为与框架的其他断言更好的一致性：

<!-- <div class="content-list" markdown="1"> -->
- `seeIsAuthenticated` 被重命名为 `assertAuthenticated` ；
- `dontSeeIsAuthenticated` 被重命名为 `assertGuest` ；
- `seeIsAuthenticatedAs` 被重命名为 `assertAuthenticatedAs` ；
- `seeCredentials` 被重命名为 `assertCredentials` ；
- `dontSeeCredentials` 被重命名为 `assertInvalidCredentials` 。
  <!-- </div> -->

#### 伪造邮件

如果你使用伪造 Mail 来请求中是否有可用的**队列** ，则应该使用 `Mail::assertQueued` 来代替 `Mail::assertSent` 。 这种区别允许你明确声明邮件已在队列中等待发送，而不是在请求期间发送。

### 翻译

#### `LoaderInterface`

`Illuminate\Translation\LoaderInterface` 该接口已被移动至 `Illuminate\Contracts\Translation\Loader`。

### 验证

#### 验证方法

所有的验证器的验证方法已由 `protected` 改为 `public` 。

### 视图

#### 动态的 「with」 变量

当允许动态的 `__call` 方法与视图共享变量时，该变量将会自动使用驼峰式命名。举例如下：

    return view('pool')->withMaximumVotes(100);

`maximumVotes` 变量可以在模板中访问，如下所示：

    {{ $maximumVotes }}
### 其他

你还可以查看 `laravel/laravel` 在 GitHub 中的更改。虽然这些更改不是必需的，但你可能希望将这些文件与应用程序保持同步。本升级指南中将介绍其中一些更改，而其他的则不会被介绍，例如更改配置文件或注释。你可以使用 [GitHub 的比较工具](https://github.com/laravel/laravel/compare/5.4...master) 轻松查看你在意的变更的内容。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@我做我的梦](https://laravel-china.org/users/18485)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/18485_1503368993.jpeg?imageView2/1/w/100/h/100">  |  翻译  | A new to Laravel，and hope Laravel is the last framework for me. |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  |  Review  | Stay Hungry. Stay Foolish. |


---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
