# Laravel 的表单验证机制详解

- [简介](#introduction)
- [快速上手](#validation-quickstart)
    - [定义路由](#quick-defining-the-routes)
    - [创建控制器](#quick-creating-the-controller)
    - [编写验证逻辑](#quick-writing-the-validation-logic)
    - [显示验证错误](#quick-displaying-the-validation-errors)
    - [有关可选字段的注意事项](#a-note-on-optional-fields)
- [表单请求验证](#form-request-validation)
    - [创建表单请求](#creating-form-requests)
    - [授权表单请求](#authorizing-form-requests)
    - [自定义错误消息](#customizing-the-error-messages)
- [手动创建表单验证](#manually-creating-validators)
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
## Introduction

Laravel 提供了多种不同的验证方法来对应用程序传入的数据进行验证。默认情况下，Laravel 的基类控制器使用 `ValidatesRequests` Trait，它提供了方便的方法使用各种强大的验证规则来验证传入的 HTTP 请求数据。

<a name="validation-quickstart"></a>
## 快速上手

为了了解 Laravel 强大验证特性，我们先来看看一个完整的表单验证并返回错误消息的示例。

<a name="quick-defining-the-routes"></a>
### 定义路由

首先，我们假定在 `routes/web.php` 文件中定义了以下路由：

    Route::get('post/create', 'PostController@create');

    Route::post('post', 'PostController@store');

`GET` 路由会显示一个用于创建新博客文章的表单，`POST` 路由则会将新的博客文章保存到数据库。

<a name="quick-creating-the-controller"></a>
### 创建控制器

下一步，我们来看一个处理这些路由的简单的控制器。我们将 `store` 方法置空：

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
            // 验证以及保存博客发表文章...
        }
    }

<a name="quick-writing-the-validation-logic"></a>
### 编写验证逻辑

现在我们准备开始编写 `store` 逻辑方法来验证我们博客发布的新文章。我们将使用 `Illuminate\Http\Request` 对象提供的 `validate` 方法 。如果验证通过，你的代码就可以正常的运行。若验证失败，则会抛出异常错误消息并自动将一个对应的错误响应返回给用户。在一般的 HTTP 请求下，都会生成一个重定向响应，而对于 AJAX 请求则会发送 JSON 响应。

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

如你所见，我们将所需的验证规则传递至 `validate` 方法中。另外再提醒一次，如果验证失败，将会自动生成一个对应的响应。如果验证通过，那我们的控制器将会继续正常运行。

#### 在第一次验证失败后停止

有时，你希望在某个属性第一次验证失败后停止运行验证规则。为了达到这个目的，附加 `bail` 规则到该属性：

    $this->validate($request, [
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
    ]);

在这个例子里，如果 `title` 字段没有通过 `required` 的验证规则，那么 `unique` 这个规则将不会被检测了。将按规则被分配的顺序来验证规则。

#### 嵌套属性的注解

如果你的 HTTP 请求包含一个 「嵌套的」 参数，你可以在验证规则中通过 「点」 语法来指定这些参数。

    $this->validate($request, [
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>
### 显示验证错误

如果本次请求的参数未通过我们指定的验证规则呢？正如前面所提到的，Laravel 会自动把用户重定向到先前的位置。另外，所有的验证错误会被自动 [闪存至 session](/docs/{{version}}/session#flash-data)。

再者，请注意在 `GET` 路由中，我们无需显式的将错误信息和视图绑定起来。这是因为 Lavarel 会检查在 Session 数据中的错误信息，然后如果错误信息存在的话，则自动将它们与视图绑定起来。变量 `$errors` 会成为 `Illuminate\Support\MessageBag` 的一个实例对象。要获取关于这个对象的更多信息，请[查阅这个文档](#working-with-error-messages)。

> {tip} `$errors` 变量被 `Illuminate\View\Middleware\ShareErrorsFromSession` 中间件绑定到视图，该中间件由 `web` 中间件组提供。**当这个中间件被应用后，在你的视图中就可以获取到 `$error` 变量**，可以使你方便的假定 `$errors` 变量总是已经被定义好并且可以安全的使用。

所以，在我们的例子中，当验证失败的时候，用户将会被重定向到 `create` 方法，让我们在视图中显示错误信息：

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
### 有关可选字段的注意事项

默认情况下，Laravel 会在你的应用中的全局中间件栈中包含 `TrimStrings` 和 `ConvertEmptyStringsToNull` 中间件。这些中间件在 `App\Http\Kernel` 类中。因此，如果您不希望验证程序将「null」值视为无效的，您通常需要将「可选」的请求字段标记为 `nullable`。

    $this->validate($request, [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

在这个例子里，我们指定 `publish_at` 字段可以为 `null` 或者一个有效的日期格式。如果 `nullable` 的修饰词没有添加到规则定义中，验证器会认为 `null` 是一个无效的日期格式。



<a name="quick-ajax-requests-and-validation"></a>
#### AJAX 请求验证

在这个例子中，我们使用一种传统的方式来将数据发送到应用程序上，然而很多程序使用AJAX发送请求。当我们在 AJAX 的请求中使用 `validate` 方法时，Laravel 并不会生成一个重定向响应，而是会生成一个包含所有错误验证的 JSON 响应。这个 JSON 响应会包含一个 422 HTTP 状态码被发送出去。

<a name="form-request-validation"></a>
## 表单请求验证

<a name="creating-form-requests"></a>
### 创建表单请求

在更复杂的验证情境中，你可能会想要创建一个「表单请求（ form request ）」。表单请求是一个自定义的请求类，里面包含着验证逻辑。要创建一个表单请求类，可使用 Artisan 命令行命令 `make:request` ：

    php artisan make:request StoreBlogPost

新生成的类保存在 `app/Http/Requests` 目录下。如果这个目录不存在，那么将会在你运行 `make:request` 命令时创建出来。让我们添加一些验证规则到 `rules` 方法中：

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

验证规则是如何运行的呢？你所需要做的就是在控制器方法中利用类型提示传入请求。传入的请求会在控制器方法被调用前进行验证，意思就是说你不会因为验证逻辑而把控制器弄得一团糟：

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

如果验证失败，就会生成一个重定向响应把用户返回到先前的位置。这些错误会被闪存到 Session，所以这些错误都可以显示出来。如果进来的是 AJAX 请求的话，则会传回一个 HTTP 响应，其中包含了 422 状态码和验证错误的 JSON 数据。

#### 添加表单请求后钩子

如果你想在表单请求「之后」添加钩子，你可以使用 `withValidator` 方法。这个方法接收一个完整的验证类，允许你在实际判断验证规则调之前调用验证类的所有方法：

    /**
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

表单的请求类内包含了 `authorize` 方法。在这个方法中，你可以确认用户是否真的通过了授权，以便更新指定数据。比方说，你可以判断，当一个用户试图去更新一篇文章的评论时，他是否具备对应的权限：

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

由于所有的表单请求都是扩展于基础的 Laravel 中的 request 类，所以我们可以使用 `user` 方法去获取当前认证登录的用户。同时请注意上述例子中对 `route` 方法的调用。这个方法授权你获取调用的路由规则中的 URI 参数，譬如下面例子中的`｛comment｝`参数：

    Route::post('comment/{comment}');

如果 `authorize` 方法返回 `false`，则会自动返回一个 HTTP 响应，其中包含 403 状态码，而你的控制器方法也将不会被运行。

如果你打算在应用程序的其它部分处理授权逻辑，只需从 `authorize` 方法返回 `true` ：

    /**
     * 判断用户是否有权限做出此请求。
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }


<a name="customizing-the-error-messages"></a>
### 自定义错误消息

你可以通过重写表单请求的 `messages` 方法来自定义错误消息。此方法必须返回一个数组，其中含有成对的属性或规则以及对应的错误消息：

    /**
     * 获取已定义验证规则的错误消息。
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
## 手动创建验证请求

如果你不想要使用 request 对象中的 `validate` 方法，你可以手动通过`validator` [Facade](/docs/{{version}}/facades) 创建一个 validator 实例。Facade 中的 `make` 方法生成一个新的 `validator` 实例：

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

第一个传给 `make` 方法的参数是验证数据。第二个参数则是数据的验证规则。

如果请求没有通过验证，则可以使用 `withErrors` 方法把错误消息闪存到 Session。在进行重定向之后，`$errors` 变量可以在视图中自动共用，让你可以轻松地显示这些消息并返回给用户。`withErrors` 方法接收 validator、`MessageBag`，或 PHP `array`。

<a name="automatic-redirection"></a>
### 自动重定向

如果你想手动创建一个验证器实例，但希望继续享用 request 对象中 `validates` 方法提供的自动跳转功能，那么你可以调用一个现存的验证器实例中的 `validate` 方法。如果验证失败了，用户会被自动重定向，或者在 AJAX 请求中，一个 JSON 格式的响应将会被返回：

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validate();

<a name="named-error-bags"></a>
### 命名错误包

如果你在一个页面中有多个表单，你也许会希望命名错误信息包 `MessageBag` ，错误信息包允许你从指定的表单中接收错误信息。简单的给 `withErrors` 方法传递第二个参数作为一个名字：

    return redirect('register')
                ->withErrors($validator, 'login');

然后你能从 `$errors` 变量中获取到 `MessageBag` 实例：

    {{ $errors->login->first('email') }}

<a name="after-validation-hook"></a>
### 验证后钩子

验证器允许你在验证完成之后附加回调函数。这使得你可以容易的执行进一步验证，甚至可以在消息集合中添加更多的错误信息。使用它只需在验证实例中使用 `after` 方法：

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

调用 `Validator` 实例的 `errors` 方法，会得到一个 `Illuminate\Support\MessageBag` 的实例，里面有许多可让你操作错误消息的便利方法。`$errors` 变量可以自动的被所有的视图获取，并且它是一个MessageBag类的实例。自动对所有视图可用的 `$errors` 变量也是 `MessageBag` 类的一个实例。

#### 查看特定字段的第一个错误消息

如果要查看特定字段的第一个错误消息，可以使用 `first` 方法：

    $errors = $validator->errors();

    echo $errors->first('email');

#### 查看特定字段的所有错误消息

如果你想以数组的形式获取指定字段的所有信息，则可以使用 `get` 方法：

    foreach ($errors->get('email') as $message) {
        //
    }

如果你正在验证的一个表单字段类型是数组，你可以使用 `*` 来获取每个元素的所有错误信息：

    foreach ($errors->get('attachments.*') as $message) {
        //
    }

#### 查看所有字段的所有错误消息

如果你想要得到所有字段的消息数组，则可以使用 `all` 方法：

    foreach ($errors->all() as $message) {
        //
    }

#### 判断特定字段是否含有错误消息

可以使用 `has` 方法来检测一个给定的字段是否存在错误信息：

    if ($errors->has('email')) {
        //
    }

<a name="custom-error-messages"></a>
### 自定义错误消息

如果有需要的话，你也可以自定义错误的验证消息来取代默认的验证消息。有几种方法可以指定自定义消息。首先，你需要先通过传递三个参数到 `Validator::make` 方法来自定义验证消息：

    $messages = [
        'required' => 'The :attribute field is required.',
    ];

    $validator = Validator::make($input, $rules, $messages);

在这个例子中，`:attribute` 占位符会被通过验证的字段实际名称所取代。除此之外，你还可以使用验证提示消息中的其他占位符。例如：

    $messages = [
        'same'    => 'The :attribute and :other must match.',
        'size'    => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute must be between :min - :max.',
        'in'      => 'The :attribute must be one of the following types: :values',
    ];

#### 指定自定义消息到特定的属性

有时候你可能只想对特定的字段自定义错误消息。只需在属性名称后加上「.」符号和指定验证的规则即可：

    $messages = [
        'email.required' => 'We need to know your e-mail address!',
    ];

<a name="localization"></a>
#### 在语言文件中指定自定义的消息提示

多数情况下，你会在语言文件中指定自定义的消息提示，而不是将定制的消息传递给 `Validator` 。实现它需要在语言文件 `resources/lang/xx/validation.php` 中，将定制的消息添加到 `custom` 数组。

    'custom' => [
        'email' => [
            'required' => 'We need to know your e-mail address!',
        ],
    ],

#### 在语言文件中自定义属性

如果希望将验证消息的`:attribute` 部分替换为自定义属性名称，则可以在 `resources/lang/xx/validation.php` 语言文件的 `attributes` 数组中指定自定义名称：

    'attributes' => [
        'email' => 'email address',
    ],

<a name="available-validation-rules"></a>
## 可用的验证规则

以下是所有可用的验证规则清单与功能：

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

验证字段值是否为 _yes_、 _on_、 _1_、或 _true_。这在确认「服务条款」是否同意时相当有用。

<a name="rule-active-url"></a>
#### active_url

根据 PHP 函数 `dns_get_record`，判断要验证的字段必须具有有效的 A 或 AAAA 记录。

<a name="rule-after"></a>
#### after:_date_

验证字段是否是在指定日期之后。这个日期将会通过 `strtotime` 函数来验证。

    'start_date' => 'required|date|after:tomorrow'

作为替换 `strtotime` 传递的日期字符串，你可以指定其它的字段来比较日期：

    'finish_date' => 'required|date|after:start_date'

<a name="rule-after-or-equal"></a>
#### after\_or\_equal:_date_

验证字段必需是等于指定日期或在指定日期之后。更多信息请参见 [after](#rule-after) 规则。

<a name="rule-alpha"></a>
#### alpha

验证字段值是否仅包含字母字符。

<a name="rule-alpha-dash"></a>
#### alpha_dash

验证字段值是否仅包含字母、数字、破折号（ - ）以及下划线（ _ ）。

<a name="rule-alpha-num"></a>
#### alpha_num

验证字段值是否仅包含字母、数字。

<a name="rule-array"></a>
#### array

验证字段必须是一个 PHP 数组。

<a name="rule-before"></a>
#### before:_date_

验证字段是否是在指定日期之前。这个日期将会通过 `strtotime` 函数来验证。

<a name="rule-before-or-equal"></a>
#### before\_or\_equal:_date_

验证字段是否是在指定日期之前。这个日期将会使用 PHP `strtotime` 函数来验证。

<a name="rule-between"></a>
#### between:_min_,_max_

验证字段值的大小是否介于指定的 _min_ 和 _max_ 之间。字符串、数字、数组或是文件大小的计算方式和 [`size`](#rule-size) 规则相同。

<a name="rule-boolean"></a>
#### boolean

验证字段值是否能够转换为布尔值。可接受的参数为 `true`、`false`、`1`、`0`、`"1"` 以及 `"0"`。

<a name="rule-confirmed"></a>
#### confirmed

验证字段值必须和 `foo_confirmation` 的字段值一致。例如，如果要验证的字段是 `password`，就必须和输入数据里的 `password_confirmation` 的值保持一致。

<a name="rule-date"></a>
#### date

验证字段值是否为有效日期，会根据 PHP 的 `strtotime` 函数来做验证。

<a name="rule-date-format"></a>
#### date_format:_format_

验证字段值符合指定的日期格式 (_format_)。你应该只使用 `date` 或 `date_format` 当中的 **其中一个** 用于验证，而不应该同时使用两者。

<a name="rule-different"></a>
#### different:_field_

验证字段值是否和指定的字段 (_field_) 有所不同。

<a name="rule-digits"></a>
#### digits:_value_

验证字段值是否为 _numeric_ 且长度为 _value_。

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

验证字段值的长度是否在 _min_ 和 _max_ 之间。

<a name="rule-dimensions"></a>
#### dimensions

验证的文件必须是图片并且图片比例必须符合规则：

    'avatar' => 'dimensions:min_width=100,min_height=200'

可用的规则为： _min\_width_， _max\_width_ ， _min\_height_ ， _max\_height_ ， _width_ ， _height_ ， _ratio_ 。

比例应该使用宽度除以高度的方式出现。能够使用 3/2 这样的形式设置，也可以使用 1.5 这样的浮点方式：

    'avatar' => 'dimensions:ratio=3/2'


由于此规则需要多个参数，因此您可以 `Rule::dimensions` 方法来构造规则：

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'avatar' => [
            'required',
            Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
        ],
    ]);

<a name="rule-distinct"></a>
#### distinct

当你在验证数组的时候，你可以指定某个值必须是唯一的：

    'foo.*.id' => 'distinct'

<a name="rule-email"></a>
#### email

验证字段值是否符合 e-mail 格式。

<a name="rule-exists"></a>
#### exists:_table_,_column_

验证字段值是否存在指定的数据表中。

#### Exists 规则的基本使用方法

    'state' => 'exists:states'

#### 指定一个特定的字段名称

    'state' => 'exists:states,abbreviation'

有时，您可能需要指定要用于 `exists` 查询的特定数据库连接。你可以使用点「.」语法将数据库连接名称添加到数据表前面来实现这个目的：

    'email' => 'exists:connection.staff,email'

如果您想自定义由验证规则执行的查询，您可以使用 `Rule` 类流畅地定义规则。在这个例子中，我们还将使用数组指定验证规则，而不是使用 `|` 字符来分隔它们：

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

必须是成功上传的文件。

<a name="rule-filled"></a>
#### filled

验证的字段必须带有内容。

<a name="rule-image"></a>
#### image

验证字段文件必须为图片格式（ jpeg、png、bmp、gif、或 svg ）。

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

验证字段值是否有在指定的列表里面。因为这个规则通常需要你 `implode` 一个数组，`Rule::in` 方法可以用来流利地构造规则：

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'zones' => [
            'required',
            Rule::in(['first-zone', 'second-zone']),
        ],
    ]);

<a name="rule-in-array"></a>
#### in_array:_anotherfield_

验证的字段必须存在于 _anotherfield_ 的值中。

<a name="rule-integer"></a>
#### integer

验证字段值是否是整数。

<a name="rule-ip"></a>
#### ip

验证字段值是否符合 IP address 的格式。

#### ipv4

验证字段值是否符合 IPv4 的格式。

#### ipv6

验证字段值是否符合 IPv6 的格式。

<a name="rule-json"></a>
#### json

验证字段是否是一个有效的 JSON 字符串。

<a name="rule-max"></a>
#### max:_value_

字段值必须小于或等于 _value_。字符串、数字、数组或是文件大小的计算方式和 [`size`](#rule-size) 规则相同。

<a name="rule-mimetypes"></a>
#### mimetypes:_text/plain_,...

验证的文件必须是这些 MIME 类型中的一个：

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

要确定上传文件的MIME类型，会读取文件的内容，并且框架将尝试猜测 MIME 类型，这可能与客户端提供的 MIME 类型不同。

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

验证字段文件的 MIME 类型是否符合列表中指定的格式。

#### MIME 规则基本用法

    'photo' => 'mimes:jpeg,bmp,png'

即使你可能只需要验证指定扩展名，但此规则实际上会验证文件的 MIME 类型，其通过读取文件的内容以猜测它的 MIME 类型。

完整的 MIME 类型及对应的扩展名列表可以在下方链接找到：[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="rule-min"></a>
#### min:_value_

字段值必须大于或等于 _value_。字符串、数字、数组或是文件大小的计算方式和 [`size`](#rule-size) 规则相同。

<a name="rule-nullable"></a>
#### nullable

验证的字段可以为 `null`。这在验证基本数据类型，如字符串和整型这些能包含 `null` 值的数据类型中特别有用。

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

验证字段值必须不在给定的值列表中出现。`Rule::notIn`方法在构建规则的时候也许有用：

	use Illuminate\Validation\Rule;
	
	Validator::make($data, [
	    'toppings' => [
	        'required',
	        Rule::notIn(['sprinkles', 'cherries']),
	    ],
	]);

<a name="rule-numeric"></a>
#### numeric

验证字段值是否为数字。

<a name="rule-present"></a>
#### present

验证的字段必须出现，但数据可以为空。

<a name="rule-regex"></a>
#### regex:_pattern_

验证字段值是否符合指定的正则表达式。

**Note:** 当使用 `regex` 规则时，你必须使用数组，而不是使用管道分隔符，特别是当正则表达式含有管道符号时。

<a name="rule-required"></a>
#### required

验证字段必须存在输入数据，且不为空。字段符合下方任一条件时即为「空」：

<div class="content-list" markdown="1">

- 该值为 `null`.
- 该值为空字符串。
- 该值为空数组或空的 `可数` 对象。
- 该值为没有路径的上传文件。

</div>

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,...

如果指定的其它字段（ _anotherfield_ ）等于任何一个 _value_ 时，此字段为必填。

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

如果指定的其它字段（ _anotherfield_ ）等于任何一个 _value_ 时，此字段为不必填。

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

如果指定的字段中的 _任意一个_ 有值且不为空，则此字段为必填。

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

如果指定的 _所有_ 字段都有值，则此字段为必填。

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

如果缺少 _任意一个_ 指定的字段，则此字段为必填。

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

如果所有指定的字段 _都没有_ 值，则此字段为必填。

<a name="rule-same"></a>
#### same:_field_

验证字段值和指定的 字段（ _field_ ） 值是否相同。

<a name="rule-size"></a>
#### size:_value_

验证字段值的大小是否符合指定的 _value_ 值。对于字符串来说，_value_ 为字符数。对于数字来说，_value_ 为某个整数值。对于数组来说， _size_ 对应的是数组的 `count` 函数值。对文件来说，_size_ 对应的是文件大小（单位 kb ）。

<a name="rule-string"></a>
#### string

验证字段值的类型是否为字符串。如果你允许字段的值为 `null` ，那么你应该将 `nullable` 规则附加到字段中。

<a name="rule-timezone"></a>
#### timezone

验证字段值是否是有效的时区，会根据 PHP 的 `timezone_identifiers_list` 函数来判断。

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

在指定的数据表中，验证字段必须是唯一的。如果没有指定 `column`，将会使用字段本身的名称。

**指定一个特定的字段名称：**

    'email' => 'unique:users,email_address'

**自定义数据库连接**

有时，您可能需要为验证程序所做的数据库查询设置自定义连接。如上面所示，如上所示，将 `unique：users` 设置为验证规则将使用默认数据库连接来查询数据库。如果要修改数据库连接，请使用「点」语法指定连接和表名：

    'email' => 'unique:connection.users,email_address'

**强迫 Unique 规则忽略指定 ID：**

有时候，你希望在进行字段唯一性验证时对指定 ID 进行忽略。例如，在「更新个人资料」页面会包含用户名、邮箱和地点。这时你会想要验证更新的 E-mail 值是否为唯一的。如果用户仅更改了用户名字段而没有改 E-mail 字段，就不需要抛出验证错误，因为此用户已经是这个 E-mail 的拥有者了。

为了指示验证器忽略用户的ID，我们将使用 `Rule` 类流畅地定义规则。 在这个例子中，我们还将通过数组来指定验证规则，而不是使用 `|` 字符来分隔：

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::unique('users')->ignore($user->id),
        ],
    ]);

如果你的数据表使用的主键名称不是 `id`，则可以在调用 `ignore` 方法时指定列的名称：

    'email' => Rule::unique('users')->ignore($user->id, 'user_id')

**增加额外的 Where 语句：**

你也可以通过 `where` 方法指定额外的查询约束条件。例如，我们添加 `account_id` 为 `1` 约束条件：

    'email' => Rule::unique('users')->where(function ($query) {
        $query->where('account_id', 1);
    })

<a name="rule-url"></a>
#### url

验证字段必需是有效的 URL 格式。

<a name="conditionally-adding-rules"></a>
## 按条件增加规则

#### 当字段存在的时候进行验证

在某些情况下，你可能 **只想** 在输入数据中有此字段时才进行验证。可通过增加 `sometimes` 规则到规则列表来实现：

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

在上面的例子中，`email` 字段的验证只会在 `$data` 数组有此字段时才会进行。

> {tip} 如果你尝试验证一个总是存在但是可能为空的字段，请查阅 [可选字段的说明](#a-note-on-optional-fields)。

#### 复杂的条件验证

有时候你可能希望增加更复杂的验证条件，例如，你可以希望某个指定字段在另一个字段的值超过 100 时才为必填。或者当某个指定字段有值时，另外两个字段要拥有符合的特定值。增加这样的验证条件并不难。首先，利用你熟悉的 _static rules_ 来创建一个 `Validator` 实例：

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

假设我们有一个专为游戏收藏家所设计的网页应用程序。如果游戏收藏家收藏超过一百款游戏，我们会希望他们来说明下为什么他们会拥有这么多游戏。比如说他们有可能经营了一家游戏分销商店，或者只是为了享受收集的乐趣。为了在特定条件下加入此验证需求，可以在 `Validator` 实例中使用 `sometimes` 方法。

    $v->sometimes('reason', 'required|max:500', function ($input) {
        return $input->games >= 100;
    });

传入 `sometimes` 方法的第一个参数是我们要用条件认证的字段名称。第二个参数是我们想使用的验证规则。`闭包` 作为第三个参数传入，如果其返回 `true`，则额外的规则就会被加入。这个方法可以轻松的创建复杂的条件式验证。你甚至可以一次对多个字段增加条件式验证：

    $v->sometimes(['reason', 'cost'], 'required', function ($input) {
        return $input->games >= 100;
    });

> {tip} 传入 `闭包` 的 `$input` 参数是 `Illuminate\Support\Fluent` 实例，可用来访问你的输入或文件对象。

<a name="validating-arrays"></a>
## 验证数组

验证基于数组的表单输入字段并不一定是一件痛苦的事情。你可以使用「.」来验证一个数组中的属性。例如，如果 HTTP 请求中包含一个 `photos[profile]` 字段，你可以使用下面的方法：

	$validator = Validator::make($request->all(), [
	    'photos.profile' => 'required|image',
	]);


你也可以验证数组中的每一个元素。要验证指定数组输入字段中的每一个 email 是否唯一，可以这么做：

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);

同理，你在语言文件定义验证信息的时候可以使用星号 `*` 字符，可以更加容易的在基于数组格式的字段中使用相同的验证信息：

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],

<a name="custom-validation-rules"></a>
## 自定义验证规则

<a name="using-rule-objects"></a>
### 使用规则对象

Laravel 提供了许多有用的验证规则。但你可能想自定义一些规则。注册自定义验证规则的方法之一，就是使用规则对象。要生成一个新的规则对象，可以使用 `make:rule` Artisan 命令。接下来，我们使用这个命令来生成一个验证字符串是否是大写的验证对象。Laravel 会将新的规则对象存放在 `app/Rules` 目录：

	php artisan make:rule Uppercase

一旦规则对象生成了，我们就可以定义它的行为。一个规则对象包含两个方法： `passes` 和 `message` 。 `passes`方法接收属性值和名称，以及根据属性值是否符合规则而返回 `true` 或者 `false` 。 `message` 方法返回验证不通过时应该使用的错误信息。

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

当然，如果你希望从翻译文件中返回验证错误信息，你可以从 `message` 方法中调用 `trans` 辅助函数。

    /**
     * 获取验证错误信息。
     *
     * @return string
     */
    public function message()
    {
        return trans('validation.uppercase');
    }

一旦规则对象被定义好后，你可以通过传递一个规则实例的方式，将其和其他验证规则附加到一个验证器：

    use App\Rules\Uppercase;

    $request->validate([
        'name' => ['required', new Uppercase],
    ]);

<a name="using-extensions"></a>
### 使用扩展

另外一个注册自定义验证规则的方法，就是使用 `Validator` [Facade](/docs/{{version}}/facades) 中的 `extend` 方法。让我们在 [服务提供者](/docs/{{version}}/providers) 中使用这个方法来注册自定义的验证规则：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Validator;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 启动任意应用程序服务。
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
         * 注册服务容器。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

自定义的验证闭包接收四个参数：要被验证的属性名称 `$attribute`，属性的值 `$value`，传入验证规则的参数数组 `$parameters`，及 `Validator` 实例。

除了使用闭包，你也可以传入类和方法到 `extend` 方法中：

    Validator::extend('foo', 'FooValidator@validate');

#### 自定义错误消息

另外你可能还需要为自定义规则来定义一个错误消息。这可以通过使用自定义内联消息数组或是在验证语言包中加入新的规则来实现。此消息应该被放在数组的第一级，而不是被放在 `custom` 数组内，这是仅针对特定属性的错误消息:

    "foo" => "你的输入是无效的!",

    "accepted" => ":attribute 必须被接受。",

    // 其余的验证错误消息...

当你在创建自定义验证规则时，你可能需要定义占位符来取代错误消息。你可以像上面所描述的那样通过 `Validator` Facade 来使用 `replacer` 方法创建一个自定义验证器。通过 [服务提供者](/docs/{{version}}/providers) 中的 `boot` 方法可以实现：

    /**
     * 启动任意应用程序服务。
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

#### 隐式扩展功能

默认情况下，若有一个类似 [`required`](#rule-required) 这样的规则，当此规则被验证的属性不存在或包含空值时，其一般的验证规则（包括自定扩展功能）都将不会被运行。例如，当 integer 规则的值为 null 时 [`unique`](#rule-unique) 将不会被运行：

    $rules = ['name' => 'unique'];

    $input = ['name' => null];

    Validator::make($input, $rules)->passes(); // true

如果要在属性为空时依然运行此规则，则此规则必须暗示该属性为必填。要创建一个「隐式」扩展功能，可以使用 `Validator::extendImplicit()` 方法：

    Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });

> {note} 一个「隐式」扩展功能只会 _暗示_ 该属性为必填。它的实际属性是否为无效属性或空属性主要取决于你。


## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| Cloes  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/6187_1477389867.jpg?imageView2/1/w/100/h/100">  |  翻译  |  我的[github](https://github.com/cloes)  |



--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org