# Laravel 的加密解密机制

- [介绍](#introduction)
- [设置](#configuration)
- [基本用法](#using-the-encrypter)

<a name="introduction"></a>
## 介绍

Laravel 是利用 OpenSSL 去提供 AES-256 和 AES-128 的加密。强烈建议您使用 Laravel 自己的加密机制，而不是尝试自己的「自制」加密算法。 Laravel 所有加密之后的结果都会使用消息认证码 (MAC) 去签署，所以一旦被加密就无法再改变。

<a name="configuration"></a>
## 设置

在使用 Laravel 加密之前, 你必须先设置 `config/app.php` 配置文件中的 `key` 选项。由于 Artisan 控制台会使用 PHP 的安全机制为你随机生成 key ，你可以直接使用 `php artisan key:generate` 命令去生成 key 。如果没有适当地设置这个值，所有被 Laravel 加密的值都将是不安全的。

<a name="using-the-encrypter"></a>
## 基本用法

#### 加密一个值

你可以借助 encrypt 辅助函数来加密一个值。这些值都会使用 OpenSSL 与 `AES-256-CBC` 来进行加密。此外，所有加密过后的值都会被签署文件消息验证码 (MAC)，以检测加密字符串是否被篡改过：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 存储用户保密信息
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $user->fill([
                'secret' => encrypt($request->secret)
            ])->save();
        }
    }

#### 不进行序列化的加密解密方法

加密值在加密期间通过 `serialize` 传递，这也就允许对对象和数组进行加密。由此，非PHP客户端接收到加密值将需要 `unserialize` 数据。如果您希望在不进行序列化的情况下加密和解密值，可以使用 `Crypt` facade 的 `encryptString` 和 `decryptString` 方法：

    use Illuminate\Support\Facades\Crypt;

    $encrypted = Crypt::encryptString('Hello world.');

    $decrypted = Crypt::decryptString($encrypted);

#### 解密一个值

你可以借助 `decrypt` 辅助函数来解密一个值。如果值不能被正确解密，例如当 MAC 无效时，将抛出 `Illuminate\Contracts\Encryption\DecryptException` 异常：

    use Illuminate\Contracts\Encryption\DecryptException;

    try {
        $decrypted = decrypt($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
	
## 译者署名
| 用户名                                      | 头像                                       | 职能   | 签名                                       |
| ---------------------------------------- | ---------------------------------------- | ---- | ---------------------------------------- |
| [@GanymedeNil](https://github.com/GanymedeNil) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/6859_1487055454.jpg?imageView2/1/w/100/h/100"> | 翻译   | 争做一个 Full Stack Developer  [@GanymedeNil](http://weibo.com/jinhongyang) |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org