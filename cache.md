# 缓存

- [配置信息](#configuration)
- [缓存的使用](#cache-usage)
    - [取得一个缓存的实例](#obtaining-a-cache-instance)
    - [从缓存中获取项目](#retrieving-items-from-the-cache)
    - [存放项目到缓存中](#storing-items-in-the-cache)
    - [删除缓存中的项目](#removing-items-from-the-cache)
- [加入自定义的缓存驱动](#adding-custom-cache-drivers)
- [缓存标签](#cache-tags)
    - [写入被标记的缓存项目](#storing-tagged-cache-items)
    - [取得被标记的缓存项目](#accessing-tagged-cache-items)
- [缓存事件](#cache-events)

<a name="configuration"></a>
## 配置信息

Laravel 提供了统一的 API 给各种不同的缓存系统，缓存的配置文件都放在 `config/cache.php` 中，在这个文件中，你可以指定默认想用哪个缓存驱动，Laravel 支持当前流行的缓存后端，如 [Memcached](http://memcached.org) 和 [Redis](http://redis.io)。

缓存配置文件还包含了其他的选项，你可以在文件中找到这些选项，请确保你都有读过这些选项上方的说明。Laravel 默认采用的缓存驱动是 `file`，这个驱动保存了串行化的缓存对象在文件系统中，对于大型应用程序而言，Laravel 比较建议你使用内存缓存，例如 Memcached 或 APC。

### 场景布置

#### 数据库

当使用 `database` 这个缓存驱动，你需要配置一个数据库表来放置缓存项目，下面是表结构：

    Schema::create('cache', function($table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });

#### Memcached

使用 Memcached 做缓存需要先安装 [Memcached PECL 扩展包](http://pecl.php.net/package/memcached)。

默认的[配置文件](#configuration)采用以 [Memcached::addServer](http://php.net/manual/en/memcached.addserver.php) 为基础的 TCP/IP：

    'memcached' => [
        [
            'host' => '127.0.0.1',
            'port' => 11211,
            'weight' => 100
        ],
    ],

你可能也会配置 `host` 选项到 UNIX 的 socket 路径中，如果你有这么做，记得 `port` 选项要设为 `0`：

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],

#### Redis

在你选择使用 Redis 作为 Laravel 的缓存前，你需要通过 Composer 预先安装 `predis/predis` 扩展包 (~1.0)。

更多有关配置 Redis 的信息，请参考 [Laravel 的文档页面](/docs/{{version}}/redis#configuration)。

<a name="cache-usage"></a>
## 缓存的使用

<a name="obtaining-a-cache-instance"></a>
### 取得一个缓存的实例

`Illuminate\Contracts\Cache\Factory` 和 `Illuminate\Contracts\Cache\Repository` [contracts](/docs/{{version}}/contracts) 提供了访问 Laravel 缓存服务的机制， 而 `Factory` contract 则为你的应用程序提供了访问所有缓存驱动的机制，`Repository` contract  是典型的缓存驱动实现，它会依照你的缓存配置文件变化。

你也需要使用 `Cache` facade，我们将会在此文档中介绍，`Cache` facade 提供了方便又简洁的方法访问缓存实例。

例如，我们试着在一个控制器中引用 `Cache` facade：

    <?php

    namespace App\Http\Controllers;

    use Cache;
    use Illuminate\Routing\Controller;

    class UserController extends Controller
    {
        /**
         * 显示应用程序中所有用户的列表。
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

#### 访问多个缓存保存

你可以通过 `store` 方法来访问缓存实例，传入 `store` 方法的键对应你在缓存配置文件中驱动：

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 10);

<a name="retrieving-items-from-the-cache"></a>
### 从缓存中获取项目

在 `Cache` facade 中，`get` 方法可以用来取出缓存中的项目，缓存不存在的话返回 `null`，`get` 方法接受第二个参数，作为找不到项目时返回的预设值：

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');


你甚至可能传入一个`闭包`作为默认值，当指定的项目不存在缓存中时，闭包将会被返回，传入一个闭包允许你延迟从数据库或外部服务中取出值：

    $value = Cache::get('key', function() {
        return DB::table(...)->get();
    });

#### 确认项目存在

`has` 方法可以用来检查一个项目是否存在于缓存中：

    if (Cache::has('key')) {
        //
    }

#### 递增与递减值

`increment` 和 `decrement` 方法可以用来调整缓存中的整数项目值，这两个方法都可以选择性的传入第二个参数，用来指示要递增或递减多少：

    Cache::increment('key');

    Cache::increment('key', $amount);

    Cache::decrement('key');

    Cache::decrement('key', $amount);

#### 取出或更新

有时候，你可能会想从缓存中取出一个项目，但也想在取出的项目不存在时存入一个默认值，例如，你可能会想从缓存中取出所有用户，当找不到用户时，从数据库中将这些用户取出并放入缓存中，你可以使用 `Cache::remember` 方法达到目的：

    $value = Cache::remember('users', $minutes, function() {
        return DB::table('users')->get();
    });

如果那个项目不存在缓存中，则返回给 `remember` 方法的闭包将会被运行，而且闭包的运行结果将会被存放在缓存中。

你可能也会结合 `remember` 和 `forever` 这两个方法来 ”永久“ 存储缓存：

    $value = Cache::rememberForever('users', function() {
        return DB::table('users')->get();
    });

#### 取出与删除

如果你需要从缓存中取出一个项目并删除它，你可能会使用 `pull` 方法，与 `get` 相似，如果对象不存在缓存中，`pull` 方法将会返回 `null`：

    $value = Cache::pull('key');

<a name="storing-items-in-the-cache"></a>
### 存放项目到缓存中

你可以使用 `Cache` facade 的 `put` 方法来存放项目到缓存中，你需要使用第三个参数来设定缓存的存放时间：

    Cache::put('key', 'value', $minutes);

如果不指定分钟数直到存放的项目过期，你也可能传递一个 PHP 的 `DateTime` 实例来表示该缓存项目过期的时间点：

    $expiresAt = Carbon::now()->addMinutes(10);

    Cache::put('key', 'value', $expiresAt);

`add` 方法只会把还不存在缓存中的项目放入缓存，如果成功存放，会返回 `true`，否返回 `false`：

    Cache::add('key', 'value', $minutes);

`forever` 方法可以用来存放永久的项目到缓存中，这些值必须被手动的删除，这可以通过 `forget` 方法达成：

    Cache::forever('key', 'value');

<a name="removing-items-from-the-cache"></a>
### 删除缓存中的项目

你可以使用 `forget` 方法从缓存中移除一个项目：

    Cache::forget('key');

也使用 `flush` 方法清除所有缓存：

    Cache::flush();

清空缓存 **并不会** 遵从缓存的前缀，并会将缓存中所有的项目删除。在清除与其他应用程序共用的缓存时应谨慎考虑这一点。

<a name="adding-custom-cache-drivers"></a>
## 加入自定义的缓存驱动

我们可以在 `Cache` facade 中使用 `extend` 方法自定义缓存驱动来扩充 Laravel 缓存，它被用来绑定一个自定义驱动的解析器到管理者上，通常这可以通过[服务容器](/docs/{{version}}/providers)来完成。

例如，要注册一个名为「mongo」的缓存驱动：

    <?php

    namespace App\Providers;

    use Cache;
    use App\Extensions\MongoStore;
    use Illuminate\Support\ServiceProvider;

    class CacheServiceProvider extends ServiceProvider
    {
        /**
         * 运行注册后的启动服务。
         *
         * @return void
         */
        public function boot()
        {
            Cache::extend('mongo', function($app) {
                return Cache::repository(new MongoStore);
            });
        }

        /**
         * 在容器中注册绑定。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

第一个传给 `extend` 方法的参数是驱动的名称，这个名称要与你在 `config/cache.php` 配置文件中，`driver` 选项指定的名称相同，第二个参数是一个应返回一个 `Illuminate\Cache\Repository` 实例的闭包，这个闭包会被传入一个 `$app` 实例，这个实例是属于类[服务容器](/docs/{{version}}/container)。

调用 `Cache::extend` 的工作可以在新加入的 Laravel 应用程序中默认的 `App\Providers\AppServiceProvider` 的 `boot` 方法中完成，或者你可以创建你自己的服务提供者来管理扩展功能（只是请别忘了在 `config/app.php` 中的服务提供者数组注册这个提供者）。

为了创建我们的自定义缓存驱动，首先需要实现 `Illuminate\Contracts\Cache\Store` [contract](/docs/{{version}}/contracts)。因此我们的 MongoDB 缓存实现大概会长这样子：

    <?php

    namespace App\Extensions;

    class MongoStore implements \Illuminate\Contracts\Cache\Store
    {
        public function get($key) {}
        public function put($key, $value, $minutes) {}
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}
        public function getPrefix() {}
    }

我们只需要通过一个 MongoDB 的连接来实现这些方法，一旦我们完成实现，我们就可以接着完成注册我们的自定义驱动：

    Cache::extend('mongo', function($app) {
        return Cache::repository(new MongoStore);
    });

如果你想让自定义扩展驱动为默认，只需要更新 `config/cache.php` 配置文件中的 `driver` 选项为驱动的 `key` 即可，如这个例子的 `mongo`。

如果你不知道要将你的自定义缓存驱动代码放置在何处，可以考虑将它放在 Packagist 上！或者你可以在你的 `app` 目录下创建一个 `Extension` 的命名空间。但是请记住，Laravel 没有硬性规定的应用程序结构，你可以依照你的喜好任意组织你的应用程序。

<a name="cache-tags"></a>
## 缓存标签

> **注意：** 缓存标签并不支持使用 `file` 或 `dababase` 的缓存驱动。此外，当在缓存使用多个标签并「永久」写入时，像是 `memcached` 的驱动性能会是最佳的，且会自动清除旧的纪录。

<a name="storing-tagged-cache-items"></a>
### 写入被标记的缓存项目

缓存标签允许你在缓存中标记关联的项目，并清空所有已分配指定标签的缓存值。你可以通过传递一组标签名称的有序数组，以访问被标记的缓存。举例来说，让我们访问一个被标记的缓存并 `put` 值给它：

	Cache::tags(['people', 'artists'])->put('John', $john, $minutes);

	Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);

当然，你不必限制于 `put` 方法。你可以在利用标签时使用任何缓存保存系统的方法。

<a name="accessing-tagged-cache-items"></a>
### 取得被标记的缓存项目

若要取得一个被标记的缓存项目，只要传递一样的标签有串行表至 `tags` 方法：

   $john = Cache::tags(['people', 'artists'])->get('John');

   $anne = Cache::tags(['people', 'authors'])->get('Anne');

你可以清空已分配单一标签或是一组标签列表中的所有项目。例如，下方的语法会将被标记 `people`、`authors`，或两者的缓存给移除。所以，`Anne` 与 `John` 都从缓存中被移除：

	Cache::tags(['people', 'authors'])->flush();

相反的，下方的语法只会删除被标记为 `authors` 的缓存，所以 `Anne` 会被移除，但 `John` 不会。

	Cache::tags('authors')->flush();

<a name="cache-events"></a>
## 缓存事件

你可以监听到缓存做每一次操作的触发[事件](/docs/{{version}}/events)。一般来说，你必须将事件监听器放置在 `EventServiceProvider` 的 `boot` 方法中：

    /**
     * 为你的应用程序注册任何其它事件。
     *
     * @param  \Illuminate\Contracts\Events\Dispatcher  $events
     * @return void
     */
    public function boot(DispatcherContract $events)
    {
        parent::boot($events);

        $events->listen('cache.hit', function ($key, $value) {
            //
        });

        $events->listen('cache.missed', function ($key) {
            //
        });

        $events->listen('cache.write', function ($key, $value, $minutes) {
            //
        });

        $events->listen('cache.delete', function ($key) {
            //
        });
    }
