# Eloquent： 修改器

- [简介](#introduction)
- [访问器 & 修改器](#accessors-and-mutators)
    - [定义一个访问器](#defining-an-accessor)
    - [定义一个修改器](#defining-a-mutator)
- [日期转换器](#date-mutators)
- [属性类型转换](#attribute-casting)
    - [数组 & JSON 转换](#array-and-json-casting)
    

<a name="introduction"></a>
## 简介

当你在 Eloquent 模型实例中获取或设置某些属性值的时候，访问器和修改器允许你对 Eloquent 属性值进行格式化。例如，你可能想要使用 [Laravel 加密器](/docs/{{version}}/encryption) 来加密一个即将被保存在数据库中的值，当你从 Eloquent 模型访问该属性时，其值将被自动解密。

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
    
如你所见，字段的原始值被传递到访问器中，允许你对它进行处理并返回结果。如果想获取被修改后的值，你可以在模型实例上访问 `first_name` 属性：

    $user = App\User::find(1);

    $firstName = $user->first_name;

当然，你也可以通过已有的属性，使用访问器返回新的计算值：

```
/**
 * 获取用户名全称
 *
 * @return string
 */
public function getFullNameAttribute()
{
  return "{$this->first_name} {$this->last_name}";
}
```

<a name="defining-a-mutator"></a>
### 定义一个修改器

若要定义一个修改器，则须在模型上定义一个 `setFooAttribute` 方法。要访问的 `Foo` 字段需使用「驼峰式」来命名。让我们再来定义 `first_name` 属性的修改器。当我们尝试在模型上设置 `first_name` 的值时，该修改器将被自动调用：

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

修改器会获取属性已经被设置的值，允许你操作该值并将其设置到 Eloquent 模型内部的 `$attributes` 属性上。举个例子，如果我们尝试将 `first_name` 属性设置成 `Sally`：

    $user = App\User::find(1);

    $user->first_name = 'Sally';

在这个例子中，`setFirstNameAttribute` 方法在调用的时候会接收 `Sally` 这个值作为参数。接着修改器会使用 `strtolower` 函数并将其值设置到内部的 `$attributes` 数组。

<a name="date-mutators"></a>
## 日期转换器

默认情况下，Eloquent 将会把 `created_at` 和 `updated_at` 字段转换成 [Carbon](https://github.com/briannesbitt/Carbon) 实例，它继承了 PHP 原生的 DateTime 类，并提供了各种有用的方法。你可以通过重写模型的 `$dates` 属性，自行定义哪些日期类型字段会被自动转换，或者完全禁止所有日期类型字段的转换：

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
    
当某个字段被认为是日期格式时，你或许想将其数值设置成一个 UNIX 时间戳、日期字符串（`Y-m-d`）、日期时间（ `date-time` ）字符串，当然还有 `DateTime` 或 `Carbon` 实例，并且让日期值自动正确地保存到你的数据库中：

    $user = App\User::find(1);

    $user->deleted_at = Carbon::now();

    $user->save();

就如上面所说的，当获取到的属性包含在 `$dates` 属性时，都将会自动转换成 [Carbon](https://github.com/briannesbitt/Carbon) 实例，允许你在属性上使用任意的 `Carbon` 方法：

    $user = App\User::find(1);

    return $user->deleted_at->getTimestamp();
    
#### 日期格式

默认情况下，时间戳将会以 `'Y-m-d H:i:s'` 的形式格式化。如果你想要自定义自己的时间戳格式，可在模型中设置 `$dateFormat` 属性。该属性决定了日期属性应以何种格式被保存到数据表中，以及当模型被序列化成数组或是 JSON 格式时，这些日期属性以何种格式被保存：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 模型的日期字段的保存格式。
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

<a name="attribute-casting"></a>
## 属性类型转换

`$casts` 属性在模型中提供了一个便利的方法来将属性转换为常见的数据类型。`$casts` 属性应是一个数组，且数组的键是那些需要被转换的属性名称，值则是你希望转换的数据类型。支持转换数据类型有：

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

例如，让我们转换 `is_admin` 属性，将整数（`0` 或 `1`）转换为布尔值：

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

现在当你访问 `is_admin` 属性时，它将会被转换成布尔值类型，即便保存在数据库里的的值是一个整数类型：

    $user = App\User::find(1);

    if ($user->is_admin) {
        //
    }
    
<a name="array-and-json-casting"></a>
### 数组 & JSON 转换

如果一个字段是以被序列化的 JSON 来存储在数据库中, `array` 类型转换将会非常有用。例如，当你在 Eloquent 模型上访问的某个属性在数据库里是一个 `JSON` 或 `TEXT` 字段类型，它包含了被序列化的 JSON，而且你对该字段添加了 `array` 类型转换，那么它将会自动反序列化成一个 PHP 数组：

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
    
一旦类型转换被定义，你就可以访问 `options` 属性，它将会自动把 JSON 反序列化成一个 PHP 数组。当你设置 `options` 属性的值时，接收到的数组将会被自动序列化成 JSON 以便进行保存：

    $user = App\User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();
## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@Ucer](http://codehaoshi.com)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/16042_1493680817.jpeg?imageView2/1/w/200/h/200">  |  翻译  | Php 工程师，[Code好事](http://codehaoshi.com) |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org
