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

Artisan 是 Laravel 的命令行接口，它提供了许多实用的命令来帮助你开发 Laravel 应用， 要查看含有所有的 Artisan 命令的列表，可以使用 `list` 命令：

    php artisan list

每个命令也包含了「帮助」界面，它会显示并概述命令可使的参数及选项。只需要在命令前面加上 `help` 即可显示命令帮助界面：

    php artisan help migrate

#### Laravel REPL

所有的 Laravel 应用都包括 Tinker ，一个基于 [PsySH](https://github.com/bobthecow/psysh) 开发的 REPL 包。 `Tinker` 让你可以在命令行中与你整个的 Laravel 应用进行交互，包括 Eloquent ORM ，任务，事件等等。运行 `tinker`  Artisan 命令进入 Tinker 环境：

    php artisan tinker

<a name="writing-commands"></a>
## 编写命令

除 Artisan 提供的命令之外，还可以创建自定义命令。自定义命令默认存储在 `app/Console/Commands` 目录，当然，你也可以修改 composer.json 文件来指定你想要存放的目录。

<a name="generating-commands"></a>
### 生成命令

要创建一个新的命令，可以使用 `make:command` 命令。这个命令会 `app/Console/Commands` 目录里面创建一个命令类。 不必担心不存在这个目录，因为该目录会在第一次运行 `ake:command` 命令时自动创建。生成的命令将会包括所有默认存在的属性和方法：

    php artisan make:command SendEmails

<a name="command-structure"></a>
### 命令结构

命令生成以后，应先填写类的 `signature` 和 `description` 属性，之后在使用 `list` 命令的时候可以显示出来。执行命令的时候会调用 `handle` 方法，可以把你的命令逻辑写到这个方法中。

> {tip} 为了更好的代码复用，保持你的控制台代码轻量并让它们延迟到应用服务中完成任务是个不错的做法。在下面的例子中，请注意到我们注入了一个服务类来完成发送邮件的重任。

让我们看一个简单的命令例子。注意：Command 类构造器允许注入任何依赖。Laravel 的 [服务容器](/docs/{{version}}/container) 将会自动注入构造函数中的所有带类型约束的依赖：

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

基于命令的闭包提供一个替代用类定义命令的方法。同样的，路由闭包是控制器的一种替代方法，就像命令闭包可以替换命令类。在 `app/Console/Kernel.php` 文件的 `commands` 方法中， Laravel 加载了 `routes/console.php` 文件：

    /**
     * 为项目注册一个基于命令的闭包。
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

虽然这个文件没有定义 HTTP 路由，它定义了基于入口点（路由）的控制台命令到你的应用中，在这个文件中，你可以使用 `Artisan::command` 方法定义所有基于路由的闭包，`command` 方法接收两个参数： [命令签名](#defining-input-expectations) 和一个接收命令参数和选项的闭包：

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    });

闭包绑定底层的命令实例，因此您可以完全访问通常可以在完整命令类中访问的所有辅助方法。

#### 类型提示依赖

除了接收命令的参数和选项外，命令闭包也可以使用类型提示你想要从 [服务容器](/docs/{{version}}/container) 中解决的其他依赖关系：

    use App\User;
    use App\DripEmailer;

    Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

#### 闭包命令描述

当定义一个基于命令的闭包时，你可以使用 describe 方法来为命令添加描述。这个描述将会在你执行 php artisan list 或 php artisan help 命令时显示：When defining a Closure based command, you may use the `describe` method to add a description to the command. This description will be displayed when you run the `php artisan list` or `php artisan help` commands:

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');

<a name="defining-input-expectations"></a>
## 定义预期的输入

在你编写控制台命令时，通常通过参数和选项收集用户输入，Laravel 使这项操作变得很方便，你可以在命令里使用 `signature` 属性定义你期望的用户输入。 `signature` 属性通过一个类似路由风格的语法让用户为命令定义名称，参数和选项。

<a name="arguments"></a>
### 参数

所有用户提供的参数及选项都被包含在大括号中。在下面的例子中，这个命令会定义了一个 **必须** 的参数： `user` ：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

您也可以创建可选参数，定义参数的默认值：

    // Optional argument...
    email:send {user?}

    // Optional argument with default value...
    email:send {user=foo}

<a name="options"></a>
### 选项

选项，类似于参数，是用户输入的另一种格式，选项在命令前有两个连字符号 (`--`) 的前缀，有两种类型的选项：接收一个值和不接受值。选项不接收一个值作为布尔值的 「开关」 。让我们看一个关于这种类型的选项的例子：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

在这个例子中,当调用 Artisan 命令时， `--queue` 这个开关可以被明确的指定。如果 `--queue` 开关被传递时，这个开关的值为 `true` ，否则为 `false` ：

    php artisan email:send 1 --queue

<a name="options-with-values"></a>
#### 有值的选项

接下来，我们看下需要传递值的选项。如果用户必须为选项指定一个值，在选项的后面加上一个 `=` 的后缀：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

在这个例子中， 用户可以像下面这个例子给选项传递一个值：

    php artisan email:send 1 --queue=default

您可以在选项后面设定一个默认值。如果用户没有传递值，将会采用默认的值：

    email:send {user} {--queue=default}

<a name="option-shortcuts"></a>
#### 选项快捷键

定义选项时，可以分配一个快捷键。你可以在选项前指定并且使用一个 `|` 分隔符将简写和完整选项名分开：

    email:send {user} {--Q|queue}

<a name="input-arrays"></a>
### 数组输入

如果你想定义参数或选项接收数组输入，你可以使用 `*` 符号。首先，我们先看一个数组参数的实例：

    email:send {user*}

调用此方法时， `user` 参数传送给命令行。例如，下面这个命令将会为 `user` 的值设置为 `['foo', 'bar']` ：

    php artisan email:send foo bar

在定义选项接收数组输入时，传递给命令的每个选项值应以选项名称为前缀：

    email:send {user} {--id=*}

    php artisan email:send --id=1 --id=2

<a name="input-descriptions"></a>
### 输入描述

您可以为参数和选项分配描述，使用冒号将参数与描述分离开来。如果你需要一个小的额外的空间来定义你的命令，可以进行多行定义：

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

在命令执行时，你可以使用 `argument` 和 `option` 方法获取命令的参数和选项：

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

选项可以像参数一样使用 `option` 方法获取， 获取所有的选项作为一个 array ，调用 `options` 方法：

    // Retrieve a specific option...
    $queueName = $this->option('queue');

    // Retrieve all options...
    $options = $this->options();

如果参数或选项不存在，将会返回 `null` 。

<a name="prompting-for-input"></a>
### 交互式输入

除了显示输出，您还可以要求用户在您的命令执行过程中提供输入。 `ask` 方法将会使用给定问题提示用户，接收用户的输入，然后将用户输入返回到你的命令：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

`secret` 方法和 `ask` 方法类似，但是用户输入在终端中是不可见的，这个方法在需要用户输入像密码这样的敏感信息的时候是很有用的：

    $password = $this->secret('What is the password?');

#### 请求确认

如果你要用户提供一些简单的确认信息，你可以使用 `confirm` 方法，默认情况下，该方法返回 `false` ，当然，如果用户输入 `y` 或者 `yes` 这个方法将会返回 `true` 。

    if ($this->confirm('Do you wish to continue?')) {
        //
    }

#### 自动补全

`anticipate` 方法可用于为可能的选择提供自动补全功能，用户仍然可以忽略这个提示任意选择回答：

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

#### 多重选择

如果你要给用户一组预设选择，可以使用 `choice` 方法，你可以设置当用户没有选择任何选项时返回的默认值：

If you need to give the user a predefined set of choices, you may use the `choice` method. You may set the default value to be returned if no option is chosen:

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $default);

<a name="writing-output"></a>
### 编写输出

可以使用 `line` 、`info` 、 `comment` 、 `question` 和 `error` 方法来发送输出到终端。每个方法都有适当的 ANSI 颜色来作为他们的标识。例如，使用 `info` 方法在终端显示一条绿色文本消息给用户：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

显示错误信息， 使用 `error` 方法。 错误信息通常显示为红色：

    $this->error('Something went wrong!');

如果您想在控制台无色的输出，请使用 `line` 方法：

    $this->line('Display this on the screen');

#### 数据表布局

`table` 方法使输出多行/列格式的数据变得简单，只需要将头和行传递给该方法，宽度和高度将基于给定数据自动计算：

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### 进度条

对需要较长时间来运行的任务，显示进度指示很有用。使用输出对象，我们可以启动，推进和停止进度条。首先，定义进程将遍历的步骤总数。然后，在处理每个项目后推进进度栏：

    $users = App\User::all();

    $bar = $this->output->createProgressBar(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

更多信息请查阅 [Symfony Progress Bar 组件的文档](https://symfony.com/doc/2.7/components/console/helpers/progressbar.html) 。

<a name="registering-commands"></a>
## 注册命令

由于在控制台内核的 `commands` 方法中调用了`load`方法，所以 `app / Console / Commands` 目录下的所有命令都将自动向 Artisan 中注册。 实际上，你可以自由地调用 `load` 方法来扫描其他目录 Artisan 命令：

    /**
     * 注册应用的命令。
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
        $this->load(__DIR__.'/MoreCommands');

        // ...
    }

您还可以通过将类名添加到 `app/Console/Kernel.php` 文件的 `$command` 属性中进行手动注册命令。当 Artisan 启动时，此属性中列出的所有命令将由 [服务容器](/docs/{{version}}/container) 解析，并在 Artisan 中注册：

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>
## 程序内部调用命令

Sometimes you may wish to execute an Artisan command outside of the CLI. For example, you may wish to fire an Artisan command from a route or controller. You may use the `call` method on the `Artisan` facade to accomplish this. The `call` method accepts the name of the command as the first argument, and an array of command parameters as the second argument. The exit code will be returned:
有时您可能希望在 CLI 之外执行 Artisan 命令。例如，您可能希望从路由或控制器触发 Artisan 命令。 您可以在 `Artisan` facade 上使用 `call` 方法来完成。 `call` 方法接受命令的名称作为第一个参数，命令参数的数组作为第二个参数。 退出代码将被退回：

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

在 `Artisan` facade 上使用 `queue` 方法，甚至可以排列 Artisan 命令，以便他们在后台由 [队列服务器](/docs/{{version}}/queues) 进行处理。 在使用此方法之前，请确保已配置队列并运行了队列侦听器：

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

如果您需要指定不接受字符串值的选项的值，例如 `migrate：refresh` 命令中的 `--force` 标志，你可以传递 `true` 或 `false` ：

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="calling-commands-from-other-commands"></a>
### 命令中调用其它命令

有时候你希望在一个已存在的 Artisan 命令中调用其它命令。你可以使用 `call` 方法， `call` 方法接受命令名称和命令参数的数组：

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

如果你想要调用其它控制台命令并阻止所有输出，可以使用 `callSilent` 命令。 `callSilent` 和 `call` 方法用法一样：

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

## 译者署名
| 用户名                                      | 头像                                       | 职能   | 签名                                       |
| ---------------------------------------- | ---------------------------------------- | ---- | ---------------------------------------- |
| [@laravelleon](https://laravel-china.org/users/18113) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/18113_1503316311.png?imageView2/1/w/200/h/200"> | 翻译   | You may delay , but the time will not . [@Leonzai](https://github.com/leonzai/) at Github |



