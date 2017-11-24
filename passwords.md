# Laravel 的密码重置功能

- [简介](#introduction)
- [数据库注意事项](#resetting-database)
- [路由](#resetting-routing)
- [视图](#resetting-views)
- [重置密码后](#after-resetting-passwords)
- [自定义](#password-customization)

<a name="introduction"></a>
## 简介

> {tip} **想要快速上手此功能？** 只需在新的 Laravel 应用中运行 `php artisan make:auth` 命令，然后在浏览器中打开 `http://your-app.dev/register`，或者给应用分配任何一个 URL。这个命令会负责构建整个身份验证系统，包括重置密码！

大部分 web 应用为用户提供了重置密码的功能。 Laravel 并不是强迫每个应用程序都要使用这个，仅仅只是想提供方法来发送密码提醒和执行密码重置。

> {note} 在使用 Laravel 的密码重置功能之前，你的用户模型必须使用 `Illuminate\Notifications\Notifiable` trait。

<a name="resetting-database"></a>
## 数据库注意事项

开始之前，请确认你的 `App\User` 模型实现了 `Illuminate\Contracts\Auth\CanResetPassword` 契约。当然，Laravel 框架中包含的 `App\User` 模型已经实现了这个接口，并且使用 `Illuminate\Auth\Passwords\CanResetPassword` trait 来包含实现接口所需的方法。

#### 生成重置令牌的表迁移

接下来，必须创建一个表来存储密码重置令牌。因为 Laravel 已经自带了用于生成这张表的迁移，就存放在 `database/migrations` 目录下。因此只需要运行数据库迁移：

    php artisan migrate

<a name="resetting-routing"></a>
## 路由

Laravel 在 `Auth\ForgotPasswordController` 和 `Auth\ResetPasswordController` 这两个类中分别实现了通过邮件发送重置密码链接和重置密码的逻辑。执行密码重置所需的所有路由都可以使用 Artisan 命令 `make:auth` 生成：

    php artisan make:auth

<a name="resetting-views"></a>
## 视图

当 Laravel 执行 `make:auth` 命令时，会在 `resources/views/auth/passwords` 目录下生成重置密码所需要的视图文件。你可以根据需要随意修改这些视图文件。

<a name="after-resetting-passwords"></a>
## 重置密码后

一旦你生成了用于重置用户密码的路由和视图，你就可以在浏览器中访问 `/password/reset` 这个路由来重置密码。框架中的 `ForgotPasswordController` 已经包含通过邮件发送重置密码链接的逻辑，而 `ResetPasswordController` 包含重置密码的逻辑。

重置密码后，用户就会自动登录并重定向到 `/home`。你可以修改 `ResetPasswordController` 中定义的 `redirectTo` 属性来自定义重置密码后重定向的位置：

    protected $redirectTo = '/dashboard';

> {note} 默认情况下，重置密码的令牌一小时后会失效。你可以修改 `config/auth.php` 文件中的 `expire` 选项来修改这个过期时间。

<a name="password-customization"></a>
## 自定义

#### 自定义认证看守器

在 `auth.php` 配置文件中，你可以配置多个「看守器」，可用于定义多个用户表的身份验证行为。可以通过在 `ResetPasswordController`  控制器上重写 `guard` 方法来指定你想使用的自定义的看守器。这个方法需要返回一个看守器实例：

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### 自定义密码 Broker

在 `auth.php` 配置文件中，你可以配置多个密码 「brokers」，用于多个用户表上的密码重置。你可以通过重写 `ForgotPasswordController` 和 `ResetPasswordController` 中的 `broker` 方法来选择你想使用的自定义 broker：

    use Illuminate\Support\Facades\Password;

    /**
     * 在密码重置期间获取使用代理。
     *
     * @return PasswordBroker
     */
    protected function broker()
    {
        return Password::broker('name');
    }

#### 自定义用于重置的邮件

你可以轻松地修改用于向用户发送密码重置链接的通知类。首先，重写 `User` 模型中的 `sendPasswordResetNotification` 方法。在这个方法中，你可以选择任何的通知类来发送通知。这个方法的第一个参数是用于重置密码的令牌 `$token` ：

    /**
     * 发送密码重置通知。
     *
     * @param  string  $token
     * @return void
     */
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new ResetPasswordNotification($token));
    }

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@lockdown56](https://laravel-china.org/users/7083) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/7083_1503468748.jpg"> | 翻译 | 农闲出来敲代码，农忙回家种玉米 |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | Review | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
