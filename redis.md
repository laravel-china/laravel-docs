# Laravel 的 Redis 使用指南

- [简介](#introduction)
    - [配置](#configuration)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [与 Redis 交互](#interacting-with-redis)
    - [管道命令](#pipelining-commands)
- [发布与订阅](#pubsub)

<a name="introduction"></a>
## 简介

[Redis](http://redis.io) 是一个开源的高级键值对存储库。它的键值可以包含 [字符串](http://redis.io/topics/data-types#strings)、[哈希](http://redis.io/topics/data-types#hashes)、[列表](http://redis.io/topics/data-types#lists)、[集合](http://redis.io/topics/data-types#sets) 和 [有序集合](http://redis.io/topics/data-types#sorted-sets) 这些数据类型，因此它通常被称为数据结构服务器。

在使用 Laravel 的 Redis 之前，你需要通过 Composer 安装 `predis/predis`  扩展包：

    composer require predis/predis

或者，你可以通过 PECL 安装 [PhpRedis](https://github.com/phpredis/phpredis) PHP 扩展。这个扩展安装起来比较复杂，但对于大量使用 Redis 的应用程序来说可能会产生更好的性能。


<a name="configuration"></a>
### 配置


应用程序的 Redis 配置都在配置文件 `config/database.php` 中。在这个文件里，你可以看到 `redis` 数组里面包含了应用程序使用的 Redis 服务器：

    'redis' => [

        'client' => 'predis',

        'default' => [
            'host' => env('REDIS_HOST', 'localhost'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => 0,
        ],

    ],

默认的服务器配置应该足以进行开发。当然，你也可以根据使用的环境来随意更改这个数组。只需在配置文件中给每个 Redis 服务器指定名称、host 和 port 即可。

#### 集群配置

如果你的程序使用 redis 服务器集群，你应该在 redis 配置文件中使用 `clusters` 键来定义这些集群：

    'redis' => [

        'client' => 'predis',

        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', 'localhost'),
                    'password' => env('REDIS_PASSWORD', null),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],

    ],

默认情况下，集群可以实现跨节点间客户端共享，允许你实现节点池以及创建大量可用内存。这里要注意，客户端共享不会处理失败的情况；因此，这个功能主要适用于从另一个主数据库获取的缓存数据。如果要使用 redis 原生集群，要在配置文件的 `options` 键中如下指定：

    'redis' => [

        'client' => 'predis',

        'options' => [
            'cluster' => 'redis',
        ],

        'clusters' => [
            // ...
        ],

    ],

<a name="predis"></a>
### Predis

除了默认的 `Host`、`port`、`database` 和 `password` 这些服务配置选项之外，Predis 还支持为每个 redis 服务器定义其他的 [连接参数](https://github.com/nrk/predis/wiki/Connection-Parameters)。如果要使用这些额外的配置选项，就将它们添加到配置文件 `config/database.php` 的 Redis 服务器配置中：

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_write_timeout' => 60,
    ],

<a name="phpredis"></a>
### PhpRedis

> {note} 如果你是通过 PECL 安装 Redis PHP 扩展，就需要重命名 `config/app.php` 文件里 Redis 的别名。

如果要使用 Phpredis 扩展，就需要将配置文件 `config/database.php` 中 Redis 配置的 `client` 选项更改为 `phpredis`：

    'redis' => [

        'client' => 'phpredis',

        // Rest of Redis configuration...
    ],

除了默认的 `Host`、`port`、`database` 和 `password` 这些服务配置项之外，Phpredis 还支持以下几个额外的连接参数：`persistent`、`prefix`、`read_timeout` 和 `timeout`。你可以将这些选项加到配置文件 `config/database.php` 中 redis 服务器配置项下：

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_timeout' => 60,
    ],

<a name="interacting-with-redis"></a>
## 与 Redis 交互

你可以调用 `Redis` [facade](/docs/{{version}}/facades) 上的各种方法来与 `Redis` 进行交互。`Redis` facade 支持动态方法，这意味着你可以在 facade 上调用任何  [Redis 命令](http://redis.io/commands)，还能将该命令直接传递给 Redis。在本例中，通过调用 `Redis` facade 上的 `get` 方法来调用 Redis 的 `GET` 命令：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Redis;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Redis::get('user:profile:'.$id);

            return view('user.profile', ['user' => $user]);
        }
    }

也就是说，你可以在 `Redis` facade 上调用任何的 Redis 命令。Laravel 使用魔术方法将传递命令给 Redis 服务器，因此只需传递 Redis 命令所需的参数即可：

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);
或者，你也可以使用 `command` 方法将命令传递给服务器，它接受命令的名称作为其第一个参数，并将值的数组作为其第二个参数：

    $values = Redis::command('lrange', ['name', 5, 10]);

#### 使用多个 Redis 连接

你可以通过 `Redis::connection` 方法来获取 Redis 实例：

    $redis = Redis::connection();
这会返回一个默认的 redis 服务器的实例。你也可以将连接或者集群的名称传递给 `connection` 方法，来获取在 Redis 配置文件中定义的特定的服务器或者集群：

    $redis = Redis::connection('my-connection');

<a name="pipelining-commands"></a>
### 管道命令

如果你需要在一个操作中向服务器发送很多命令，推荐你使用管道命令。`pipeline` 方法接收一个带有 Redis 实例的 `闭包` 。你可以将所有的命令发送给这个 Redis 实例，它们都会一次过执行完：

    Redis::pipeline(function ($pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>
## 发布与订阅

Laravel 为 Redis 的 `publish` 及 `subscribe` 提供了方便的接口。这些 Redis 命令让你可以监听指定「频道」上的消息。你可以从另一个应用程序发布消息给另一个应用程序，甚至使用其它编程语言，让应用程序和进程之间能够轻松进行通信。

首先，我们使用 `subscribe` 方法设置频道监听器。我们将这个方法调用放在 [Artisan 命令](/docs/{{version}}/artisan) 中，因为调用 `subscribe` 方法会启动一个长时间运行的进程：

    <?php

    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Support\Facades\Redis;

    class RedisSubscribe extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Subscribe to a Redis channel';

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            Redis::subscribe(['test-channel'], function ($message) {
                echo $message;
            });
        }
    }

现在，我们可以使用 `publish` 方法发布消息到频道：

    Route::get('publish', function () {
        // Route logic...

        Redis::publish('test-channel', json_encode(['foo' => 'bar']));
    });

#### 通配符订阅

使用 `psubscribe` 方法可以订阅通配符频道，可以用来在所有频道上获取所有消息。`$channel` 名称将作为第二个参数传递给提供的回调 `闭包` ：

    Redis::psubscribe(['*'], function ($message, $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function ($message, $channel) {
        echo $message;
    });

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@daydaygo](https://github.com/daydaygo) | <img class="avatar-66 rm-style" src="http://qiniu.daydaygo.top/lol-timo-panda.png"> | 翻译 | [Coder at Work](http://blog.daydaygo.top) |
| [@大袋鼠](https://github.com/FaithPatrick)  | <img class="avatar-66 rm-style" src="https://avatars1.githubusercontent.com/u/17744239"> | 校对 | [暮光博客](https://muguang.me/) |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | Review | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org

