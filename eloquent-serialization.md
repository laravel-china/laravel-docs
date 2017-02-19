# Eloquent: 序列化

- [简介](#introduction)
- [序列化模型 & 集合](#serializing-models-and-collections)
    - [序列化成数组](#serializing-to-arrays)
    - [序列化成 JSON](#serializing-to-json)
- [隐藏来自 JSON 的属性](#hiding-attributes-from-json)
- [添加参数到 JSON 中](#appending-values-to-json)

<a name="introduction"></a>
## 简介

当你在创建 JSON API 的时候，经常会需要将模型和关联转换成数组或 JSON。Eloquent 提供了一些便捷的方法来让我们可以完成这些转换，以及控制哪些属性需要被包括在序列化中。

<a name="serializing-models-and-collections"></a>
## 序列化模型 & 集合

<a name="serializing-to-arrays"></a>
### 序列化成数组

如果要将模型还有其加载的 [关联](/docs/{{version}}/eloquent-relationships) 转换成一个数组，则可以使用 toArray 方法。这个方法是递归的，因此，所有属性和关联（包含关联中的关联）都会被转换成数组：

    $user = App\User::with('roles')->first();

    return $user->toArray();

你也可以将整个 [集合](/docs/{{version}}/eloquent-collections) 转换成数组：

    $users = App\User::all();

    return $users->toArray();

<a name="serializing-to-json"></a>
### 序列化成 JSON

如果要将模型转换成 JSON，则可以使用 `toJson` 方法。如同 `toArray` 方法一样，`toJson` 方法也是递归的。因此，所有的属性以及关联都会被转换成 JSON：

    $user = App\User::find(1);

    return $user->toJson();

或者，你也可以强制把一个模型或集合转型成一个字符串，它将会自动调用 `toJson` 方法：

    $user = App\User::find(1);

    return (string) $user;

当模型或集合被转型成字符串时，模型或集合便会被转换成 `JSON` 格式，因此你可以直接从应用程序的路由或者控制器中返回 `Eloquent` 对象：

    Route::get('users', function () {
        return App\User::all();
    });

<a name="hiding-attributes-from-json"></a>
## 隐藏来自 JSON 的属性

有时候你可能会想要限制包含在模型数组或 JSON 表示中的属性，比如说密码。则可以通过在模型中增加 `$hidden` 属性定义来实现：


    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 在数组中想要隐藏的属性。
         *
         * @var array
         */
        protected $hidden = ['password'];
    }

> {note} 当你要对关联进行隐藏时，需使用关联的 **方法** 名称，而不是它的动态属性名称。

另外，你也可以使用 `visible` 属性来定义应该包含在你的模型数组和 JSON 表示中的属性白名单。白名单外的其他属性将隐藏，不会出现在转换后的数组或 JSON 中：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 在数组中可见的属性。
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }

#### 临时修改属性的可见度

你可以在模型实例后使用 `makeVisible` 方法来显示通常隐藏的属性，且为了便于使用，`makeVisible` 方法会返回一个模型实例：

    return $user->makeVisible('attribute')->toArray();

相应的，你可以在模型实例后使用 `makeHidden ` 方法来隐藏通常显示的属性：

    return $user->makeHidden('attribute')->toArray();

<a name="appending-values-to-json"></a>
## 添加参数到 JSON 中

有时候，在转换模型到 数组 或 JSON 时，你希望添加一个在数据库中没有对应字段的属性。首先你需要为这个值定义一个 [访问器](/docs/{{version}}/eloquent-mutators) ：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 为用户获取管理者的标记。
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] == 'yes';
        }
    }

访问器创建成功后，只需添加该属性到改模型的 `appends` 属性中。注意，属性名称通常遵循 「[Snake Case](https://en.wikipedia.org/wiki/Snake_case)」 的命名方式，即是访问器的名称是基于 「[Camel Case](https://en.wikipedia.org/wiki/Camel_case)」 的命名方式。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 访问器被附加到模型数组的形式。
         *
         * @var array
         */
        protected $appends = ['is_admin'];
    }

一旦属性被添加到 `appends` 清单，便会将模型中的数组和 JSON 这两种形式都包含进去。在 `appends` 数组中的属性也遵循模型中 `visible` 和 `hidden` 设置。
## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@skyverd](https://laravel-china.org/users/79)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/79_1427370664.jpeg?imageView2/1/w/100/h/100">  |  翻译  | 全桟工程师，[时光博客](https://skyverd.com) |
