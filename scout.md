# Laravel 的搜索系统 Scout

- [简介](#introduction)
- [安装](#installation)
    - [队列](#queueing)
    - [驱动之前](#driver-prerequisites)
- [配置](#configuration)
    - [配置模型索引](#configuring-model-indexes)
    - [配置可搜索的数据](#configuring-searchable-data)
- [索引](#indexing)
    - [批量导入](#batch-import)
    - [添加记录](#adding-records)
    - [更新记录](#updating-records)
    - [删除记录](#removing-records)
    - [暂停索引](#pausing-indexing)
- [搜索](#searching)
    - [Where 语句](#where-clauses)
    - [分页](#pagination)
- [自定义引擎](#custom-engines)

<a name="introduction"></a>
## 简介

Laravel Scout 为 [Eloquent 模型](/docs/{{version}}/eloquent) 的全文搜索提供了一个简单的、基于驱动程序的解决方案。使用模型观察员，Scout 会自动同步你的搜索索引和 Eloquent 记录。

目前，Scout 自带了一个 [Algolia](https://www.algolia.com/) 驱动；而编写自定义驱动程序很简单，你可以自由地使用自己的搜索实现来扩展 Scout。

<a name="installation"></a>
## 安装

首先，使用 composer 包管理器来安装 Scout：

    composer require laravel/scout

接下来，你需要将 `ScoutServiceProvider` 添加到你的配置文件 `config/app.php` 的 `providers` 数组中：

    Laravel\Scout\ScoutServiceProvider::class,

注册好 Scout 的服务提供器之后，你还需使用Artisan 命令 `vendor:publish` 生成 Scout 的配置文件。这个命令会在你的 `config` 目录下生成 `scout.php` 配置文件：

    php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"


最后，将 `Laravel\Scout\Searchable` trait 加到你想要搜索的模型中。这个 trait 会注册一个模型观察者来保持搜索驱动程序与模型的同步：

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

虽然不是严格要求使用 Scout，但在使用这个库之前，强烈建议你配置一个 [队列驱动程序](/docs/{{version}}/queues)。使用队列来对处理 Scout 将所有模型信息同步到搜索索引的操作，会极大的减少 Web 页面的响应时间。

一旦配置了队列驱动程序，就要在 `config/scout.php` 配置文件中设置 `queue` 的值为 `true`：

    'queue' => true,

<a name="driver-prerequisites"></a>
### 驱动之前

#### Algolia

使用 Algolia 驱动时，需要在 `config/scout.php` 配置文件配置你的 Algolia `id` 和 `secret` 凭证。配置好凭证之后， 还需要使用 Composer 包管理器安装 Algolia PHP SDK ：

    composer require algolia/algoliasearch-client-php

<a name="configuration"></a>
## 配置

<a name="configuring-model-indexes"></a>
### 配置模型索引

每个 Eloquent 模型与给定的搜索「索引」同步，这个「索引」包含该模型的所有可搜索记录。换句话说，你可以把每一个「索引」设想为一张 MySQL 数据表。默认情况下，每个模型都会被持久化到与模型的「表」名（通常是模型名称的复数形式）相匹配的索引。你也可以通过覆盖模型上的 `searchableAs` 方法来自定义模型的索引：

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * Get the index name for the model.
         *
         * @return string
         */
        public function searchableAs()
        {
            return 'posts_index';
        }
    }

<a name="configuring-searchable-data"></a>
### 配置可搜索的数据

默认情况下，「索引」会从模型的 `toArray` 方法中读取数据来做持久化。如果要自定义同步到搜索索引的数据，可以覆盖模型上的 `toSearchableArray` 方法：

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * Get the indexable data array for the model.
         *
         * @return array
         */
        public function toSearchableArray()
        {
            $array = $this->toArray();

            // Customize array...

            return $array;
        }
    }

<a name="indexing"></a>
## 索引

<a name="batch-import"></a>
### 批量导入

如果你想要将 Scout 安装到现有的项目中，你可能已经有了想要导入搜索驱动的数据库记录。使用 Scout 提供的Artisan 命令 `import` 把所有现有记录导入到搜索索引里：

    php artisan scout:import "App\Post"

<a name="adding-records"></a>
### 添加记录

当你将 `Laravel\Scout\Searchable`  trait 添加到模型中，你需要做的是 `save` 一个模型实例，它就会自动添加到你的搜索索引。如果你已经将 Scout 配置为 [使用队列](#queueing)，那这个操作会在后台由你的队列工作进程来执行：

    $order = new App\Order;

    // ...

    $order->save();

#### 使用队列添加

如果你想通过 Eloquent 查询构造器将模型集合添加到搜索索引中，你也可以在 Eloquent 查询构造器上链式调用 `searchable` 方法。`searchable` 会把构造器的查询 [结果分块](/docs/{{version}}/eloquent#chunking-results) 并且将记录添加到你的搜索索引里。同样的，如果你已经配置 Scout 为使用队列，则所有的数据块将在后台由你的队列工作进程添加：

    // 使用 Eloquent 查询构造器增加...
    App\Order::where('price', '>', 100)->searchable();

    // 使用模型关系增加记录...
    $user->orders()->searchable();

    // 使用集合增加记录...
    $orders->searchable();

`searchable` 方法可以被看做是「更新插入」的操作。换句话说，如果模型记录已经在你的索引里了，它就会被更新。如果搜索索引中不存在，则将其添加到索引中。

<a name="updating-records"></a>
### 更新记录

要更新可搜索的模型，只需要更新模型实例的属性并将模型 `save` 到数据库。Scout 会自动将更新同步到你的搜索索引中：

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

简单地使用 `delete` 从数据库中删除该模型就可以移除索引里的记录。这种删除形式甚至与 [软删除](/docs/{{version}}/eloquent#soft-deleting) 的模型兼容:

    $order = App\Order::find(1);

    $order->delete();

如果你不想在删除记录之前检索模型，可以在 Eloquent 查询实例或集合上使用 `unsearchable` 方法：

    // 通过 Eloquent 查询删除...
    App\Order::where('price', '>', 100)->unsearchable();

    // 通过模型关系删除...
    $user->orders()->unsearchable();

    // 通过集合删除...
    $orders->unsearchable();

<a name="pausing-indexing"></a>
### 暂停索引

你可能需要在执行一批 Eloquent 操作的时候，不同步模型数据到搜索索引。此时你可以使用 `withoutSyncingToSearch` 方法来执行此操作。这个方法接受一个立即执行的回调。该回调中所有的操作都不会同步到模型的索引：

    App\Order::withoutSyncingToSearch(function () {
        // 执行模型动作...
    });

<a name="searching"></a>
## 搜索

你可以使用 `search` 方法来搜索模型。`search` 方法接受一个用于搜索模型的字符串。你还需在搜索查询上链式调用 `get` 方法，才能用给定的搜索语句查询与之匹配的 Eloquent 模型：

    $orders = App\Order::search('Star Trek')->get();
Scout 搜索返回 Eloquent 模型的集合，因此你可以直接从路由或控制器返回结果，它们会被自动转换成 JSON 格式：

    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return App\Order::search($request->search)->get();
    });

如果你想在它们返回 Eloquent 模型前得到原结果，你应该使用`raw` 方法:

    $orders = App\Order::search('Star Trek')->raw();

搜索查询通常会在模型的 [`searchableAs`](#configuring-model-indexes) 方法指定的索引上执行。当然，你也可以使用 `within` 方法指定应该搜索的自定义索引:

    $orders = App\Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();

<a name="where-clauses"></a>
### Where 语句

Scout 允许你在搜索查询中增加简单的「where」语句。目前，这些语句只支持基本的数值等式检查，并且主要是用于根据拥有者的 ID 进行的范围搜索查询。由于搜索索引不是关系型数据库，因此当前不支持更高级的「where」语句：

    $orders = App\Order::search('Star Trek')->where('user_id', 1)->get();

<a name="pagination"></a>
### 分页

除了检索模型的集合，你也可以使用 `paginate` 方法对搜索结果进行分页。这个方法会返回一个就像 [传统的 Eloquent 查询分页](/docs/{{version}}/pagination) 一样的 `Paginator`  实例：

    $orders = App\Order::search('Star Trek')->paginate();

你可以通过将数量作为第一个参数传递给 `paginate` 方法来指定每页检索多少个模型：

    $orders = App\Order::search('Star Trek')->paginate(15);

获取到检索结果后，就可以使用 [Blade](/docs/{{version}}/blade) 来渲染分页链接从而显示结果，就像传统的 Eloquent 查询分页一样：

    <div class="container">
        @foreach ($orders as $order)
            {{ $order->price }}
        @endforeach
    </div>

    {{ $orders->links() }}

<a name="custom-engines"></a>
## 自定义引擎

#### 写引擎

如果内置的 Scout 搜索引擎不能满足你的需求，你可以写自定义的引擎并且将它注册到 Scout。你的引擎需要继承 `Laravel\Scout\Engines\Engine` 抽象类，这个抽象类包含了你自定义的引擎必须要实现的五种方法：

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function map($results, $model);

在 `Laravel\Scout\Engines\AlgoliaEngine` 类里查看这些方法会对你有较大的帮助。这个类会为你在学习如何在自定义引擎中实现这些方法提供一个好的起点。

#### 注册引擎

一旦你写好了自定义引擎，你可以用 Scout 引擎管理的 `extend` 方法将它注册到 Scout。你只需要从 `AppServiceProvider` 下的 `boot` 方法或者应用中使用的任何一个服务提供器调用 `extend` 方法。举个例子，如果你写好了一个 `MySqlSearchEngine`，你可以像这样去注册它：

    use Laravel\Scout\EngineManager;

    /**
     * 引导任何应用服务。
     *
     * @return void
     */
    public function boot()
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }

引擎注册后，你可以在 `config/scout.php` 配置文件中指定它为默认的 Scout `driver`：

    'driver' => 'mysql',

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
| ---| --- | --- | --- |
| [@Insua](https://phphub.org/users/3853) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/3853_1457586148.jpeg?imageView2/1/w/100/h/100"/> | 翻译 | happay coding with laravel+vue |


---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
