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

开始之前，请确认你的 `App\User` 模型实现了 `Illuminate\Contracts\Auth\CanResetPassword` 这个契约。当然，Laravel 框架中包含的 `App\User` 模型已经实现了这个接口，and uses the `Illuminate\Auth\Passwords\CanResetPassword` trait to include the methods needed to implement the interface.

#### Generating The Reset Token Table Migration

Next, a table must be created to store the password reset tokens. The migration for this table is included with Laravel out of the box, and resides in the `database/migrations` directory. So, all you need to do is run your database migrations:

    php artisan migrate

<a name="resetting-routing"></a>
## Routing

Laravel includes `Auth\ForgotPasswordController` and `Auth\ResetPasswordController` classes that contains the logic necessary to e-mail password reset links and reset user passwords. All of the routes needed to perform password resets may be generated using the `make:auth` Artisan command:

    php artisan make:auth

<a name="resetting-views"></a>
## Views

Again, Laravel will generate all of the necessary views for password reset when the `make:auth` command is executed. These views are placed in `resources/views/auth/passwords`. You are free to customize them as needed for your application.

<a name="after-resetting-passwords"></a>
## After Resetting Passwords

Once you have defined the routes and views to reset your user's passwords, you may simply access the route in your browser at `/password/reset`. The `ForgotPasswordController` included with the framework already includes the logic to send the password reset link e-mails, while the `ResetPasswordController` includes the logic to reset user passwords.

After a password is reset, the user will automatically be logged into the application and redirected to `/home`. You can customize the post password reset redirect location by defining a `redirectTo` property on the `ResetPasswordController`:

    protected $redirectTo = '/dashboard';

> {note} By default, password reset tokens expire after one hour. You may change this via the password reset `expire` option in your `config/auth.php` file.

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

