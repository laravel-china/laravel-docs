# Laravel 升级索引

- [从 5.3 升级到 5.4.0](#upgrade-5.4.0)

<a name="upgrade-5.4.0"></a>
## 从 5.3 升级到 5.4.0 

#### 预计升级耗时: 1-2 小时

> {note} 我们尽量罗列出每一个不兼容的变更。但因为其中一些不兼容变更只存在于框架很不起眼的地方，事实上只有一小部分会真正影响到你的应用程序。

### 更新依赖

在 `composer.json` 文件中将你的 `laravel/framework` 更新为 `5.4.*`。 此外, 你应该更新你的 `phpunit/phpunit` 依赖关系到 `~5.7`。

#### 删除编译的服务文件

如果存在，你可以删除 `bootstrap/cache/compiled.php` 文件。 它不再被框架使用。

#### 刷新缓存

升级所有软件包之后，你应该运行 `php artisan view:clear` 来避免与删除 `Illuminate\View\Factory::getFirstLoop()` 相关的 Blade 错误。此外，你需要运行 `php artisan route:clear` 来刷新路由缓存。

#### Laravel Cashier

Laravel Cashier 已经兼容 Laravel 5.4。

#### Laravel Passport

为了兼容 Laravel 5.4 和 [Axios](https://github.com/mzabriskie/axios) JavaScript 库，Laravel Passport 已经发布了 `2.0.0` 。如果你是从 Laravel 5.3 升级，并使用了预置的 Passport Vue 组件，你应该确保 Axios 库在你的应用程序中作为 `axios` 全局可用。

#### Laravel Scout

Laravel Scout `3.0.0` 已经发布，以提供与 Laravel 5.4 的兼容性。

#### Laravel Socialite

Laravel Socialite `3.0.0` 已经发布，以提供与 Laravel 5.4 的兼容性。

#### Laravel Tinker

为了继续使用 `tinker` Artisan 命令，你还应该安装 `laravel/tinker` 包:

    composer require laravel/tinker

一旦软件包被安装，你应该添加 `Laravel\Tinker\TinkerServiceProvider::class` 到 `config/app.php` 配置文件中的 `providers`数组。

#### Guzzle

Laravel 5.4 需要 Guzzle 6.0或者更高版本。

### 授权

#### `getPolicyFor` 方法

在之前的版本中，调用 `Gate::getPolicyFor($class)` 方法的时候，如果没有找到对应的 policy，会抛出异常。 现在, 该方法在给定类中找不到policy的时候会返回  `null`。 如果你直接调用这个方法，请确保你在重构代码中新增了对 `null`的检查:

```php
$policy = Gate::getPolicyFor($class);

if ($policy) {
    // code that was previously in the try block
} else {
    // code that was previously in the catch block
}
```

### Blade

#### `@section` 转码

在 Laravel 5.4，传递给 section 的内联内容会自动进行转码：

    @section('title', $content)

如果你想要在 section 中渲染原生内容，你必须使用传统的「长形式」声明该 section。

    @section('title')
        {!! $content !!}
    @stop

### 引导程序

如果你在 HTTP 或 Console 启动类中手动覆盖了 `$bootstrappers` 数组，需要将 `DetectEnvironment` 重命名为 `LoadEnvironmentVariables`。

### 广播

#### 频道模型绑定

在 Laravel 5.3 中定义频道名称占位符时，使用 `*` 字符。在 Laravel 5.4 中， 你应该使用 `{foo}` 风格来定义这些占位符，如路由：

    Broadcast::channel('App.User.{userId}', function ($user, $userId) {
        return (int) $user->id === (int) $userId;
    });

### 集合

#### `every` 方法

`every` 方法的功能被合并到 `nth` 方法中以匹配被 Lodash 定义的方法名。

#### `random` 方法

调用 `$collection->random(1)` 现在会返回一个包含单个item的新的集合实例。在之前版本中，这个方法会返回单个对象。如果没有提供参数，该方法将只返回单个对象。

### 容器

#### 别名 via `bind` / `instance`

在之前的 Laravel 版本中，你可以将数组作为第一个参数传递给 `bind` 或者 `instance` 方法来注册别名：

    $container->bind(['foo' => FooContract::class], function () {
        return 'foo';
    });

然而，这种行为已经在 Laravel 5.4 中删除。要注册别名，你应该使用 `alias` 方法：

    $container->alias(FooContract::class, 'foo');

#### 使用 \ 绑定类

将不再支持通过带有 \ 的类名绑定类到容器。因为该功能需要在底层容器中进行大量的字符串格式化调用。取而代之的，只需要通过下面这种不带 \ 的方式绑定即可：
    
    $container->bind('Class\Name', function () {
        //
    });
    
    $container->bind(ClassName::class, function () {
        //
    });

#### `make` 方法参数

容器的 `make` 方法不再接受第二个数组参数，因为该特性表明了代码的坏味道。我们需要以更加直观的方式来构造对象。

#### 解决回调

容器的 `resolving` 和 `afterResolving` 方法现在必须提供一个类名或绑定键作为方法的第一个参数：

    $container->resolving('Class\Name', function ($instance) {
        //
    });

    $container->afterResolving('Class\Name', function ($instance) {
        //
    });

#### `share` 方法已删除

`share` 方法已经从容器中删除。这是一个遗留的方法，多年未被列出来。如果你使用这个方法，你应该开始使用 `singleton` 方法：

    $container->singleton('foo', function () {
        return 'foo';
    });

### 控制台

#### The `Illuminate\Console\AppNamespaceDetectorTrait` Trait

如果你直接引用 `Illuminate\Console\AppNamespaceDetectorTrait` trait，更新你的代码替换为 `Illuminate\Console\DetectsApplicationNamespace`。

### 数据库

#### 自定义连接

如果你之前为 `db.connection.{driver-name}` 绑定了一个服务容器绑定以便解析自定义数据库连接实例，现在需要在 `AppServiceProvider` 的 `register` 方法中使用 `Illuminate\Database\Connection::resolverFor` 方法：

    use Illuminate\Database\Connection;

    Connection::resolverFor('driver-name', function ($connection, $database, $prefix, $config) {
        //
    });

#### Fetch 模式

Laravel不再支持在配置文件中定制PDO 「fetch mode」 的功能。取而代之，  `PDO::FETCH_OBJ` 一直可以使用。如果你仍然想要为你的应用程序自定义提取模式，你可以监听新的 `Illuminate\Database\Events\StatementPrepared` 事件：

    Event::listen(StatementPrepared::class, function ($event) {
        $event->statement->setFetchMode(...);
    });

### Eloquent

#### 日期转换

日期转换 `date` 现在会将日期转化为 `Carbon` 对象并调用该对象上的  `startOfDay` 方法，如果你想保留日期的时间部分，需要使用 `datetime` 转换。

#### 外键约束

在定义关联关系的时候如果外键没有被显式指定，Eloquent 将会使用关联模型的表名和主键名来构建外键。这适用于大多数应用，这不是行为的改变。例如：

    public function user()
    {
        return $this->belongsTo(User::class);
    }

就像以前的 Laravel 版本一样，这种关系通常使用 `user_id` 作为外键。但是，如果你重写 `User` 模型的 `getKeyName` 方法，情况就会变得不一样，例如：

    public function getKeyName()
    {
        return 'key';
    }

在这种情况下，Laravel 会以你自定义的主键名为准，外键名将会是 `user_key` 而不是 `user_id`。

#### 一对一 / 多的 `createMany`

`hasOne` or `hasMany` 关系的 `createMany` 方法现在返回一个集合对象而不是一个数组。

#### 相关模型连接

相关模型现在将使用与父模型相同的连接。例如，你执行以下查询：

    User::on('example')->with('posts');

Eloquent 将查询 `example` 连接上的 posts 表，而不是默认的数据库连接。如果你想从默认连接读取 `posts` 关系，你应该明确地设置模型的连接到你的应用程序的默认连接。

####`create` & `forceCreate` 方法

`Model::create` & `Model:: forceCreate` 方法已经移动到  `Illuminate\Database\Eloquent\Builder` 类，以便为在多个连接上创建模型提供更好的支持。但是，如果要在自己的模型中扩展这些方法，则需要修改实现以在构建器上调用 `create` 方法。例如：

    public static function create(array $attributes = [])
    {
        $model = static::query()->create($attributes);

        // ...

        return $model;
    }

####`hydrate` 方法

如果你现在传递自定义的连接名到该方法，需要使用 `on` 方法：

    User::on('connection')->hydrate($records);

####`hydrateRaw` 方法

`Model::hydrateRaw` 方法已经被重命名为 `fromQuery` ，如果你传递自定义的连接名到该方法，需要使用 `on` 方法：

    User::on('connection')->fromQuery('...');

#### `whereKey` 方法

`whereKey($id)` 方法现在将为给定的主键值添加一个 「where」 子句。 在之前的版本中，这将属于动态 「where」子句构建器，并为 「key」 列添加 「where」子句。如果你使用 `whereKey` 方法为 `key` 列动态添加一个条件，你现在应该使用  `where('key', ...)` 。

#### `factory` Helper

调用 `factory(User::class, 1)->make()` 或 `factory(User::class, 1)->create()` 现在将返回一个集合。以前，这将返回单个模型。如果没有传入数量的话，返回的仍然只是单个模型。

### 事件

#### Contract 变更

如果你在应用程序或者包中手动实现 `Illuminate\Contracts\Events\Dispatcher` 接口，则应该将 `fire` 方法重命名为 `dispatch`。

#### 事件优先级

不再支持事件处理器 「优先级」 ，从未记录的功能通常表示事件功能的滥用。相反，请考虑使用一系列同步方法调用。或者，你可以从另一个事件的处理程序中分派一个新事件，以确保给定事件的处理程序不在相关的处理程序之后触发。

#### 通配符事件处理器签名

通配符事件处理器现在接收事件名作为第一个参数以及事件数据作为第二个参数，  `Event::firing` 方法被移除：

    Event::listen('*', function ($eventName, array $data) {
        //
    });

#### `kernel.handled` 事件

`kernel.handled` 事件现在是一个基于对象的事件，使用  `Illuminate\Foundation\Http\Events\RequestHandled` 类。

#### `locale.changed` 事件

`locale.changed` 事件现在是一个基于对象的事件，使用 `Illuminate\Foundation\Events\LocaleUpdated` 类。

#### `illuminate.log` 事件

`illuminate.log` 事件现在是一个基于对象的事件，使用  `Illuminate\Log\Events\MessageLogged` 类。

### 异常

`Illuminate\Http\Exception\HttpResponseException` 已被重命名为  `Illuminate\Http\Exceptions\HttpResponseException` 。注意  `Exceptions` 现在是复数。 类似的，  `Illuminate\Http\Exception\PostTooLargeException` 已经被重命名为  `Illuminate\Http\Exceptions\PostTooLargeException`。

### 邮件

#### `Class@method` 语法

不再支持使用 `Class@method` 语法发邮件。例如：

    Mail::send('view.name', $data, 'Class@send');

如果你是以这种方式发送邮件，需要将这些调用做转化 [mailables](/docs/{{version}}/mail) 。

#### 新的配置选项

为了支持 Laravel 5.4 版本的 Markdown 邮件组件，你需要添加如下区域配置到 `mail` 配置文件底部：

    'markdown' => [
        'theme' => 'default',

        'paths' => [
            resource_path('views/vendor/mail'),
        ],
    ],

#### 使用闭包将邮件推送到队列

为了将邮件推送到队列，必须使用 [mailable](/docs/{{version}}/mail) 提供的方法。 使用 `Mail::queue` 和 `Mail::later` 方法将邮件推送到队列，不再支持使用闭包来配置邮件消息。该功能需要使用指定的库来序列化闭包，因为PHP原生不支持这一功能特性。

### Redis

#### 改进的集群支持

Laravel 5.4 引入了改进的 Redis 集群支持。 如果你正在使用 Redis 集群，则应该将集群连接置于 `config/database.php` 配置文件的 Redis 部分中的 `clusters`  配置选项内：

    'redis' => [

        'client' => 'predis',

        'options' => [
            'cluster' => 'redis',
        ],

        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', '127.0.0.1'),
                    'password' => env('REDIS_PASSWORD', null),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],

    ],

### 路由

#### Post 大小中间件

`Illuminate\Foundation\Http\Middleware\VerifyPostSize` 类被重命名为  `Illuminate\Foundation\Http\Middleware\ValidatePostSize`。

#### `middleware` 方法

`Illuminate\Routing\Router` 类的 `middleware` 方法已重命名为 `aliasMiddleware()` 。 似乎大多数应用都不会直接手动调用这个方法，因为这个方法只会被 HTTP kernel 调用，用于注册定义在 `$routeMiddleware` 数组中的路由级中间件。

#### `Route` 方法

`Illuminate\Routing\Route` 类的 `getUri` 方法已删除。你应该使用 `uri` 方法。

`Illuminate\Routing\Route` 类的 `getMethods` 方法已经删除。你应该使用  `methods` 方法。

`Illuminate\Routing\Route` 类的 `getParameter` 方法已经删除。你应该使用 `getParameter` 方法。

`Illuminate\Routing\Route` 类的 `getPath` 方法已经删除。你应该使用 `uri` 方法。

### 会话

#### Symfony兼容性

Laravel 的会话处理程序不再实现 Symfony 的 `SessionInterface` 。 
实现这个接口需要我们实现框架不需要的无关特性。取而代之，已经定义了新的 `Illuminate\Contracts\Session\Session` 接口，并且可以使用。还应该修改一下代码：

所有调用 `->set()` 方法应该更改为 `->put()` 。通常，Laravel 应用从不调用  `set` 方法，因为它从未在 Laravel 文档中记录。不过，谨慎起见，这里我们依然罗列出来。

所有调用 `->getToken()` 方法的地方需要修改为 `->token()` 。

所有调用 `$request->setSession()` 方法的地方需要求改为  `setLaravelSession()` 。

### 测试

Laravel 5.4 的测试层已经被重成更加简单和更加轻量级。如果你想继续使用 Laravel  5.3 中的测试层，你可以安装 `laravel/browser-kit-testing` [package](https://github.com/laravel/browser-kit-testing) 到你的应用程序。该包提供了与 Laravel 5.3 测试层的完全兼容。实际上，你可以同时运行 Laravel 5.3 和 Laravel 5.4 的测试层。

#### 在一个应用里面运行 Laravel 5.3 & 5.4 的测试

在开始之前，你应该将 `Tests` 命名空间添加到 `composer.json` 文件中的 `autoload-dev` 块中。这将允许 Laravel 自动加载你使用 Laravel 5.4 测试生成器生成的任何新测试：

    "psr-4": {
        "Tests\\": "tests/"
    }

接下来, 安装 `laravel/browser-kit-testing` 包：

    composer require laravel/browser-kit-testing

安装软件包后，创建 `tests/TestCase.php` 文件的副本，并将其作为 `BrowserKitTestCase.php` 保存到你的 `tests` 文件夹中。然后，修改该文件以扩展  `Laravel\BrowserKitTesting\TestCase` 类。完成之后，你应该在你的 `tests` 目录中有两个基本测试类： `TestCase.php` 和 `BrowserKitTestCase.php` 。为了正确加载你的  `BrowserKitTestCase` 类，你可能需要将它添加到你的 `composer.json` 文件：

    "autoload-dev": {
        "classmap": [
            "tests/TestCase.php",
            "tests/BrowserKitTestCase.php"
        ]
    },

在 Laravel 5.3 上编写的测试将扩展 `BrowserKitTestCase` 类，而使用 Laravel 5.4 测试层的任何新测试将扩展 `TestCase` 类。你的 `BrowserKitTestCase` 类应该如下所示：

    <?php

    use Illuminate\Contracts\Console\Kernel;
    use Laravel\BrowserKitTesting\TestCase as BaseTestCase;

    abstract class BrowserKitTestCase extends BaseTestCase
    {
        /**
         * 应用的基础 URL 。
         *
         * @var string
         */
        public $baseUrl = 'http://localhost';

        /**
         * 创建应用。
         *
         * @return \Illuminate\Foundation\Application
         */
        public function createApplication()
        {
            $app = require __DIR__.'/../bootstrap/app.php';

            $app->make(Kernel::class)->bootstrap();

            return $app;
        }
    }

创建好这些类之后，确保更新所有已编写的测试类继承自 `BrowserKitTestCase` 类，这将得所有基于 Laravel 5.3 编写的测试类在 Laravel 5.4 中得以继续运行。如果你选择，你可以慢慢地开始将它们移植到新的 [Laravel 5.4 测试语法](/docs/5.4/http-tests) 或者  [Laravel Dusk](/docs/5.4/dusk)。

> {note} 如果你正在编写新的测试，并且希望它们使用 Laravel 5.4 的测试层，确保继承自  `TestCase` 类。

#### 在已升级应用中安装 Duck

如果你想在已升级应用中安装 Laravel Duck ，首先通过 Composer 安装：

    composer require laravel/dusk

记下来，你需要在 `tests` 目录中创建一个 `CreatesApplication` trait ，该 trait 用于为测试用例创建新的应用实例。该 trait 代码如下： 

    <?php

    use Illuminate\Contracts\Console\Kernel;

    trait CreatesApplication
    {
        /**
         * 创建应用。
         *
         * @return \Illuminate\Foundation\Application
         */
        public function createApplication()
        {
            $app = require __DIR__.'/../bootstrap/app.php';

            $app->make(Kernel::class)->bootstrap();

            return $app;
        }
    }

> {note} 如果你的测试类位于某个命名空间下，并使用 PSR-4 自动加载标准来加载 `tests`  目录，则应该将 `CreatesApplication` trait 放置在合适的命名空间下。

完成了这些准备工作，你就可以按照正常的 [Dusk 安装指南](/docs/{{version}}/dusk#installation) 进行操作。

#### 环境

Laravel 5.4 不需要在每个测试类中手动强制设置 `putenv('APP_ENV=testing')`，取而代之，框架使用从 `.env` 文件中载入 `APP_ENV` 变量。

#### 时间伪造

`Event` 伪造的 `assertFired` 方法应该更新为 `assertDispatched`，并且  `assertNotFired` 方法应该更新为 `assertNotDispatched` 。方法的签名没有变更。

#### 邮件伪造

`Mail` 伪造在 Laravel 5.4 中极大的简化。现在应该使用 `assertSent` 方法以及 `hasTo`，`hasCc` 等辅助函数即可即可，不再需要调用 `assertSentTo` 方法：

    Mail::assertSent(MailableName::class, function ($mailable) {
        return $mailable->hasTo('email@example.com');
    });

### 翻译

#### `{Inf}` 占位符

如果你在使用 `{Inf}` 占位符处理字符串复数格式的翻译，你需要更新为使用 `*` 字符：

    {0} First Message|{1,*} Second Message

### URL 生成

#### `forceSchema` 方法

`Illuminate\Routing\UrlGenerator` 类中的 `forceSchema` 方法已经重命名为  `forceScheme`。

### 验证

#### 日期格式验证

日期格式验证现在更加严格，并支持对 PHP [date function](http://php.net/manual/en/function.date.php) 中列出的占位符进行校验。 在之前版本中，时区占位符 `P` 会接受所有的时区格式； 然而，在 Laravel 5.4 中，每种时区格式像在 PHP 文档一样，都有一个唯一的占位符。

#### 方法名

`addError` 方法已经重命名为 `addFailure`。此外，`doReplacements` 方法重命名为  `makeReplacements`。通常，这些改变只有当你去扩展 `Validator` 类时才会有影响。

### 杂项

我们同样鼓励你去翻阅 `laravel/laravel` [GitHub 仓库](https://github.com/laravel/laravel) 中的修改记录。虽然很多修改不是必需的，你可能希望这些文件与你的应用程序同步。另外的一些变更将在本升级指南中介绍，但是其他更改不会。例如更改配置文件或者注释。你可以通过 [Github comparison tool](https://github.com/laravel/laravel/compare/5.3...master) 很轻松的查看到这些变更，并选择对你重要的更新。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@skyverd](https://laravel-china.org/users/79)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/79_1427370664.jpeg?imageView2/1/w/100/h/100">  |  翻译  | 全桟工程师，[时光博客](https://skyverd.com) |