# 升级导引

- [升级到 5.1.11](#upgrade-5.1.11)
- [升级到 5.1.0](#upgrade-5.1.0)
- [升级到 5.0.16](#upgrade-5.0.16)
- [从 4.2 升级到 5.0](#upgrade-5.0)
- [从 4.1 升级到 4.2](#upgrade-4.2)
- [从 4.1.x 以前版本升级到 4.1.29](#upgrade-4.1.29)
- [从 4.1.25 以前版本升级到 4.1.26](#upgrade-4.1.26)
- [从 4.0 升级到 4.1](#upgrade-4.1)

<a name="upgrade-5.1.11"></a>
## 升级到 5.1.11

Laravel 5.1.11 包含了对于[授权](/docs/{{version}}/authorization)及[授权策略](/docs/{{version}}/authorization#policies)的支持。要将这些功能添加到你现有的 Laravel 5.1 应用程序是相当容易的。

> **注意：**这些升级是**可选的**，忽略它们并不会影响你的应用程序。

#### 创建 Policies 目录

首先，在你的应用程序创建一个空的 `app/Policies` 目录。

#### 创建并注册 AuthServiceProvider 与 Gate Facade

在你的 `app/Providers` 目录创建一个 `AuthServiceProvider`。你可以 [从 GitHub](https://raw.githubusercontent.com/laravel/laravel/master/app/Providers/AuthServiceProvider.php) 获取此文件作为默认的内容，请注意，如果你的应用程序使用自定的命名空间的话，请修改提供者的命名空间。创建完成后，请务必在你的 `app.php` 配置文件的 `providers` 数组注册它。

同样的，你必须在你的 `app.php` 配置文件的 `aliases` 数组注册 `Gate` facade：

    'Gate' => Illuminate\Support\Facades\Gate::class,

#### 更新用户模型

然后，在你的 `App\User` 模型使用 `Illuminate\Foundation\Auth\Access\Authorizable` trait 及 `Illuminate\Contracts\Auth\Access\Authorizable` contract：

    <?php

    namespace App;

    use Illuminate\Auth\Authenticatable;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Auth\Passwords\CanResetPassword;
    use Illuminate\Foundation\Auth\Access\Authorizable;
    use Illuminate\Contracts\Auth\Authenticatable as AuthenticatableContract;
    use Illuminate\Contracts\Auth\Access\Authorizable as AuthorizableContract;
    use Illuminate\Contracts\Auth\CanResetPassword as CanResetPasswordContract;

    class User extends Model implements AuthenticatableContract,
                                        AuthorizableContract,
                                        CanResetPasswordContract
    {
        use Authenticatable, Authorizable, CanResetPassword;
    }

#### 更新基础控制器

接着，更新 `App\Http\Controllers\Controller` 基础控制器，让它使用 `Illuminate\Foundation\Auth\Access\AuthorizesRequests` trait：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Foundation\Bus\DispatchesJobs;
    use Illuminate\Routing\Controller as BaseController;
    use Illuminate\Foundation\Validation\ValidatesRequests;
    use Illuminate\Foundation\Auth\Access\AuthorizesRequests;

    abstract class Controller extends BaseController
    {
        use AuthorizesRequests, DispatchesJobs, ValidatesRequests;
    }

<a name="upgrade-5.1.0"></a>
## 升级到 5.1.0

#### 估计升级需时：少于 1 小时

### 更新 `bootstrap/autoload.php`

将 `bootstrap/autoload.php` 里的 `$compiledPath` 变量按照以下方式更新：

    $compiledPath = __DIR__.'/cache/compiled.php';

### 创建 `bootstrap/cache` 目录

在你的 `bootstrap` 目录里，创建一个 `cache` 目录 (`bootstrap/cache`)。放置一个有以下内容的 `.gitignore` 文件在这个目录：

    *
    !.gitignore

这个目录必须为可写入的，框架会暂时存放如 `compiled.php`、`routes.php`、`config.php` 和 `services.json` 的最佳化文件在此目录。

### 添加 `BroadcastServiceProvider` 提供者

在你的 `config/app.php` 配置文件里面，增加 `Illuminate\Broadcasting\BroadcastServiceProvider` 到 `providers` 数组里。

### 认证

如果你有使用内置的 `AuthController`，他使用了 `AuthenticatesAndRegistersUsers` trait，你会需要对新用户如何创建跟验证做一些修改。

首先，你不再需要传递 `Guard` 和 `Registrar` 实例到基底构造器。你可以从控制器的构造器完全移除这些依赖。

第二，已经不再需要 Laravel 5.0 中使用的 `App\Services\Registrar` 类。你可以简单的从这个类直接复制你的 `validator` 和 `create` 方法粘贴至你的 `AuthController`。这些方法不需要做其它修改。然而，你必须确定有在你的 `AuthController` 顶端引入 `Validator` facade 跟你的 `User` 模型。

#### 密码控制器

内置的 `PasswordController` 的构造器不再需要任何依赖。你可以把 5.0 下需要的依赖都移除。

### 验证

如果你在基底控制器类上重写了 `formatValidationErrors` 方法，你现在应该把类型提示改成 `Illuminate\Contracts\Validation\Validator` contract 来取代具体的 `Illuminate\Validation\Validator` 实例。

同样地，如果你在基底表单请求类上重写了 `formatErrors` 方法，你现在应该把类型提示改成 `Illuminate\Contracts\Validation\Validator` contract 来取代具体的 `Illuminate\Validation\Validator` 实例。

### Eloquent

#### `create` 方法

Eloquent 的 `create` 方法现在可以不带任何参数调用。如果你有在自己的模型重写了 `create` 方法，请把 `$attributes` 参数的默认值设置成空数组：

    public static function create(array $attributes = [])
    {
        // 你的自定义实现
    }

#### `find` 方法

如果你在自己的模型中重写了 `find` 方法并在方法中调用了 `parent::find()`，现在则应该把它改成调用 Eloquent 查询语句构造器上的 `find` 方法：

    public static function find($id, $columns = ['*'])
    {
        $model = static::query()->find($id, $columns);

        // ...

        return $model;
    }

#### `lists` 方法

`lists` 方法现在返回一个 `Collection` 实例而不是 Eloquent 查找用的一般数组。如果你想要把 `Collection` 转换成一般数组，请使用 `all` 方法：

    User::lists('id')->all();

请注意查询语句构造器的 `lists` 方法返回的是一个数组。

#### 日期格式

以前，Eloquent 日期字段的保存格式可以借助重写模型上的 `getDateFormat` 方法来修改。现在仍然可以这么做。然而，为了方便起见你可以直接在模型上指定 `$dateFormat` 属性来取代重写方法。

当序列化模型成 `array` 或 JSON 时，也会采用该日期格式。当从 Laravel 5.0 迁移到 5.1 时，这可能会改变你的 JSON 序列化的日期字段格式。要想针对序列化模型设置特定的日期格式，你需要在你的模型上重写 `serializeDate(DateTime $date)` 方法。这个方法可以让你在不改变日期字段保存格式的情况下，精细的控制 Eloquent 序列化格式。

### 集合类

#### `sort` 方法

`sort` 方法现在会返回全新的集合实例，而不是修改原有的集合：

    $collection = $collection->sort($callback);

#### `sortBy` 方法

`sortBy` 方法现在会返回一个全新的集合实例而不会去改动到现有的集合：

    $collection = $collection->sortBy('name');

#### `groupBy` 方法

`groupBy` 方法现在会返回 `Collection` 实例给父 `Collection` 的每一个元素。如果你想要把所有元素转换回一般数组，你可以通过 `map` 来处理：

    $collection->groupBy('type')->map(function($item)
    {
        return $item->all();
    });

#### `lists` 方法

`lists` 方法现在返回一个 `Collection` 实例而不是一个一般数组。如果你想要把 `Collection` 转换成一般数组，请使用 `all` 方法：

    $collection->lists('id')->all();

### 命令和处理进程

`app/Commands` 目录已经被改名成 `app/Jobs`。然而你并不需要把所有的命令移动到新的位置，且可以继续用 `make:command` 和 `handler:command` Artisan 命令来生成类。

`app/Handlers` 目录已经被改名成 `app/Listeners` 并且现在只包含事件监听者。不需要移动或重命名现有的命令和事件处理进程，并且可以继续使用 `handler:event` 命令来生成事件处理进程。

借由 Laravel 5.0 目录结构提供的向下兼容性，你可以先升级你的应用程序到 Laravel 5.1，然后在你或你的团队方便的时候再慢慢地将你的事件跟命令升级到它们的新位置上。

### Blade

`createMatcher`、`createOpenMatcher` 和 `createPlainMatcher` 方法已经从 Blade 编译器移除。在 Laravel 5.1，请使用新的 `directive` 方法来创建自定义的 Blade 标签。请查看[扩展 blade](/docs/{{version}}/blade#extending-blade) 文档来了解更多信息。

### 测试

添加 protected `$baseUrl` 属性到 `tests/TestCase.php` 文件中：

    protected $baseUrl = 'http://localhost';

### 语言包

第三方扩展包发布语言包的默认目录已经改变。必须从 `resources/lang/packages/{locale}/{namespace}` 移动所有的第三方扩展包语言包到 `resources/lang/vendor/{namespace}/{locale}` 目录。例如，`Acme/Anvil` 扩展包命名空间为 `acme/anvil::foo` 的英文语言包应该从 `resources/lang/packages/en/acme/anvil/foo.php` 移动到 `resources/lang/vendor/acme/anvil/en/foo.php`。

### Amazon 网络服务 SDK

如果你有使用 AWS SQS 队列驱动或 AWS SES 电子邮件驱动，你应该升级你安装的 AWS PHP SDK 到 3.0 版本。

如果你有使用 Amazon S3 文件系统驱动，你将需要借助 Composer 更新对应的文件系统扩展包：

- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`

### 弃用的功能

以下的 Laravel 功能已经被弃用， 并将会在 2015 十二月发布的 Laravel 5.2 中被完全移除：

<div class="content-list" markdown="1">
- 路由过滤器已经被弃用，转而使用[中间件](/docs/{{version}}/middleware)。
- `Illuminate\Contracts\Routing\Middleware` contract 已经被弃用。你的中间件上不需要任何 contract。此外，`TerminableMiddleware` contract 也已经被弃用。不要实现接口，简单地定义一个 `terminate` 方法在你的中间件上就好。
- `Illuminate\Contracts\Queue\ShouldBeQueued` contract 已经被弃用而用 `Illuminate\Contracts\Queue\ShouldQueue` 取代。
- Iron.io 的「推送队列」已经被弃用而用一般的 Iron.io 队列和[队列监听者](/docs/{{version}}/queues#running-the-queue-listener)取代。
- `Illuminate\Foundation\Bus\DispatchesCommands` trait 已经被弃用并改名成 `Illuminate\Foundation\Bus\DispatchesJobs`。
- `Illuminate\Container\BindingResolutionException` 被移到 `Illuminate\Contracts\Container\BindingResolutionException`。
- 服务容器的 `bindShared` 方法已经被弃用，转而使用 `singleton` 方法取代。
- Eloquent 和查询语句构造器的 `pluck` 方法已被弃用并改名为 `value`。
- 集合的 `fetch` 方法已经被弃用，转而用 `pluck` 方法取代。
- `array_fetch` 辅助函数已经被弃用，转而用 `array_pluck` 方法取代。
</div>

<a name="upgrade-5.0.16"></a>
## 升级到 5.0.16

在你的 `bootstrap/autoload.php` 文件中，更新 `$compiledPath` 变量为：

    $compiledPath = __DIR__.'/../vendor/compiled.php';

<a name="upgrade-5.0"></a>
## 从 4.2 升级到 5.0

### 全新安装，然后迁移

推荐的升级方式是创建一个全新的 Laravel `5.0` 项目，然后复制你 `4.2` 网站特定的应用程序文件到此新的应用程序。其中包含控制器、路由、Eloquent 模型、Artisan 命令、资源文件，和其它专属于你的应用程序代码。

开始前，请先在你的本地环境中[安装一个新的 Laravel 5 应用程序](/docs/{{version}}/installation)到一个全新的目录中。不要安装超过 5.0 的任何版本，因为我们需要先完成迁移至 5.0 的步骤。我们将会在后面详细探讨各部分的详细迁移过程。

### Composer 依赖与扩展包

别忘了复制其它额外的 Composer 依赖扩展包到你的 5.0 应用程序内。其中包含如 SDKs 的第三方代码。

部分 Laravel 专用扩展包也许不兼容于刚发布的 Laravel 5。请向扩展包的维护者确认该扩展包支持 Laravel 5 的版本。在你添加完应用程序需要的所有额外 Composer 依赖后，请运行 `composer update`。

### 命名空间

默认情况下，Laravel 4 应用程序不会在应用程序的代码中使用命名空间。如所有的 Eloquent 模型和控制器都简单地存在于「全局」的命名空间中。为了更快速的迁移，Laravel 5 也允许你将这些类继续保留在全局的命名空间内。

### 配置

#### 迁移环境变量

复制新的 `.env.example` 文件到 `.env`，在 `5.0` 中这相当于原本的 `.env.php`。在此设置合适的值，如 `APP_ENV` 和 `APP_KEY` (你的加密密钥)、数据库凭证、缓存驱动与 session 驱动。

此外，将你原本的 `.env.php` 文件中自定义的设置值都复制并移到 `.env` (本机环境的实际设置值) 和 `.env.example` (给其他团队成员的范本教程)。

更多关于环境配置的信息，请查看[完整文档](/docs/{{version}}/installation#environment-configuration)。

> **注意：**在部署你的 Laravel 5 应用程序之前，你需要先在正式主机上放置 `.env` 文件并给其设置好适当的值。

#### 配置文件

Laravel 5.0 不再使用 `app/config/{environmentName}/` 目录结构来为指定环境提供配置文件。取而代之的是，将环境对应的各种设置值移到 `.env`，并接着借助 `env('key', 'default value')` 来获取配置文件的值。你可以在 `config/database.php` 配置文件中看到相关例子。

将配置文件放在 `config/` 目录下，来代表所有环境共用的配置文件，或是在使用 `env()` 来获取对应该环境的设置值。

请记住，如果你添加了其它的键值到 `.env` 文件中，记得在 `.env.example` 文件内也要添加相对应的键值。这将可以帮助其他团队成员创建他们自己的 `.env` 文件。

### 路由

复制粘贴你原本的 `routes.php` 文件到 `app/Http/routes.php`。

### 控制器

下一步，将你所有的控制器移到 `app/Http/Controllers` 目录下。因为在本指南中我们不打算迁移到完整的命名空间，请将 `app/Http/Controllers` 目录添加到 `composer.json` 的 `classmap` 属性中。接下来，你可以从 `app/Http/Controllers/Controller.php` 基底抽象类中移除命名空间。请确认你迁移过来的控制器都继承这个基类。

在你的 `app/Providers/RouteServiceProvider.php` 文件，设置 `namespace` 属性为 `null`。

### 路由过滤器

将过滤器逻辑绑定从 `app/filters.php` 复制到 `app/Providers/RouteServiceProvider.php` 的 `boot()` 方法。并在 `app/Providers/RouteServiceProvider.php` 添加 `use Illuminate\Support\Facades\Route;` 来继续使用 `Route` Facade。

你不需要移动任何 Laravel 4.0 中的默认过滤器，如 `auth` 和 `csrf`。他们已经内置其中，只是换作以中间件形式出现。那些在路由或控制器内有使用到旧有默认过滤器的 (例如，`['before' => 'auth']`) 请修改使用新的中间件 (例如，`['middleware' => 'auth']`)。

过滤器在 Laravel 5 中没有被移除。你仍然可以绑定并借由 `before` 和 `after` 使用你自己自定义的过滤器。

### 全局 CSRF

默认情况下，所有路由都会启用[CSRF 保护](/docs/{{version}}/routing#csrf-protection)。如果你想要关闭它们或是只想在特定路由手动开启，请从 `App\Http\Kernel` 的 `middleware` 数组移除这行：

    'App\Http\Middleware\VerifyCsrfToken',

如果你想要在其它地方使用它，添加此行到 `$routeMiddleware`：

    'csrf' => 'App\Http\Middleware\VerifyCsrfToken',

现在你可以在路由内使用 `['middleware' => 'csrf']` 添加中间件到各个路由/控制器上。想要了解更多关于中间件的信息，请查看[完整文档](/docs/{{version}}/middleware)。

### Eloquent 模型

你可以随意创建一个新的 `app/Models` 目录来放置你的 Eloquent 模型。同样地，这个目录必须添加到 `composer.json` 文件的 `classmap` 属性中。

将正在使用 `SoftDeletingTrait` 的模型更改为使用 `Illuminate\Database\Eloquent\SoftDeletes`。

#### Eloquent 缓存

Eloquent 不再提供 `remember` 方法来缓存查找结果。现在你有责任手动地使用 `Cache::remember` 方法缓存查找结果。要了解更多关于缓存的信息，请查看[完整文档](/docs/{{version}}/cache)。

### 用户认证模型

要使用 Laravel 5 的认证系统，请遵循以下指引来升级你的 `User` 模型：

**从你的 `use` 区块删除以下内容：**

```php
use Illuminate\Auth\UserInterface;
use Illuminate\Auth\Reminders\RemindableInterface;
```

**添加以下内容到你的 `use` 区块：**

```php
use Illuminate\Auth\Authenticatable;
use Illuminate\Auth\Passwords\CanResetPassword;
use Illuminate\Contracts\Auth\Authenticatable as AuthenticatableContract;
use Illuminate\Contracts\Auth\CanResetPassword as CanResetPasswordContract;
```

**移除 UserInterface 和 RemindableInterface 接口。**

**标记类实现以下接口：**

```php
implements AuthenticatableContract, CanResetPasswordContract
```

**在类声明引入以下 traits：**

```php
use Authenticatable, CanResetPassword;
```

**如果你有使用它们，请把 `Illuminate\Auth\Reminders\RemindableTrait` 和 `Illuminate\Auth\UserTrait` 从你的 use 区块和类声明中移除。**

### Cashier 的用户需要做的修改

[Laravel Cashier](/docs/{{version}}/billing) 使用的 trait 和接口名称已作修改。trait 请改用 `Laravel\Cashier\Billable` 取代 `BillableTrait`。接口实现请用 `Laravel\Cashier\Contracts\Billable` 取代 `Laravel\Cashier\BillableInterface`。不需要修改其它任何方法。

### Artisan 命令

将你所有的命令类从原本的 `app/commands` 目录移到新的 `app/Console/Commands` 目录。接下来，把 `app/Console/Commands` 目录添加到 `composer.json` 文件的 `classmap` 属性中。

然后，把你的 Artisan 命令清单从 `start/artisan.php` 复制到 `app/Console/Kernel.php` 文件里的 `command` 数组中。

### 数据库迁移和数据填充

因为你的数据库里应该已经有 users 表了，请删除 Laravel 5.0 内置的两个迁移文件。

将你所有的迁移文件从原本的 `app/database/migrations` 目录移到新的 `database/migrations`。你所有的数据填充档也应该从 `app/database/seeds` 移到 `database/seeds`。

### 全局 IoC 绑定

如果你在 `start/global.php` 有任何的[服务容器](/docs/{{version}}/container)绑定，请将它们全部移至 `app/Providers/AppServiceProvider.php` 文件的 `register` 方法。你可能需要引入 `App` facade。

你也可以选择将这些绑定，依照类拆分到各别的服务提供者中。

### 视图

将你的视图从 `app/views` 移到新的 `resources/views` 目录。

### Blade 标签修改

基于安全考量，Laravel 5.0 会把所有 `{{ }}` 和 `{{{ }}}` Blade 标签的输出的特殊字符都进行转译。新的 `{!! !!}` 标签则被采用来显示原始未转译的输出。当你**有足够把握**来保证显示的原始输出内容是安全的，那么升级你的应用程序的最安全方法是只使用新的 `{!! !!}` 标签。

然而，如果你**必须**使用旧的 Blade 语法，请在  `AppServiceProvider@register` 的结尾加入以下几行：

```php
\Blade::setRawTags('{{', '}}');
\Blade::setContentTags('{{{', '}}}');
\Blade::setEscapedContentTags('{{{', '}}}');
```

上述设置你不该轻易使用，这将使你的应用程序更加容易遭受 XSS 攻击。而且用 `{{--` 注释将无法作用。

### 语言包

将你的语言包从 `app/lang` 移到新的 `resources/lang` 目录。

### 公开目录

从你的 `4.2` 应用程序的 `public` 目录内复制公开的 assets 到新应用程序的 `public` 目录内。请确定有保留 `5.0` 版本的 `index.php`。

### 测试

将你的测试从 `app/tests` 移到新的 `tests` 目录。

### 杂项文件

复制项目内任何其它的文件，例如： `.scrutinizer.yml`, `bower.json` 以及其它类似的工具配置文件。

你可以将 Sass、Less 或 CoffeeScript 移动到任何你想放置的地方。`resources/assets` 目录是一个不错的默认位置。

### 表单和 HTML 辅助函数

如果你使用表单或 HTML 辅助函数，你将会看到 `class 'Form' not found` 或 `class 'Html' not found` 的错误。表单和 HTML 辅助函数已经在 Laravel 5.0 中被弃用。然而，有些社区导向的替代品可供替代，例如：[Laravel Collective](http://laravelcollective.com/docs/{{version}}/html) 维护的这些。

举例来说，你可以把 `"laravelcollective/html": "~5.0"` 添加到你的 `composer.json` 的 `require` 区块。

你也需要表单和 HTML 的 facades 以及服务提供者。编辑 `config/app.php` 并添加这行到 'providers' 数组内：

    'Collective\Html\HtmlServiceProvider',

接着，添加这几行到 'aliases' 数组内：

    'Form' => 'Collective\Html\FormFacade',
    'Html' => 'Collective\Html\HtmlFacade',

### 缓存管理员

如果你的代码有注入 `Illuminate\Cache\CacheManager` 来获取非 Facade 版本的 Laravel 缓存，请改成注入 `Illuminate\Contracts\Cache\Repository` 取代。

### 分页

替换所有的 `$paginator->links()` 为 `$paginator->render()`。

依次替换所有的 `$paginator->getFrom()` 和 `$paginator->getTo()` 为 `$paginator->firstItem()` 和 `$paginator->lastItem()`。

从 `$paginator->getPerPage()`、`$paginator->getCurrentPage()`、`$paginator->getLastPage()` 和 `$paginator->getTotal()` 移除「get」前缀 (例如：`$paginator->perPage()`)。

### Beanstalk 队列

Laravel 5.0 现在需要 `"pda/pheanstalk": "~3.0"` 取代原本的 `"pda/pheanstalk": "~2.1"`。

### Remote

Remote 组件已被弃用。

### Workbench

Workbench 组件已被弃用。

<a name="upgrade-4.2"></a>
## 从 4.1 升级到 4.2

### PHP 5.4+

Laravel 4.2 需要 PHP 5.4.0 或更高的版本。

### 默认加密

在你的 `app/config/app.php` 配置文件中添加一个新的 `cipher` 选项。这个选项的值应该为 `MCRYPT_RIJNDAEL_256`。

    'cipher' => MCRYPT_RIJNDAEL_256

这个设置可以被用来调整 Laravel 加密工具使用的默认加密方法。

> **注意：**在 Laravel 4.2，默认的加密方法为 `MCRYPT_RIJNDAEL_128` (AES)，它认为是最安全的加密方法。必须将加密方法改回 `MCRYPT_RIJNDAEL_256` 来解密在 Laravel 4.1 以前版本下加密的 cookies 和值。

### 现在在可以软删除的模型上使用 Traits

如果你有使用可以软删除的模型，`softDeletes` 属性已经被移除。现在你必须使用 `SoftDeletingTrait` 如下：

    use Illuminate\Database\Eloquent\SoftDeletingTrait;

    class User extends Eloquent
    {
        use SoftDeletingTrait;
    }

你也必须手动地添加 `deleted_at` 字段到你的 `dates` 属性中：

    class User extends Eloquent
    {
        use SoftDeletingTrait;

        protected $dates = ['deleted_at'];
    }

而所有软删除的 API 使用方式保持不变。

> **注意：**`SoftDeletingTrait` 无法在模型基类下被使用。他必须用在一个实际的模型类。

### 视图 / 分页的环境名称修改

如果你直接参照 `Illuminate\View\Environment` 或 `Illuminate\Pagination\Environment` 类， 请更新你的代码将其改为参照 `Illuminate\View\Factory` 和 `Illuminate\Pagination\Factory`。改名后的这两个类更可以代表他们的功能。

### 额外的参数 On Pagination Presenter

如果你扩展了 `Illuminate\Pagination\Presenter` 类，抽象方法 `getPageLinkWrapper` 的参数表变成要加上 `rel` 参数：

    abstract public function getPageLinkWrapper($url, $page, $rel = null);

### Iron.Io 队列加密

如果你有使用 Iron.io 队列驱动，你需要添加一个新的 `encrypt` 选项到你的 queue 配置文件中：

    'encrypt' => true

<a name="upgrade-4.1.29"></a>
## 从 4.1.x 以前版本升级到 4.1.29

Laravel 4.1.29 对于所有的数据库驱动加强了 column quoting 的部分。当你的模型中**没有**使用 `fillable` 属性时，他保护你的应用程序不会受到 mass assignment 漏洞影响。如果你在模型中使用 `fillable` 属性来防范 mass assignment，你的应用程序将不会有漏洞。然而，如果你使用 `guarded` 并传递用户控制的数组到「更新」或「保存」类型的函数中，那你应该立即升级到 `4.1.29` 以避免你的应用程序遭受 mass assignment 的风险。

要升级到 Laravel 4.1.29，只要 `composer update` 即可。在这个发行版本中没有重大的更新。

<a name="upgrade-4.1.26"></a>
## 从 4.1.25 以前版本升级到 4.1.26

Laravel 4.1.26 针对「记得我」cookies 的安全性进行了更新。在此更新之前，如果一个 「记得我」cookie 被恶意用户劫持，该 cookie 在很长一段时间内仍然有效，即便真实的用户进行了重设密码或者注销等操作。

此更动需要在你的 `users` (或同等的) 数据表中添加一个额外的 `remember_token` 字段。在更新之后，当用户每次登录你的应用程序将会被给予一个全新的 token。在用户注销应用程序后，这个 token 也会被更新。这个更新的影响为：如果一个「记得我」的 cookie 被劫持，只要用户注销应用程序该 cookie 将会失效。

### 升级路径

首先，添加一个新的 `remember_token` 字段到你的 `users` 数据表中，它可为空值且为 VARCHAR(100)、TEXT 或同等的类型。

然后，如果你是使用 Eloquent 认证驱动，请更新你的 `User` 类的以下三个方法：

    public function getRememberToken()
    {
        return $this->remember_token;
    }

    public function setRememberToken($value)
    {
        $this->remember_token = $value;
    }

    public function getRememberTokenName()
    {
        return 'remember_token';
    }

> **注意：**所有现存的「记得我」sessions 在此更新后将会失效，所以应用程序的所有用户将会被迫重新认证。

### 扩展包维护者

`Illuminate\Auth\UserProviderInterface` 接口加了两个新的方法。简单的实现可以在默认驱动中找到：

    public function retrieveByToken($identifier, $token);

    public function updateRememberToken(UserInterface $user, $token);

`Illuminate\Auth\UserInterface` 也添加了三个新方法并被描述在「升级路径」部分。

<a name="upgrade-4.1"></a>
## 从 4.0 升级到 4.1

### 升级你的 Composer 依赖

要将你的应用程序升级至 Laravel 4.1，则必须将应用程序的 `composer.json` 里的 `laravel/framework` 版本更改至 `4.1.*`。

### 文件替换

将你的 `public/index.php` 文件替换为[这个从 repository 复制的全新文件](https://github.com/laravel/laravel/blob/v4.1.0/public/index.php)。

将你的 `artisan` 文件替换为[这个从 repository 复制的全新文件](https://github.com/laravel/laravel/blob/v4.1.0/artisan)。

### 添加配置文件及选项

更新 `app/config/app.php` 配置文件里的 `aliases` 和 `providers` 数组。这些数组更新后的值可以在[这个文件](https://github.com/laravel/laravel/blob/v4.1.0/app/config/app.php)中找到。请确保你自定义的 providers / aliases 和扩展包的 providers / aliases 都已添加回数组中。

[从 repository](https://github.com/laravel/laravel/blob/v4.1.0/app/config/remote.php) 添加新的 `app/config/remote.php` 文件。

在你的 `app/config/session.php` 文件里，添加新的 `expire_on_close` 设置选项。默认值应该为 `false`。

在你的 `app/config/queue.php` 文件里，添加新的 `failed` 设置区块。以下为这区块的默认值：

    'failed' => [
        'database' => 'mysql', 'table' => 'failed_jobs',
    ],

**(可选的)**  在你的 `app/config/view.php` 文件里，将 `pagination` 设置选项更新为 `pagination::slider-3`。

### 控制器的修改

如果 `app/controllers/BaseController.php` 有个 `use` 语句在最上面，将 `use Illuminate\Routing\Controllers\Controller;` 改为 `use Illuminate\Routing\Controller;`。

### 密码提醒的修改

密码提醒功能在大幅改进后已经具有更好的灵活性。你可以运行 `php artisan auth:reminders-controller` Artisan 命令来检查新的存根控制器。你也可以浏览 [更新后的文档](/docs/security#password-reminders-and-reset) 来相应更新你的应用程序。

更新你的 `app/lang/en/reminders.php` 语言文件来对应[这个新版文件](https://github.com/laravel/laravel/blob/v4.1.0/app/lang/en/reminders.php)。

### 环境侦测的修改

为了安全因素，不再使用网址域名来侦测应用程序的环境。因为这些值很容易被伪造欺骗，继而让攻击者通过请求来修改环境。你必须改为使用机器的主机名称 (在 Mac、Linux 和 Windows 下运行 `hostname` 命令的值) 来侦测环境。

### 更简单的日志文件

Laravel 目前只会产生单个的日志文件：`app/storage/logs/laravel.log`。然而，你还是可以在 `app/start/global.php` 文件设置更改它的行为。

### 移除重定向向结尾的斜线

在你的 `bootstrap/start.php` 文件中，移除对 `$app->redirectIfTrailingSlash()` 的调用。这个方法已经不再需要，因为这个功能现在已交由框架内的 `.htaccess` 文件来处理。

然后，用 [新的文件](https://github.com/laravel/laravel/blob/v4.1.0/public/.htaccess) 替换为 Apache 的 `.htaccess`，来处理结尾的斜线。

### 获取目前路由

现在通过 `Route::current()` 获取当前路由，而不是 `Route::getCurrentRoute()`。

### Composer 更新

一旦你完成以上更新，则可以运行 `composer update` 功能来更新应用程序的核心文件！如果发生 class load 错误，试着运行 `update` 命令并加上 `--no-scripts` 选项，样就像这样：`composer update --no-scripts`。

### 通配符事件监听者

通配符事件监听者不再添加事件到你的处理函数参数上。如果你需要寻找触发的事件，则可用 `Event::firing()` 来触发。
