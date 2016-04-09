# 验证

- [简介](#introduction)
- [验证快速上手](#validation-quickstart)
    - [定义路由](#quick-defining-the-routes)
    - [创建控制器](#quick-creating-the-controller)
    - [编写验证逻辑](#quick-writing-the-validation-logic)
    - [显示验证错误](#quick-displaying-the-validation-errors)
    - [AJAX 请求和验证](#quick-ajax-requests-and-validation)
- [其他验证的处理](#other-validation-approaches)
    - [手动创建验证程序](#manually-creating-validators)
    - [表单要求验证](#form-request-validation)
- [处理错误消息](#working-with-error-messages)
    - [自定义错误消息](#custom-error-messages)
- [可用的验证规则](#available-validation-rules)
- [依条件增加规则](#conditionally-adding-rules)
- [自定义验证规则](#custom-validation-rules)

<a name="introduction"></a>
## 简介

Laravel 提供了各种不同的处理方法来验证应用程序传入进来的数据。默认情况下，Laravel 的基底控制器类使用了 `ValidatesRequests` trait，提供了一种方便的方法来验证传入的 HTTP 要求和各种强大的验证规则。

<a name="validation-quickstart"></a>
## 验证快速上手

要了解有关 Laravel 强大的验证特色，让我们来看看一个完整的表单验证例子以及返回错误消息给用户。

<a name="quick-defining-the-routes"></a>
#### 定义路由

首先，让我们假设我们有下列路由定义在我们的 `app/Http/routes.php` 文件里：

    // 显示一个创建博客文章的表单...
    Route::get('post/create', 'PostController@create');

    // 保存一个新的博客文章...
    Route::post('post', 'PostController@store');

当然，`GET` 路由会显示用户用于创建一个新的博客文章的表单，同时 `POST` 路由会保存新的博客文章到数据库。

<a name="quick-creating-the-controller"></a>
#### 创建控制器

接下来，让我们看看一个简单的控制器来操作这些路由。我们现在先让 `store` 方法为空的：

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
#### 编写验证逻辑

现在我们准备填写我们的 `store` 的逻辑方法来验证我们博客发布的新文章。检查你的应用程序的基底控制器 (`App\Http\Controllers\Controller`) 类，你会看到这个类使用了 `ValidatesRequests` trait。这个 trait 在你所有的控制器里提供了方便的 `validate` 验证方法。

`validate` 方法接受 HTTP 传入的请求以及验证的规则。假设通过验证规则，你的代码可以正常的运行；但是，如果验证失败，会抛出异常错误消息自动返回给用户。在传统的 HTTP 请求情况下，会产生一个重定向响应，对于 AJAX 请求则发送 JSON 响应。

为了更能够理解 `validate` 方法，让我们先回到 `store` 方法：

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

        // 博客发表文章是有效的，保存到数据库...
    }

正如你所看到的，我们简单的传递当次 HTTP 请求及所需的验证规则至 `validate` 方法中。再提醒一次，如果验证失败，将会自动产生一个对应的响应。如果通过验证，那我们的控制器会继续正常的运行。

#### 对于嵌套属性的提醒

如果你的 HTTP 请求包含了「嵌套」参数，你可以在验证规则中使用「点」语法指定他们：

    $this->validate($request, [
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>
#### 显示验证错误

所以，如果当次请求的参数未通过我们指定的验证规则呢？正如前面所提到的，Laravel 会自动把用户重定向到先前的位置。另外，所有的验证错误会被自动[快闪至 session](/docs/{{version}}/session#flash-data)。

同样的，注意我们不需要在 `GET ` 路由明确的绑定错误消息至视图。这是因为 Laravel 会自动检查 session 内的错误数据，如果错误存在的话，会自动绑定这些错误消息到视图。**所以，请注意到 `$errors` 变量在每次请求的所有视图中都将可以使用**，让你可以方便假设 `$errors` 变量已被定义且可以安全地使用。`$errors` 变量是 `Illuminate\Support\MessageBag` 的实例。有关此对象的详细信息，[请查阅它的文档](#working-with-error-messages)。

所以，在我们的例子中，当验证失败，用户将被重定向到我们的控制器 `create` 方法，让我们在视图中显示错误的消息：

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

    <!-- 创建文章的表单 -->

<a name="quick-customizing-the-flashed-error-format"></a>
#### 自定义快闪的错误消息格式

当验证失败时，假设你想要自定义验证的错误格式到你的闪存，重写 `formatValidationErrors` 在你的控制器里。别忘了将 `Illuminate\Contracts\Validation\Validator` 类引入到文件的上方：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Foundation\Bus\DispatchesJobs;
    use Illuminate\Contracts\Validation\Validator;
    use Illuminate\Routing\Controller as BaseController;
    use Illuminate\Foundation\Validation\ValidatesRequests;

    abstract class Controller extends BaseController
    {
        use DispatchesJobs, ValidatesRequests;

        /**
         * {@inheritdoc}
         */
        protected function formatValidationErrors(Validator $validator)
        {
            return $validator->errors()->all();
        }
    }

<a name="quick-ajax-requests-and-validation"></a>
### AJAX 请求和验证

在这个例子中，我们用一种传统形式将数据发送到应用程序。然而，许多应用程序使用 AJAX 请求。当我们使用 `validate` 方法在 AJAX 的请求中，Laravel 不会产生一个重定向的响应。相反的，Laravel 会产生一个包含所有错误验证的 JSON 响应。这个 JSON 响应会发送一个 422 HTTP 的状态码。

<a name="other-validation-approaches"></a>
## 其他验证的处理

<a name="manually-creating-validators"></a>
### 手动创建验证程序

如果你不想要使用 `ValidatesRequests` trait 的 `validate` 方法，你可以手动创建一个 validator 实例通过 `Validator::make` 方法在 [facade](/docs/{{version}}/facades) 产生一个新的 validator 实例：

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

            // 保存博客发表文章...
        }
    }

第一个参数传给 `make` 方法是根据数据的验证。第二个参数应该应用在数据的验证规则。

在确认之后，假设要求没有通过验证， 你可以使用 `withErrors` 方法把错误消息闪存到 session。使用这个方法时，在重定向之后，`$errors` 变量可以自动的共用在你的视图中，让你轻松地显示这些消息并返回给用户。`withErrors` 方法接受 validator、`MessageBag`，或者是 PHP `array`。

#### 命名错误清单

假如在一个页面中有许多的表单，你可能希望为 `MessageBag` 的错误命名。这可以让你获取特定表单的所有错误消息，只要在 `withErrors` 的第二个参数设置名称即可：

    return redirect('register')
                ->withErrors($validator, 'login');

然后你就可以从一个 `$errors` 变量中，获取已命名的 `MessageBag` 实例：

    {{ $errors->login->first('email') }}

#### 验证后的挂勾

在验证完成之后，validator 可以让你附加返回消息。可以让你更简单的做更进一步的验证以及增加更多的错误消息让消息变成一个集合。在 validator 实例使用 `after` 方法作为开始：

    $validator = Validator::make(...);

    $validator->after(function($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });

    if ($validator->fails()) {
        //
    }

<a name="form-request-validation"></a>
### 表单请求验证

在更复杂的验证情境中，你可能会想创建一个「表单请求（ form request ）」。表单请求是一个自定义的请求类，里面包含验证的逻辑。要创建一个表单请求类，使用 Artisan 命令行命令 `make:request` ：

    php artisan make:request StoreBlogPostRequest

新产生的类档会放在 `app/Http/Requests` 目录下。让我们加入一些验证规则到 `rules` 方法中：

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

所以，验证的规则会如何被运行？你所需要的只有在控制器方法，利用类型提示传入请求。进入的请求会在控制器方法被调用前进行验证，意思说你不会因为验证逻辑而把控制器弄得一团糟：

    /**
     * 保存传入的博客发表文章。
     *
     * @param  StoreBlogPostRequest  $request
     * @return Response
     */
    public function store(StoreBlogPostRequest $request)
    {
        // 传入的请求是有效的...
    }

假设验证失败，会产生一个重定向响应把用户返回到先前的位置。这些错误会被闪存到 session，所以这些错误都可以被显示。如果进来的是 AJAX 请求的话，而是会传回一个 HTTP 响应，包含 422 状态码，并包含验证错误的 JSON 数据。

#### 授权表单请求

表单的请求类内包含了 `authorize` 方法。在这个方法中，你可以确认用户是否真的通过授权，可以更新特定数据。打个比方，当一个用户试图更新博客文章的评论，他确实是这篇评论的拥有者吗？例如：

    /**
     * 判断用户是否有权限做出此请求。
     *
     * @return bool
     */
    public function authorize()
    {
        $commentId = $this->route('comment');

        return Comment::where('id', $commentId)
                      ->where('user_id', Auth::id())->exists();
    }

注意到上面例子中调用 `route` 的方法。这个方法可以帮助你，获取路由被调用时传入的 URI 参数，像是如下例子的 `{comment}` 参数：

    Route::post('comment/{comment}');

如果 `authorize` 方法返回 `false`，会自动返回一个 HTTP 响应，包含 403 状态码， 而你的控制器方法将不会被运行。

如果你打算在应用程序的其他部分处理授权逻辑，只要从 `authorize` 方法返回 `true` ：

    /**
     * 判断用户是否有权限做出此请求。
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

#### 自定义闪存的错误消息格式

如果你想要自定义验证失败时，闪存到 session 的验证错误的格式， 在你的基底要求 (`App\Http\Requests\Request`) 重写 `formatErrors`。别忘了文件上方要引入 `Illuminate\Contracts\Validation\Validator` 类：

    /**
     * {@inheritdoc}
     */
    protected function formatErrors(Validator $validator)
    {
        return $validator->errors()->all();
    }

#### 自定错误消息

你可以通过重写表单请求的 `messages` 方法自定错误消息。此方法必须返回一个数组，含有成对的属性与规则及对应的错误消息：

    /**
     * 获取已定义验证规则的错误消息。
     *
     * @return array
     */
    public function messages()
    {
        return [
            'title.required' => '标题是必填的',
            'body.required'  => '消息是必填的',
        ];
    }

<a name="working-with-error-messages"></a>
## 处理错误消息

调用一个 `Validator` 实例的 `errors` 方法，会得到一个 `Illuminate\Support\MessageBag` 的实例，里面有许多方便的方法让你操作错误消息。

#### 查看特定字段的第一个错误消息

如果要查看特定字段的第一个错误消息，可以使用 `first` 方法：

    $messages = $validator->errors();

    echo $messages->first('email');

#### 查看特定字段的所有错误消息

如果你想要简单得到一个所有的消息数组在特定的字段，可以使用 `get` 方法：

    foreach ($messages->get('email') as $message) {
        //
    }

#### 查看所有字段的所有错误消息

如果你想要得到所有字段的消息数组，可以使用 `all` 方法：

    foreach ($messages->all() as $message) {
        //
    }

#### 判断特定字段是否有错误消息

    if ($messages->has('email')) {
        //
    }

#### 获取格式化后的错误消息

    echo $messages->first('email', '<p>:message</p>');

#### 获取所有格式化后的错误消息

    foreach ($messages->all('<li>:message</li>') as $message) {
        //
    }

<a name="custom-error-messages"></a>
### 自定义错误消息

如果有需要，你可以自定义错误的验证消息来取代默认的验证消息。有几种方法可以来自定义指定的消息。首先，你需要先自定义验证消息，通过三个参数传到 `Validator::make` 的方法：

    $messages = [
        'required' => ':attribute 的字段是必要的。',
    ];

    $validator = Validator::make($input, $rules, $messages);

在这个例子中，`:attribute` 通过验证字段的的实际名称，预留的字段会被取代。你还可以使用其他默认字段的验证消息。例如：

    $messages = [
        'same'    => ':attribute 和 :other 必须相同。',
        'size'    => ':attribute 必须是 :size。',
        'between' => ':attribute 必须介于 :min - :max。',
        'in'      => ':attribute 必须是以下的类型之一： :values。',
    ];

#### 指定自定义消息到特定的属性

有时候你可能想要对特定的字段自定义错误消息。在你的属性名称后，加上「.」符号，并加上指定验证的规则：

    $messages = [
        'email.required' => '我们需要知道你的 e-mail 地址！',
    ];

<a name="localization"></a>
#### 在语系档中自定义指定消息

在许多情况下，你可能希望在语系档中，被指定的特定属性的自定义消息不是被直接传到 `Validator `。所以把你的消息加入到 `resources/lang/xx/validation.php` 语言档中的 `custom` 数组。

    'custom' => [
        'email' => [
            'required' => '我们需要知道你的 e-mail 地址！',
        ],
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
[Alpha](#rule-alpha)
[Alpha Dash](#rule-alpha-dash)
[Alpha Numeric](#rule-alpha-num)
[Array](#rule-array)
[Before (Date)](#rule-before)
[Between](#rule-between)
[Boolean](#rule-boolean)
[Confirmed](#rule-confirmed)
[Date](#rule-date)
[Date Format](#rule-date-format)
[Different](#rule-different)
[Digits](#rule-digits)
[Digits Between](#rule-digits-between)
[E-Mail](#rule-email)
[Exists (Database)](#rule-exists)
[Image (File)](#rule-image)
[In](#rule-in)
[Integer](#rule-integer)
[IP Address](#rule-ip)
[JSON](#rule-json)
[Max](#rule-max)
[MIME Types (File)](#rule-mimes)
[Min](#rule-min)
[Not In](#rule-not-in)
[Numeric](#rule-numeric)
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

验证字段值是否为 _yes_、_on_、_1_、或 _true_。这在确认「服务条款」是否同意时很有用。

<a name="rule-active-url"></a>
#### active_url

验证字段值是否为一个有效的网址，会通过 PHP 的 `checkdnsrr` 函数来验证。

<a name="rule-after"></a>
#### after:_date_

验证字段是否是在指定日期之后。这个日期将会通过 `strtotime` 函数验证。

    'start_date' => 'required|date|after:tomorrow'

相反的传入日期字符串通过 `strtotime` 来确认，你可以指定其他的字段来比较日期：

    'finish_date' => 'required|date|after:start_date'

<a name="rule-alpha"></a>
#### alpha

验证字段值是否仅包含字母字符。

<a name="rule-alpha-dash"></a>
#### alpha_dash

验证字段值是否仅包含字母、数字、破折号（ - ）以及底线（ _ ）。

<a name="rule-alpha-num"></a>
#### alpha_num

验证字段值是否仅包含字母、数字。

<a name="rule-array"></a>
#### array

验证字段必须是一个 PHP `array`。

<a name="rule-before"></a>
#### before:_date_

验证字段是否是在指定日期之前。这个日期将会使用 PHP `strtotime` 函数验证。

<a name="rule-between"></a>
#### between:_min_,_max_

验证字段值的大小是否介于指定的 _min_ 和 _max_之间。字符串、数值或是文件大小的计算方式和 [`size`](#rule-size) 规则相同。

<a name="rule-boolean"></a>
#### boolean

验证字段值必须要能够转型为布尔值。可接受的参数为 `true`、`false`、`1`、`0`、`"1"` 以及 `"0"`。

<a name="rule-confirmed"></a>
#### confirmed

验证字段值必须和 `foo_confirmation` 命名型式的字段其值一致。例如，如果要验证的字段是 `password`，就必须和输入数据里的 `password_confirmation` 栏为值相同。

<a name="rule-date"></a>
#### date

验证字段值是有效的日期，会根据 PHP 的 `strtotime` 函数做验证。

<a name="rule-date-format"></a>
#### date_format:_format_

验证字段值符合定义的日期 _格式_，通过 PHP 的 `date_parse_from_format` 函数验证。

<a name="rule-different"></a>
#### different:_field_

验证字段值是否和指定的 _字段（ field ）_ 不同。

<a name="rule-digits"></a>
#### digits:_value_

验证字段值为 _numeric_ 且长度为 _value_。

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

验证字段值的长度在 _min_ 和 _max_ 之间。

<a name="rule-email"></a>
#### email

验证字段值符合 e-mail 格式。

<a name="rule-exists"></a>
#### exists:_table_,_column_

验证字段值存在指定的数据表中。

#### Exists 规则的基本使用方法

    'state' => 'exists:states'

#### 指定一个特定的字段名称

    'state' => 'exists:states,abbreviation'

也可以指定更多的条件，它们会被加到「where」查找语句里：

    'email' => 'exists:staff,email,account_id,1'

你也可以传递 `NULL` 或 `NOT_NULL` 至「where」语句：

    'email' => 'exists:staff,email,deleted_at,NULL'

    'email' => 'exists:staff,email,deleted_at,NOT_NULL'

<a name="rule-image"></a>
#### image

验证字段文件必须为图片格式（ jpeg、png、bmp、gif、 或 svg ）。

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

验证字段值有在指定的清单里。

<a name="rule-integer"></a>
#### integer

验证字段值是整数。

<a name="rule-ip"></a>
#### ip

验证字段值符合 IP address 的格式。

<a name="rule-json"></a>
#### json

验证字段必须是一个有效的 JSON 字符串。

<a name="rule-max"></a>
#### max:_value_

验证字段值的大小是否小于或等于 _value_ 。字符串、数值或是文件大小的计算方式和 [`size`](#rule-size) 规则相同。

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

验证字段文件的 MIME 类型必须符合清单里。

#### MIME 规则基本用法

    'photo' => 'mimes:jpeg,bmp,png'

即便你只需要指定的扩展名，但此规则实际上验证了文件的 MIME 类型，通过读取文件的内容，并猜测它的 MIME 类型。

完整的 MIME 类型及对应的扩展名清单可以在下方链接找到：[http://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](http://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="rule-min"></a>
#### min:_value_

验证字段值的大小是否小于或等于 _value_。字符串、数值或是文件大小的计算方式和 [`size`](#rule-size) 规则相同。

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

验证字段值不在指定的清单里。

<a name="rule-numeric"></a>
#### numeric

验证字段值是否为数值。

<a name="rule-regex"></a>
#### regex:_pattern_

验证字段值符合指定的正则表达式。

**注意：**当使用 `regex` 规则时，你必须使用数组，而不该用管道分隔规则，特别是当正则表达式含有管道符号时。

<a name="rule-required"></a>
#### required

验证字段必须存在输入数据且不为空。字段符合下方任一条件时「空」为真：

- 该值为 `null`。
- 该值为空字符串。
- 该值为空数组或空的`可数`对象。
- 该值为没有路径的上传文件。

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,...

如果指定的_其它字段（ anotherfield ）_等于任何一个 _value_ 时，此字段为必填。

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

如果指定的_其它字段（ anotherfield ）_等于任何一个 _value_ 时，则此字段不为必填。

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

如果指定的字段之中，_任一_个有值，则此字段为必填。

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

如果指定的_所有_字段都有值，则此字段为必填。

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

如果缺少_任何一个_指定的字段，则此字段为必填。

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

如果_所有_指定的字段都没有值，则此字段为必填。

<a name="rule-same"></a>
#### same:_field_

验证字段值和指定的_字段（ field ）_值相同。

<a name="rule-size"></a>
#### size:_value_

验证字段值的大小需符合指定 _value_ 值。对于字符串来说，_value_ 为字符数。对于数字来说，_value_ 为某个整数值。对文件来说，_size_ 对应到的是文件大小（单位 kb ）。

<a name="rule-string"></a>
#### string

验证字段值的类型是否为字符串。

<a name="rule-timezone"></a>
#### timezone

验证字段值是个有效的时区，会根据 PHP 的 `timezone_identifiers_list` 函数来判断。

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

在指定的数据表，验证字段中必须是唯一的。如果没有指定 `column`，将会使用字段本身的名称。

**指定一个特定的字段名称：**

    'email' => 'unique:users,email_address'

**自定义数据库连接**

有时候你可能需要自定义一个连接，通过 Validator 对数据库进行查找。如上面所示，设置 `unique:users` 作为验证规则，通过默认数据库连接来做数据库查找。如果要重写验证规则，将指定的连接后在表单名称后加上「.」：

    'email' => 'unique:connection.users,email_address'

**强迫 Unique 规则忽略特定 ID：**

有时候，你希望在验证字段的时候，忽略指定的 ID。例如，在「更新个人数据」时包含用户名、信箱等等。当然，需要先验证 e-mail 是否为唯一的。因为用户只能更改名称字段而不是 e-mail 字段，你不需要去验证错误，因为 e-mail 已经是这个用户的拥有者。假设用户提供的 e-mail 已经被不同的用户使用，你需要抛出验证错误。用特定的规则来忽略用户 ID，你应该把发送 ID 当作第三个参数：

    'email' => 'unique:users,email_address,'.$user->id

如果你的数据表使用的主建名称不是 `id`，那么你可以在第四个参数指定它：

    'email' => 'unique:users,email_address,'.$user->id.',user_id'

**增加额外的 Where 语句：**

也可以指定更多的条件到「where」查找语句：

    'email' => 'unique:users,email_address,NULL,id,account_id,1'

上述规则中，只有 `account_id` 为 `1` 的数据列会被包含在 unique 规则的验证。

<a name="rule-url"></a>
#### url

验证字段值必须符合 URL 格式，根据 PHP 的 `filter_var` 函数。

<a name="conditionally-adding-rules"></a>
## 依条件增加规则

在某些情况下，你可能**只想**在输入数据中有此字段时，才进行验证。只要增加 `sometimes` 规则到进规则列表，就可以快速达成：

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

在上面的例子中，`email` 字段的验证，只会在 `$data` 数组有此字段才会进行。

#### 复杂的条件验证

有时候你可能希望增加更复杂的验证条件，例如，你可以希望某个指定字段，在另一个字段的值有超过 100 时才为必填。或者你当某个指定字段有值时，另外两个字段要符合特定值。增加这样的验证条件并不痛苦。首先，利用你熟悉的 _static rules_ 创建一个 `Validator` 实例：

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

假设我们的网页应用程序是专为游戏收藏家所设计。如果游戏收藏家收藏超过一百款游戏，我们希望他们说明为什么他们拥有这么多游戏。像是，可能他们经营一家二手游戏商店，或是他们可能只是享受收集的乐趣。为了在特定条件下，加入此验证需求，我们可以在 `Validator` 实例使用 `sometimes` 方法。

    $v->sometimes('reason', 'required|max:500', function($input) {
        return $input->games >= 100;
    });

传入 `sometimes` 方法的第一个参数，是我们要依条件认证的字段名称。第二个参数是我们想加入验证规则。`闭包`作为第三个参数传入，如果其返回 `true` 额外的规则就会被加入。这个方法可以轻松的创建复杂的条件式验证。你甚至可以一次对多个字段增加条件式验证：

    $v->sometimes(['reason', 'cost'], 'required', function($input) {
        return $input->games >= 100;
    });

> **注意：**传入`闭包`的 `$input` 参数是 `Illuminate\Support\Fluent` 实例，可以用来作为获取你的输入和文件的对象。

<a name="custom-validation-rules"></a>
## 自定义验证规则

Laravel 提供了很多有用的验证规则；但是，你可能希望自定义一些规则。注册自定义的验证规则的方法之一，就是使用 `Validator::extend` 方法 [facade](/docs/{{version}}/facades) 让我们使用这个方法在[服务提供者](/docs/{{version}}/providers)来自定义注册的验证规则：

    <?php

    namespace App\Providers;

    use Validator;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 启动所有应用程序服务。
         *
         * @return void
         */
        public function boot()
        {
            Validator::extend('foo', function($attribute, $value, $parameters, $validator) {
                return $value == 'foo';
            });
        }

        /**
         * 注册服务提供者。
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

在你的自定义的规则中，需要定义错误消息。你可以自定义消息数组或是在验证语系档中加入新的规则。这个消息应该被放在第一个级别的数组，而不是放在 `custom` 数组， 这是仅对特定属性的错误消息:

    "foo" => "你的输入是无效的！",

    "accepted" => ":attribute 必须被接受。",

    // 其余的验证错误消息...

当你在创建自定义的验证规则时，你可能需要定义保留字段来取代错误消息。你可以创建自定义的验证器，像上面所描述的通过 `Validator` facade 来使用 `replacer` 的方法。你可以通过[服务提供者](/docs/{{version}}/providers)中的 `boot` 方法这么做：

    /**
     * 启动所有应用程序服务。
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend(...);

        Validator::replacer('foo', function($message, $attribute, $rule, $parameters) {
            return str_replace(...);
        });
    }

#### 隐式扩展功能

默认情况中，当验证一个属性时，如同定义的 [`required`](#rule-required) 规则，若不存在或包含空值，则一般的验证规则，包含自定扩展功能，都不会被运行。例如，[`integer`](#rule-integer) 规则当值为 `null` 将不被运行：

    $rules = ['count' => 'integer'];

    $input = ['count' => null];

    Validator::make($input, $rules)->passes(); // true

若要当属性为空时依然运行该规则，那么该规则必须暗示属性为必填。要创建一个「隐式」扩展功能，可以使用 `Validator::extendImplicit()` 方法：

    Validator::extendImplicit('foo', function($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });

> **注意：**一个「隐式」扩展功能只会_暗示_该属性为必填。不论它实际上是无效或是空的属性都取决你。
