# 加密

- [介绍](#introduction)
- [设置](#configuration)
- [基本用法](#basic-usage)

<a name="introduction"></a>
## 介绍

Laravel 是利用 OpenSSL 去提供 AES-256 和 AES-128 的加密。你完全可以使用 Laravel 自带的加密机制，而不用再另外去创建加密算法。Laravel 所有加密之后的结果都会使用消息认证码 (MAC) 去签署，所以一旦被加密就无法再改变。

<a name="configuration"></a>
## 设置

在使用 Laravel 的加密器前，你应该先设置 `config/app.php` 配置文件中的 `key` 选项。由于 Artisan 控制台会使用 PHP 的安全机制为你随机生成 key，你可以直接使用 `php artisan key:generate` 命令去生成 key。如果没有适当地设置这个值，所有被 Laravel 加密的值都将是不安全的。

<a name="basic-usage"></a>
## 基本用法

#### 加密一个值

你可以借助 `Crypt` [facade](/docs/{{version}}/facades) 来加密一个值。这些值都会使用 OpenSSL 与 `AES-256-CBC` 来进行加密。此外，所有加密过后的值都会被签署文件消息验证码 (MAC)，以检测加密字符串是否被篡改过。

例如，我们可以使用 `encrypt` 方法加密机密信息，并把它保存在 [Eloquent 模型](/docs/{{version}}/eloquent) 中：

    <?php

    namespace App\Http\Controllers;

    use Crypt;
    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 保存用户的机密消息。
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $user->fill([
                'secret' => Crypt::encrypt($request->secret)
            ])->save();
        }
    }

> **注意：** 为了能加密对象和数组，加密方法中使用 `serialize` 函数，所以，非 PHP 客户端在解密数据以后需要对数据进行 `unserialize` 反序列化处理。

#### 解密一个值

当然，你可以使用 `Crypt` facade 上的 `decrypt` 方法来解密值。如果该值无法被适当地解密，例如文档消息验证码无效等因素，将会抛出一个 `Illuminate\Contracts\Encryption\DecryptException` 异常：

    use Illuminate\Contracts\Encryption\DecryptException;

    try {
        $decrypted = Crypt::decrypt($encryptedValue);
    } catch (DecryptException $e) {
        //
    }

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@silvercell](https://github.com/silvercell)  | <img class="avatar-66 rm-style" src="https://avatars2.githubusercontent.com/u/20363459?v=3&u=2234d736aa27209a2e986d4d789f95c6d110aa0c&s=140">  |  翻译  | [你今天吃药了吗？](http://www.cxdog.com) |
| [@buer](https://github.com/buer0)  | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/22141008?v=3&u=f14a9d540240e1d39079dc1319eb146a91aabfa8&s=140">  | 翻译 | [已放弃治疗！](http://www.cxdog.com) |
