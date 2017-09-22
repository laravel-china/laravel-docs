# Laravel 的哈希加密

- [简介](#introduction)
- [基本用法](#basic-usage)

<a name="introduction"></a>
## 简介

Laravel `Hash` [Facade](/docs/{{version}}/facades) 提供安全的 Bcrypt 哈希保存用户密码。 如果应用程序中使用了 Laravel 内置的 `LoginController` 和 `RegisterController` 类，它们将自动使用 Bcrypt 进行注册和身份验证。

> {tip} Bcrypt 是哈希密码的理想选择，因为它的「加密系数」可以任意调整，这意味着生成哈希所需的时间可以随着硬件功率的增加而增加。

<a name="basic-usage"></a>
## 基本用法

你可以通过调用 `Hash` Facade 的 `make` 方法来填写密码：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use App\Http\Controllers\Controller;

    class UpdatePasswordController extends Controller
    {
        /**
         *  更新用户密码
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // Validate the new password length...

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

`make` 方法还能使用 `rounds` 选项来管理 bcrypt 哈希算法的加密系数。然而，大多数应用程序还是能接受默认值的：

```
$hashed = Hash::make('password', [
    'rounds' => 12
]);
```

#### 根据哈希值验证密码

`check` 方法可以验证给定的纯文本字符串对应于给定的散列。 如果使用 [Laravel 内置的](/docs/{{version}}/authentication) `LoginController`，则不需要直接使用该方法，因为该控制器会自动调用此方法：

    if (Hash::check('plain-text', $hashedPassword)) {
        // 密码对比...
    }

#### 检查密码是否需要重新加密

`needsRehash` 函数允许你检查已加密的密码所使用的加密系数是否被修改：

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@GanymedeNil](https://github.com/GanymedeNil) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/6859_1487055454.jpg"> | 翻译 | 争做一个 Full Stack Developer  [@GanymedeNil](http://weibo.com/jinhongyang) |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | Review | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
