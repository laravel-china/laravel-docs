# Laravel 的 Artisan 命令行工具

- [介绍](#introduction)
- [编写命令](#writing-commands)
    - [生成命令](#generating-commands)
    - [命令结构](#command-structure)
    - [闭包命令](#closure-commands)
- [定义预期的输入](#defining-input-expectations)
    - [参数](#arguments)
    - [选项](#options)
    - [输入数组](#input-arrays)
    - [输入说明](#input-descriptions)
- [I/O 命令](#command-io)
    - [获取输入](#retrieving-input)
    - [输入提示](#prompting-for-input)
    - [编写输出](#writing-output)
- [注册命令](#registering-commands)
- [程序内部调用命令](#programmatically-executing-commands)
    - [命令中调用其它命令](#calling-commands-from-other-commands)

<a name="introduction"></a>
## 介绍

`Artisan` 是 Laravel 的命令行接口的名称， 它提供了许多实用的命令来帮助你开发 Laravel 应用， 要查看所有的 `Artisan` 命令列表，可以使用 `list` 命令：

    php artisan list

每个命令也包含了「帮助」界面，它会显示并概述命令可使的参数及选项。只需要在命令前面加上 `help` 即可显示命令帮助界面：

    php artisan help migrate

#### Laravel REPL

所有的 Laravel 应用都包括 Tinker，一个基于 [PsySH](https://github.com/bobthecow/psysh) 开发的 REPL 包。Tinker 让你可以在命令行中与你整个的 Laravel 应用进行交互，包括 Eloquent ORM，任务，事件等等。运行 `tinker` 命令进入 Tinker 环境：

    php artisan tinker

<a name="writing-commands"></a>
## 编写命令

除了 `Artisan` 提供的命令之外，还可以创建自定义命令。自定义命令默认存储在 `app/Console/Commands` 目录，当然，你也可以修改 `composer.json` 配置来指定你想要存放的目录。

<a name="generating-commands"></a>
### 生成命令

要创建一个新的命令，可以使用 `make:command` 命令。这个命令会创建一个命令类并存放在 `app/Console/Commands` 目录。 不必担心不存在这个目录，运行 `make:command` 命令时会首先创建这个目录。生成的命令将会包括所有默认存在的属性和方法：

    php artisan make:command SendEmails

接下来，你需要在 Artisan CLI 里执行之前[注册命令](#registering-commands)。

<a name="command-structure"></a>
### 命令结构

命令生成以后，应先填写类的 `signature` 和 `description` 属性，之后可以在使用 `list` 命令是显示出来。执行命令是调用 `handle` 方法，可以把你的命令逻辑放到这个方法中。

> {tip} 大部分的代码复用，保持你的代码轻量并让它们延迟到应用服务中完成任务是个不错的实践。在下面这个例子中，我们注入了一个服务类去执行发送邮件的繁重任务。

让我们看这个简单的命令例子，`Command` 类构造器允许注入需要的依赖，Laravel 的[服务容器](/docs/{{version}}/container) 将会自动注入构造函数中所有带类型约束的依赖：

    <?php

    namespace App\Console\Commands;

    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'email:send {user}';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Send drip e-mails to a user';

        /**
         * The drip e-mail service.
         *
         * @var DripEmailer
         */
        protected $drip;

        /**
         * Create a new command instance.
         *
         * @param  DripEmailer  $drip
         * @return void
         */
        public function __construct(DripEmailer $drip)
        {
            parent::__construct();

            $this->drip = $drip;
        }

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            $this->drip->send(User::find($this->argument('user')));
        }
    }

<a name="closure-commands"></a>
### 闭包命令

闭包命令提供一个替代定义命令方法的类。同样的路由闭包是控制器的一种替代方法，这种命令闭包可以替换命令类。使用 `app/Console/Kernel.php` 文件的 `commands` 方法，需要 Laravel 在 `routes/console.php` 注册：


    /**
     * Register the Closure based commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

虽然这个文件没有定义 HTTP 路由，它定义了基于控制台的入口点（路由）到你的应用中，在这个文件中，你可以使用 `Artisan::command` 方法定义所有基于路由的闭包，`command` 方法接收两个参数：[命令签名](#defining-input-expectations)和一个接收命令参数和选项的闭包：

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    });

闭包绑定下面的命令实例，因此你可以访问所有的辅助方法，您也可以访问一个完整的命令类。

#### 类型提示依赖

除了接收命令的参数和选项外，命令闭包也可以使用类型提示来指定 [服务容器](/docs/{{version}}/container) 之外的额外依赖：

    use App\User;
    use App\DripEmailer;

    Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

#### 闭包命令描述

当定义一个基于命令的闭包时，你可以使用 `describe` 方法来为命令添加描述。这个描述将会在你执行 `php artisan list` 或 `php artisan help` 命令时显示：

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');

<a name="defining-input-expectations"></a>
## 定义预期的输入

在你编写控制台命令时，通常通过参数和选项收集用户输入，Laravel 使这项操作变得很方便，你可以在命令里使用 `signature` 属性。 `signature` 属性通过一个类似路由风格的语法让用户为命令定义名称，参数和选项。

<a name="arguments"></a>
### 参数

所有用户提供的参数及选项都包在大括号中。如以下例子，此命令会定义一个 **必须** 的参数： `user` ：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

您也可以创建可选参数，并定义参数的默认值：

    // Optional argument...
    email:send {user?}

    // Optional argument with default value...
    email:send {user=foo}

<a name="options"></a>
### 选项

选项，和参数一样，也是用户输入的一种格式，不过当使用选项时，需要在命令前加两个连字符号 (`--`) 的前缀，有两种类型的选项：接收一个值和不接受值。选项不接收一个值作为布尔值的 `switch` 。让我们看一个选项的例子：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

在这个例子中,当调用 `Artisan` 命令时，`--queue` 这个选项可以被明确的指定。如果 `--queue` 被当成输入时，这个选项的的值为  `true` ，否则这个值为 `false` ：

    php artisan email:send 1 --queue

<a name="options-with-values"></a>
#### 有值的选项

接下来，我们看下有值的选项。如果用户必须为选项指定一个值，会在选项的后面加一个 `=` 的后缀：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

在这个例子中， 用户可以想下面这个例子传递一个值：

    php artisan email:send 1 --queue=default

您可以通过指定选项名称后的默认值将默认值分配给选项。如果用户没有输入一个值，将会采用默认的值：

    email:send {user} {--queue=default}

<a name="option-shortcuts"></a>
#### 选项快捷键

当定义一个定义选项时，可以分配一个快捷键。你可以在选项前使用一个 `|` 分隔符将简写和完整选项名分开：

    email:send {user} {--Q|queue}

<a name="input-arrays"></a>
### 数组输入

如果你想使用数组输入方式定义参数或选项，你可以使用 `*` 符号。首先，我们先看一个数组输入的实例：

    email:send {user*}

调用此方法时，`user` 参数通过命令行输入。例如，下面这个命令将会为 `user` 设置 `['foo', 'bar']` ：

    php artisan email:send foo bar

在定义一个使用数组输入时，每个输入选项值的命令都要在选项名称前加前缀：

    email:send {user} {--id=*}

    php artisan email:send --id=1 --id=2

<a name="input-descriptions"></a>
### 输入描述

您可以通过在使用一个冒号分离参数来分配输入参数和选项的说明。如果你需要一个小的额外的空间来定义你的命令，可以多行定义你的扩展：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';

<a name="command-io"></a>
## I/O 命令

<a name="retrieving-input"></a>
### 获取输入

在命令执行的时，你可以像下面这样使用 `argument` 和 `option` 方法获取参数和选项：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

如果您需要获取所有参数作为一个 `array`，调用 `arguments` 方法：

    $arguments = $this->arguments();

选项可以像参数一样使用 `option` 方法检索， 获取所有的选项作为一个 `array` ，调用 `options` 方法：

    // Retrieve a specific option...
    $queueName = $this->option('queue');

    // Retrieve all options...
    $options = $this->options();

如果参数或选项不存在，将会返回 `null` 。

<a name="prompting-for-input"></a>
### 交互式输入

除了显示输出，您还可以要求用户在您的命令执行过程中提供输入。`ask` 方法将会使用给定问题提示用户，接收输入，然后返回用户输入到命令：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

`secret` 方法和 `ask` 方法类似，但是用户输入在终端中是不可以见的，这个方法在需要用户输入像密码这样的敏感信息是很有用的：

    $password = $this->secret('What is the password?');

#### 请求确认

如果你要用户提供的确认信息，你可以使用 `confirm` 方法，默认情况下，该方法返回 `false`，当然，如果你输入 `y` 这个方法将会返回 `true`。

    if ($this->confirm('Do you wish to continue?')) {
        //
    }

#### 自动完成

`anticipate` 方法可用于为可能的选项提供自动完成功能，用户仍然可以选择，忽略这个提示：

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

#### 多重选择

如果你要给用户一组预设选择，可以使用 `choice` 方法，你可以设置默认选项当用户没有选择时：

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $default);

<a name="writing-output"></a>
### 编写输出

使用 `line` 、`info` 、 `comment` 、 `question` 和 `error` 方法来发送输出到终端。每个方法都有适当的 ANSI 颜色来作为他们的标识。例如，要显示一条信息消息给用户，使用 `info` 方法。通常，在终端显示为绿色：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

显示错误信息， 使用 `error` 方法。 错误信息显示红色：

    $this->error('Something went wrong!');

使用 `line` 方法可以像平常一样，没有颜色输出。

    $this->line('Display this on the screen');

#### 数据表布局

 table 方法使输出多行/列格式的数据变得简单，只需要将头和行传递给该方法，宽度和高度将基于给定数据自动计算：

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### 进度条

对需要较长时间运行的任务，显示进度指示器很有用，使用该输出对象，我们可以开始、前进以及停止该进度条。在开始进度时你必须定义步数，然后每走一步进度条前进一格：

    $users = App\User::all();

    $bar = $this->output->createProgressBar(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

更多信息请查阅[Symfony Progress Bar 组件的文档](https://symfony.com/doc/2.7/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>
## 注册命令

命令编写完成后，需要注册 Artisan 后才能使用。注册文件为  `app/Console/Kernel.php` 。在这个文件中, 你会在 `commands` 属性中看到一个命令列表。要注册你的命令，只需将其加到该列表中即可。当 Artisan 启动时，所有罗列在这个 属性的命令，都会被 [服务容器](/docs/{{version}}/container) 解析并向 Artisan 注册：

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>
## 程序内部调用命令

有时候你可能希望在 CLI 之外执行 Artisan 命令，例如，你可能希望在路由或控制器中触发 Artisan 命令，你可以使用 `Artisan` facade 上的 `call` 方法来完成。`call` 方法接收被执行的命令名称作为第一个参数，命令参数数组作为第二个参数，退出代码被返回：

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

在 `Artisan` facade 使用 `queue` 方法，可以将 Artisan 命令给后台的 [队列服务器](/docs/{{version}}/queues) 运行,在这之前，确保您已配置了您的队列，并正在运行队列：

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

如果需要指定非接收字符串选项的值，如 `migrate:refresh`  命令的 `--force` 值，你可以传递一个 `true` 或者 `false`：

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="calling-commands-from-other-commands"></a>
### 命令中调用其它命令

有时候你希望从一个已存在的 Artisan 命令中调用其它命令。你可以使用 `call` 方法， `call` 方法接受命令名称和命令参数的数组：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    }

如果你想要调用其它控制台命令并阻止其所有输出，可以使用 `callSilent` 命令。`callSilent` 和 `call` 方法用法一样：

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
