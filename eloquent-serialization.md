# Eloquent: 序列化

- [简介](#introduction)
- [基本用法](#basic-usage)
- [隐藏来自 JSON 的属性](#hiding-attributes-from-json)
- [添加一些值到 JSON](#appending-values-to-json)

<a name="introduction"></a>
## 简介

当你在创建 JSON APIs 的时候，经常需要将模型和关联转换成数组或是 JSON。Eloquent 提供了一些方便的方法让我们可以做这些转换，以及控制哪些属性要包括在你的序列化中。

<a name="basic-usage"></a>
## 基本用法

#### 将模型转换成一个数组

如果要将模型还有它加载的[关联](/docs/{{version}}/eloquent-relationships)转换成一个数组，你可以使用 `toArray` 方法。这个方法是递归的，因此，所有属性和所有关联（包含关联中的关联）都会被转换成数组：

    $user = App\User::with('roles')->first();

    return $user->toArray();

你也可以将 [集合](/docs/{{version}}/eloquent-collections) 转换成数组：

    $users = App\User::all();

    return $users->toArray();

#### 将模型转换成 JSON

如果要将模型转换成 JSON，你可以使用 `toJson` 方法。如同 `toArray`，`toJson` 方法也是递归的，所以，所有的属性以及关联都将被转换成 JSON：

    $user = App\User::find(1);

    return $user->toJson();

或者，你可以强制把一个模型或集合转型成一个字符串，它将会自动的调用 `toJson` 方法：

    $user = App\User::find(1);

    return (string) $user;

当模型或集合被转型成字符串时，会转换成 JSON 格式，所以你可以直接从应用程序的路由或者控制器返回 Eloquent 对象：

    Route::get('users', function () {
        return App\User::all();
    });

<a name="hiding-attributes-from-json"></a>
## 隐藏来自 JSON 的属性

有时候你可能会想要限制包含在模型数组或 JSON 表示法的属性，像是密码。要达成这个目地，你可以在模型中增加 `$hidden` 属性定义：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 应该在数组中隐藏的属性。
         *
         * @var array
         */
        protected $hidden = ['password'];
    }

> **注意：** 当你要隐藏关联时，要使用关联的 **方法** 名称，而不是它的动态属性的名称。

另外，你也可以使用 `visible` 属性定义应该包含在你的模型数组和 JSON 表示法中的属性白名单：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 应该在数组中可见的属性。
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }

<a name="appending-values-to-json"></a>
## 添加一些值到 JSON

有时候，你可能需要添加在数据库没有对应字段的数组属性。要达成这个目地，首先必须为这个值定义一个 [访问器](/docs/{{version}}/eloquent-mutators)：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 为用户取得管理者的标记。
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] == 'yes';
        }
    }

一旦你创建了访问器，就可以把将属性名称添加到模型中的 `appends` 属性：

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

一旦属性被添加到 `appends` 清单，在模型的数组和 JSON 两种形式都将会包含进去。在 `appends` 数组中的属性也遵循模型中 `visible` 和 `hidden` 的设置。
