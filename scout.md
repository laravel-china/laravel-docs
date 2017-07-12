# Laravel Scout

- [简介](#introduction)
- [安装](#installation)
    - [队列](#queueing)
    - [驱动的必要设置](#driver-prerequisites)
- [配置](#configuration)
    - [配置模型索引](#configuring-model-indexes)
    - [配置可检索的数据](#configuring-searchable-data)
- [索引](#indexing)
    - [批量导入](#batch-import)
    - [添加记录](#adding-records)
    - [更新记录](#updating-records)
    - [删除记录](#removing-records)
    - [暂停索引](#pausing-indexing)
- [检索](#searching)
    - [Where 语句](#where-clauses)
    - [分页](#pagination)
- [自定义引擎](#custom-engines)

<a name="introduction"></a>
## 简介

Laravel Scout 是针对 [Eloquent 模型](/docs/{{version}}/eloquent) 开发的基于驱动的全文检索系统。Scout 使用模型观察者时会自动保持你的检索索引与你的 Eloquent 记录同步。

目前，Scout 带着一个 [Algolia](https://www.algolia.com/) 驱动；然而，扩展 Scout 并不难，你可以通过自定义驱动来自由的扩展 Scout。

<a name="installation"></a>
## 安装

首先，使用 composer 包管理器来安装 Scout：

    composer require laravel/scout

接下来，你需要将 `ScoutServiceProvider` 添加到你的 `config/app.php` 配置文件的 `providers` 数组中：

    Laravel\Scout\ScoutServiceProvider::class,

注册好 Scout 的服务提供者之后，你可以使用 `vendor:publish` Artisan 命令生成 Scout 的配置文件。这个命令会在你的 `config` 目录下生成 `scout.php` 配置文件：

    php artisan vendor:publish

最后，将 `Laravel\Scout\Searchable` trait 加到你想要做检索的模型，这个 trait 会注册一个模型观察者来保持模型同步到检索的驱动：

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;
    }

<a name="queueing"></a>
### 队列

虽然 Scort 没有限制你必须使用队列，但是建议你为 Scort 配置一个 [队列驱动](/docs/{{version}}/queues)。使用队列来对处理 Scout 对数据模型的索引，将会极大的提高你的页面响应时间。

一旦你配置了队列驱动，在你的 `config/scout.php` 配置文件中设置 `queue` 的值为 `true`：

    'queue' => true,

<a name="driver-prerequisites"></a>
### 驱动必要设置

#### Algolia

当你使用 Algolia 驱动时，你需要在你的 `config/scout.php` 配置文件配置你的 Algolia `id` 和 `secret` 认证资料。配置好认证资料以后， 你还需要使用 composer 包管理器安装 Algolia PHP SDK ：

    composer require algolia/algoliasearch-client-php

<a name="configuration"></a>
## 配置

<a name="configuring-model-indexes"></a>
### 配置模型索引

每一个 Eloquent 模型都会同步到对应的一个检索「索引」中，「索引」里包含了此模型的所有可检索的记录。你可以把每一个「索引」设想为一张 MySQL 数据表。默认情况下，「索引」的名称与模型对应数据表名称一致，也就是说，是模型名称的复数形式。当然，你也可以在模型类使用 `searchableAs` 方法来重写「索引」名称：

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * 得到该模型索引的名称。
         *
         * @return string
         */
        public function searchableAs()
        {
            return 'posts_index';
        }
    }

<a name="configuring-searchable-data"></a>
### 配置可检索的数据

默认情况下，「索引」会从模型的 `toArray` 方法中读取数据来做数据持久化。你也可以通过重写 `toSearchableArray` 方法来自定义数据到「索引」的同步：

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * 得到该模型可索引数据的数组。
         *
         * @return array
         */
        public function toSearchableArray()
        {
            $array = $this->toArray();

            // 自定义数组数据...

            return $array;
        }
    }

<a name="indexing"></a>
## 索引

<a name="batch-import"></a>
### 批量导入

如果你想要将 Scout 安装到已经存在的项目里，那你也需要将已经在数据库里的数据导入到搜索引擎里。你可以使用 Scout 提供的 `import` Artisan 命令把现有的模型数据导入到「索引」里：

    php artisan scout:import "App\Post"

<a name="adding-records"></a>
### 添加记录

当你将 `Laravel\Scout\Searchable` trait 添加到模型之后，你只需要 `save` 一个模型实例，它就会自动的添加到你检索的索引里。如果你的 Scout 配置里 [使用队列](#queueing) 这个操作会在后台由你的 queue worker 执行：

    $order = new App\Order;

    // ...

    $order->save();

#### 使用队列添加

如果你想使用 Eloquent 构造器添加模型的集合到你的检索索引里，你也可以在 Eloquent 构造器上链式调用 `searchable` 方法。`searchable` 会把构造器的查询 [结果分块](/docs/{{version}}/eloquent#chunking-results) 并且将记录添加到你的检索索引里。同样的，如果你配置 Scout 使用队列，所有的数据块将会在后台由你的 queue workers 添加：

    // 使用 Eloquent 查询语句增加...
    App\Order::where('price', '>', 100)->searchable();

    // 你也可以使用模型关系增加记录...
    $user->orders()->searchable();

    // 你也可以使用集合增加记录...
    $orders->searchable();

`searchable` 方法可以被看做是「增量更新」的操作。换句话说，如果一个模型的记录已经在你的索引里了，它就会被更新，如果不在，就会被插入。

<a name="updating-records"></a>
### 更新记录

你只需要更新一个模型实例的属性并且 `save` 这个模型到你的数据库里，就更新了这个可检索的模型。Scout 会自动更新这个变化到你的检索索引里：

    $order = App\Order::find(1);

    // 更新 order...

    $order->save();

你也可以在 Eloquent 查询语句上使用 `searchable` 方法来更新一个模型的集合。如果这个模型不存在你检索的索引里，就会被创建：

    // 使用 Eloquent 查询语句更新...
    App\Order::where('price', '>', 100)->searchable();

    // 你也可以使用模型关系更新...
    $user->orders()->searchable();

    // 你也可以使用集合更新...
    $orders->searchable();

<a name="removing-records"></a>
### 删除记录

简单的使用 `delete` 从数据库删除模型就可以移除索引里的记录。这种移除的方式也兼容 [软删除](/docs/{{version}}/eloquent#soft-deleting) 模型:

    $order = App\Order::find(1);

    $order->delete();

或者说是你不想在删除记录之前检索模型，你也可以在 Eloquent 查询语句的实例或集合上使用 `unsearchable` 方法：

    // 使用 Eloquent 查询语句移除索引...
    App\Order::where('price', '>', 100)->unsearchable();

    // 你也可以使用模型关系移除索引...
    $user->orders()->unsearchable();

    // 你也可以使用集合移除索引...
    $orders->unsearchable();

<a name="pausing-indexing"></a>
### 暂停索引

当你想要对 Eloquent 模型完成一系列的操作并且不想将它们的数据同步到检索的索引里时，你也可以使用 `withoutSyncingToSearch` 方法。这个方法接受一个立即执行的回调函数。函数内部所有的操作都不会被同步到模型的索引里：

    App\Order::withoutSyncingToSearch(function () {
        // 对模型进行操作...
    });

<a name="searching"></a>
## 检索

你可以对一个模型使用 `search` 方法来检索。这个 search 方法接受一个将要在模型里检索的简单的字符串。之后你可以在搜索语句上链式调用 `get` 方法得到匹配的 Eloquent 模型：

    $orders = App\Order::search('Star Trek')->get();

当 Scout 检索到数据后，会返回一个 Eloquent 模型的集合，你也可以在路由或控制器上直接返回数据，这样它会被自动解析成 JSON 格式：

    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return App\Order::search($request->search)->get();
    });

<a name="where-clauses"></a>
### Where 语句

Scout 允许你增加一个简单的「where」语句链接到搜索语句上。目前，这些语句只支持简单的数字相等检查，并且主要用来查询范围内的拥有者的 ID。由于检索索引是非关系型数据库，更高级的「where」暂时不支持：

    $orders = App\Order::search('Star Trek')->where('user_id', 1)->get();

<a name="pagination"></a>
### 分页

除了检索模型的集合，你也可以使用 `paginate` 方法对检索结果进行分页。这个方法会返回一个 `Paginator` 实例就像你 [对传统的 Eloquent 查询语句进行分页](/docs/{{version}}/pagination):

    $orders = App\Order::search('Star Trek')->paginate();

你可以通过通过 `paginate` 方法的第一个参数来指定检索的结果每页要显示的数量：

    $orders = App\Order::search('Star Trek')->paginate(15);

一旦你获取到结果，就可以对结果进行显示，就像你对传统的 Eloquent 查询语句进行分页一样，使用 [Blade](/docs/{{version}}/blade) 来渲染页面的链接：

    <div class="container">
        @foreach ($orders as $order)
            {{ $order->price }}
        @endforeach
    </div>

    {{ $orders->links() }}

<a name="custom-engines"></a>
## 自定义引擎

#### 写一个引擎

如果 Scout 内建的引擎不能满足你的需求，你可以写你自定义的引擎并且将它注册到 Scout。你的引擎需要扩展 `Laravel\Scout\Engines\Engine` 抽象类，这个抽象类包含了五种你自定义的引擎必须要实现的方法：

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function map($results, $model);

在 `Laravel\Scout\Engines\AlgoliaEngine`　类里回顾这些方法会对你有较大的帮助。这个类将为你提供一个良好的学习起点，学习如何在你自己的引擎中实现这些方法。

#### 注册引擎

一旦你写好了自己的引擎，你可以用 Scout 管理引擎的 `extend` 方法将它注册到 Scout。你只需要在你的 `AppServiceProvider` 下的 `boot` 方法调用 `extend` 方法，或者是你的应用下使用的其他任何一个服务提供者。举个例子，如果你写好了一个 `MySqlSearchEngine`，你可以像这样去注册它：

    use Laravel\Scout\EngineManager;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }

一旦你的引擎注册好了，你可以在 `config/scout.php` 配置文件中指定它为默认的 Scout `driver`：

    'driver' => 'mysql',

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@贺钧威](https://phphub.org/users/5711)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5711_1473489317.jpg?imageView2/1/w/100/h/100">  |  翻译  | 感谢[BlueStone](http://bluestoneapp.thexrverge.com/)翻译支持，[@贺钧威](https://github.com/HejunweiCoder/) at Github  |



--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org] 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/2752/laravel-53-document-translation-completed)。
> 
> 文档永久地址： http://d.laravel-china.org