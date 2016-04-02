# 数据库: 迁移

- [简介](#introduction)
- [产生迁移](#generating-migrations)
- [迁移结构](#migration-structure)
- [运行迁移](#running-migrations)
    - [还原迁移](#rolling-back-migrations)
- [编写迁移](#writing-migrations)
    - [创建数据表](#creating-tables)
    - [重命名与删除数据表](#renaming-and-dropping-tables)
    - [创建字段](#creating-columns)
    - [修改字段](#modifying-columns)
    - [移除字段](#dropping-columns)
    - [创建索引](#creating-indexes)
    - [移除索引](#dropping-indexes)
    - [外键约束](#foreign-key-constraints)

<a name="introduction"></a>
## 简介

迁移是一种数据库的版本控制，让团队能够轻松的修改跟共享应用程序的数据库结构。迁移通常会搭配 Laravel 的结构建构器，让你可以轻松的建构应用程序的数据库结构。

Laravel 的 `Schema` [facade](/docs/{{version}}/facades) 提供了在数据库创建和操作数据表的相关支持。它对所有 Laravel 所支持的数据库系统共用了同样一目了然、流畅的 API。

<a name="generating-migrations"></a>
## 产生迁移

可以使用 `make:migration` [Artisan 命令](/docs/{{version}}/artisan) 创建迁移：

    php artisan make:migration create_users_table

新的迁移文件将会放置在 `database/migrations` 目录中。每个迁移文件名称都包含了一个时间戳记，让 Laravel 能够确认迁移的顺序。

`--table` 和 `--create` 选项可用来指定数据表的名称，或是该迁移会创建新的数据表。这些选项只需预先在产生迁移建置文件时填入指定的数据表：

    php artisan make:migration add_votes_to_users_table --table=users

    php artisan make:migration create_users_table --create=users

如果你想为产生的迁移指定一个自定的输出路径，你可以在运行 `make:migration` 命令时使用 `--path` 选项。提供的路径必须相对于你应用程序的基本路径。

<a name="migration-structure"></a>
## 迁移结构

一个迁移类会包含两个方法：`up` 和 `down`。`up` 方法用于在数据库内增加新的数据表表、字段、或索引，而 `down` 方法则必须简单的反向运行 `up` 方法的操作。

这两个方法中你可以使用 Laravel 结构建构器明确的创建及修改数据表。若要了解`结构`建构器中所有可用的方法，[请查阅它的文档](#creating-tables)。例如：让我们瞧瞧创建一张 `flights` 数据表的例子：

    <?php

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;

    class CreateFlightsTable extends Migration
    {
        /**
         * 运行迁移。
         *
         * @return void
         */
        public function up()
        {
            Schema::create('flights', function (Blueprint $table) {
                $table->increments('id');
                $table->string('name');
                $table->string('airline');
                $table->timestamps();
            });
        }

        /**
         * 还原迁移。
         *
         * @return void
         */
        public function down()
        {
            Schema::drop('flights');
        }
    }


<a name="running-migrations"></a>
## 运行迁移

要运行你应用程序中所有未完成的迁移，可以使用 `migrate` Artisan 命令。如果你使用 [Homestead 虚拟主机](/docs/{{version}}/homestead)，你应该在你的虚拟机中运行下方的命令：

    php artisan migrate

如果在你运行时出现「class not found」的错误，请试着在运行 `composer dump-autoload` 命令后再次运行一次迁移命令。

#### 在上线环境强制运行迁移

一些迁移的操作是有破坏性的，意思是它们可能会导致你失去数据。为了保护你上线环境的数据库运行这些命令，你会在这些命令被运行之前，系统将会提示你进行确认。若要忽略提示强制运行命令，你可以使用 `--force` 标记：

    php artisan migrate --force

<a name="rolling-back-migrations"></a>
### 还原迁移

若要还原迁移至上一个「操作」，你可以使用 `rollback` 命令。请注意，此还原是回复到上一次运行的「批量」迁移，其中可能包括多笔的迁移文件：

    php artisan migrate:rollback

`migrate:reset` 命令会还原应用程序的所有迁移：

    php artisan migrate:reset

#### 单个命令还原或运行迁移

`migrate:refresh` 命令首先会还原你数据库的所有迁移，接着再运行 `migrate` 命令。此命令能有效的重新创建整个数据库：

    php artisan migrate:refresh

    php artisan migrate:refresh --seed

<a name="writing-migrations"></a>
## 编写迁移

<a name="creating-tables"></a>
### 建力数据表

要创建一张新的数据表，可以使用 `Schema` facade 的 `create`方法。`create` 方法接收两个参数。第一个参数为数据表的名称，第二个参数为一个`闭包`，它接收一个用于定义新数据表的 `Blueprint` 对象：

    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
    });

当然，当创建数据表时，你可以使用任何的结构建构器的[字段方法](#creating-columns)来定义数据表的字段。

#### 检查数据表或字段是否存在

你可以使用 `hasTable` 和 `hasColumn` 方法简单的检查数据表或字段是否存在：

    if (Schema::hasTable('users')) {
        //
    }

    if (Schema::hasColumn('users', 'email')) {
        //
    }

#### 连接与保存引擎

如果你想要在一个非默认的数据库连接进行结构操作，可以使用 `connection` 方法：

    Schema::connection('foo')->create('users', function ($table) {
        $table->increments('id');
    });

若要设置数据表的保存引擎，只要在结构建构器上设置 `engine` 属性：

    Schema::create('users', function ($table) {
        $table->engine = 'InnoDB';

        $table->increments('id');
    });

<a name="renaming-and-dropping-tables"></a>
### 重命名与删除数据表

若要重命名一张已存在的数据表，可以使用 `rename` 方法：

    Schema::rename($from, $to);

要删除已存在的数据表，你可使用 `drop` 或 `dropIfExists` 方法：

    Schema::drop('users');

    Schema::dropIfExists('users');

<a name="creating-columns"></a>
### 创建字段

若要更新一张已存在的数据表，我们会使用 `Schema` facade 的 `table` 方法。如同 `create` 方法，`table` 方法接收两个参数：数据表的名称，及一个接收 `Blueprint` 实例的`闭包`，我们可以使用它为数据表增加字段：

    Schema::table('users', function ($table) {
        $table->string('email');
    });

#### 可用的字段类型

当然，结构建构器包含许多字段类型，供你建构数据表时使用：

命令  | 描述
------------- | -------------
`$table->bigIncrements('id');`  |  递增的 ID（主键），使用相当于「UNSIGNED BIG INTEGER」的型态。
`$table->bigInteger('votes');`  |  相当于 BIGINT 型态。
`$table->binary('data');`  |  相当于 BLOB 型态。
`$table->boolean('confirmed');`  | 相当于 BOOLEAN 型态。
`$table->char('name', 4);`  | 相当于 CHAR 型态，并带有长度。
`$table->date('created_at');`  |  相当于 DATE 型态。
`$table->dateTime('created_at');`  |  相当于 DATETIME 型态。
`$table->decimal('amount', 5, 2);`  |  相当于 DECIMAL 型态，并带有精度与基数。
`$table->double('column', 15, 8);`  |  相当于 DOUBLE 型态，总共有 15 位数，在小数点后面有 8 位数。
`$table->enum('choices', ['foo', 'bar']);` | 相当于 ENUM 型态。
`$table->float('amount');`  |  相当于 FLOAT 型态。
`$table->increments('id');`  |  递增的 ID (主键)，使用相当于「UNSIGNED INTEGER」的型态。
`$table->integer('votes');`  |  相当于 INTEGER 型态。
`$table->json('options');`  |  相当于 JSON 型态。
`$table->jsonb('options');`  |  相当于 JSONB 型态。
`$table->longText('description');`  |  相当于 LONGTEXT 型态。
`$table->mediumInteger('numbers');`  |  相当于 MEDIUMINT 型态。
`$table->mediumText('description');`  |  相当于 MEDIUMTEXT 型态。
`$table->morphs('taggable');`  |  加入整数 `taggable_id` 与字符串 `taggable_type`。
`$table->nullableTimestamps();`  |  与 `timestamps()` 相同，但允许 NULL。
`$table->rememberToken();`  |  加入 `remember_token` 使用 VARCHAR(100) NULL。
`$table->smallInteger('votes');`  |  相当于 SMALLINT 型态。
`$table->softDeletes();`  |  加入 `deleted_at` 字段于软删除使用。
`$table->string('email');`  |  相当于 VARCHAR 型态。
`$table->string('name', 100);`  |  相当于 VARCHAR 型态，并带有长度。
`$table->text('description');`  |  相当于 TEXT 型态。
`$table->time('sunrise');`  |  相当于 TIME 型态。
`$table->tinyInteger('numbers');`  |  相当于 TINYINT 型态。
`$table->timestamp('added_on');`  |  相当于 TIMESTAMP 型态。
`$table->timestamps();`  |  加入 `created_at` 和 `pdated_at` 字段。
`$table->uuid('id');`  |  相当于 UUID 型态。

#### 字段修饰

除了上述的字段类型列表，还有一些其它的字段「修饰」，你可以将它增加至字段。例如，若要让字段「nullable」，那么你可以使用 `nullable` 方法：

    Schema::table('users', function ($table) {
        $table->string('email')->nullable();
    });

以下列表为字段可用的修饰。此列表不包括[索引修饰](#creating-indexes)：

修饰  | 描述
------------- | -------------
`->first()`  |  将此字段放置在数据表的「第一个」（仅限 MySQL）
`->after('column')`  |  将此字段放置在其他字段「之后」（仅限 MySQL）
`->nullable()`  |  此字段允许写入 NULL 值
`->default($value)`  |  为此字段指定「默认」值
`->unsigned()`  |  设置 `integer` 字段为 `UNSIGNED`

<a name="changing-columns"></a>
<a name="modifying-columns"></a>
### 修改字段

#### 先决条件

在修改字段之前，务必在你的 `composer.json` 的增加 `doctrine/dbal` 依赖。Doctrine DBAL 函数库被用于判断目前的字段状态及创建调整指定字段的 SQL 查找。

#### 更新字段属性

`change` 方法让你可以修改一个已存在字段的类型，或修改字段的属性。例如，你可以想增加字符串字段的长度。要看看 `change` 方法的作用，让我们将 `name` 字段的长度从 25 增加到 50：

    Schema::table('users', function ($table) {
        $table->string('name', 50)->change();
    });

我们也能将字段修改为 nullable：

    Schema::table('users', function ($table) {
        $table->string('name', 50)->nullable()->change();
    });

<a name="renaming-columns"></a>
#### 重命名字段

要重命名字段，你可使用结构建构器的 `renameColumn` 方法。在重命名字段前，请确定你的 `composer.json` 文件内已经加入 `doctrine/dbal` 依赖：

    Schema::table('users', function ($table) {
        $table->renameColumn('from', 'to');
    });

> **注意：**数据表的 `enum` 字段目前尚未支持修改字段名称。

<a name="dropping-columns"></a>
### 移除字段

要移除字段，可使用结构建构器的 `dropColumn` 方法：

    Schema::table('users', function ($table) {
        $table->dropColumn('votes');
    });

你可以传递字段名称的数组至 `dropCloumn` 方法，移除多笔字段：

    Schema::table('users', function ($table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });

> **注意：**在 SQLite 数据库中移除字段前，你需要增加 `doctrine/dbal` 依赖至你的 `composer.json` 文件，并在你的命令行运行 `composer update` 命令安装该函数库。

> **注意：**当使用 SQLite 数据库时并不支持在单行迁移中移除或修改多笔字段。

<a name="creating-indexes"></a>
### 创建索引

结构建构器支持多种类型的索引。首先，让我们看看一个指定字段的值必须是唯一的例子。要创建索引，我们可以简单的在字段定义之后链式调用上 `unique` 方法：

    $table->string('email')->unique();

此外，你可以在定义完字段之后创建索引。例如：

    $table->unique('email');

你也可以传递一个字段的数组至索引方法来创建复合索引：

    $table->index(['account_id', 'created_at']);

#### 可用的索引类型

命令  | 描述
------------- | -------------
`$table->primary('id');`  |  加入主键。
`$table->primary(['first', 'last']);`  |  加入复合键。
`$table->unique('email');`  |  加入唯一索引。
`$table->index('state');`  |  加入基本索引。

<a name="dropping-indexes"></a>
### 移除索引

若要移除索引，你必须指定索引的名称。默认的，Laravel 会自动分配合理的名称至索引。简单地链接这些数据表名称，索引的字段名称，及索引类型。举例如下：

命令  | 描述
------------- | -------------
`$table->dropPrimary('users_id_primary');`  |  从「users」数据表移除主键。
`$table->dropUnique('users_email_unique');`  |  从「users」数据表移除唯一索引。
`$table->dropIndex('geo_state_index');`  |  从「geo」数据表移除基本索引。

<a name="foreign-key-constraints"></a>
### 外键约束

Laravel 也为创建外键约束提供支持，这是用于在数据库层面强制参考完整性。例如，让我们定义 `posts` 数据表内有个 `user_id` 字段要参考至 `users` 数据表的 `id` 字段：

    Schema::table('posts', function ($table) {
        $table->integer('user_id')->unsigned();

        $table->foreign('user_id')->references('id')->on('users');
    });

你也可以指定约束的「on delete」及「on update」作为所需的操作：

    $table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

要移除外键，你可以使用 `dropForeign` 方法。外键约束与索引采用相同的命名方式。所以，我们可以链接数据表明与约束的字段，接着在该名称加上「_foreign」后缀：

    $table->dropForeign('posts_user_id_foreign');
