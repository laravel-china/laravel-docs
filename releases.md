# 发行说明

- [支持政策](#support-policy)
- [Laravel 5.1.11](#laravel-5.1.11)
- [Laravel 5.1.4](#laravel-5.1.4)
- [Laravel 5.1](#laravel-5.1)
- [Laravel 5.0](#laravel-5.0)
- [Laravel 4.2](#laravel-4.2)
- [Laravel 4.1](#laravel-4.1)

<a name="support-policy"></a>
## 发行说明

Laravel 5.1 LTS 版本会提供两年的 BUG 修复及三年的安全性修复，LTS 版本是 Laravel 能提供的支持及维护最长时间支持的发行版。

对于一般的版本，会提供六个月的 BUG 修复及一年的安全性修复。

<a name="laravel-5.1.11"></a>
## Laravel 5.1.11

Laravel 5.1.11 推出了内置的[授权](/docs/{{version}}/authorization)支持！通过回调或授权策略类方便的组织你应用程序的授权逻辑，使用简单明了的方法对行为进行授权。

更多的信息请参考[授权的文档](/docs/{{version}}/authorization)。

<a name="laravel-5.1.4"></a>
## Laravel 5.1.4

Laravel 5.1.4 为框架推出了简单的登录限制。查阅 [认证的文档](/docs/{{version}}/authentication#authentication-throttling)以获取更多信息。

<a name="laravel-5.1"></a>
## Laravel 5.1

Laravel 5.1 继续以 Laravel 5.0 改进而成，通过采用 PSR-2 及添加事件广播、中间件参数、Artisan 的改进及其它等等。

### PHP 5.5.9+

由于 PHP 5.4 将在九月「结束寿命」，不再接收来自 PHP 开发团队的安全性更新，Laravel 需要 PHP 5.5.9 或更高的版本。PHP 5.5.9 可以兼容最新版本的 PHP 函数库，像是 Guzzle 及 AWS SDK。

### LTS

Laravel 5.1 是 Laravel 生态系统内 **长期支持** 的第一个版本。Laravel 5.1 会获得两年的 BUG 修复及三年的安全性修复，此策略也使 Laravel 能更好的服务于较大型的企业客户及消费者。

### PSR-2

[PSR-2 代码风格指南](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) 已经被 Laravel 框架采用为默认的代码风格指南。此外，所以的生成器都已进行更新，以生成 PSR-2 兼容的语法。

### 文档

Laravel 文档的每一页已被精心审阅，并得到显著的改善。所有的代码例子也进行了严密检查，并以提供更多的上下文关联性与情境。

### 事件广播

在许多现代的 Web 应用程序，WebSockets 都用来实现即时更新用户状态，当在服务器上更新一些数据，WebSocket 连接通常发送一个消息通过客户端处理。

为了协助你创建这些类型的应用程序，Laravel 让你经由 WebSocket 连接来「广播」你的事件，广播事件让你能够在服务器端代码和客户端 JavaScript 框架间分享相同的事件名称。

了解更多关于事件广播，请查阅[事件的文档](/docs/{{version}}/events#broadcasting-events)。

### 中间件参数

中间件接收自定义传参，例如，如果应用程序要在运行特定操作之前，检查当前登录的用户是否具备该操作的「角色」，可以创建 `RoleMiddleware` 来接收角色名称作为额外的传参。

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class RoleMiddleware
    {
        /**
         * 运行请求过滤。
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // 重定向...
            }

            return $next($request);
        }

    }

在路由中可使用冒号 `:` 来区隔中间件名称与指派参数，多个参数可使用逗号作为分隔：

    Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
        //
    }]);

关于中间件的更多信息，请查阅[中间件的文档](/docs/{{version}}/middleware)。

### 测试改进

Laravel 内置的测试功能已得到显著的改善。各种新的方法提供了流畅、简明的接口与应用程序进行交互，并检查响应。例如，查看下方的测试：

    public function testNewUserRegistration()
    {
        $this->visit('/register')
             ->type('Taylor', 'name')
             ->check('terms')
             ->press('Register')
             ->seePageIs('/dashboard');
    }

关于测试的更多信息，请查阅[测试的文档](/docs/{{version}}/testing)。

### 模型工厂

Laravel 现在提供一个简单的方式创建仿真的 Eloquent 模型，使用[模型工厂](/docs/{{version}}/testing#model-factories)。模型工厂让你简单的为 Eloquent 模型定义一组「默认」的属性，并为你的测试或数据填充生成测试模型实例。模型工厂也使用高级的 [Faker](https://github.com/fzaninotto/Faker) PHP 函数库来生成随机属性的数据：

    $factory->define(App\User::class, function ($faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->email,
            'password' => str_random(10),
            'remember_token' => str_random(10),
        ];
    });

更多关于模型工厂的消息，请查阅[它的文档](/docs/{{version}}/testing#model-factories)。

### Artisan 的改进

Artisan 命令现在可以使用简单的方式定义，相似于路由的「署名」，提供了一个非常简单的接口来定义命令行的参数及选项。举个例子，你可以定义一个简单的命令及它的选项，如下：

    /**
     * 命令行的名字及署名。
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--force}';

更多关于定义 Artisan 命令的消息，请参考 [Artisan 的文档](/docs/{{version}}/artisan)。

### 文件夹结构

为了更好地表明目的，`app/Commands` 目录已经被更名为 `app/Jobs`。此外，`app/Handler` 目录已经被合并成一个只包含事件侦听器的 `app/Listeners` 目录。但是，这不是一个重大的改变，你不必更新成新的文件夹结构也能使用 Laravel 5.1。

### 加密

在 Laravel 之前的版本，加密是通过 `mcrypt` PHP 扩展功能进行处理。不过，从 Laravel 5.1 起，加密通过更积极维护的 `openssl` 扩展功能进行处理。

<a name="laravel-5.0"></a>
## Laravel 5.0

Laravel 5.0 在默认的项目上引进了新的应用程序架构。新的架构提供了更好的基础在 Laravel 中创建强健的应用程序，以及在应用程序中全面采用新的自动加载标准（PSR-4）。首先，来查看一些主要更动：

### 新的目录结构

旧的 `app/models` 目录已经完全被移除。相对的，你所有的代码都放在 `app` 目录下，以及默认情况下使用 `App` 命名空间。这个默认的命名空间可以快速的使用新的 `app:name` Artisan 命令来做更改。

控制器、中间件，以及请求（Laravel 5.0 中新型态的类），现在分门别类的放在 `app/Http` 目录下，因为他们都与应用程序的 HTTP 传输层相关。除了一个路由设置的文件外，所有的中间件现在都分开成为独自的类档。

新的 `app/Providers` 目录取代了旧版 Laravel 4.x `app/start` 里的文件。这些服务提供者为应用程序提供了各种的引导功能，像是错误处理，日志纪录，路由加载等等。当然，你可以任意的创建新的服务提供者到应用程序中。

应用程序的语言文件和视图都被移到 `resources` 目录下。

### Contracts

所有 Laravel 主要组件实现所用的接口都放在 `illuminate/contracts` 保存库中。这个保存库没有其它的外部依赖。这些方便、集成的接口，可以让你用来让依赖注入变得低耦合，将可以简单作为 Laravel Facades 的替代选项。

更多关于 contracts 的信息，参考[完整文档](/docs/{{version}}/contracts)。

### 路由缓存

如果你的应用程序全部都是使用控制器路由，你可以使用新的 `route:cache` Artisan 命令来大幅度地加快注册路由。这对于有 100 个以上路由规则的应用程序很有用，可以**大幅度地**加快应用程序这部分的处理速度。

### 路由中间件

除了像 Laravel 4 风格的路由「过滤器」，Laravel 5 现在有 HTTP 中间件，而原本的认证和 CSRF 「过滤器」已经改写成中间件。中间件提供了单个、一致的接口取代了各种过滤器，让你在请求进到应用程序前，可以简单地检查甚至拒绝请求。

更多关于中间件的信息，参考[完整文档](/docs/{{version}}/middleware)。

### 控制器方法注入

除了之前有的建构函数注入外，你现在可以在控制器方法使用类型提示进行依赖注入。[服务容器](/docs/{{version}}/container)会自动注入依赖，即使路由包含了其它参数也不成问题：

    public function createPost(Request $request, PostRepository $posts)
    {
        //
    }

### 认证基本架构

用户注册，认证，以及重设密码的控制器现在已经默认含括了，包含相对应的视图，放在 `resources/views/auth`。除此之外，「users」数据表迁移也已经默认存在框架中了。这些简单的资源，可以让你快速开发应用程序的点子，而不用陷在编写认证模板的泥淖上。认证相关的视图可以经由 `auth/login` 以及 `auth/register` 路由访问。`App\Services\Auth\Registrar` 服务会负责处理用户认证和添加的相关逻辑。

### 事件对象

你现在可以将事件定义成对象，而不是仅使用字符串。例如，瞧瞧以下的事件：
    <?php

    class PodcastWasPurchased
    {
        public $podcast;

        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }
    }

这个事件可以像一般使用那样被派发：

    Event::fire(new PodcastWasPurchased($podcast));

当然，你的事件处理会收到事件的对象而不是数据的列表：

    <?php

    class ReportPodcastPurchase
    {
        public function handle(PodcastWasPurchased $event)
        {
            //
        }
    }

更多关于使用事件的信息，参考[完整文档](/docs/{{version}}/events)。

### 命令及队列

除了 Laravel 4 形式的队列任务，Laravel 5 以简单的命令对象作为队列任务。这些命令放在 `app/Commands` 目录下。下面是个例子的命令：

    <?php

    class PurchasePodcast extends Command implements SelfHandling, ShouldBeQueued
    {

        use SerializesModels;

        protected $user, $podcast;

        /**
         * Create a new command instance.
         *
         * @return void
         */
        public function __construct(User $user, Podcast $podcast)
        {
            $this->user = $user;
            $this->podcast = $podcast;
        }

        /**
         * Execute the command.
         *
         * @return void
         */
        public function handle()
        {
            // Handle the logic to purchase the podcast...

            event(new PodcastWasPurchased($this->user, $this->podcast));
        }
    }

Laravel 的基底控制器使用了新的 `DispatchesCommands` trait，让你可以简单的派发命令运行。

    $this->dispatch(new PurchasePodcastCommand($user, $podcast));

当然，你也可以将命令视为同步运行（而不会被放到队列里）的任务。事实上，使用命令是个好方式，让你可以封装应用程序需要运行的复杂任务。更多相关的信息，参考 [command bus](/docs/{{version}}/bus) 文档。

### 数据库队列

`database` 队列驱动现在已经包含在 Laravel 中了，提供了简单的本地端队列驱动，让你除了数据库相关软件外不需安装其它扩展包。

### Laravel 调度器

过去，开发者可以生成 Cron 设置，用以调度所有他们想要运行的命令行命令。然而，这是件很头痛的事情，因为你的命令行调度不在版本控制中，而你必须登录到服务器里加入 Cron 设置。让生活变得简单点。Laravel 命令行调度，让你可以流畅而且具有表达性的定义在 Laravel 里面，定义你的命令调度，而且只需要在服务器里设置一个 Cron 设置。

它会看起来如下：

    $schedule->command('artisan:command')->dailyAt('15:00');

当然，快参考[完整文档](/docs/{{version}}/artisan#scheduling-artisan-commands)学习所有调度相关知识。

### Tinker 与 Psysh

`php artisan tinker` 命令现在使用 Justin Hileman 的 [Psysh](https://github.com/bobthecow/psysh)，一个 PHP 更强大的 REPL。如果你喜欢 Laravel 4 的 Boris，你也会喜欢上 Psysh。更好的是，它可以在 Windows 上运行！要开始使用，只要输入：

    php artisan tinker

### DotEnv

比起一堆令人困惑的、嵌套的环境配置档目录，Laravel 5 现在使用了 Vance Lucas 的 [DotEnv](https://github.com/vlucas/phpdotenv)。这个扩展包提供了超级简单的方式管理配置文件，并且让 Laravel 5 环境侦测变得轻松。更多的细节，参考完整的[配置文件文档](/docs/{{version}}/configuration#environment-configuration)。

### Laravel Elixir

Jeffrey Way 的 Laravel Elixir 提供了一个流畅、口语化的接口，可以编译以及合并 assets。如果你曾经因为学习 Grunt 或 Gulp 而被吓到，不必再害怕了。Elixir 让使用 Gulp 编译 Less、Sass 及 CoffeeScript 变得简单。它甚至可以帮你运行测试！

更多关于 Elixir 的信息，参考[完整文档](/docs/{{version}}/elixir)。

### Laravel Socialite

Laravel Socialite 是个选用的，Laravel 5.0 以上兼容的扩展包，提供了无痛的 OAuth 认证。目前 Socialite 支持 Facebook、Twitter、Google 以及 GitHub。它写起来可能像这样：

    public function redirectForAuth()
    {
        return Socialize::with('twitter')->redirect();
    }

    public function getUserFromProvider()
    {
        $user = Socialize::with('twitter')->user();
    }

不用再花上数小时编写 OAuth 的认证流程。只要几分钟！查看[完整文档](/docs/{{version}}/authentication#social-authentication) 里有所有的细节。

### 文件系统集成

Laravel 现在包含了强大的[文件系统](https://github.com/thephpleague/flysystem)（一个文件系统的抽象函数库），提供了无痛的集成，把本地端文件系统、Amazon S3 和 Rackspace 云存储集成在一起，
有统一且优雅的 API！现在要将文件存到 Amazon S3 相当简单：

    Storage::put('file.txt', 'contents');

更多关于 Laravel 文件系统集成，参考[完整文档](/docs/{{version}}/filesystem)。

### Form Requests

Laravel 5.0 引进了 **form requests**，是继承自 `Illuminate\Foundation\Http\FormRequest` 的类。这些 request 对象可以和控制器方法依赖注入结合，提供一个不需模版的方法，来验证用户输入。让我们深入点，看一个 `FormRequest` 的例子：

    <?php

    namespace App\Http\Requests;

    class RegisterRequest extends FormRequest
    {
        public function rules()
        {
            return [
                'email' => 'required|email|unique:users',
                'password' => 'required|confirmed|min:8',
            ];
        }

        public function authorize()
        {
            return true;
        }
    }

定义好类后，我们可以在控制器动作里使用类型提示：

    public function register(RegisterRequest $request)
    {
        var_dump($request->input());
    }

当 Laravel 的服务容器辨别出要注入的类是个 `FormRequest` 实例，该请求将会被**自动验证**。意味着，当你的控制器动作被调用了，你可以安全的假设 HTTP 的请求输入己经被验证过，根据你在 form request 类里自定的规则。甚至，若这个请求验证不通过，一个 HTTP 重定向（可以自定），会自动发出，错误消息可以被闪存到 session 中或是转换成 JSON 返回。**表单验证再简单不过如此。**更多关于 `FormRequest` 验证，参考[文档](/docs/{{version}}/validation#form-request-validation)。

### 简易控制器请求验证

Laravel 5 基底控制器包含一个 `ValidatesRequests` trait。这个 trait 包含了一个简单的 `validate` 方法可以验证请求。如果对你的应用程序来说 `FormRequests` 太复杂了，参考这个：

    public function createPost(Request $request)
    {
        $this->validate($request, [
            'title' => 'required|max:255',
            'body' => 'required',
        ]);
    }

如果验证失败，会抛出异常以及返回适当的 HTTP 响应到浏览器。验证错误信息会被闪存到 session！而如果请求是 AJAX 请求，Laravel 会自动返回 JSON 格式的验证错误信息。

更多关于这个新方法的信息，参考[这个文档](/docs/{{version}}/validation#controller-validation)。

### 新的生成器

因应新的应用程序默认架构，框架添加了 Artisan generator 命令。使用 `php artisan list` 瞧瞧更多细节。

### 配置文件缓存

你现在可以通过 `config:cache` 命令将所有的配置文件缓存在单个文件中了。

### Symfony VarDumper

出名的 `dd` 辅助函数，其可以在调试时输出变量信息，已经升级成使用令人惊艳的 Symfony VarDumper 扩展包。它提供了颜色标记的输出，甚至数组可以自动缩合。在项目中试试下列代码：

    dd([1, 2, 3]);

<a name="laravel-4.2"></a>
## Laravel 4.2

此发行版本的完整更动列表可以从一个 4.2 的完整安装下，运行 `php artisan changes` 命令，或者 [Github 上的更动纪录](https://github.com/laravel/framework/blob/4.2/src/Illuminate/Foundation/changes.json)。此纪录仅含括主要的强化更新和此发行的更动部分。

> **附注:** 在 4.2 发布周期间，许多小的 BUG 修正与功能强化被整并至各个 4.1 的子发行版本中。所以最好确认 Laravel 4.1 版本的更新列表。

### PHP 5.4 需求

Laravel 4.2 需要 PHP 5.4 以上的版本。此 PHP 更新版本让我们可以使用 PHP 的新功能：traits 来为像是 [Laravel 收银台](/docs/billing) 来提供更具表达力的接口。PHP 5.4 也比 PHP 5.3 带来显著的速度及性能提升。

### Laravel Forge

Larvel Forge，一个网页应用程序，提供一个简单的接口去创建管理你云端上的 PHP 服务器，像是 Linode、DigitalOcean、Rackspace 和 Amazon EC2。支持自动化 nginx 设置、SSH 密钥管理、Cron job 自动化、通过 NewRelic & Papertrail 服务器监控、「推送部署」、Laravel queue worker 设置等等。Forge 提供最简单且更实惠的方式来部署所有你的 Laravel 应用程序。

默认 Laravel 4.2 的安装里，`app/config/database.php` 配置文件默认已为 Forge 设置完成，让在平台上的全新应用程序更方便部署。

关于 Laravel Forge 的更多信息可以在[官方 Forge 网站](https://forge.laravel.com)上找到。

### Laravel Homestead

Laravel Homestead 是一个为部署健全的 Laravel 和 PHP 应用程序的官方 Vagrant 环境。绝大多数的封装包的依赖与软件在发布前已经部署处理完成，让封装包可以极快的被启用。Homestead 包含 Nginx 1.6、PHP 5.5.12、MySQL、Postres、Redis、Memcached、Beanstalk、Node、Gulp、Grunt 和 Bower。Homestead 包含一个简单的 `Homestead.yaml` 配置文件，让你在单个个封装包中管理多个 Laravel 应用程序。

默认的 Laravel 4.2 安装中包含的 `app/config/local/database.php` 配置文件使用 Homestead 的数据库作为默认。让 Laravel 初始化安装与设置更为方便。

官方文档已经更新并包含在 [Homestead 文档](/docs/homestead) 中。

### Laravel 收银台

Laravel 收银台是一个简单、具表达性的资源库，用来管理 Stripe 的订阅帐务。虽然在安装中此组件依然是选用，我们依然将收银台文档包含在主要 Laravel 文档中。此收银台发布版本带来了数个错误修正、多货币支持还有支持最新的 Stripe API。

### Queue Workers 常驻程序

Artisan `queue:work` 命令现在支持 `--daemon` 参数让 worker 可以以「常驻程序」启用。代表 worker 可以持续的处理队列工作不需要重启框架。这让一个复杂的应用程序部署过程中，使得 CPU 的使用有显著的降低。

更多关于 Queue Workers 常驻程序信息请详阅 [queue 文档](/docs/queues#daemon-queue-worker)。

### Mail API Drivers

Laravel 4.2 为 `Mail` 函数采用了新的 Mailgun 和 Mandrill API 驱动。对许多应用程序而言，他提供了比 SMTP 更快也更可靠的方法来递送邮件。新的驱动使用了 Guzzle 4 HTTP 资源库。

### 软删除 Traits

对于软删除和全作用域更简洁的方案，PHP 5.4 的 `traits` 提供了一个更加简洁的软删除架构和全局作用域，这些新架构为框架提供了更有扩展性的功能，并且让框架更加简洁。

更多关于软删除的文档请见: [Eloquent documentation](/docs/eloquent#soft-deleting).

### 更为方便的 认证(auth) & Remindable Traits

得益于 PHP 5.4 traits，我们有了一个更简洁的用户认证和密码提醒接口，这也让 `User` 模型文档更加精简。

### "简易分页"

一个新的 `simplePaginate` 方法已被加入到查找以及 Eloquent 查找器中。让你在分页视图中，使用简单的「上一页」和「下一页」链接查找更为高效。

### 迁移确认

在正式环境中，破坏性的迁移动作将会被再次确认。如果希望取消提示字符确认请使用 `--force` 参数。

<a name="laravel-4.1"></a>
## Laravel 4.1

### 完整更动列表

此发行版本的完整更动列表，可以在版本 4.1 的安装中命令行运行 `php artisan changes` 获取，或者浏览 [Github 更动档](https://github.com/laravel/framework/blob/4.1/src/Illuminate/Foundation/changes.json) 中了解。其中只记录了该版本比较主要的强化功能和更动。

### 新的 SSH 组件

一个全新的 `SSH` 组件在此发行版本中登场。此功能让你可以轻易的 SSH 至远程服务器并运行命令。更多信息，可以参阅 [SSH 组件文档](/docs/ssh)。

新的 `php artisan tail` 命令就是使用这个新的 SSH 组件。更多的信息，请参阅 `tail` [命令集文档](http://laravel.com/docs/ssh#tailing-remote-logs)。

### Boris In Tinker

如果你的系统支持 [Boris REPL](https://github.com/d11wtq/boris)，`php artisan thinker` 命令将会使用到它。系统中也必须先行安装好 `readline` 和 `pcntl` 两个 PHP 扩展包。如果你没这些扩展包，从 4.0 之后将会使用到它。

### Eloquent 强化

Eloquent 添加了新的 `hasManyThrough` 关系链。想要了解更多，请参见 [Eloquent 文档](/docs/eloquent#has-many-through)。

一个新的 `whereHas` 方法也同时登场，他将允许[检索基于关系模型的约束](/docs/eloquent#querying-relations)。

### 数据库读写分离

Query Builder 和 Eloquent 目前通过数据库层，已经可以自动做到读写分离。更多的信息，请参考 [文档](/docs/database#read-write-connections)。

### 队列排序

队列排序已经被支持，只要在 `queue:listen` 命令后将队列以逗号分隔送出。

### 失败队列作业处理

现在队列将会自动处理失败的作业，只要在 `queue:listen` 后加上 `--tries` 即可。更多的失败作业处理可以参见 [队列文档](/docs/queues#failed-jobs)。

### 缓存标签

缓存「区块」已经被「标签」取代。缓存标签允许你将多个「标签」指向同一个缓存对象，而且可以清空所有被指定某个标签的所有对象。更多使用缓存标签信息请见 [缓存文档](/docs/cache#cache-tags)。

### 更具弹性的密码提醒

密码提醒引擎已经可以提供更强大的开发弹性，如：认证密码、显示状态消息等等。使用强化的密码提醒引擎，更多的信息 [请参阅文档](/docs/security#password-reminders-and-reset)。

### 强化路由引擎

Laravel 4.1 拥有一个完全重新编写的路由层。API 一样不变。然而与 4.0 相比，速度快上 100%。整个引擎大幅的简化，且对于路由表达式的编译大大减少对 Symfony Routing 的依赖。

### 强化 Session 引擎

此发行版本中，我们亦发布了全新的 Session 引擎。如同路由增进的部分，新的 Session 曾更加简化且更快速。我们不再使用 Symfony 的 Session 处理工具，并且使用更简单、更容易维护的自定义解法。

### Doctrine DBAL

如果你有在你的迁移中使用到 `renameColumn`，之后你必须在 `composer.json` 里加 `doctrine/dbal` 进依赖扩展包中。此扩展包不再默认包含在 Laravel 之中。
