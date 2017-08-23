# Laravel 的密码重设功能

- [简介](#introduction)
- [数据库注意事项](#resetting-database)
- [路由](#resetting-routing)
- [视图](#resetting-views)
- [重置密码后](#after-resetting-passwords)
- [密码自定义](#password-customization)

<a name="introduction"></a>
## 简介

> {tip} **想要快速上手此功能？** 首先在新的 Laravel 应用中运行 `php artisan make:auth` 命令， 然后在浏览器中打开 `http://your-app.dev/register`，或者任意一个在应用中分配的 URL。这个命令将会生成包括密码重置在内的整个认证系统。

大部分 web 应用为用户提供了充值密码的功能。Laravel 提供了方便的方法用于发送密码提醒并完成密码重置，而不需要在每个应用中重新实现。

> {note} 在使用 Laravel 的密码重置功能之前，你必须在你的用户模型中使用 `Illuminate\Notifications\Notifiable` 这个 trait，也就是加入 `use Illuminate\Notifications\Notifiable` 这行代码，框架中自带的用户模型 App\User 中已添加。

<a name="resetting-database"></a>
## 数据库注意事项

开始之前，请确认你的 `App\User` 模型实现了 `Illuminate\Contracts\Auth\CanResetPassword` 这个契约。当然，Laravel 框架中包含的 `App\User` 模型已经实现了这个接口，并且需要使用 `Illuminate\Auth\Passwords\CanResetPassword` 这个 trait 引入实现这个接口的方法。

#### 生成重置令牌的数据表

接下来，我们需要创建保存密码重置令牌的数据表。因为 Laravel 已经自带了用于生成这张表的迁移，就存放在 `database/migrations` 目录下， 因此你只需要运行下面的命令就能完成相应数据表的创建：

    php artisan migrate

<a name="resetting-routing"></a>
## 路由

Laravel 在 `Auth\ForgotPasswordController` 和 `Auth\ResetPasswordController` 这两个类中包含了发送重置密码链接邮件和重置密码的必要逻辑。所有重置密码需要用到的路由都会通过执行 `make:auth` 这个 Artisan 命令生成：

    php artisan make:auth

<a name="resetting-views"></a>
## 视图

另外，当 Laravel 执行 `make:auth` 命令时，会在 `resources/views/auth/passwords` 目录下生成充值密码所需要的视图文件。当然你可以根据项目需求随意修改这些视图文件。

<a name="after-resetting-passwords"></a>
## 重置密码后

一旦你生成了用于重置用户密码的路由和视图，你就可以在浏览器中访问 `/password/reset` 这个路由来重置密码。框架中的 `ForgotPasswordController` 已经实现了通过邮件发送重置密码链接的逻辑，`ResetPasswordController` 实现了重置密码的逻辑。

重置密码后，用户就会自动登录并重定向到 `/home`。你可以修改 `ResetPasswordController` 中的 `redirectTo` 属性，从而自定义重置密码后重定向到的位置：

    protected $redirectTo = '/dashboard';

> {note} 默认情况下，重置密码的令牌一小时后会失效。你可以修改 `config/auth.php` 文件中的 `expire` 选项来修改这个过期时间。

<a name="password-customization"></a>
## Customization

#### Authentication Guard Customization

In your `auth.php` configuration file, you may configure multiple "guards", which may be used to define authentication behavior for multiple user tables. You can customize the included `ResetPasswordController` to use the guard of your choice by overriding the `guard` method on the controller. This method should return a guard instance:

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### Password Broker Customization

In your `auth.php` configuration file, you may configure multiple password "brokers", which may be used to reset passwords on multiple user tables. You can customize the included `ForgotPasswordController` and `ResetPasswordController` to use the broker of your choice by overriding the `broker` method:

    use Illuminate\Support\Facades\Password;

    /**
     * Get the broker to be used during password reset.
     *
     * @return PasswordBroker
     */
    protected function broker()
    {
        return Password::broker('name');
    }

#### Reset Email Customization

You may easily modify the notification class used to send the password reset link to the user. To get started, override the `sendPasswordResetNotification` method on your `User` model. Within this method, you may send the notification using any notification class you choose. The password reset `$token` is the first argument received by the method:

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

