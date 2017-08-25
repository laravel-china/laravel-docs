# Laravel 升级索引

- [从 5.4 升级至 5.5.0](#upgrade-5.5.0)

<a name="upgrade-5.5.0"></a>

## 从 5.4 升级到 5.5.0

#### 预计升级耗时：1小时

> {note}我们尽量罗列出每一个不兼容的变更。但因为其中一些不兼容变更只存在于框架很不起眼的地方，事实上只有一小部分会真正影响到你的应用程序。

### 更新依赖

在 `composer.json` 文件中将您的 `laravel/framework` 更新为 `5.5.*` 。 此外, 您应该更新 `phpunit/phpunit` 依赖关系到 `~6.0`。

#### Laravel Dusk

Laravel Dusk `2.0.0` 已经发布，该版本同时兼容 Laravel 5.5 和 headless Chrome 模式的测试。

#### Pusher

Pusher 事件的广播驱动现在要求 Pusher SDK 的版本为 `~3.0`。

### Artisan

#### `fire` 方法

该方法已重命名为 `handle` 方法。

### 用户授权

#### `authorizeResouce` 控制器方法

当将一个复合的模型名称传递给 `authorizeResource` 方法时，路由将会以「蛇底法」（snake case）的方式对其进行分割，与资源控制器的行为相匹配。

#### `before` 策略方法

如果类中不包含名称与当前确认的功能名相同的方法， `before` 方法将不会被调用。

### 缓存

#### 数据库驱动

如果您正在使用数据库缓存驱动，在您第一次部署升级至 Laravel 5.5 的应用时，应先运行 `php artisan cache:clear` 命令。

### Eloquent

#### `belongsToMany` 方法

如果您在您的 Eloquent 模型中重写了 `belongsToMany` 方法，您应当更新您的方法签名来映射新增的参数：

    /**
     * 定义一个多对多的关系。
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

#### Model `is` 方法

如果您重写了 Eloquent 模型的 `is` 方法 ，您应该移除方法中 `Model` 的类型提示 。这样 `is` 方法就能将 `null` 作为参数接收。

    /**
     * 判断两个模型是否具有相同的ID且属于同一个表。
     *
     * @param  \Illuminate\Database\Eloquent\Model|null  $model
     * @return bool
     */
    public function is($model)
    {
        //
    }

#### Model `$events` 属性

由于大量的用户需要定义名为 `events` 的关联，这将与旧的属性名相冲突，所以模型中的 `$events` 属性已被重命名为 `$dispatchesEvents`。

#### Pivot `$parent` 属性

原本在 `Illuminate\Database\Eloquent\Relations\Pivot` 类中的 protected `$parent` 属性 已被重命名为 `$pivotParent` 。

#### Relationship `create` 方法

`BelongsToMany` ，`HasOneOrMany` ， 以及 `MorphOneOrMany` 类中， `create` 方法为 `$attributes` 参数提供了一个默认值。如果您重写了这些方法，您应更新您的签名以与新的定义相匹配。

    public function create(array $attributes = [])
    {
        //
    }

#### Soft Deleted 模型

当删除一个 「soft deleted」 模型时，该模型的 `exists` 属性将保持 `true` 。

#### `withCount` 列的格式化

当使用别名时，`withCount` 方法将不再自动在生成的列名中添加 `_count`。例如，在 Laravel 5.4 中，以下查询会将一个 `bar_count` 列添加到结果中：

    $users = User::withCount('foo as bar')->get();

但是在 Laravel 5.5 中，别名将严格按照给定的方式使用。如果您想在结果栏中添加 `_count` ，则必须在定义该别名时就指定该后缀：

    $users = User::withCount('foo as bar_count')->get();

### 异常处理格式

在 Laravel 5.5 中，所有的异常，包括验证异常，都被异常处理机制转换成 HTTP 响应。另外，JSON 验证错误的默认格式已经更改。新的格式符合以下规则：

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

如果您想沿用 Laravel 5.4 的 JSON 错误提示格式，只需将以下方法添加到 `App\Exceptions\Handler` 类中即可：

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

这一改变同样作用于通过 JSON 进行的失败的身份验证所返回的错误信息格式。在 Laravel 5.5 中，JSON 身份验证失败返回的错误信息将遵循上述的新格式。

#### Form 表单请求的注意事项

现在您在自定义个人表单请求的响应格式时，应重写该表单请求的 `failedValidation` 方法 ，并抛出一个包含您自定义响应的 `HttpResponseException` 例子。

    use Illuminate\Http\Exceptions\HttpResponseException;

    /**
     * 验证失败的处理尝试
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

`files` 方法现在与 `allFiles` 方法类似，会返回一个关于 `SqlFileInfo` 对象的数组。之前的 `files` 方法会返回一个关于路径名称的字符串数组。

### 邮件

#### 未用的参数

未使用的参数 `$data` 和 `$callback` 已从 `Illuminate\Contracts\Mail\MailQueue` 合约中的 `queue` 和 `later` 方法中移除。

    /**
     * 队列中加入一条新的需发送邮件
     *
     * @param  string|array|MailableContract  $view
     * @param  string  $queue
     * @return mixed
     */
    public function queue($view, $queue = null);

    /**
     * 队列中加入一条新的需在n秒后发送的邮件
     *
     * @param  \DateTimeInterface|\DateInterval|int  $delay
     * @param  string|array|MailableContract  $view
     * @param  string  $queue
     * @return mixed
     */
    public function later($delay, $view, $queue = null);

### 请求

#### `has` 方法

`$request->has` 方法现在对于空字符串和 `null` 将返回 `true` 。新添加了一个 `$request->filled` 方法提供了之前的 `has` 方法的功能。

#### `intersect` 方法

`intersect` 方法已被移除。您可以在调用 `array_filter` 时使用 `$request->only` 来代替该方法：

    return array_filter($request->only('foo'));

#### `only` 方法

`only` 方法现在只会返回请求的有效载荷中实际存在的属性。如果您想保留该方法的旧功能，可以使用 `all` 方法来替代。

    return $request->all('foo');

#### The `request()` Helper

`request` helper 将不再检索嵌套的密钥。如果有需要，可以使用 request 中的 `input` 方法来达成此目的：

    return request()->input('filters.date');

### 测试

#### 授权声明

为了更好的保证框架声明的一致性，一些授权声明被重命名了：

<!-- <div class="content-list" markdown="1"> -->
- `seeIsAuthenticated` 重命名为 `assertAuthenticated` ；
- `dontSeeIsAuthenticated` 重命名为 `assertGuest` ；
- `seeIsAuthenticatedAs` 重命名为 `assertAuthenticatedAs` ；
- `seeCredentials` 重命名为 `assertCredentials` ；
- `dontSeeCredentials` 重命名为 `assertInvalidCredentials` 。
<!-- </div> -->

#### 邮件伪造

如果您在请求期间使用 Mail Fake 来确定是否有可用的邮件 **队列** ，那么您现在应该使用 `Mail :: assertQueued` 而不是 `Mail :: assertSent` 。 这种区别允许您明确声明邮件已在后台排队等待发送，而不是在请求期间发送。

### 翻译

#### 关于 `LoaderInterface`

`Illuminate\Translation\LoaderInterface` 该接口已被移动至 `Illuminate\Contracts\Translation\Loader`。

### 验证

#### 验证方法

所有的验证方法已由 `protected` 改为 `public` 。

### 视图

#### 关于 "with" 后的动态变量

当允许动态的 `__call` 方法与一个视图共享变量时，该变量将会自动以驼峰法命名。举例如下：

    return view('pool')->withMaximumVotes(100);

变量 `maximumVotes` 在模板中将以该形式被调用：

    {{ $maximumVotes }}

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@我做我的梦](https://laravel-china.org/users/18485)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/18485_1503368993.jpeg?imageView2/1/w/100/h/100">  |  翻译  | A new to Laravel，and hope Laravel is the last framework for me. |
