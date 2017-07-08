# Laravel 的错误和日志记录

- [简介](#introduction)
- [配置](#configuration)
    - [显示错误信息](#error-detail)
    - [日志存储](#log-storage)
    - [日志等级](#log-severity-levels)
    - [自定义 Monolog 设置](#custom-monolog-configuration)
- [异常处理](#the-exception-handler)
    - [Report 方法](#report-method)
    - [Render 方法](#render-method)
- [HTTP 异常](#http-exceptions)
    - [自定义错误页面](#custom-http-error-pages)
- [记录](#logging)

<a name="introduction"></a>
## 简介

当您启动一个新的 Laravel 项目时，错误和异常处理就已为您配置。 应用程序触发的所有异常都被 `App\Exceptions\Handler` 类记录下来，然后渲染给用户。 我们将在本文档中深入介绍此类。

Laravel 使用功能强大的 [Monolog](https://github.com/Seldaek/monolog) 库进行日志处理。Laravel 配置了多几种日志处理 handler ，方便您在单个日志文件、多个交替日志文件之间进行选择写入或将错误信息写入系统日志。

<a name="configuration"></a>
## 配置
<a name="error-detail"></a>
### 显示错误信息
`config/app.php` 文件的 `debug` 选项，决定了是否向用户显示错误信息。默认情况下，此选项设置为存储在 `.env` 文件中的  `APP_DEBUG` 环境变量中。

开发环境下，应该将 `APP_DEBUG` 环境变量设置为 `true` 。在您的生产环境中，此值应始终为  `false` 。如果在生产中将该值设置为 `true` ，则可能会将敏感的配置值暴露给应用程序的最终用户。

<a name="log-storage"></a>
### 日志存储
开箱即用，Laravel 支持 `single` 、`daily` 、 `syslog` 和 `errorlog` 日志模式。要配置 Laravel 使用的存储机制，应该修改 `config/app.php` 配置文件中的 `log` 选项。例如，如果您希望使用每日一个日志文件而不是单个文件，则应将 `app` 配置文件中的 `log` 值设置为 `daily`：

    'log' => 'daily'

#### 日志保存天数限制

使用 `daily` 日志模式时，Laravel 将只保留五天默认的日志文件。如果你想调整保留文件的数量，您可以添加一个 `log_max_files` 配置项目到 `APP` 配置文件：

    'log_max_files' => 30

<a name="log-severity-levels"></a>
### 日志等级

使用 Monolog 时，日志消息可能具有不同的日志等级。默认情况下，Laravel 将所有日志级别写入存储。但是，在生产环境中，您可能希望通过将 `log_level` 选项添加到 `app.php` 配置文件中来配置应记录的最低日志等级。

一旦配置了此选项，Laravel 将记录大于或等于指定日志等级的所有级别。例如，默认将 `log_level` 设置为 `error` 那么将会记录 error , critical , alert 和 emergency 日志信息：

    'log_level' => env('APP_LOG_LEVEL', 'error'),

> {tip} Monolog 识别以下日志等级 - 从低到高为: `debug` , `info` , `notice` , `warning` , `error` , `critical` , `alert` , `emergency`。


<a name="custom-monolog-configuration"></a>
### 自定义 Monolog 设置

如果你想让你的应用程序完全控制 Monolog ，可以使用应用程序的 `configureMonologUsing` 方法。你应该放置一个回调方法到 `bootstrap/app.php` 文件中，在文件返回 `$app` 变量之前，调用这个方法：

    $app->configureMonologUsing(function ($monolog) {
        //eg. return $monolog->pushHandler(new \Monolog\Handler\StreamHandler(storage_path('logs/lumen-' . date('Ymd') . '.log')));
        return $monolog->pushHandler(...);
    });

    return $app;

<a name="the-exception-handler"></a>
## 异常处理

<a name="report-method"></a>
### Report 方法

所有异常都由 `App\Exceptions\Handler` 类处理。 这个类包含两个方法：`report` 和 `render` 。 我们将详细研究这些方法。 `report` 方法用于记录异常或将其发送到外部服务，如 [Bugsnag](https://bugsnag.com) 或 [Sentry](https://github.com/getsentry/sentry-laravel) 。默认情况下，`report` 方法只是将异常传递给记录异常的基类。然而，你可以自由选择任何方式进行处理。

例如，如果您需要以不同的方式报告不同类型的异常，您可以使用 PHP `instanceof` 比较运算符：

    /**
     * 报告或记录异常
     *
     * 这是一个很棒的位置向 Sentry ，Bugsnag 等发送异常。
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

异常 handler 的 `$dontReport` 属性包含不会记录的异常类型数组。例如，404错误导致的异常以及其他几种类型的错误不会写入您的日志文件。您可以根据需要向此数组添加其他异常类型：


    /**
     * 不应报告的异常类型列表
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

`render` 方法负责将异常转换成 HTTP 响应发送给浏览器。默认情况下，异常会传递给为您生成响应的基类。但是，您可以自由检查异常类型或返回您自己的自定义响应：

    /**
     * 渲染异常并添加到 HTTP 响应中。
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

一些异常描述了来自服务器的 HTTP 错误代码。例如这可能是「找不到页面」 错误（404），「未授权错误」（401）或甚至开发者生成的500错误。你可以使用 `abort` 函数，在应用程序中的任何地方生成这样的响应：

    abort(404);

`abort`函数将立即创建一个被渲染的异常。此外，您还可以提供响应文本：

    abort(403, 'Unauthorized action.');

<a name="custom-http-error-pages"></a>
### 自定义错误页面

Laravel 可以轻松地显示各种HTTP状态代码的自定义错误页面。例如，如果您要自定义404 HTTP状态代码的错误页面，请创建一个 `resources/views/errors/404.blade.php` 。此文件将会用于渲染所有404错误。此目录中的视图文件命名应与它们对应的HTTP状态代码匹配。由 `abort` 函数引发的 `HttpException` 实例将作为 `$exception` 变量传递给视图。


<a name="logging"></a>
## 记录

Laravel 在强大的 [Monolog](https://github.com/seldaek/monolog) 库上提供了一个简单的抽象层。默认情况下，Laravel 日志目录为 `storage/logs` 。您可以使用 `Log` [facade](/docs/{{version}}/facades) :将信息写入日志：



    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Support\Facades\Log;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 显示给定用户的配置文件
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

该日志记录器提供八种 [RFC 5424](https://tools.ietf.org/html/rfc5424) :定义的日志级别: emergency ，alert ，critical, error ，warning ，notice ，info 和 debug 。

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

#### 上下文信息

将上下文数据以数组格式传递给日志方法。此上下文数据将被格式化并与日志消息一起显示：

    Log::info('User failed to login.', ['id' => $user->id]);

#### 访问底层 Monolog 实例

Monolog 还有多种其他的处理 handler ，你可以用来记录。如果需要，您可以访问 Laravel 底层的 Monolog 实例：

    $monolog = Log::getMonolog();

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@e421083458](https://github.com/e421083458)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/10802_1486368142.jpeg?imageView2/1/w/100/h/100">  |  翻译  | Github求star，[@e421083458](https://github.com/e421083458/) at Github  |
