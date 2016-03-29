# 加密

- [设置](#configuration)
- [基本用法](#basic-usage)

<a name="configuration"></a>
## 设置

在使用 Laravel 的加密器前，你应该先设置 `config/app.php` 配置文件中的 `key` 选项，设置值需要是 32 个字符的随机字符串。如果没有适当地设置这个值，所有被 Laravel 加密的值将是不安全的。

<a name="basic-usage"></a>
## 基本用法

#### 加密一个值

你可以借由 `Crypt` [facade](/docs/{{version}}/facades) 加密一个值。所有被加密的值都会使用 OpenSSL 与 `AES-256-CBC` 加密。此外，所有加密过后的值都会被签署文档消息鉴别码 (MAC)，以侦测加密字符串是否被窜改。

例如，我们可以使用 `encrypt` 方法加密机密信息，并把它保存在 [Eloquent 模型](/docs/{{version}}/eloquent)中：

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

#### 解密一个值

当然，你可以使用 `Crypt` facade 上的 `decrypt` 方法来解密值。如果该值无法被适当地解密，像是文档消息鉴别码无效等因素，将会抛出一个 `Illuminate\Contracts\Encryption\DecryptException`：

    use Illuminate\Contracts\Encryption\DecryptException;

    try {
        $decrypted = Crypt::decrypt($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
