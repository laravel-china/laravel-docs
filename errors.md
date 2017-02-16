# Laravel 的错误和日志记录

- [简介](#introduction)
- [Configuration](#configuration)
    - [Error Detail](#error-detail)
    - [Log Storage](#log-storage)
    - [Log Severity Levels](#log-severity-levels)
    - [Custom Monolog Configuration](#custom-monolog-configuration)
- [The Exception Handler](#the-exception-handler)
    - [Report Method](#report-method)
    - [Render Method](#render-method)
- [HTTP Exceptions](#http-exceptions)
    - [Custom HTTP Error Pages](#custom-http-error-pages)
- [Logging](#logging)

<a name="introduction"></a>
## 简介

当您启动一个新的Laravel项目时，错误和异常处理就已为您配置。 应用程序触发的所有异常都被`App\Exceptions\Handler`类记录下来，然后渲染给用户。 我们将在本文档中深入介绍此类。

Laravel使用功能强大的[Monolog](https://github.com/Seldaek/monolog) 库进行日志处理。 Laravel配置了多几种日志处理handler，方便您在单个日志文件、多个交替日志文件之间进行选择写入，或将错误信息写入系统日志。

<a name="configuration"></a>
## 配置
<a name="error-detail"></a>
### 显示错误信息
`config / app.php` 文件的`debug`选项，决定了是否向用户显示错误信息。默认情况下，此选项设置为存储在 `.env` 文件中的  `APP_DEBUG` 环境变量中。

开发环境下，应该将`APP_DEBUG`环境变量设置为`true`。在您的生产环境中，此值应始终为 `false`。如果在生产中将该值设置为`true`，则可能会将敏感的配置值暴露给应用程序的最终用户。

<a name="log-storage"></a>
### 日志存储
开箱即用，Laravel支持 `single` 、`daily` 、 `syslog` 和 `errorlog` 日志模式。要配置Laravel使用的存储机制，应该修改config/app.php配置文件中的`log`选项。 例如，如果您希望使用每日一个日志文件而不是单个文件，则应将 `app` 配置文件中的 `log` 值设置为 `daily`：

    'log' => 'daily'

#### 日志保存天数限制

When using the `daily` log mode, Laravel will only retain five days of log files by default. If you want to adjust the number of retained files, you may add a `log_max_files` configuration value to your `app` configuration file:
使用`daily`日志模式时，Laravel将只保留五天默认的日志文件。如果你想调整保留文件的数量，您可以添加一个` log_max_files `配置项目到 ` APP `配置文件：

    'log_max_files' => 30

<a name="log-severity-levels"></a>
### 日志等级

When using Monolog, log messages may have different levels of severity. By default, Laravel writes all log levels to storage. However, in your production environment, you may wish to configure the minimum severity that should be logged by adding the `log_level` option to your `app.php` configuration file.

使用Monolog时，日志消息可能具有不同的日志等级。 默认情况下，Laravel将所有日志级别写入存储。 但是，在生产环境中，您可能希望通过将`log_level`选项添加到`app.php`配置文件中来配置应记录的最低日志等级。


Once this option has been configured, Laravel will log all levels greater than or equal to the specified severity. For example, a default `log_level` of `error` will log **error**, **critical**, **alert**, and **emergency** messages:

一旦配置了此选项，Laravel将记录大于或等于指定日志等级的所有级别。 例如，默认将`log_level` 设置为 `error` 那么将会记录 error, critical, alert 和 emergency 日志信息：

    'log_level' => env('APP_LOG_LEVEL', 'error'),

> {tip} Monolog recognizes the following severity levels - from least severe to most severe: `debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert`, `emergency`.

> {tip} Monolog 识别以下日志等级 - 从低到高为: `debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert`, `emergency`.


<a name="custom-monolog-configuration"></a>
### 自定义Monolog设置

If you would like to have complete control over how Monolog is configured for your application, you may use the application's `configureMonologUsing` method. You should place a call to this method in your `bootstrap/app.php` file right before the `$app` variable is returned by the file:

如果你想让你的应用程序完全控制Monolog，可以使用应用程序的`configureMonologUsing`方法。 你应该放置一个回调方法到 `bootstrap/app.php`文件中，在文件返回`$ app`变量之前，调用这个方法：

    $app->configureMonologUsing(function ($monolog) {
        $monolog->pushHandler(...);
    });

    return $app;

<a name="the-exception-handler"></a>
## 异常处理

<a name="report-method"></a>
### Report方法

All exceptions are handled by the `App\Exceptions\Handler` class. This class contains two methods: `report` and `render`. We'll examine each of these methods in detail. The `report` method is used to log exceptions or send them to an external service like [Bugsnag](https://bugsnag.com) or [Sentry](https://github.com/getsentry/sentry-laravel). By default, the `report` method simply passes the exception to the base class where the exception is logged. However, you are free to log exceptions however you wish.

所有异常都由 `App\Exceptions\Handler` 类处理。 这个类包含两个方法：`report`和`render`。 我们将详细研究这些方法。 `report`方法用于记录异常或将其发送到外部服务，如[Bugsnag](https://bugsnag.com) 或[Sentry](https://github.com/getsentry/sentry-laravel) 。 默认情况下，`report`方法只是将异常传递给记录异常的基类。 然而，你可以自由选择任何方式进行处理。

For example, if you need to report different types of exceptions in different ways, you may use the PHP `instanceof` comparison operator:

例如，如果您需要以不同的方式报告不同类型的异常，您可以使用PHP `instanceof`比较运算符：

    /**
     * 报告或记录异常
     *
     * 这是一个很棒的位置向Sentry，Bugsnag等发送异常。
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

#### Ignoring Exceptions By Type通过类型忽略异常

The `$dontReport` property of the exception handler contains an array of exception types that will not be logged. For example, exceptions resulting from 404 errors, as well as several other types of errors, are not written to your log files. You may add other exception types to this array as needed:

异常handler的`$dontReport`属性包含不会记录的异常类型数组。 例如，404错误导致的异常以及其他几种类型的错误不会写入您的日志文件。 您可以根据需要向此数组添加其他异常类型：


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
### Render方法

The `render` method is responsible for converting a given exception into an HTTP response that should be sent back to the browser. By default, the exception is passed to the base class which generates a response for you. However, you are free to check the exception type or return your own custom response:

`render`方法负责将异常转换成 HTTP 响应发送给浏览器。 默认情况下，异常会传递给为您生成响应的基类。 但是，您可以自由检查异常类型或返回您自己的自定义响应：


    /**
     * 渲染异常并添加到HTTP响应中。
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

Some exceptions describe HTTP error codes from the server. For example, this may be a "page not found" error (404), an "unauthorized error" (401) or even a developer generated 500 error. In order to generate such a response from anywhere in your application, you may use the `abort` helper:

一些异常描述了来自服务器的HTTP错误代码。 例如这可能是“找不到页面” 错误（404），“未授权错误”（401）或甚至开发者生成的500错误。 你可以使用`abort`函数，在应用程序中的任何地方生成这样的响应：

    abort(404);

The `abort` helper will immediately raise an exception which will be rendered by the exception handler. Optionally, you may provide the response text:
`abort`函数将立即引发被异常handler渲染的异常。 此外，您还可以提供响应文本：

    abort(403, 'Unauthorized action.');

<a name="custom-http-error-pages"></a>
### 自定义错误页面

Laravel makes it easy to display custom error pages for various HTTP status codes. For example, if you wish to customize the error page for 404 HTTP status codes, create a `resources/views/errors/404.blade.php`. This file will be served on all 404 errors generated by your application. The views within this directory should be named to match the HTTP status code they correspond to. The `HttpException` instance raised by the `abort` function will be passed to the view as an `$exception` variable.

Laravel可以轻松地显示各种HTTP状态代码的自定义错误页面。 例如，如果您要自定义404 HTTP状态代码的错误页面，请创建一个 `resources/views/errors/404.blade.php`。 此文件将会用于渲染所有404错误。 此目录中的视图文件命名应与它们对应的HTTP状态代码匹配。 由`abort`函数引发的`HttpException`实例将作为`$exception`变量传递给视图。


<a name="logging"></a>
## 记录

Laravel provides a simple abstraction layer on top of the powerful [Monolog](https://github.com/seldaek/monolog) library. By default, Laravel is configured to create a log file for your application in the `storage/logs` directory. You may write information to the logs using the `Log` [facade](/docs/{{version}}/facades):

Laravel在强大的[Monolog](https://github.com/seldaek/monolog) 库上提供了一个简单的抽象层。 默认情况下，Laravel日志目录为 `storage/logs`。 您可以使用 `Log` [facade](/docs/{{version}}/facades): 将信息写入日志：


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

The logger provides the eight logging levels defined in [RFC 5424](https://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** and **debug**.
该日志记录器提供八种 [RFC 5424](https://tools.ietf.org/html/rfc5424):定义的日志级别: emergency ，alert ，critical, error ，warning ，notice ，info 和 debug 。

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

#### 上下文信息
An array of contextual data may also be passed to the log methods. This contextual data will be formatted and displayed with the log message:
将上下文数据以数组格式传递给日志方法。 此上下文数据将被格式化并与日志消息一起显示：

    Log::info('User failed to login.', ['id' => $user->id]);

#### 访问底层 Monolog 实例

Monolog has a variety of additional handlers you may use for logging. If needed, you may access the underlying Monolog instance being used by Laravel:
Monolog还有多种其他的处理handler，你可以用来记录。 如果需要，您可以访问Laravel底层的Monolog实例：

    $monolog = Log::getMonolog();
