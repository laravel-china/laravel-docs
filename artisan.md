# Artisan 命令行

- [介绍](#introduction)
- [编写命令](#writing-commands)
    - [命令结构](#command-structure)
- [命令的输入与输出](#command-io)
    - [定义预期的输入](#defining-input-expectations)
    - [获取输入](#retrieving-input)
    - [为输入加上提示](#prompting-for-input)
    - [编写输出](#writing-output)
- [注册命令](#registering-commands)
- [使用代码来调用命令](#calling-commands-via-code)

<a name="introduction"></a>
## 介绍

Artisan 是 Laravel 的命令行接口的名称，它提供了许多实用的命令来帮助你开发 Laravel 应用，它是由强大的 Symfony Console 组件所驱动的。你可以使用 `list` 命令来列出所有可用的 Artisan 命令：

    php artisan list

每个命令也包含了「帮助」界面，它会显示并概述命令可以使用的参数及选项。只要在命令前面加上 `help` 即可显示帮助界面：

    php artisan help migrate

<a name="writing-commands"></a>
## 编写命令

除了使用 Artisan 本身所提供的命令之外，你也可以定制自己的命令，这也是 Artisan 强大的地方之一。你可以将你的自定义命令保存在 `app/Console/Commands` 目录中；当然，只要你在 `composer.json` 文件中的配置了自动加载，你就可以自由选择你想要放置的地方。

若要创建新的命令，你可以使用 `make:console` Artisan 命令生成一个默认的脚本来帮助你开始编写：

    php artisan make:console SendEmails

上面的这个命令会产生一个放置在 `app/Console/Commands/SendEmails.php` 的类。创建命令时，`--command` 这个参数可以用来指定要使用的终端命令名称：

    php artisan make:console SendEmails --command=emails:send

<a name="command-structure"></a>
### 命令结构

一旦生成这个命令，你应该先填写类的 `signature` 和 `description` 这两个属性，它们会被显示在 `list` 界面中。

命令运行的时候 `handle` 方法会被调用，因此你可以将程序逻辑放置在这个方法中，接下来让我们来看一个例子。

我们可以在构造器中注入需要的依赖，Laravel 的[服务容器](/docs/{{version}}/container) 将会自动注入有类型提示的依赖到构造器中。

为了更好的代码重用性，还有可读性，通常我们都会把处理业务逻辑的代码抽离放到一个功能类里。

    <?php

    namespace App\Console\Commands;

    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * 命令行的名称及用法。
         *
         * @var string
         */
        protected $signature = 'email:send {user}';

        /**
         * 命令行的概述。
         *
         * @var string
         */
        protected $description = 'Send drip e-mails to a user';

        /**
         * 滴灌电子邮件服务。
         *
         * @var DripEmailer
         */
        protected $drip;

        /**
         * 创建新的命令实例。
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
         * 运行命令。
         *
         * @return mixed
         */
        public function handle()
        {
            $this->drip->send(User::find($this->argument('user')));
        }
    }

<a name="command-io"></a>
## 命令的输入与输出

<a name="defining-input-expectations"></a>
### 定义预期的输入

当编写命令行时，从参数或是选项中得到用户的输入是一种普遍的做法。借助命令的 `signature` 属性，让你很方便的定义希望从用户获得的输入格式。`signature` 属性可用来定义命令的名字、参数及选项，具有单一、表达清晰、与路由相似的语法等特性。

所有提供给用户的参数及选项都包在大括号中。如以下例子，此命令会定义一个**必须的**参数：`user`：

    /**
     * 命令行的名称及用法。
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

你也可以使用选择性的参数和定义默认的值给选择性的参数（注意括号内的符号）：

    // 选择性的参数...
    email:send {user?}

    // 选择性的参数及默认的值...
    email:send {user=foo}

选项就跟参数一样，同样也是用户输入的一种格式，不过当使用选项时，需要在命令行加入两个连字符号（`--`），我们可以像这样子定义选项：

    /**
     * 命令行的名称及用法。
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

在这个例子中，当调用 Artisan 命令时，`--queue` 这个选项可以被明确的指定。如果 `--queue` 被当成输入时，这个选项的值会是 `true`，如果没有指定时，这个选项的值将会是 `false`：

    php artisan email:send 1 --queue

你也可以借助在这个选项后面加个 `=` 来为选项明确指定值：

    /**
     * 命令行的名称及用法。
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

在这个例子中，用户可以为这个参数传入一个值：

    php artisan email:send 1 --queue=default

你也可以指定默认值给选项：

    email:send {user} {--queue=default}

如果要在定义选项时给予简写，你可以在选项名称之前指定简写，并使用 | 分隔符号将它与完整的选项名称隔开：

    email:send {user} {--Q|queue}

#### 输入的概述

借助加入冒号 `:` 及概述，你也可以为输入参数及选项加上概述：

    /**
     * 命令行的名称及用法。
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : 用户的 ID }
                            {--queue= : 这个工作是否该进入队列}';

<a name="retrieving-input"></a>
### 获取输入

我们可以通过调用 `argument` 及 `option` 方法来获取到用输入的参数和选项数据。

使用 `argument` 方法来获取参数的值：

    /**
     * 命令行的处理逻辑
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

如果你需要将所有的参数汇聚成一个`数组`，不加参数调用 `argument` 即可：

    $arguments = $this->argument();

而获取选项就跟参数一样简单，除了使用的方法变为 `option`。就像 `argument` 方法一样，你可以调用 `option` 不加任何参数，即可获取所有的
选项并将之转为一个`数组`：

    // 获取特定的选择
    $queueName = $this->option('queue');

    // 获取所有选择
    $options = $this->option();

如果参数或选项不存在，将会返回 `null`。

<a name="prompting-for-input"></a>
### 让用户回答

除了显示输出，你也可以在命令运行期间，要求用户输入值。`ask` 方法将会用提供的问题来提示用户，并且接受他们的输入，接着返回用户的输入回命令：

    /**
     * 命令行的处理逻辑
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('你是名字是?');
    }

`secret` 方法就如同 `ask` 方法一般，但是用户的输入将不会显示在命令行。这个方法适合要求提供如密码的敏感信息时：

    $password = $this->secret('密码是？');

#### 让用户确认

如果你需要用户做简单的确认，你可以使用 `confirm` 方法。默认的情况下，这个方法会返回 `false`。如果用户对这个提示输入 `y`，那这个方法将会返回 `true`：

    if ($this->confirm('你希望继续吗? [y|N]')) {
        //
    }

#### 让用户做选择

`anticipate` 方法可被用于为可能的选择提供自动完成。用户仍可以选择任何答案而不理会这些选择。

    $name = $this->anticipate('你的名字是?', ['Taylor', 'Dayle']);

如果你需要提供用户一组事先定义的选择，你可以使用 `choice` 方法。用户会选择答案的索引，但是返回给你的会是答案的值。你可以设置返回的默认值来防止没有任何东西被选择的情况：

    $name = $this->choice('你的名字是?', ['Taylor', 'Dayle'], false);

<a name="writing-output"></a>
### 编写输出

使用 `line`、`info`、`comment`、`question` 和 `error` 方法来发送输出到终端。每个方法都有适当的 ANSI 颜色来表达它们的目的。

使用 `info` 方法来发送信息消息给用户，并在终端以绿色呈现。

    /**
     * 命令行的处理逻辑
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('把我显示在界面上');
    }

使用 `error` 方法来发送错误消息给用户，并在终端以红色呈现。

    $this->error('有东西出问题了！');

如果你想显示纯文本的控制列输出，可以使用 `line` 方法。`line` 方法不会接收任何特殊的颜色：

    $this->line('把我显示在界面上');

#### 数据库表布局

`table` 方法让格式化多行与多列的数据变得简单。只要发送标头和行到这个方法，宽跟高将会基于给的数据做动态的计算:

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### 进度条

对于需要长时间运行的任务，可以使用进度条来提示用户。使用 output 对象，我们可以开始、前进、停止进度条，当开始运行时你需要定义总共有几个阶段，然后每阶段完成后就让进度条前进：

    $users = App\User::all();

    // 多少个任务
    $bar = $this->output->createProgressBar(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        // 一个任务处理完了，可以前进一点点了
        $bar->advance();
    }

    $bar->finish();


想得到更多信息，请查看 [Symfony Progress Bar 组件的文档](http://symfony.com/doc/2.7/components/console/helpers/progressbar.html)。

<a name="registering-commands"></a>
## 注册命令

一旦你的命令编写完成，你需要注册 Artisan 后才能使用。注册文件为 `app/Console/Kernel.php`。

在这个文件中，你会在 `commands` 属性里找到命令的清单。要注册你的命令，只要简单的在此清单加入类的名称。当 Artisan 启动时，所有罗列在这个
属性的命令，都会被[服务容器](/docs/{{version}}/container)解析并向 Artisan 注册：

    protected $commands = [
        Commands\SendEmails::class,
    ];

<a name="calling-commands-via-code"></a>
## 使用代码调用命令

有时候你想在命令行接口外运行 Artisan 命令。例如，你希望在路由或控制器触发 Artisan 命令。只需利用 `Artisan` facade 的 `call` 方法即可做到。`call` 方法的第一个参数为命令的名称，第二个参数为数组型态的命令输入。退出码将会被返回：

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1,
            '--queue' => 'default'
        ]);

        //
    });

在 `Artisan` facade 使用 `queue` 方法，可以将一堆 Artisan 命令放进队列，好让它们能在后台被你的[队列服务器](/docs/{{version}}/queues) 运行：

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1,
            '--queue' => 'default'
        ]);

        //
    });

如果你需要指定非接收字符串选项的值，像是 `migrate:refresh` 命令的 `--force` 标记，你可以传递一个 `true` 或 `false` 的布尔值：

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

### 在命令中调用其他命令

有时候你会希望在命令中调用其他已存在的命令。你可以使用 `call` 方法来实现。`call` 方法接受命令名称和命令参数的数组：

    /**
     * 命令行的处理逻辑
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('email:send', [
            'user' => 1,
            '--queue' => 'default'
        ]);

        //
    }

如果你想要调用其他命令并忽略它所有的输出，则可以使用 `callSilent` 命令。`callSilent` 方法有和 `call` 方法一样的用法：

    $this->callSilent('email:send', [
        'user' => 1,
        '--queue' => 'default'
    ]);
