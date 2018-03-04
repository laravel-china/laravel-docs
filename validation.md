# Laravel 的表单验证机制详解

- [简介](#introduction)
- [快速验证](#validation-quickstart)
    - [定义路由](#quick-defining-the-routes)
    - [创建控制器](#quick-creating-the-controller)
    - [编写验证逻辑](#quick-writing-the-validation-logic)
    - [显示验证错误](#quick-displaying-the-validation-errors)
    - [可选字段上的注意事项](#a-note-on-optional-fields)
- [表单请求验证](#form-request-validation)
    - [创建表单请求](#creating-form-requests)
    - [授权表单请求](#authorizing-form-requests)
    - [自定义错误消息](#customizing-the-error-messages)
- [手动创建验证器](#manually-creating-validators)
    - [自动重定向](#automatic-redirection)
    - [命名错误包](#named-error-bags)
    - [验证后钩子](#after-validation-hook)
- [处理错误消息](#working-with-error-messages)
    - [自定义错误消息](#custom-error-messages)
- [可用的验证规则](#available-validation-rules)
- [按条件增加规则](#conditionally-adding-rules)
- [验证数组](#validating-arrays)
- [自定义验证规则](#custom-validation-rules)
    - [使用规则对象](#using-rule-objects)
    - [使用扩展](#using-extensions)

<a name="introduction"></a>
## 简介

Laravel 提供了几种不同的方法来验证传入应用程序的数据。默认情况下，Laravel 的控制器基类使用 `ValidatesRequests` Trait，它提供了一种方便的方法使用各种强大的验证规则来验证传入的 HTTP 请求。

<a name="validation-quickstart"></a>
## 快速验证

为了解 Laravel 强大的验证功能，我们先来看看一个完整的验证表单并返回错误消息的示例。

<a name="quick-defining-the-routes"></a>
### 定义路由

首先，假设我们在 `routes/web.php` 文件中定义了以下路由：

    Route::get('post/create', 'PostController@create');

    Route::post('post', 'PostController@store');

`GET` 路由用来显示一个供用户创建新的博客文章的表单，`POST` 路由则是会将新的博客文章保存到数据库。

<a name="quick-creating-the-controller"></a>
### 创建控制器

下一步，我们来看一个处理这些路由的控制器。我们将 `store` 方法置空：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * 显示创建博客文章的表单。
         *
         * @return Response
         */
        public function create()
        {
            return view('post.create');
        }

        /**
         * 保存一个新的博客文章。
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // 验证以及保存博客文章...
        }
    }

<a name="quick-writing-the-validation-logic"></a>
### 编写验证逻辑

现在我们准备开始在 `store` 方法中编写逻辑来验证新的博客文章。为此，我们将使用 `Illuminate\Http\Request` 对象提供的 `validate` 方法 。如果验证通过，你的代码就可以正常的运行。但是如果验证失败，就会抛出异常，并自动将对应的错误响应返回给用户。在典型的 HTTP 请求的情况下，会生成一个重定向响应，而对于 AJAX 请求则会发送 JSON 响应。

让我们接着回到 `store` 方法来深入理解 `validate` 方法：

    /**
     * 保存一篇新的博客文章。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $this->validate($request, [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        // 文章内容是符合规则的，存入数据库
    }

如你所见，我们将所需的验证规则传递至 `validate` 方法中。另外再提醒一次，如果验证失败，会自动生成一个对应的响应。如果验证通过，那我们的控制器将会继续正常运行。

#### 在第一次验证失败后停止

有时，你希望在某个属性第一次验证失败后停止运行验证规则。为了达到这个目的，附加 `bail` 规则到该属性：

    $this->validate($request, [
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
    ]);

在这个例子里，如果 `title` 字段没有通过 `unique`，那么不会检查 `max` 规则。规则会按照分配的顺序来验证。

#### 关于数组数据的注意事项

如果你的 HTTP 请求包含一个 「嵌套」 参数（即数组），那你可以在验证规则中通过 「点」 语法来指定这些参数。

    $this->validate($request, [
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>
### 显示验证错误

如果传入的请求参数未通过给定的验证规则呢？正如前面所提到的，Laravel 会自动把用户重定向到先前的位置。另外，所有的验证错误信息会被自动 [闪存至 session](/docs/{{version}}/session#flash-data)。

重申一次，我们不必在 `GET` 路由中将错误消息显式绑定到视图。因为 Lavarel 会检查在 Session 数据中的错误信息，并自动将其绑定到视图（如果存在）。而其中的变量 `$errors` 是 `Illuminate\Support\MessageBag` 的一个实例。要获取关于这个对象的更多信息，请 [查阅这个文档](#working-with-error-messages)。

> {tip} `$errors` 变量被由Web中间件组提供的 `Illuminate\View\Middleware\ShareErrorsFromSession` 中间件绑定到视图。**当这个中间件被应用后，在你的视图中就可以获取到 `$errors` 变量**，可以使一直假定 `$errors` 变量存在并且可以安全地使用。

所以，在我们的例子中，当验证失败的时候，用户将会被重定向到控制器的 `create` 方法，让我们在视图中显示错误信息：

    <!-- /resources/views/post/create.blade.php -->

    <h1>创建文章</h1>

    @if (count($errors) > 0)
        <div class="alert alert-danger">
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    <!-- 创建文章表单 -->

<a name="a-note-on-optional-fields"></a>
### 可选字段上的注意事项

默认情况下，Laravel 在你应用的全局中间件堆栈中包含在 `App\Http\Kernel` 类中的 `TrimStrings` 和 `ConvertEmptyStringsToNull` 中间件。因此，如果你不希望验证程序将 `null` 值视为无效的，那就将「可选」的请求字段标记为 `nullable`。

    $this->validate($request, [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

在这个例子里，我们指定 `publish_at` 字段可以为 `null` 或者一个有效的日期格式。如果 `nullable` 的修饰词没有被添加到规则定义中，验证器会认为 `null` 是一个无效的日期格式。

<a name="quick-ajax-requests-and-validation"></a>
#### AJAX 请求 & 验证

在这个例子中，我们使用传统的表单将数据发送到应用程序。但实际情况中，很多程序都会使用 AJAX 来发送请求。当我们对 AJAX 的请求中使用 `validate` 方法时，Laravel 并不会生成一个重定向响应，而是会生成一个包含所有验证错误信息的 JSON 响应。这个 JSON 响应会包含一个 HTTP 状态码 422 被发送出去。

<a name="form-request-validation"></a>
## 表单请求验证

<a name="creating-form-requests"></a>
### 创建表单请求

面对更复杂的验证情境中，你可以创建一个「表单请求」来处理更为复杂的逻辑。表单请求是包含验证逻辑的自定义请求类。可使用 Artisan 命令 `make:request` 来创建表单请求类：

    php artisan make:request StoreBlogPost

新生成的类保存在 `app/Http/Requests` 目录下。如果这个目录不存在，运行 `make:request` 命令时它会被创建出来。让我们添加一些验证规则到 `rules` 方法中：

    /**
     * 获取适用于请求的验证规则。
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

验证规则是如何运行的呢？你所需要做的就是在控制器方法中类型提示传入的请求。在调用控制器方法之前验证传入的表单请求，这意味着你不需要在控制器中写任何验证逻辑：

    /**
     * 保存传入的博客文章。
     *
     * @param  StoreBlogPost  $request
     * @return Response
     */
    public function store(StoreBlogPost $request)
    {
        // The incoming request is valid...
    }

如果验证失败，就会生成一个让用户返回到先前的位置的重定向响应。这些错误也会被闪存到 Session 中，以便这些错误都可以在页面中显示出来。如果传入的请求是 AJAX，会向用户返回具有 422 状态代码和验证错误信息的 JSON 数据的 HTTP 响应。

#### 添加表单请求后钩子

如果你想在表单请求「之后」添加钩子，可以使用 `withValidator` 方法。这个方法接收一个完整的验证构造器，允许你在验证结果返回之前调用任何方法：

    /**
     * 配置验证器实例。
     *
     * @param  \Illuminate\Validation\Validator  $validator
     * @return void
     */
    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add('field', 'Something is wrong with this field!');
            }
        });
    }

<a name="authorizing-form-requests"></a>
### 授权表单请求

表单请求类内也包含了 `authorize` 方法。在这个方法中，你可以检查经过身份验证的用户确定其是否具有更新给定资源的权限。比方说，你可以判断用户是否拥有更新文章评论的权限：

    /**
     * 判断用户是否有权限做出此请求。
     *
     * @return bool
     */
    public function authorize()
    {
        $comment = Comment::find($this->route('comment'));

        return $comment && $this->user()->can('update', $comment);
    }

由于所有的表单请求都是继承了 Laravel 中的请求基类，所以我们可以使用 `user` 方法去获取当前认证登录的用户。同时请注意上述例子中对 `route` 方法的调用。这个方法允许你在被调用的路由上获取其定义的 URI 参数，譬如下面例子中的 `{comment}` 参数：

    Route::post('comment/{comment}');

如果 `authorize` 方法返回 `false`，则会自动返回一个包含 403 状态码的 HTTP 响应，也不会运行控制器的方法。

如果你打算在应用程序的其它部分也能处理授权逻辑，只需从 `authorize` 方法返回 `true` ：

    /**
     * 判断用户是否有权限进行此请求。
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }


<a name="customizing-the-error-messages"></a>
### 自定义错误消息

你可以通过重写表单请求的 `messages` 方法来自定义错误消息。此方法应该如下所示返回属性/规则对数组及其对应错误消息：

    /**
     * 获取已定义的验证规则的错误消息。
     *
     * @return array
     */
    public function messages()
    {
        return [
            'title.required' => 'A title is required',
            'body.required'  => 'A message is required',
        ];
    }

<a name="manually-creating-validators"></a>
## 手动创建验证器

如果你不想要使用请求上使用 `validate` 方法，你可以通过 `validator` [Facade](/docs/{{version}}/facades) 手动创建一个验证器实例。用 Facade 上的 `make` 方法生成一个新的验证器实例：

    <?php

    namespace App\Http\Controllers;

    use Validator;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * 保存一篇新的博客文章。
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $validator = Validator::make($request->all(), [
                'title' => 'required|unique:posts|max:255',
                'body' => 'required',
            ]);

            if ($validator->fails()) {
                return redirect('post/create')
                            ->withErrors($validator)
                            ->withInput();
            }

            // 保存文章
        }
    }

传给 `make` 方法的第一个参数是要验证的数据。第二个参数则是该数据的验证规则。

如果请求没有通过验证，则可以使用 `withErrors` 方法把错误消息闪存到 Session。使用这个方法进行重定向之后，`$errors` 变量会自动与视图中共享，你可以将这些消息显示给用户。`withErrors` 方法接收验证器、`MessageBag` 或 PHP `array`。

<a name="automatic-redirection"></a>
### 自动重定向

如果想手动创建验证器实例，又想利用请求中 `validates` 方法提供的自动重定向，那么你可以在现有的验证器实例上调用 `validate` 方法。如果验证失败，用户会自动重定向，如果是 AJAX 请求，将会返回 JSON 格式的响应：

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validate();

<a name="named-error-bags"></a>
### 命名错误包

如果你一个页面中有多个表单，你可以命名错误信息的 `MessageBag` 来检索特定表单的错误消息。只需给 `withErrors` 方法传递一个名字作为第二个参数：

    return redirect('register')
                ->withErrors($validator, 'login');

然后你能从 `$errors` 变量中获取命名的 `MessageBag` 实例：

    {{ $errors->login->first('email') }}

<a name="after-validation-hook"></a>
### 验证后钩子

验证器还允许你添加在验证完成之后运行的回调函数。以便你进行进一步的验证，甚至是在消息集合中添加更多的错误消息。使用它只需在验证实例上使用 `after` 方法：

    $validator = Validator::make(...);

    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });

    if ($validator->fails()) {
        //
    }

<a name="working-with-error-messages"></a>
## 处理错误消息

在 `Validator` 实例上调用 `errors` 方法后，会得到一个 `Illuminate\Support\MessageBag` 实例，该实例具有各种方便的处理错误消息的方法。`$errors` 变量是自动提供给所有视图的 `MessageBag` 类的一个实例。

#### 查看特定字段的第一个错误消息

如果要查看特定字段的第一个错误消息，可以使用 `first` 方法：

    $errors = $validator->errors();

    echo $errors->first('email');

#### 查看特定字段的所有错误消息

如果你想以数组的形式获取指定字段的所有错误消息，则可以使用 `get` 方法：

    foreach ($errors->get('email') as $message) {
        //
    }

如果要验证表单的数组字段，你可以使用 `*` 来获取每个数组元素的所有错误消息：

    foreach ($errors->get('attachments.*') as $message) {
        //
    }

#### 查看所有字段的错误消息

如果你想要得到所有字段的错误消息，可以使用 `all` 方法：

    foreach ($errors->all() as $message) {
        //
    }

#### 判断特定字段是否含有错误消息

可以使用 `has` 方法来检测一个给定的字段是否存在错误消息：

    if ($errors->has('email')) {
        //
    }

<a name="custom-error-messages"></a>
### 自定义错误消息

如果有需要的话，你也可以自定义错误消息取代默认值进行验证。有几种方法可以指定自定义消息。首先，你可以将自定义消息作为第三个参数传递给 `Validator::make` 方法：

    $messages = [
        'required' => 'The :attribute field is required.',
    ];

    $validator = Validator::make($input, $rules, $messages);

在这个例子中，`:attribute` 占位符会被验证字段的实际名称取代。除此之外，你还可以在验证消息中使用其他占位符。例如：

    $messages = [
        'same'    => 'The :attribute and :other must match.',
        'size'    => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute must be between :min - :max.',
        'in'      => 'The :attribute must be one of the following types: :values',
    ];

#### 为给定属性指定自定义消息

有时候你可能只想为特定的字段自定义错误消息。只需在属性名称后使用「点」语法来指定验证的规则即可：

    $messages = [
        'email.required' => 'We need to know your e-mail address!',
    ];

<a name="localization"></a>
#### 在语言文件中指定自定义消息

现实中大多数情况下，我们可能不仅仅只是将自定义消息传递给 `Validator`，而是想要会使用不同的语言文件来指定自定义消息。实现它需要在 `resources/lang/xx/validation.php` 语言文件中将定制的消息添加到 `custom` 数组。

    'custom' => [
        'email' => [
            'required' => 'We need to know your e-mail address!',
        ],
    ],

#### 在语言文件中指定自定义属性

如果要使用自定义属性名称替换验证消息的 `:attribute` 部分，就在 `resources/lang/xx/validation.php` 语言文件的 `attributes` 数组中指定自定义名称：

    'attributes' => [
        'email' => 'email address',
    ],

<a name="available-validation-rules"></a>
## 可用的验证规则

以下是所有可用的验证规则及其功能的清单：

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">

[Accepted](#rule-accepted)
[Active URL](#rule-active-url)
[After (Date)](#rule-after)
[After Or Equal (Date)](#rule-after-or-equal)
[Alpha](#rule-alpha)
[Alpha Dash](#rule-alpha-dash)
[Alpha Numeric](#rule-alpha-num)
[Array](#rule-array)
[Before (Date)](#rule-before)
[Before Or Equal (Date)](#rule-before-or-equal)
[Between](#rule-between)
[Boolean](#rule-boolean)
[Confirmed](#rule-confirmed)
[Date](#rule-date)
[Date Equals](#rule-date-equals)
[Date Format](#rule-date-format)
[Different](#rule-different)
[Digits](#rule-digits)
[Digits Between](#rule-digits-between)
[Dimensions (Image Files)](#rule-dimensions)
[Distinct](#rule-distinct)
[E-Mail](#rule-email)
[Exists (Database)](#rule-exists)
[File](#rule-file)
[Filled](#rule-filled)
[Image (File)](#rule-image)
[In](#rule-in)
[In Array](#rule-in-array)
[Integer](#rule-integer)
[IP Address](#rule-ip)
[JSON](#rule-json)
[Max](#rule-max)
[MIME Types](#rule-mimetypes)
[MIME Type By File Extension](#rule-mimes)
[Min](#rule-min)
[Nullable](#rule-nullable)
[Not In](#rule-not-in)
[Numeric](#rule-numeric)
[Present](#rule-present)
[Regular Expression](#rule-regex)
[Required](#rule-required)
[Required If](#rule-required-if)
[Required Unless](#rule-required-unless)
[Required With](#rule-required-with)
[Required With All](#rule-required-with-all)
[Required Without](#rule-required-without)
[Required Without All](#rule-required-without-all)
[Same](#rule-same)
[Size](#rule-size)
[String](#rule-string)
[Timezone](#rule-timezone)
[Unique (Database)](#rule-unique)
[URL](#rule-url)

</div>

<a name="rule-accepted"></a>
#### accepted

验证的字段必须为 _yes_、 _on_、 _1_、或 _true_。这在确认「服务条款」是否同意时相当有用。

<a name="rule-active-url"></a>
#### active_url

相当于使用了 PHP 函数 `dns_get_record`，验证的字段必须具有有效的 A 或 AAAA 记录。

<a name="rule-after"></a>
#### after:_date_

验证的字段必须是给定日期后的值。这个日期将会通过 PHP 函数 `strtotime` 来验证。

    'start_date' => 'required|date|after:tomorrow'
你也可以指定其它的字段来比较日期：

    'finish_date' => 'required|date|after:start_date'

<a name="rule-after-or-equal"></a>
#### after\_or\_equal:_date_

验证的字段必须等于给定日期之后的值。更多信息请参见 [after](#rule-after) 规则。

<a name="rule-alpha"></a>
#### alpha

验证的字段必须完全是字母的字符。

<a name="rule-alpha-dash"></a>
#### alpha_dash

验证的字段可能具有字母、数字、破折号（ - ）以及下划线（ _ ）。

<a name="rule-alpha-num"></a>
#### alpha_num

验证的字段必须完全是字母、数字。

<a name="rule-array"></a>
#### array

验证的字段必须是一个 PHP 数组。

<a name="rule-before"></a>
#### before:_date_

验证的字段必须是给定日期之前的值。这个日期将会通过 PHP 函数 `strtotime` 来验证。

<a name="rule-before-or-equal"></a>
#### before\_or\_equal:_date_

验证的字段必须是给定日期之前或之前的值。这个日期将会使用 PHP 函数 `strtotime` 来验证。

<a name="rule-between"></a>
#### between:_min_,_max_

验证的字段的大小必须在给定的 _min_ 和 _max_ 之间。字符串、数字、数组或是文件大小的计算方式都用 [`size`](#rule-size) 方法进行评估。

<a name="rule-boolean"></a>
#### boolean

验证的字段必须能够被转换为布尔值。可接受的参数为 `true`、`false`、`1`、`0`、`"1"` 以及 `"0"`。

<a name="rule-confirmed"></a>
#### confirmed

验证的字段必须和 `foo_confirmation` 的字段值一致。例如，如果要验证的字段是 `password`，输入中必须存在匹配的 `password_confirmation` 字段。

<a name="rule-date"></a>
#### date

验证的字段值必须是通过 PHP 函数 `strtotime` 校验的有效日期。

<a name="rule-date-equals"></a>

#### date_equals:*date*

验证的字段必须等于给定的日期。该日期会被传递到 PHP 函数 `strtotime`。

<a name="rule-date-format"></a>
#### date_format:_format_

验证的字段必须与给定的格式相匹配。你应该只使用 `date` 或 `date_format` **其中一个**用于验证，而不应该同时使用两者。

<a name="rule-different"></a>
#### different:_field_

验证的字段值必须与字段 (_field_) 的值不同。

<a name="rule-digits"></a>
#### digits:_value_

验证的字段必须是数字，并且必须具有确切的值。

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

验证的字段的长度必须在给定的 _min_ 和 _max_ 之间。

<a name="rule-dimensions"></a>
#### dimensions

验证的文件必须是图片并且图片比例必须符合规则：

    'avatar' => 'dimensions:min_width=100,min_height=200'

可用的规则为： _min\_width_、 _max\_width_ 、 _min\_height_ 、 _max\_height_ 、 _width_ 、 _height_ 、 _ratio_。

比例应该使用宽度除以高度的方式来约束。这样可以通过 3/2 这样的语句或像 1.5 这样的浮点的约束：

    'avatar' => 'dimensions:ratio=3/2'
由于此规则需要多个参数，因此你可以 `Rule::dimensions` 方法来构造可读性高的规则：

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'avatar' => [
            'required',
            Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
        ],
    ]);

<a name="rule-distinct"></a>
#### distinct

验证数组时，指定的字段不能有任何重复值。

    'foo.*.id' => 'distinct'

<a name="rule-email"></a>
#### email

验证的字段必须符合 e-mail 地址格式。

<a name="rule-exists"></a>
#### exists:_table_,_column_

验证的字段必须存在于给定的数据库表中。

#### Exists 规则的基本使用方法

    'state' => 'exists:states'

#### 指定自定义字段名称

    'state' => 'exists:states,abbreviation'

如果你需要指定 `exists` 方法用来查询的数据库。你可以通过使用「点」语法将数据库的名称添加到数据表前面来实现这个目的：

    'email' => 'exists:connection.staff,email'

如果要自定义验证规则执行的查询，可以使用 `Rule` 类来定义规则。在这个例子中，我们使用数组指定验证规则，而不是使用 `|` 字符来分隔它们：

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::exists('staff')->where(function ($query) {
                $query->where('account_id', 1);
            }),
        ],
    ]);

<a name="rule-file"></a>
#### file

验证的字段必须是成功上传的文件。

<a name="rule-filled"></a>
#### filled

验证的字段在存在时不能为空。

<a name="rule-image"></a>
#### image

验证的文件必须是一个图像（ jpeg、png、bmp、gif、或 svg ）。

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

验证的字段必须包含在给定的值列表中。因为这个规则通常需要你 `implode` 一个数组，`Rule::in` 方法可以用来构造规则：

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'zones' => [
            'required',
            Rule::in(['first-zone', 'second-zone']),
        ],
    ]);

<a name="rule-in-array"></a>
#### in_array:_anotherfield_

验证的字段必须存在于另一个字段（anotherfield）的值中。

<a name="rule-integer"></a>
#### integer

验证的字段必须是整数。

<a name="rule-ip"></a>
#### ip

验证的字段必须是 IP 地址。

#### ipv4

验证的字段必须是 IPv4 地址。

#### ipv6

验证的字段必须是 IPv6 地址。

<a name="rule-json"></a>
#### json

验证的字段必须是有效的 JSON 字符串。

<a name="rule-max"></a>
#### max:_value_

验证中的字段必须小于或等于 _value_。字符串、数字、数组或是文件大小的计算方式都用 [`size`](#rule-size) 方法进行评估。

<a name="rule-mimetypes"></a>
#### mimetypes:_text/plain_,...

验证的文件必须与给定 MIME 类型之一匹配：

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

要确定上传文件的 MIME 类型，会读取文件的内容来判断 MIME 类型，这可能与客户端提供的 MIME 类型不同。

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

验证的文件必须具有与列出的其中一个扩展名相对应的 MIME 类型。

#### MIME 规则基本用法

    'photo' => 'mimes:jpeg,bmp,png'
即使你可能只需要验证指定扩展名，但此规则实际上会验证文件的 MIME 类型，其通过读取文件的内容以猜测它的 MIME 类型。

这个过程看起来只需要你指定扩展名，但实际上该规则是通过读取文件的内容并判断其 MIME 的类型来验证的。

可以在以下链接中找到完整的 MIME 类型列表及其相应的扩展名：

[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="rule-min"></a>
#### min:_value_

验证中的字段必须具有最小值。字符串、数字、数组或是文件大小的计算方式都用 [`size`](#rule-size) 方法进行评估。

<a name="rule-nullable"></a>
#### nullable

验证的字段可以为 `null`。这在验证基本数据类型时特别有用，例如可以包含空值的字符串和整数。

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

验证的字段不能包含在给定的值列表中。`Rule::notIn` 方法可以用来构建规则：

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'toppings' => [
            'required',
            Rule::notIn(['sprinkles', 'cherries']),
        ],
    ]);

<a name="rule-numeric"></a>
#### numeric

验证的字段必须是数字。

<a name="rule-present"></a>
#### present

验证的字段必须存在于输入数据中，但可以为空。

<a name="rule-regex"></a>
#### regex:_pattern_

验证的字段必须与给定的正则表达式匹配。

**注意**： 当使用 `regex` 规则时，你必须使用数组，而不是使用 `|` 分隔符，特别是如果正则表达式包含 `|` 字符。

<a name="rule-required"></a>
#### required

验证的字段必须存在于输入数据中，而不是空。如果满足以下条件之一，则字段被视为「空」：

<div class="content-list" markdown="1">

- 该值为 `null`.
- 该值为空字符串。
- 该值为空数组或空的 `可数` 对象。
- 该值为没有路径的上传文件。

</div>

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,...

如果指定的其它字段（ _anotherfield_ ）等于任何一个 _value_ 时，被验证的字段必须存在且不为空。

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

如果指定的其它字段（ _anotherfield_ ）等于任何一个 _value_ 时，被验证的字段不必存在。

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

只要在指定的其他字段中有任意一个字段存在时，被验证的字段就必须存在并且不能为空。

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

只有当所有的其他指定字段全部存在时，被验证的字段才必须存在并且不能为空。

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

只要在其他指定的字段中有任意一个字段不存在，被验证的字段就必须存在且不为空。

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

只有当所有的其他指定的字段都不存在时，被验证的字段才必须存在且不为空。

<a name="rule-same"></a>
#### same:_field_

给定字段必须与验证的字段匹配。

<a name="rule-size"></a>
#### size:_value_

验证的字段必须具有与给定值匹配的大小。对于字符串来说，_value_ 对应于字符数。对于数字来说，_value_ 对应于给定的整数值。对于数组来说， _size_ 对应的是数组的 `count` 值。对文件来说，_size_ 对应的是文件大小（单位 kb ）。

<a name="rule-string"></a>
#### string

验证的字段必须是字符串。如果要允许该字段的值为 `null` ，就将 `nullable` 规则附加到该字段中。

<a name="rule-timezone"></a>
#### timezone

验证的字段必须是有效的时区标识符，会根据 PHP 函数 `timezone_identifiers_list` 来判断。

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

验证的字段在给定的数据库表中必须是唯一的。如果没有指定 `column`，将会使用字段本身的名称。

**指定自定义字段名称：**

    'email' => 'unique:users,email_address'

**自定义数据库连接**

有时，你可能需要为验证程序创建的数据库查询设置自定义连接。上面的例子中，将 `unique：users` 设置为验证规则，等于使用默认数据库连接来查询数据库。如果要对其进行修改，请使用「点」语法指定连接和表名：

    'email' => 'unique:connection.users,email_address'

**强迫 Unique 规则忽略指定 ID：**

如果你想在进行字段唯一性验证时忽略指定 ID 。例如，在「更新个人资料」页面会包含用户名、邮箱和地点。这时你会想要验证更新的 E-mail 值是否唯一。如果用户仅更改了用户名字段而没有改 E-mail 字段，就不需要抛出验证错误，因为此用户已经是这个 E-mail 的拥有者了。

使用 `Rule` 类定义规则来指示验证器忽略用户的 ID。 这个例子中通过数组来指定验证规则，而不是使用 `|` 字符来分隔：

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::unique('users')->ignore($user->id),
        ],
    ]);

如果你的数据表使用的主键名称不是 `id`，那就在调用 `ignore` 方法时指定字段的名称：

    'email' => Rule::unique('users')->ignore($user->id, 'user_id')

**增加额外的 Where 语句：**

你也可以通过 `where` 方法指定额外的查询条件。例如，我们添加 `account_id` 为 `1` 的约束：

    'email' => Rule::unique('users')->where(function ($query) {
        $query->where('account_id', 1);
    })

<a name="rule-url"></a>
#### url

验证的字段必须是有效的 URL。

<a name="conditionally-adding-rules"></a>
## 按条件增加规则

#### 存在才验证

在某些情况下，只有在该字段存在于输入数组中时，才可以对字段执行验证检查。可通过增加 `sometimes` 到规则列表来实现：

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

在上面的例子中，`email` 字段只有在 `$data` 数组中存在时才会被验证。

> {tip} 如果你尝试验证应该始终存在但可能为空的字段，请查阅 [可选字段的注意事项](#a-note-on-optional-fields)。

#### 复杂的条件验证

有时候你可能需要增加基于更复杂的条件逻辑的验证规则。例如，你可以希望某个指定字段在另一个字段的值超过 100 时才为必填。或者当某个指定字段存在时，另外两个字段才能具有给定的值。增加这样的验证条件并不难。首先，使用静态规则创建一个 `Validator` 实例：

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

假设我们有一个专为游戏收藏家所设计的网页应用程序。如果游戏收藏家收藏超过一百款游戏，我们会希望他们来说明下为什么他们会拥有这么多游戏。比如说他们有可能经营了一家游戏分销商店，或者只是为了享受收集的乐趣。为了在特定条件下加入此验证需求，可以在 `Validator` 实例中使用 `sometimes` 方法。

    $v->sometimes('reason', 'required|max:500', function ($input) {
        return $input->games >= 100;
    });

传入 `sometimes` 方法的第一个参数是要用来验证的字段名称。第二个参数是我们想使用的验证规则。`闭包` 作为第三个参数传入，如果其返回 `true`，则额外的规则就会被加入。这个方法可以轻松地创建复杂的条件验证。你甚至可以一次对多个字段增加条件验证：

    $v->sometimes(['reason', 'cost'], 'required', function ($input) {
        return $input->games >= 100;
    });

> {tip} 传入 `闭包` 的 `$input` 参数是 `Illuminate\Support\Fluent` 的一个实例，可用来访问你的输入或文件对象。

<a name="validating-arrays"></a>
## 验证数组

验证表单的输入为数组的字段也不难。你可以使用「点」语法来验证数组中的属性。例如，如果传入的 HTTP 请求中包含 `photos[profile]` 字段，可以如下验证：

    $validator = Validator::make($request->all(), [
        'photos.profile' => 'required|image',
    ]);


你还可以验证数组中的每个元素。例如，要验证指定数组输入字段中的每一个 email 是唯一的，可以这么做：

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);

同理，你可以在语言文件定义验证信息时使用 `*` 字符，为基于数组的字段使用单个验证消息：

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],

<a name="custom-validation-rules"></a>
## 自定义验证规则

<a name="using-rule-objects"></a>
### 使用规则对象

Laravel 提供了许多有用的验证规则，同时也支持自定义规则。注册自定义验证规则的方法之一，就是使用规则对象。可以使用 Artisan 命令 `make:rule` 来生成新的规则对象。接下来，让我们用这个命令生成一个验证字符串是大写的规则。Laravel 会将新的规则存放在 `app/Rules` 目录中：

    php artisan make:rule Uppercase

一旦创建了规则，我们就可以定义它的行为。规则对象包含两个方法： `passes` 和 `message` 。 `passes` 方法接收属性值和名称，并根据属性值是否符合规则而返回 `true` 或者 `false`。 `message` 应返回验证失败时应使用的验证错误消息：

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;

    class Uppercase implements Rule
    {
        /**
         * 判断验证规则是否通过。
         *
         * @param  string  $attribute
         * @param  mixed  $value
         * @return bool
         */
        public function passes($attribute, $value)
        {
            return strtoupper($value) === $value;
        }

        /**
         * 获取验证错误信息。
         *
         * @return string
         */
        public function message()
        {
            return 'The :attribute must be uppercase.';
        }
    }

当然，如果你希望从翻译文件中返回一个错误信息，你可以从 `message` 方法中调用辅助函数 `trans`：

    /**
     * 获取验证错误信息。
     *
     * @return string
     */
    public function message()
    {
        return trans('validation.uppercase');
    }

一旦规则对象被定义好后，你可以通过将规则对象的实例传递给其他验证规则来将其附加到验证器：

    use App\Rules\Uppercase;

    $request->validate([
        'name' => ['required', new Uppercase],
    ]);

<a name="using-extensions"></a>
### 使用扩展

另外一个注册自定义验证规则的方法，就是使用 `Validator` [Facade](/docs/{{version}}/facades) 中的 `extend` 方法。让我们在 [服务提供器](/docs/{{version}}/providers) 中使用这个方法来注册自定义验证规则：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Validator;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 引导任何应用服务。
         *
         * @return void
         */
        public function boot()
        {
            Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
                return $value == 'foo';
            });
        }

        /**
         * 注册服务提供器。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

自定义的验证闭包接收四个参数：要被验证的属性名称 `$attribute`、属性的值 `$value`、传入验证规则的参数数组 `$parameters`、及 `Validator` 实例。

除了使用闭包，你也可以传入类和方法到 `extend` 方法中：

    Validator::extend('foo', 'FooValidator@validate');

#### 自定义错误消息

你还需要为自定义规则定义错误消息。这可以通过使用自定义内联消息数组或是在验证语言文件中加入新的规则来实现。此消息应该被放在数组的第一级，而不是被放在 `custom` 数组内，这是仅针对特定属性的错误消息:

    "foo" => "你的输入是无效的!",

    "accepted" => ":attribute 必须被接受。",

    // 其余的验证错误消息...

创建自定义验证规则时，可能需要为错误消息定义自定义替换占位符。你可以像上面所描述的那样通过 `Validator` Facade 来使用 `replacer` 方法创建一个自定义验证器。你可以在 [服务提供器](/docs/{{version}}/providers) 中的 `boot` 方法中执行此操作：

    /**
     * 引导任何应用服务。
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend(...);

        Validator::replacer('foo', function ($message, $attribute, $rule, $parameters) {
            return str_replace(...);
        });
    }

#### 隐式扩展

默认情况下，当所要验证的属性不存在或包含由 [`required`](#rule-required) 规则定义的空值时，将不会运行正常的验证规则（包括自定义扩展）。例如，[`unique`](#rule-unique) 规则不会针对 `null` 运行：

    $rules = ['name' => 'unique'];

    $input = ['name' => null];

    Validator::make($input, $rules)->passes(); // true
即使属性为空的规则也可以运行，该规则必须意味着该属性是必需的。要创建这样一个「隐式」扩展，可以使用 `Validator::extendImplicit()` 方法：

    Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });

> {note} 「隐式」扩展只 _暗示_ 该属性是必需的。是否使一个丢失或为空的属性无效主要取决于你。


## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| Cloes | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/6187_1477389867.jpg?imageView2/1/w/100/h/100"> | 翻译   | 我的[github](https://github.com/cloes) |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | Review | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
