# 错误与日志

- [简介](#introduction)
- [设置](#configuration)
- [错误处理](#the-exception-handler)
    - [报告方法](#report-method)
    - [呈现方法](#render-method)
- [HTTP 异常](#http-exceptions)
    - [自定义 HTTP 错误页面](#custom-http-error-pages)
- [日志](#logging)

<a name="introduction"></a>
## 简介

当你开始一个新的 Laravel 项目时，Laravel 已经帮你设置好错误和异常的处理。另外，Laravel 也集成了 [Monolog](https://github.com/Seldaek/monolog) 日志函数库，Monolog 支持和提供多种强大的日志处理。

<a name="configuration"></a>
## 设置

#### 错误细节

你的应用程序通过 `config/app.php` 配置文件中的 `debug` 设置选项来控制浏览器显示错误的细节。默认情况下，此设置选项是参照于保存在 `.env` 文件的 `APP_DEBUG` 环境变量。

在开发的时候，你应该将 `APP_DEBUG` 环境变量设置为 `true`。在你的上线环境中，这个值应该永远为 `false`。

#### 日志模式

Laravel 提供立即可用的 `single`、`daily`、`syslog` 和 `errorlog` 日志模式。例如，如果你想要每天保存一个日志档，而不是单一的文件，你可以简单地在 `config/app.php` 配置文件内设置 `log` 变量：

    'log' => 'daily'

#### 自定义 Monolog 设置

如果你想要完全控制 Monolog，你可以使用应用程序的 `configureMonologUsing` 方法来设置。你应该在 `bootstrap/app.php` 文件返回 `$app` 变量之前调用这个方法：

    $app->configureMonologUsing(function($monolog) {
        $monolog->pushHandler(...);
    });

    return $app;

<a name="the-exception-handler"></a>
## 错误处理

所有的异常处理都是通过 `App\Exceptions\Handler` 类。这个类包含了两个方法：`report` 和 `render`。我们将研究这些方法的细节。

<a name="report-method"></a>
### 报告方法

`report` 方法用来记录异常或将它们发送到像是 [BugSnag](https://bugsnag.com) 的外部服务。默认情况下，当异常被记录时，`report` 方法只是简单的发送异常到基类，当然，你也可以自由的记录异常。

例如，如果你需要以不同的方式报告不同类型的异常，你可以使用 PHP `instanceof` 比较运算符：

    /**
     * 报告或记录一个异常。
     *
     * 这里是个把异常送至 Sentry、Bugsnag 等等的好地方。
     *
     * @param  \Exception  $e
     * @return void
     */
    public function report(Exception $e)
    {
        if ($e instanceof CustomException) {
            //
        }

        return parent::report($e);
    }

#### 借由类型忽略异常

异常处理中的 `$dontReport` 属性是一个数组，包含不需要被记录的异常类型。默认情况下，由 404 错误导致的异常结果并不会被记录到日志文件。你可以根据你的需求增加其他异常类型到这个数组。

<a name="render-method"></a>
### 呈现方法

`render` 方法负责将给定的异常转换成 HTTP 回应再发送到浏览器。默认情况下，异常会被发送到基类并帮你产生回应。然而，你可以自由的检查异常类型或返回自定义的回应：

    /**
     * 呈现异常到 HTTP 回应。
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $e
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $e)
    {
        if ($e instanceof CustomException) {
            return response()->view('errors.custom', [], 500);
        }

        return parent::render($request, $e);
    }

<a name="http-exceptions"></a>
## HTTP 异常

有一些异常是描述来自服务器的 HTTP 错误码。例如，这可能是个「找不到页面」错误（404），「未授权错误」（401），或甚至是开发者导致的 500 错误。你可以使用以下的方法，在你的应用程序任何地方产生回应：

    abort(404);

`abort` 方法通过错误处理，将会立即引发异常，并且呈现错误。或者，你可以提供回应的文本消息：

    abort(403, 'Unauthorized action.');

你可以在请求的生命周期中任何时间点使用这个方法。

<a name="custom-http-error-pages"></a>
### 自定义 HTTP 错误页面

你可以简单的对于各种不同的 HTTP 状态码返回自定义的错误视图。例如，如果你想要自定义 HTTP 404 状态码的错误视图，创建一个 `resources/views/errors/404.blade.php`。应用程序将会使用这个视图处理所有发生的 404 错误。

在这个目录下的视图，命名应该匹配对应到 HTTP 状态码。

<a name="logging"></a>
## 日志

Laravel 日志工具在强大的 [Monolog](http://github.com/seldaek/monolog) 函数库上提供一层简单的功能。Laravel 默认为应用程序创建每天的日志档并保存在 `storage/logs` 目录。你可以使用 `Log` [facade](/docs/{{version}}/facades) 写入信息到你的日志：

    <?php

    namespace App\Http\Controllers;

    use Log;
    use App\User;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 显示给定用户的个人数据。
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

日志工具提供了定义在 [RFC 5424](http://tools.ietf.org/html/rfc5424) 的八个级别： **emergency**、**alert**、**critical**、**error**、**warning**、**notice**、**info** 和 **debug**。

    Log::emergency($error);
    Log::alert($error);
    Log::critical($error);
    Log::error($error);
    Log::warning($error);
    Log::notice($error);
    Log::info($error);
    Log::debug($error);

#### 上下文消息

传入上下文相关的数据数组到日志方法里。此上下文的相关数据会进行格式化并显示在日志消息：

    Log::info('User failed to login.', ['id' => $user->id]);

#### 访问 Monolog 底层实例

Monolog 支持各式各样的附加处理方法，如果有需要，你可以访问 Laravel 底层的 Monolog 实例：

    $monolog = Log::getMonolog();
