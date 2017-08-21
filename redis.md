# Laravel 的 Redis 使用指南

- [简介](#introduction)
    - [配置](#configuration)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [基本用法](#interacting-with-redis)
    - [管道化命令](#pipelining-commands)
- [发布与订阅](#pubsub)

<a name="introduction"></a>
## 简介

[Redis](http://redis.io) 是一款开源且先进的键值对数据库。由于它的键指向的数据包含了 [字符串](http://redis.io/topics/data-types#strings)、[哈希](http://redis.io/topics/data-types#hashes)、[列表](http://redis.io/topics/data-types#lists)、[集合](http://redis.io/topics/data-types#sets) 和 [有序集合](http://redis.io/topics/data-types#sorted-sets) 这些数据类型，因此常被用作数据结构服务器。

在使用 Redis 之前，你需要通过 Composer 安装 `predis/predis`  扩展包：

    composer require predis/predis

还有一种选择，你可以通过 PECL 安装 [PhpRedis](https://github.com/phpredis/phpredis) PHP 扩展。这个扩展安装起来更复杂，但是你可以在你的程序重度使用 redis 时获得一定的性能提升。


<a name="configuration"></a>
### 配置


应用程序的 Redis 配置都在 `config/database.php` 配置文件中。在这个文件里，你可以看到 `redis` 数组里面包含了应用程序使用的 Redis 服务器：

    'redis' => [

        'client' => 'predis',

        'default' => [
            'host' => env('REDIS_HOST', 'localhost'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => 0,
        ],

    ],

默认的服务器配置对于开发来说应该足够了。当然，你也可以根据使用的环境来随意更改数组。只需给每个 Redis 服务器指定名称、host 和 port 即可。


> 译者注： 关于 Redis 多连接的配置，请参阅 - [Laravel 下配置 Redis 让缓存、Session 各自使用不同的 Redis 数据库](https://laravel-china.org/topics/2466)。

#### redis 集群配置

如果你的程序使用 redis 服务器集群，你应该在 redis 配置文件中使用 `clusters` 键来定义：

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

默认情况下，集群可以实现跨节点间客户端共享，允许你实现节点池以及创建大量可用内存。然而，注意客户端共享并没有处理失败情况；因此，主要适用于从另一个主要的数据源来建立缓存数据。如果你喜欢使用 redis 原生集群，你需要在配置文件中配置 `options` 键：

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

除了默认的 `Host`，`port`，`database` 和 `password` 服务配置项之外，Predis 还可以为每个 redis 定义其他的 [连接参数](https://github.com/nrk/predis/wiki/Connection-Parameters)。要使用这些额外的配置选项，只需将它们添加到你的 `config/database.php` 配置文件的 Redis 服务器配置项中即可：

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_write_timeout' => 60,
    ],

<a name="phpredis"></a>
### PhpRedis

> {note} 如果你是通过 PECL 安装 Redis PHP 扩展，则需要重命名 `config/app.php` 文件里的 Redis 别名。

要使用 Phpredis 扩展，你需要将 `client` 选项配置为 `phpredis`。这个选项可以在 `config/database.php` 配置文件中找到：

    'redis' => [

        'client' => 'phpredis',

        // Rest of Redis configuration...
    ],

除了默认的 `Host`，`port`，`database` 和 `password` 服务配置项之外，Phpredis 还支持下列额外连接配置：`persistent`，`prefix`，`read_timeout` 和 `timeout`。你可以将这些选项加到 `config/database.php` 配置文件中 redis 服务器配置项下：

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_timeout' => 60,
    ],

<a name="interacting-with-redis"></a>
## 基本用法

你可以通过调用 `Redis` [facade](/docs/{{version}}/facades) 的各种方法与 `Redis` 进行交互。`Redis` facade 支持动态方法，意思就是指你可以在该 facade 调用任何 [Redis 命令](http://redis.io/commands)，该命令会直接传递给 Redis。在本例中，我们会通过 `Redis` facade 的 `get` 方法来调用 Redis 的 `GET` 命令：

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

如上所述，你可以在 `Redis` facade 调用任何的 Redis 命令。Laravel 使用魔术方法来传递命令至 Redis 服务器，所以可以简单的传递 Redis 命令所需要的参数：

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

另外，你也可以通过 `command` 方法传递命令至服务器，它接收命令的名称作为第一个参数，第二个参数则为值的数组：

    $values = Redis::command('lrange', ['name', 5, 10]);

#### 使用多个 Redis 连接

你可以通过 `Redis::connection` 方法来得到 Redis 实例：

    $redis = Redis::connection();

这会返回配置项中的默认的 redis 服务器。你也可以传递连接或者集群的名字给 `connection` 方法，来获取在 Redis 配置文件中配置的特定的服务器或者集群：

    $redis = Redis::connection('my-connection');

<a name="pipelining-commands"></a>
### 管道化命令

当你想要在单次操作中发送多个命令至服务器时则可以使用管道化命令。 `pipeline` 方法接收一个参数：带有 Redis 实例的 `闭包` 。你可以发送所有的命令至此 Redis 实例，它们都会在单次操作中运行：

    Redis::pipeline(function ($pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>
## 发布与订阅

Laravel 也对 Redis 的 `publish` 及 `subscribe` 提供了方便的接口。这些 Redis 命令让你可以监听指定「频道」的消息。你可以从另一个应用程序发布消息至频道，甚至使用另一种编程语言，让应用程序或进程之间容易沟通。

首先，让我们通过 `Redis` 来使用 `subscribe` 方法在一个频道设置侦听器。我们会将方法调用放置于一个 [Artisan 命令](/docs/{{version}}/artisan) 中，因为调用 `subscribe` 方法会启动一个长时间运行的进程：

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

现在，我们可以通过 `publish` 方法发布消息至该频道：

    Route::get('publish', function () {
        // Route logic...

        Redis::publish('test-channel', json_encode(['foo' => 'bar']));
    });

#### 通配符订阅

你可以使用 `psubscribe` 方法订阅一个通配符频道，这在对所有频道获取所有消息时相当有用。 `$channel` 名称会被传递至该方法提供的回调 `闭包` 的第二个参数：

    Redis::psubscribe(['*'], function ($message, $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function ($message, $channel) {
        echo $message;
    });
    
## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@daydaygo](https://github.com/daydaygo)  | <img class="avatar-66 rm-style" src="http://qiniu.daydaygo.top/lol-timo-panda.png">  | 翻译 | [Coder at Work](http://blog.daydaygo.top) |
| [@大袋鼠](https://github.com/FaithPatrick)  | <img class="avatar-66 rm-style" src="https://avatars1.githubusercontent.com/u/17744239">  | 校对 | [暮光博客](https://muguang.me/) |


