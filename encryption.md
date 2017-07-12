# 加密

- [设置](#configuration)
- [基本用法](#basic-usage)

<a name="configuration"></a>
## 设置

在使用 Laravel 的加密器前，你应该先设置 `config/app.php` 配置文件中的 `key` 选项，设置值需要是 32 个字符的随机字符串。如果没有适当地设置这个值，所有被 Laravel 加密的值都将是不安全的。

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





--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译。
> 
> 文档永久地址： http://d.laravel-china.org