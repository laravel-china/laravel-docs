# Redis

- [简介](#introduction)
- [基本用法](#basic-usage)
    - [管线化命令](#pipelining-commands)
- [发布与订阅](#pubsub)

<a name="introduction"></a>
## 简介

[Redis](http://redis.io) 是开源且先进的键值对保存库。由于它可用的键包含了[字符串](http://redis.io/topics/data-types#strings)、[哈希](http://redis.io/topics/data-types#hashes)、[列表](http://redis.io/topics/data-types#lists)、[集合](http://redis.io/topics/data-types#sets)和[有序集合](http://redis.io/topics/data-types#sorted-sets)，因此常被称作数据结构服务器。在 Laravel 使用 Redis 之前，你必须透过 Composer 安装 `predis/predis` 扩展包（~1.0）。

<a name="configuration"></a>
### 设置

应用程序的 Redis 设置都在 `config/database.php` 配置文件中。在这个文件里，你会看到 `redis` 数组，里面有包含了应用程序使用的 Redis 服务器：

    'redis' => [

        'cluster' => false,

        'default' => [
            'host'     => '127.0.0.1',
            'port'     => 6379,
            'database' => 0,
        ],

    ],

默认的服务器设置对于开发应该是足够的。然而，你可以根据使用环境自由修改数组。只要给每个 Redis 一个名称，并且指定服务器使用的的 host 和 port。

`cluster` 选项会让 Laravel 的 Redis 客户端在所有 Redis 节点间运行客户端分片（client-side sharding），让你创建节点池，并因此拥有大量的可用内存。但是请注意，客户端分片的节点不能运行容错转移；因此，此选项主要适用于可从另一台主要数据保存库取得的缓存数据。

此外，你可以在你的 Redis 连接中定义一个 `options` 数组值，让你指定一套 Predis 的[客户端选项](https://github.com/nrk/predis/wiki/Client-Options)。

如果你的 Redis 服务器需要认证，你可以在 Redis 服务器的设置数组里加入 `password` 设置项目作为提供的密码。

> **注意：**如果你是透过 PECL 安装 Redis PHP extension，则需要重命名 `config/app.php` 文件里的 Redis 别名。

<a name="basic-usage"></a>
## 基本用法

你可以透过调用 `Redis` [facade](/docs/{{version}}/facades) 的各种方法与 Redis 进行交互。`Redis` facade 支持动态方法，意思就是指你可以在该 facade 调用任何 [Redis 命令](http://redis.io/commands)，该命令会直接传递给 Redis。在本例中，我们会透过 `Redis` facade 的 `get` 方法来调用 Redis 的 `GET` 命令：

    <?php

    namespace App\Http\Controllers;

    use Redis;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 显示给定用户的个人数据。
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

当然，如上所述，你可以在 `Redis` facade 调用任何的 Redis 命令。Laravel 使用魔术方法来传递命令至 Redis 服务器，所以可以简单的传递 Redis 命令所需要的参数：

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

另外，你也可以透过 `command` 方法传递命令至服务器，它接受命令的名称作为第一个参数，第二个参数则为值的数组：

    $values = Redis::command('lrange', ['name', 5, 10]);

#### 使用多个 Redis 链接

你可以经由 `Redis::connection` 方法得到 Redis 实例：

    $redis = Redis::connection();

你会得到一个 Redis 默认服务器的实例。如果你没有使用服务器集群，你可以在 `connection` 方法传入定义在 Redis 配置文件的服务器名称，以取得特定服务器：

    $redis = Redis::connection('other');

<a name="pipelining-commands"></a>
### 管线化命令

当你想要在单次操作中发送多个命令至服务器时可以使用管线化。`pipeline` 方法接受一个参数：接收 Redis 实例的`闭包`。你可以发送所有的命令至此 Redis 实例，它们都会在单次操作中运行：

    Redis::pipeline(function ($pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>
## 发布与订阅

Laravel 也对 Redis 的 `publish` 及 `subscribe` 提供了方便的接口。这些 Redis 命令让你可以监听给定「频道」的消息。你可以从另一个应用程序发布消息至频道，甚至使用另一种编程语言，让应用程序或进程之间容易沟通。

首先，让我们透过 `Redis` 使用 `subscribe` 方法在一个频道设置监听器。我们会将方法调用放置于一个 [Artisan 命令](/docs/{{version}}/artisan)中，因为调用 `subscribe` 方法会启动一个长时间运行的进程：

    <?php

    namespace App\Console\Commands;

    use Redis;
    use Illuminate\Console\Command;

    class RedisSubscribe extends Command
    {
        /**
         * 主控台命令的识别名称。
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * 主控台命令描述。
         *
         * @var string
         */
        protected $description = 'Subscribe to a Redis channel';

        /**
         * 运行主控台命令。
         *
         * @return mixed
         */
        public function handle()
        {
            Redis::subscribe(['test-channel'], function($message) {
                echo $message;
            });
        }
    }

现在，我们可以透过 `publish` 方法发布消息至该频道：

    Route::get('publish', function () {
        // 路由逻辑...

        Redis::publish('test-channel', json_encode(['foo' => 'bar']));
    });

#### 通配符订阅

你可以使用 `psubscribe` 方法订阅一个通配符频道，这在要在所有频道获取所有消息时相当有用。`$channel` 名称会被传递至该方法提供的回调`闭包`的第二个参数：

    Redis::psubscribe(['*'], function($message, $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function($message, $channel) {
        echo $message;
    });
