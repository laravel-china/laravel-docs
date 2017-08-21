# Laravel 的密码重设功能

- [简介](#introduction)
- [数据库注意事项](#resetting-database)
- [路由](#resetting-routing)
- [视图](#resetting-views)
- [重置密码后](#after-resetting-passwords)
- [密码自定义](#password-customization)

<a name="introduction"></a>
## 简介

> {tip} **想要快速上手此功能？**  首先在 Laravel 应用下运行 `php artisan make:auth` 命令，然后使用浏览器打开 `http://your-app.dev/register` ，或者任意一个在应用中分配的 URL 。这个命令将会生成包括密码重置在内的整个认证系统。

大部分的 web 应用都为用户提供重置密码的功能。Laravel 提供了一种非常方便的方法用于发送密码重置邮件来完成密码重置，而不需要在每个应用中重新实现。

> {note} 在使用 Laravel 密码重置功能之前, 你必须 `use` 这个  `Illuminate\Notifications\Notifiable` `trait`。

<a name="resetting-database"></a>
## 数据库注意事项

开始之前，请先确认你的 `App\User` 模型是否实现了 `Illuminate\Contracts\Auth\CanResetPassword` 协议。当然，在 Laravel 框架中 `App\User` 模型已经实现了这个接口，并使用  `Illuminate\Auth\Passwords\CanResetPassword` `trait` 来实现该接口包含的方法。

#### 生成重置令牌表迁移

接下来，我们必须先创建用来存储密码重置令牌的数据表。由于 Laravel 已经自带了这张表的迁移，就存放在 `database/migrations` 目录，因此，你仅仅只需要运行下面的命令即可完成数据表的创建：

    php artisan migrate

<a name="resetting-routing"></a>
## 路由

Laravel 在 `Auth\ForgotPasswordController` 和 `Auth\ResetPasswordController` 两个类中包含了电子邮件密码重置链接和重置用户密码的必要逻辑。你只需要使用 Artisan 命令 `make:auth` 命令即可生成密码重置所需要的路由：

    php artisan make:auth

<a name="resetting-views"></a>
## 视图

和路由一样Laravel 在执行 `make:auth` 命令时同时会在 `resources/views/auth/passwords` 生成密码重置必要视图文件。当然，你可以随意对它们进行修改。

<a name="after-resetting-passwords"></a>
## 重置密码后

一旦您定义了重置用户密码的路由和视图后，您可以在浏览器中访问 `/password/reset` 。 框架自带的 `ForgotPasswordController` 已经包含了发送密码重置链接邮件的逻辑， `ResetPasswordController` 包含了重置用户密码的逻辑。

在密码重置之后，用户将会自动登录并重定向到 `/home` 。你可以定义 `ResetPasswordController` 控制器的 `redirectTo` 属性来定制密码重置成功后的自定义跳转链接：

    protected $redirectTo = '/dashboard';

> {note} 默认情况下, 密码重置令牌在一小时内有效。你可以通过修改 `config/auth.php` 的 `expire` 选项来修改此项配置。

<a name="password-customization"></a>
## 自定义

#### 自定义认证 Guard

在配置文件 `auth.php` 中, 你可以配置多个 「guards」 参数，可以用来多用户表的独立认证。你可以通过重写  `ResetPasswordController` 控制器的 `guard` 方法来实现自定义认证 `guard` ， 同时别忘了，这个方法需要返回一个 `guard`  实例：

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### 自定义密码 Broker

在配置文件 `auth.php` 中, 可以配置多个密码 「brokers」 , 它可以用于多个用户表重置密码。你可以通过重写  `ForgotPasswordController` 和 `ResetPasswordController` 控制器的 `broker` 方法来实现你的自定义 `broker` ：

    use Illuminate\Support\Facades\Password;

    /**
     * 获取在密码重置期间使用的 broker 
     *
     * @return PasswordBroker
     */
    protected function broker()
    {
        return Password::broker('name');
    }

#### 自定义重置邮箱

你可以方便的修改通知类用以给用户发送密码重置链接。在开始之前， 重写 `User` 模型中  `sendPasswordResetNotification` 方法。在这个方法中，你可以使用你选择的任意通知类发送通知。这个方法的第一个参数为密码重置令牌  `$token` ：

    /**
     * 发送密码重置通知
     *
     * @param  string  $token
     * @return void
     */
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new ResetPasswordNotification($token));
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