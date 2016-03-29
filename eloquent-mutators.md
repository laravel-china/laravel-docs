# Eloquent：修改器

- [简介](#introduction)
- [访问器和修改器](#accessors-and-mutators)
- [日期转换器](#date-mutators)
- [属性类型转换](#attribute-casting)

<a name="introduction"></a>
## 简介

当你从模型取得 Eloquent 的属性或是设置它们的值，访问器和修改器可以让你格式化它们。例如，你可能想要使用 [Laravel 加密器](/docs/{{version}}/encryption)来加密一个被保存在数据库中的值，而在你从 Eloquent 模型访问该属性时可以自动的解密它。

除了自定义的访问器和修改器外，Eloquent 也会自动将日期字段类型转换成 [Carbon](https://github.com/briannesbitt/Carbon) 实例或甚至将[文本字段类型转换成 JSON](#attribute-casting)。

<a name="accessors-and-mutators"></a>
## 访问器和修改器

#### 定义一个访问器

若要定义一个访问器，必须在你的模型上创建一个 `getFooAttribute` 方法，而且你希望访问的 `Foo` 字段需使用「驼峰式」的方式命名。在这个例子中，我们将对 `first_name` 属性定义一个访问器。当 Eloquent 尝试取得 `first_name` 的值时，将会自动的调用访问器：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 取得用户的名字。
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }

如你所见的，字段原始的值被传递到访问器，让你可以操作并返回结果。如果要访问被修改的值，你可以简单的访问 `first_name` 属性：

    $user = App\User::find(1);

    $firstName = $user->first_name;

#### 定义一个修改器

若要定义一个修改器，必须在你的模型上定义一个 `setFooAttribute` 方法，而且你希望访问的 `Foo` 字段需使用「驼峰式」的方式命名。所以，让我们再一次定义 `first_name` 属性的修改器。当我们尝试在模型上设置 `first_name` 的值时，将会自动的调用修改器：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 设置用户的名字。
         *
         * @param  string  $value
         * @return string
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }

修改器会取得属性已经被设置的值，让你可以操作该值并设置到 Eloquent 模型内部的 `$attributes` 属性。所以，举个例子，如果我们尝试将 `first_name` 属性设置成 `Sally`：

    $user = App\User::find(1);

    $user->first_name = 'Sally';

在这个例子中，`setFirstNameAttribute` 函数会使用 `Sally` 当参数调用。修改器会对该名字使用 `strtolower` 函数并将值设置于内部的 `$attributes` 数组。

<a name="date-mutators"></a>
## 日期转换器

默认情况下，Eloquent 将会把 `created_at` 和 `updated_at` 字段转换成 [Carbon](https://github.com/briannesbitt/Carbon) 的实例，它提供了各式各样有用的方法，并继承了 PHP 原生的 `DateTime` 类。

你可以在你的模型中自定义哪些字段要自动地被修改，或甚至完全禁止修改，只要借由覆写模型的 `$dates` 属性：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         *  应该应用日期转换的属性。
         *
         * @var array
         */
        protected $dates = ['created_at', 'updated_at', 'deleted_at'];
    }

当某个字段被认为是日期，你可以将数值设置成一个 UNIX 时间戳记、日期字符串（`Y-m-d`）、日期时间（ date-time ）字符串、当然还有 `DateTime` 或 `Carbon` 实例，然后日期数值会自动正确的保存到你的数据库中：

    $user = App\User::find(1);

    $user->deleted_at = Carbon::now();

    $user->save();

如上面所述，在你的 `$dates` 属性中列出取得的属性，他们将自动转换成 [Carbon](https://github.com/briannesbitt/Carbon) 实例，让你可以在你的属性上使用任何的 Carbon 方法：

    $user = App\User::find(1);

    return $user->deleted_at->getTimestamp();

默认情况下，时间戳记将会以 `'Y-m-d H:i:s'` 格式化。如果你想要自定义你自己的时间戳记格式，在你的模型中设置 `$dateFormat` 属性。这个属性定义了时间属性该如何被保存到数据库，以及模型被串行化成一个数组或 JSON 时的格式：

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

`$casts` 属性在你的模型中提供了方便的方法将属性转换为常见的数据类型。`$casts` 属性应该是一个数组，而键是那些需要被转换的属性名称，而值则是代表你想要把字段转换成什么类型。支持的类型转换的类型有：`integer`、`real`、`float`、`double`、`string`、`boolean`、`object`、`array`、`collection`、`date` 及 `datetime`。

例如，`is_admin` 属性以整数（0 或 1）被保存在我们的数据库中，让我们把它转换为布尔值：

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

现在当你访问 `is_admin` 属性时，它将会总是被转换成布尔值，即使保存在数据库里面的值是一个整数：

    $user = App\User::find(1);

    if ($user->is_admin) {
        //
    }

#### 数组类型转换

如果原本字段保存的为被串行化的 JSON 时，那么 `array` 类型转换将会特别的有用。例如，如果你的数据库有一个 `TEXT` 字段类型包含了被串行化的 JSON，而且对该属性添加了 `array` 类型转换，当你在 Eloquent 模型上访问该属性时，它会自动被反串行化成一个 PHP 的数组：

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

一旦类型转换被定义，你可以访问 `options` 属性而它会自动地从 JSON 反串行化成一个 PHP 数组。当你设置 `options` 属性的值，给定的数组将会自动的被串行化变回成 JSON 以进行保存：

    $user = App\User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();
