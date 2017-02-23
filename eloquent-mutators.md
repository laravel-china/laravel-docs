# Eloquent: 修改器

- [简介](#introduction)
- [访问器 & 修改器](#accessors-and-mutators)
    - [定义一个访问器](#defining-an-accessor)
    - [定义一个修改器](#defining-a-mutator)
- [日期转换器](#date-mutators)
- [属性类型转换](#attribute-casting)
    - [数组 & JSON 转换](#array-and-json-casting)

<a name="introduction"></a>
## 简介

访问器和修改器可以让你修改 Eloquent 模型中的属性或者设置它们的值，例如，你可能想要使用 [Laravel 加密器](/docs/{{version}}/encryption) 来加密一个被保存在数据库中的值，当你从 Eloquent 模型访问该属性时该值将被自动解密。

除了自定义访问器和修改器之外，Eloquent 也会自动将日期字段类型转换成 [Carbon](https://github.com/briannesbitt/Carbon) 实例或将 [文本字段类型转换成 JSON](#attribute-casting)。

<a name="accessors-and-mutators"></a>
## 访问器 & 修改器

<a name="defining-an-accessor"></a>
### 定义一个访问器

若要定义一个访问器，则须在你的模型上创建一个 `getFooAttribute` 方法。要访问的 `Foo` 字段需使用「驼峰式」来命名。在这个例子中，我们将为 `first_name` 属性定义一个访问器。当 Eloquent 尝试获取 `first_name` 的值时，将会自动调用此访问器：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 获取用户的名字。
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }

如你所见，字段的原始值被传递到访问器中，让你可以编辑修改并返回结果。如果要访问被修改的值，则可以像这样来访问 `first_name` 属性：

    $user = App\User::find(1);

    $firstName = $user->first_name;

<a name="defining-a-mutator"></a>
### 定义一个修改器

若要定义一个修改器，则须在模型上定义一个 `setFooAttribute` 方法。要访问的 `Foo` 字段需使用「驼峰式」来命名。让我们再来定义 `first_name` 属性的修改器。当我们尝试在模型上设置 `first_name` 的值时，将会自动调用此修改器：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 设定用户的名字。
         *
         * @param  string  $value
         * @return void
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }

修改器会获取属性已经被设置的值，让你可以操作该值并将其设置到 Eloquent 模型内部的 `$attributes` 属性上。举个例子，如果我们尝试将 `first_name` 属性设置成 `Sally`：

    $user = App\User::find(1);

    $user->first_name = 'Sally';

在这个例子中，`setFirstNameAttribute` 函数将会使用 `Sally` 作为参数来调用。修改器会对该名字使用 `strtolower` 函数并将其值设置于内部的 `$attributes` 数组。

<a name="date-mutators"></a>
## 日期转换器

默认情况下，Eloquent 将会把 `created_at` 和 `updated_at` 字段转换成 [Carbon](https://github.com/briannesbitt/Carbon) 实例，它提供了各种各样的方法，并继承了 PHP 原生的 DateTime 类。

你可以在模型中自定义哪些字段需要被自动修改，或完全禁止修改，可通过重写模型的 `$dates` 属性来实现：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 应被转换为日期的属性。
         *
         * @var array
         */
        protected $dates = [
            'created_at',
            'updated_at',
            'deleted_at'
        ];
    }

当某个字段被认为是日期时，你或许想将其数值设置成一个 UNIX 时间戳、日期字符串（`Y-m-d`）、日期时间（ `date-time` ）字符串，当然还有 `DateTime` 或 `Carbon` 实例，然后日期数值将会被自动保存到数据库中：

    $user = App\User::find(1);

    $user->deleted_at = Carbon::now();

    $user->save();

如上所述，在 `$dates` 属性中列出的所有属性被获取到时，都将会自动转换成 [Carbon](https://github.com/briannesbitt/Carbon) 实例，让你可在属性上使用任何 `Carbon` 方法：

    $user = App\User::find(1);

    return $user->deleted_at->getTimestamp();

#### 时间格式

默认情况下，时间戳将会以 `'Y-m-d H:i:s'` 格式化。如果你想要自定义自己的时间戳格式，可在模型中设置 `$dateFormat` 属性。该属性定义了时间属性应如何被保存到数据库，以及模型应被序列化成一个数组或 JSON 格式：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 模型的数据字段的保存格式。
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

<a name="attribute-casting"></a>
## 属性类型转换

`$casts` 属性在模型中提供了将属性转换为常见的数据类型的方法。`$casts` 属性应是一个数组，且键是那些需要被转换的属性名称，值则是代表字段要转换的类型。支持的转换的类型有：

+ integer
+ real
+ float
+ double
+ string
+ boolean
+ object
+ array
+ collection
+ date
+ datetime
+ timestamp

例如，`is_admin` 属性以整数（`0` 或 `1`）被保存在我们的数据库中，让我们来把它转换为布尔值：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 应该被转换成原生类型的属性。
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }

现在当你访问 `is_admin` 属性时，它将会被转换成布尔值，即便保存在数据库里的值是一个整数：

    $user = App\User::find(1);

    if ($user->is_admin) {
        //
    }

<a name="array-and-json-casting"></a>
### 数组 & JSON 转换

若原本字段保存的是被序列化的 JSON，则 `array` 类型转换将会特别有用。例如，在你的数据库中有一个 `JSON` 或 `TEXT` 字段类型，其包含了 被序列化的 JSON，且对该属性添加了 `array` 类型转换。当你在 Eloquent 模型上访问该属性时，它将会被自动反序列化成一个 PHP 数组：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 应该被转换成原生类型的属性。
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

一旦类型转换被定义，则可以访问 `options` 属性，它将会自动把 JSON 反序列化成一个 PHP 数组。当你设置 `options` 属性的值时，指定的数组将会被自动序列化成 JSON 以便进行保存：

    $user = App\User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();
## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@skyverd](https://laravel-china.org/users/79)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/79_1427370664.jpeg?imageView2/1/w/100/h/100">  |  翻译  | 全桟工程师，[时光博客](https://skyverd.com) |