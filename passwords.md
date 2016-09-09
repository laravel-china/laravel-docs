# 重置密码

- [重置密码说明](#introduction)
- [数据库注意事项](#resetting-database)
- [路由](#resetting-routing)
- [视图](#resetting-views)
- [重置密码后](#after-resetting-passwords)
- [密码自定义](#password-customization)

<a name="introduction"></a>
## 重置密码说明

> ｛提示｝ **马上开始？** 首先在 Laravel 应用下运行 `php artisan make:auth` 命令，然后使用浏览器打开 `http://your-app.dev/register` ，或者任意一个你配的 URL 。这个命令将会生成包括密码重置在内的整个认证系统。

大部分的 web 应用都为用户提供重置密码的功能。Laravel 提供了一种方便的方法用于发送密码提示及执行密码重置，而不需要在每个应用中重新实现。


> ｛注意｝ 在使用 Laravel 密码重置功能之前, 你必须 use 这个 `Illuminate\Notifications\Notifiable` trait 。

<a name="resetting-database"></a>
## 数据库注意事项

开始之前，请先确认 `App\User` 模型是否实现了 `Illuminate\Contracts\Auth\CanResetPassword` contract 。当然，在 Laravel 框架中 `App\User` 模型已经实现了这个接口，并使用 `Illuminate\Contracts\Auth\CanResetPassword` trait 来实现该接口包含的方法。

#### 生成重置令牌表迁移

接下来，用来存储密码重置令牌的表必须被创建。 Laravel 已经自带了这张表的迁移，就存放在 `database/migrations` 目录，因此，你只需要运行数据库迁移命令：


    php artisan migrate

<a name="resetting-routing"></a>
## 路由

Laravel 在 `Auth\ForgotPasswordController` 和 `Auth\ResetPasswordController` 两个类中包含了电子邮件密码重置链接和重置用户密码的必要逻辑。所有需要密码重置的路由可以使用 `make:auth` Artisa 命令生成：

    php artisan make:auth

<a name="resetting-views"></a>
## 视图

和路由一样， Laravel 在执行 `make:auth` 命令时将会成生成密码重置时的必要视图文件。这些视图文件存放在 `resources/views/auth/passwords` 目录。你可以在应用中对生成的文件进行相应修改。

<a name="after-resetting-passwords"></a>
## 重置密码后

定义好重置用户密码的路由和视图后，你可以简单的在浏览器中访问路由 `/password/reset` 。框架自带的 `ForgotPasswordController` 已经包含了发送密码重置链接邮件的逻辑，`ResetPasswordController` 包含了重置用户密码的逻辑。

在密码重置之后，用户将会自动登录并重定向到 `/home` 。你可以定义 `ResetPasswordController` 控制器的 `redirectTo` 属性来定制密码重置成功后的自定义跳转链接：

    protected $redirectTo = '/dashboard';

> ｛注意｝ 默认情况下, 密码重置令牌在一小时内有效。你可以通过修改 `config/auth.php` 的 `expire` 选项来修改此项配置。

<a name="password-customization"></a>
## 密码自定义

#### 自定义认证 Guard

在配置文件 `auth.php` 中, 你可以配置多个 "guards" 参数，可以用来多用户表的独立认证。你可以在重写 `ResetPasswordController` 控制器的 `guard` 方法来实现自定义认证 guard ，此方法应该返回一个 guard 实例：

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### 自定义密码 Broker

在配置文件 `auth.php` 中, 可以配置多个密码 "brokers" ，它可以用于多个用户表重置密码。你可以通过重写 `ForgotPasswordController` and `ResetPasswordController` 控制器的 `broker` 方法来实现你的自定义 broker ：



    /**
     * Get the broker to be used during password reset.
     *
     * @return PasswordBroker
     */
    protected function broker()
    {
        return Password::broker('name');
    }

#### 自定义重置邮箱

你可以方便的修改通知类用以给用户发送密码重置链接。现在开始，在 `User` 模型中重写  `sendPasswordResetNotification` 方法。在这个方法中，你可以使用你选择的任意方法发送通知，密码重置 `$token` 是这个方法的接收第一个参数：

    /**
     * Send the password reset notification.
     *
     * @param  string  $token
     * @return void
     */
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new ResetPasswordNotification($token));
    }
