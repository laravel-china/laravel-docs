# 错误与日志

- [简介](#introduction)
- [设置](#configuration)
    - [错误细节](#error-detail)
    - [日志储存](#log-storage)
    - [日志严重级别](#log-severity-levels)
    - [自定义 Monolog 配置](#custom-monolog-configuration)
- [异常处理](#the-exception-handler)
    - [Report 方法](#report-method)
    - [render 方法](#render-method)
- [HTTP 异常](#http-exceptions)
    - [自定义 HTTP 错误页面](#custom-http-error-pages)
- [日志](#logging)

<a name="introduction"></a>
## 简介

当你创建一个新的 Laravel 项目时，Laravel 已经将错误和异常处理帮你配置好了。 `App\Exceptions\Handler` 类会将触发异常记入日志并返回给用户。本文会深入的对这个类进行探讨。

日志记录，Laravel 利用 [Monolog](https://github.com/Seldaek/monolog) 函数库提供多样而强大的日志处理。 Laravel 配置了几个处理程序给你，允许你选择单个日志文件或多个来系统记录错误信息。

<a name="configuration"></a>
## 设置

<a name="error-detail"></a>
#### 错误细节

你的应用程序通过 `config/app.php` 配置文件中的 `debug` 设置选项来控制浏览器对错误的细节显示。默认情况下，此设置选项是参照于保存在 `.env` 文件的 `APP_DEBUG` 环境变量。

在开发的时候，你应该将 `APP_DEBUG` 环境变量设置为 `true`。在你的上线环境中，这个值应该永远为 `false`。 如果在生产环境中将这个值设置为 `true`，你将冒风险将一些敏感配置信息暴露个最终用户。

<a name="log-storage"></a>
### 日志存储

Laravel 提供可立即使用的 `single`、`daily`、`syslog` 和 `errorlog` 日志模式。例如，如果你想要每天保存一个日志文件，而不是单个文件，则可以在 `config/app.php` 配置文件内设置 `log` 变量：

    'log' => 'daily'

#### 日志保存天数限制

当使用「日志模式」时，默认情况下会保存 5 天的日志，你可通过 `app.php` 配置文件里的配置项 `log_max_files` 来定制日志保存天数：

    'log_max_files' => 30

<a name="log-severity-levels"></a>
### 日志记录级别

使用 Monolog 时, log 信息可以有不同的严重级别。默认，Laravel 将所有级别日志写到 storage ，然而在你的生产环境中，你可能希望配置一个最小严重级别，那么你应该添加 `log_level` 选项到你的 `app.php` 配置文件。

一旦该选项被配置，Laravel 会记录所有大于或等于这个级别的日志。例如，一个默认 `log_level` 是 `error` 那么将会记录 **error**, **critical**, **alert** 和 **emergency** 信息：

    'log_level' => env('APP_LOG_LEVEL', 'error'),

> {tips} Monolog 辨识以下严重级别 - 最低到高为： `debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert`, `emergency` 。

<a name="custom-monolog-configuration"></a>
### 自定义 Monolog 设置

如果你想要完全控制 Monolog，则使用应用程序的 `configureMonologUsing` 方法。此方法应该在 `bootstrap/app.php` 文件返回 `$app` 变量之前被调用：

    $app->configureMonologUsing(function($monolog) {
        $monolog->pushHandler(...);
    });

    return $app;

<a name="the-exception-handler"></a>
## 异常处理

<a name="report-method"></a>
### Report 方法

所有异常处理都由 `App\Exceptions\Handler` 类进行。这个类包含两个方法：`report` 和 `render`。 我们将研究这些方法的细节。`report` 方法方法用于记录异常或将异常寄给外部服务如 [Bugsnag](https://bugsnag.com) 或 [Sentry](https://github.com/getsentry/sentry-laravel) 。默认， `report` 方法简单地通过传递异常到基类进行处理，然而，你可以自由选择任何方式进行处理。

例如，如果你需要将不同的异常类型报告给不同的方法，你可以使用 PHP `instanceof` 比较操作符：

    /**
     * 报告或记录异常。
     *
     * 这是一个很棒的异常发送到 Sentry ，Bugsnag ，etc 。
     *
     * @param  \Exception  $exception
     * @return void
     */
    public function report(Exception $exception)
    {
        if ($exception instanceof CustomException) {
            //
        }

        return parent::report($exception);
    }

#### 通过类型忽略异常

 `$dontReport` 属性包含一个不会被记录的异常类型数组。例如，404 异常以及其他几个类型异常不会被写到你的日志文件中，如果需要你可以添加其他异常类型这个数组：

    /**
     * A list of the exception types that should not be reported.
     *
     * @var array
     */
    protected $dontReport = [
        \Illuminate\Auth\AuthenticationException::class,
        \Illuminate\Auth\Access\AuthorizationException::class,
        \Symfony\Component\HttpKernel\Exception\HttpException::class,
        \Illuminate\Database\Eloquent\ModelNotFoundException::class,
        \Illuminate\Validation\ValidationException::class,
    ];

<a name="render-method"></a>
### Render 方法

`render` 方法负责将异常转换成 HTTP 响应发送给浏览器。默认，异常传递给生成响应的基类，然而你也可以自由的想检查异常类型或返回自定义响应：

    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof CustomException) {
            return response()->view('errors.custom', [], 500);
        }

        return parent::render($request, $exception);
    }

<a name="http-exceptions"></a>
## HTTP 异常

一些异常描述来自服务器的 HTTP 错误码。例如这个方法可以是一个「页面未找到」 错误 (404)，一个 「认证失败错误」 (401) 或者一个开发人员生成的 500 错误。为了在应用中生成一个这样的响应，你可以使用 `abort` 辅助函数：

    abort(404);

 `abort` 辅助函数将会立即引发一个被异常处理器渲染的异常。此外，你还可以提供响应文本：

    abort(403, 'Unauthorized action.');

<a name="custom-http-error-pages"></a>
### 自定义 HTTP 错误页面

Laravel 制作自定义的 HTTP 错误显示页面很简单。例如，如果你想定义一个 404 页面，创建一个 `resources/views/errors/404.blade.php` 。这个文件将会用于渲染所有的 404 错误。这个视图目录中的视图命名应该和·对于的 HTTP 状态码相匹配。 `HttpException` 实例会将 `abort` 函数传递到视图作为 `$exception` 变量.

<a name="logging"></a>
## 日志

Laravel 用强大的 [Monolog](http://github.com/seldaek/monolog) 函数库提供一个简单日志抽象层。默认，Laravel 被配置为每天为应用在 `storage/logs` 目录下创建一个日志文件。你可以使用 `Log` [facade](/docs/{{version}}/facades) 写入信息：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Support\Facades\Log;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            Log::info('Showing user profile for user: '.$id);

            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

该日志记录器提供八种 [RFC 5424](http://tools.ietf.org/html/rfc5424) 定义的日志级别: **emergency** ，**alert** ，**critical**, **error** ，**warning** ，**notice** ，**info** 和 **debug** 。

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

#### 上下午信息

一个数组上下午信息数据也会传递给日志方法，上下文信息数据也会被格式化记录在日志信息中：

    Log::info('User failed to login.', ['id' => $user->id]);

#### 访问底层 Monolog 实例

Monolog 有一个多样的日志处理器，如果你需要，你可以访问 Laravel 底层的 Monolog 实例：

    $monolog = Log::getMonolog();

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@麦索](https://github.com/dongm2ez)  | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/9032795?v=3&s=460?imageView2/1/w/100/h/100">  |  翻译  | 程序界的小学生，目前生活在北京，希望能够多结交大牛。Follow me [@dongm2ez](https://github.com/dongm2ez) at Github
