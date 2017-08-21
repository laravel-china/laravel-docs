# Laravel 的哈希加密

- [简介](#introduction)
- [基本用法](#basic-usage)

<a name="introduction"></a>
## 简介

Laravel 通过 `Hash` [facade](/docs/{{version}}/facades) 提供 Bcrypt 加密来保存用户密码。 如果您在当前的 Laravel 应用程序中使用了内置的`LoginController` 和 `RegisterController` 类，它们将自动使用 Bcrypt 进行注册和身份验证。

> {tip} 由于 Bcrypt 的 「加密系数（word fator）」可以任意调整，这使它成为最好的加密选择。这代表每一次加密的次数可以随着硬件设备的升级而增加。

<a name="basic-usage"></a>
## 基本用法

你可以通过调用 `Hash` facade 的 `make` 方法加密一个密码：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use App\Http\Controllers\Controller;

    class UpdatePasswordController extends Controller
    {
        /**
         * 更新用户密码
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

#### 根据哈希值验证密码

`check` 方法允许你通过一个指定的纯字符串跟哈希值进行验证。 如果你目前正使用[Laravel内含的](/docs/{{version}}/authentication) `LoginController` , 你可能不需要直接使用该方法，它已经包含在控制器当中并且会被自动调用：

    if (Hash::check('plain-text', $hashedPassword)) {
        // 密码对比...
    }

#### 验证密码是否须重新加密

`needsRehash` 函数允许你检查已加密的密码所使用的加密系数是否被修改：

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
	
## 译者署名
| 用户名                                      | 头像                                       | 职能   | 签名                                       |
| ---------------------------------------- | ---------------------------------------- | ---- | ---------------------------------------- |
| [@GanymedeNil](https://github.com/GanymedeNil) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/6859_1487055454.jpg?imageView2/1/w/100/h/100"> | 翻译   | 我不是Full Stack Developer 2333  [@GanymedeNil](http://weibo.com/jinhongyang) |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/3810/laravel-54-document-translation-come-and-join-the-translation)。
> 
> 文档永久地址： http://d.laravel-china.org
