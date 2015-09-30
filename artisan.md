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

#### Giving The User A Choice

The `anticipate` method can be used to provided autocompletion for possible choices. The user can still choose any answer, regardless of the choices.

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

If you need to give the user a predefined set of choices, you may use the `choice` method. The user chooses the index of the answer, but the value of the answer will be returned to you. You may set the default value to be returned if nothing is chosen:

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], false);

<a name="writing-output"></a>
### Writing Output

To send output to the console, use the `info`, `comment`, `question` and `error` methods. Each of these methods will use the appropriate ANSI colors for their purpose.

To display an information message to the user, use the `info` method. Typically, this will display in the console as green text:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

To display an error message, use the `error` method. Error message text is typically displayed in red:

    $this->error('Something went wrong!');

#### Table Layouts

The `table` method makes it easy to correctly format multiple rows / columns of data. Just pass in the headers and rows to the method. The width and height will be dynamically calculated based on the given data:

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### Progress Bars

For long running tasks, it could be helpful to show a progress indicator. Using the output object, we can start, advance and stop the Progress Bar. You have to define the number of steps when you start the progress, then advance the Progress Bar after each step:

    $users = App\User::all();

    $this->output->progressStart(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $this->output->progressAdvance();
    }

    $this->output->progressFinish();

For more advanced options, check out the [Symfony Progress Bar component documentation](http://symfony.com/doc/2.7/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>
## Registering Commands

Once your command is finished, you need to register it with Artisan so it will be available for use. This is done within the `app/Console/Kernel.php` file.

Within this file, you will find a list of commands in the `commands` property. To register your command, simply add the class name to the list. When Artisan boots, all the commands listed in this property will be resolved by the [service container](/docs/{{version}}/container) and registered with Artisan:

    protected $commands = [
        'App\Console\Commands\SendEmails'
    ];

<a name="calling-commands-via-code"></a>
## Calling Commands Via Code

Sometimes you may wish to execute an Artisan command outside of the CLI. For example, you may wish to fire an Artisan command from a route or controller. You may use the `call` method on the `Artisan` facade to accomplish this. The `call` method accepts the name of the command as the first argument, and an array of command parameters as the second argument. The exit code will be returned:

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

Using the `queue` method on the `Artisan` facade, you may even queue Artisan commands so they are processed in the background by your [queue workers](/docs/{{version}}/queues):

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

If you need to specify the value of an option that does not accept string values, such as the `--force` flag on the `migrate:refresh` command, you may pass a boolean `true` or `false`:

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

### Calling Commands From Other Commands

Sometimes you may wish to call other commands from an existing Artisan command. You may do so using the `call` method. This `call` method accepts the command name and an array of command parameters:

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

If you would like to call another console command and suppress all of its output, you may use the `callSilent` method. The `callSilent` method has the same signature as the `call` method:

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
