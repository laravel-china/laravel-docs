# Artisan 控制台

- [介绍](#introduction)
- [编写命令](#writing-commands)
    - [命令结构](#command-structure)
- [命令 I/O](#command-io)
    - [定义输入期望值](#defining-input-expectations)
    - [获取输入值](#retrieving-input)
    - [输入提示](#prompting-for-input)
    - [编写输出](#writing-output)
- [注册命令](#registering-commands)
- [通过代码调用命令](#calling-commands-via-code)

<a name="introduction"></a>
## 介绍

Artisan是包含在Laravel中命令行界面的名字。 它提供了很多有用的命令用于开发你的程序。它是基于强大的Symfony控制台组件开发。想要查看所有可用的Artisan命令，你可以使用‘list’命令：

    php artisan list

每个命令也包含了一个帮助界面去显示和描述该命令可用的参数和选项。要查看帮助界面，可以使用‘help’命令：

    php artisan help migrate

<a name="writing-commands"></a>
## 编写命令

除了Artisan提供的命令，你也可以创建自定义命令在你的程序中使用。你可以把你的自定义命令保存到`app/Console/Commands`目录中；尽管如此，你也可以选择保存到任意地方只要你的命令能被自动加载基于你的`composer.json`配置。

要编写一个新命令，你可以用这个Artisan命令`make:console`生成一个命令存根来帮助你开始：

    php artisan make:console SendEmails

上面这个命令会在`app/Console/Commands/SendEmails.php`上生成一个类。 创建命令的时候，这个`--command`选项可以用来定义终端命令名字：

    php artisan make:console SendEmails --command=emails:send

<a name="command-structure"></a>
### 命令结构

你的命令生成之后，你应该填写这个类的`signature`和`description`属性，这两个属性用来显示你的命令在`list`界面。

当你的命令被执行的时候，`handle`方法会被调用。你可以在这个方法中编写任意命令逻辑。让我们来看一个例子命令。

记住这个：我们可以在命令的构造器中注入任何我们需要的依赖。Laravel [service container](/docs/{{version}}/container)会自动注入所有类型提示的依赖在构造器中。为了更好的代码复用性，保持你的命令轻量和让他们顺从程序服务去完成他们的任务是一个好习惯。

    <?php

    namespace App\Console\Commands;

    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;
    use Illuminate\Foundation\Inspiring;

    class Inspire extends Command
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

<a name="command-io"></a>
## 命令 I/O

<a name="defining-input-expectations"></a>
### 定义输入期望值

编写命令的时候，一般是通过参数或者选项获取用户的输入。Laravel通过使用你的命令的`signature`属性让定义你期望的用户输入变得很方便。这个`signature`属性允许你通过一种简单的，有表现力的，易读的语法定义命令的名字，参数和选项。

所有用户输入的参数和选项会被包入大括号中，例如：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

在这个例子中，这个命令定义了一个必填参数：`user`。你也可以把参数设为可选和给可选参数定义默认值：

    // 可选参数...
    email:send {user?}

    // 可选参数和默认值...
    email:send {user=foo}

选项，像参数一样，也是一种用户输入形式。虽然如此，他们在命令行中输入的时候需要加两个短横线(`--`)作为前缀。我们可以这样定义选项在signature中：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

在这个例子中，这个`--queue`选项在调用这个Artisan命令的时候可以被指定。如果传递了`--queue`选项，它的值为`true`。否则，它的值为`false`： 

    php artisan email:send 1 --queue

你也可以让用户给这个选项赋值通过在选项后加等于号`=`，来指示这个值将会由用户来指定：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

这个例子中，用户可以像这样给这个选项赋值：

    php artisan email:send 1 --queue=default

你也可以给这个选项指定默认值：

    email:send {user} {--queue=default}

#### 输入描述

你可以为参数和选项定义描述如下通过冒号分割参数与描述：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';

<a name="retrieving-input"></a>
### 获取输入

当你的命令执行的时候，很明显你需要获取你的命令接收的参数及选项的值。想要获取这些值，你可以使用`argument` 和 `option`方法：

获取某个参数的值，使用`argument`方法：

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

如果你需要获取所有参数的值来作为一个数组，调用无参方法`argument`：

    $arguments = $this->argument();

通过`option`方法可以获取选项的值，就像获取参数的值一样简单。和`argument`方法一样，通过调用无参方法`option`可以获取到所有的选项值来作为一个数组：

    // Retrieve a specific option...
    $queueName = $this->option('queue');

    // Retrieve all options...
    $options = $this->option();

如果参数和选项不存在，将返回空值`null`。

<a name="prompting-for-input"></a>
### 输入提示

为了显示输出，在你的命令执行过程中你可能需要向用户请求输入一些信息。这个`ask`方法就可以通过既定的问题提示用户，然后获取用户输入的信息并返回给你的命令：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

这个`secret`方法和`ask`方法类似，但在用户输入时输入信息用户不可见。这个方法会很有用当要求用户输入一些敏感信息的时候，例如密码：

    $password = $this->secret('What is the password?');

#### 请求确认

如果你需要请求用户做一个简单确认，你可以使用`confirm`方法。默认这个方法会返回`false`。虽然如此，如果用户输入了`y`作为回应，这个方法会返回`true`。

    if ($this->confirm('Do you wish to continue? [y|N]')) {
        //
    }

#### 给用户一个选择

`anticipate`方法用来为可能的选择做自动填充。用户仍然可以填任何答案而无视这些选项。

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

如果你需要让用户必须从给定的选项中选，你可以使用`choice`方法。用户选择答案的索引，但答案的值返回给你。你可以设定返回默认值如果什么都没有选的话：

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], false);

<a name="writing-output"></a>
### 编写输出

使用`info`,`comment`,`question`和`error`方法，可以发送输出到命令行。这些方法中的每一个都会根据他们的目的使用合适的ANSI颜色来显示。

使用`info`方法来显示信息。一般情况下，在命令行中会显示为绿色文本：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

使用`error`方法来显示错误信息。一般情况下，在命令行中会显示为红色文本：

    $this->error('Something went wrong!');

#### 表格布局

`table`方法让以正确格式显示多行多列数据变得很容易。只需要把表头和记录传给这个方法。宽和高都会被动态计算出来根据传入的数据：

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### 进度条

对于耗时任务，显示一个进度条是很有帮助的。使用输出对象，我们可以开始，推进和停止进度。你必须定义总步数当开始进度的时候，然后在每一步完成之后推进进度：

    $users = App\User::all();

    $this->output->progressStart(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $this->output->progressAdvance();
    }

    $this->output->progressFinish();

想了解更多高级选项，请查看 [Symfony Progress Bar component documentation](http://symfony.com/doc/2.7/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>
## 注册命令

当你的命令完成之后，你需要注册才可以使用。这个可以在`app/Console/Kernel.php`文件中完成。

在这个文件中，你会发现一个命令列表在`commands`属性中。要注册你的命令，只需要简单的把类名加到列表中。当Artisan启动时，所有在这个属性中的命令都会被[service container](/docs/{{version}}/container)解析和用Artisan注册:

    protected $commands = [
        'App\Console\Commands\SendEmails'
    ];

<a name="calling-commands-via-code"></a>
## 通过代码调用命令

有时候你可能希望执行一个Artisan命令在命令行之外。例如，你希望触发一个Artisan命令在一个路由或者控制器里。你可以通过`Artisan` facade的`call`方法去完成它。`call`方法接收命令名作为第一个参数，命令参数数组作为第二个参数。退出代码（exit code）将被返回：

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

使用`Artisan` facade的`queue`方法，你甚至可以把命令放入队列，这样他们就可以在后台通过[queue workers](/docs/{{version}}/queues)被处理：

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

如果你需要指定的选项不接收字符串，例如`migrate:refresh`命令的`--force`选项，你可以传递一个布尔值`true` 或者 `false`：

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

### 从其他命令中调用命令

有时你希望从已有Artisan命令中调用其它命令。你可以使用`call`方法。这个`call`方法接收命令名和命令参数数组作为参数：

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

如果你想调用另一个命令并忽略它所有的输出，你可以使用`callSilent`方法。`callSilent`方法接收的参数和`call`方法一样：

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
