# Laravel 的数据库迁移 Migrations

- [简介](#introduction)
- [生成迁移](#generating-migrations)
- [迁移结构](#migration-structure)
- [运行迁移](#running-migrations)
    - [回滚迁移](#rolling-back-migrations)
- [数据表](#tables)
    - [创建数据表](#creating-tables)
    - [重命名与删除数据表](#renaming-and-dropping-tables)
- [字段](#columns)
    - [创建字段](#creating-columns)
    - [字段修饰](#column-modifiers)
    - [修改字段](#modifying-columns)
    - [移除字段](#dropping-columns)
- [索引](#indexes)
    - [创建索引](#creating-indexes)
    - [删除索引](#dropping-indexes)
    - [外键约束](#foreign-key-constraints)

<a name="introduction"></a>
## 简介

数据库迁移就像是数据库的版本控制，可以让你的团队轻松修改并共享应用程序的数据库结构。迁移通常会搭配上 Laravel 的数据库结构构造器来让你方便地构建数据库结构。如果你曾经出现过让同事手动在数据库结构中添加字段的情况，数据库迁移可以解决你这个问题。

Laravel 的 `Schema` [facade](/docs/{{version}}/facades) 对所有 Laravel 支持的数据库系统提供了创建和操作数据表的相应支持。

<a name="generating-migrations"></a>
## 生成迁移

使用 `make:migration` [Artisan 命令](/docs/{{version}}/artisan) 来创建迁移：

    php artisan make:migration create_users_table

新的迁移文件将会被放置在 `database/migrations` 目录中。每个迁移文件的名称都包含了一个时间戳，以便让 Laravel 确认迁移的顺序。

`--table` 和 `--create` 选项可用来指定数据表的名称，或是该迁移被执行时会创建的新数据表。这些选项需在预生成迁移文件时填入指定的数据表：

    php artisan make:migration create_users_table --create=users

    php artisan make:migration add_votes_to_users_table --table=users

如果你想为生成的迁移指定一个自定义输出路径，则可以在运行 `make:migration` 命令时添加 `--path` 选项。提供的路径必须是相对于应用程序的基本路径。

<a name="migration-structure"></a>
## 迁移结构

一个迁移类会包含两个方法： `up` 和 `down` 。 `up` 方法可为数据库添加新的数据表、字段或索引，而 `down` 方法则是 `up` 方法的逆操作。

你可以在这两个方法中使用 Laravel 数据库结构构造器来创建以及修改数据表。若要了解 `数据库结构` 构造器中的所有可用方法，[可查阅它的文档](#creating-tables)。以下的迁移实例会创建一张 `flights` 数据表：

    <?php

    use Illuminate\Support\Facades\Schema;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;

    class CreateFlightsTable extends Migration
    {
        /**
         * 运行数据库迁移
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
         * 回滚数据库迁移
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

使用 `migrate` Artisan 命令，来运行所有未运行过的迁移：

    php artisan migrate

> {note} 如果你使用的是 [Homestead 虚拟机](/docs/{{version}}/homestead) ， 你需要在虚拟机中执行以上命令。

#### 在线上环境强制执行迁移

一些迁移的操作是具有破坏性的，它们可能会导致数据丢失。为了保护线上环境的数据库，系统会在这些命令被运行之前显示确认提示。若要忽略此提示并强制运行命令，则可以使用 `--force` 标记：

    php artisan migrate --force

<a name="rolling-back-migrations"></a>
### 回滚迁移

若要回滚最后一次迁移，则可以使用 `rollback` 命令。此命令是对上一次执行的「批量」迁移回滚，其中可能包括多个迁移文件：

    php artisan migrate:rollback

在 `rollback` 命令后加上 `step` 参数，你可以限制回滚迁移的个数。例如，下面的命令将会回滚最后的 5 个迁移。

    php artisan migrate:rollback --step=5

`migrate:reset` 命令可以回滚应用程序中的所有迁移：

    php artisan migrate:reset

#### 使用单个命令来执行回滚或迁移

`migrate:refresh` 命令不仅会回滚数据库的所有迁移还会接着运行 `migrate` 命令。所以此命令可以有效的重新创建整个数据库：

    php artisan migrate:refresh

    // 刷新数据库结构并执行数据填充
    php artisan migrate:refresh --seed

使用 `refresh` 命令并加上 `step` 参数，你也可以限制执行回滚和再迁移的个数。比如，下面的命令会回滚并再迁移最后的 5 个迁移：

    php artisan migrate:refresh --step=5

<a name="tables"></a>
## 数据表

<a name="creating-tables"></a>
### 创建数据表

要创建一张新的数据表，可以使用 `Schema` facade 的 `create` 方法。`create` 方法接收两个参数：第一个参数为数据表的名称，第二个参数为一个 `闭包` ，此闭包会接收一个用于定义新数据表的 `Blueprint` 对象：

    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
    });

当然，在创建数据表的时候，你也可以使用任何数据库结构构造器的 [字段方法](#creating-columns) 来定义数据表的字段。

#### 检查数据表或字段是否存在

你可以方便地使用 `hasTable` 和 `hasColumn` 方法来检查数据表或字段是否存在：

    if (Schema::hasTable('users')) {
        //
    }

    if (Schema::hasColumn('users', 'email')) {
        //
    }

#### 数据库连接与存储引擎

如果你想要在一个非默认的数据库连接中进行数据库结构操作，可以使用 `connection` 方法：

    Schema::connection('foo')->create('users', function (Blueprint $table) {
        $table->increments('id');
    });

你可以在数据库结构构造器上设置 `engine` 属性来设置数据表的存储引擎：

    Schema::create('users', function (Blueprint $table) {
        $table->engine = 'InnoDB';

        $table->increments('id');
    });

<a name="renaming-and-dropping-tables"></a>
### 重命名与删除数据表

若要重命名一张已存在的数据表，可以使用 `rename` 方法：

    Schema::rename($from, $to);

要删除已存在的数据表，可使用 `drop` 或 `dropIfExists` 方法：

    Schema::drop('users');

    Schema::dropIfExists('users');

#### 重命名带外键的数据表

在重命名前，你需要检查外键的约束涉及到的数据表名需要在迁移文件中显式的提供，而不是让 Laravel 按照约定来设置一个名称。因为那样会让外键约束关联到旧的数据表上。

<a name="columns"></a>
## 字段

<a name="creating-columns"></a>
### 创建字段

使用 `Schema` facade 的 `table` 方法可以更新已有的数据表。如同 `create` 方法，`table` 方法会接收两个参数：一个是数据表的名称，另一个则是接收 `Blueprint` 实例的`闭包`。我们可以使用它来为数据表新增字段：

    Schema::table('users', function (Blueprint $table) {
        $table->string('email');
    });

#### 可用的字段类型

数据库结构构造器包含了许多字段类型，供你构建数据表时使用：

命令  | 描述
------------- | -------------
`$table->bigIncrements('id');`  |  递增 ID（主键），相当于「UNSIGNED BIG INTEGER」型态。
`$table->bigInteger('votes');`  |  相当于 BIGINT 型态。
`$table->binary('data');`  |  相当于 BLOB 型态。
`$table->boolean('confirmed');`  |  相当于 BOOLEAN 型态。
`$table->char('name', 4);`  |  相当于 CHAR 型态，并带有长度。
`$table->date('created_at');`  |  相当于 DATE 型态
`$table->dateTime('created_at');`  |  相当于 DATETIME 型态。
`$table->dateTimeTz('created_at');`  |  DATETIME (带时区) 形态
`$table->decimal('amount', 5, 2);`  |  相当于 DECIMAL 型态，并带有精度与基数。
`$table->double('column', 15, 8);`  |  相当于 DOUBLE 型态，总共有 15 位数，在小数点后面有 8 位数。
`$table->enum('choices', ['foo', 'bar']);` | 相当于 ENUM 型态。
`$table->float('amount', 8, 2);`  |  相当于 FLOAT 型态，总共有 8 位数，在小数点后面有 2 位数。
`$table->increments('id');`  | 递增的 ID (主键)，使用相当于「UNSIGNED INTEGER」的型态。
`$table->integer('votes');`  |  相当于 INTEGER 型态。
`$table->ipAddress('visitor');`  |  相当于 IP 地址形态。
`$table->json('options');`  |  相当于 JSON 型态。
`$table->jsonb('options');`  |  相当于 JSONB 型态。
`$table->longText('description');`  |  相当于 LONGTEXT 型态。
`$table->macAddress('device');`  |  相当于 MAC 地址形态。
`$table->mediumIncrements('id');`  |  递增 ID (主键) ，相当于「UNSIGNED MEDIUM INTEGER」型态。
`$table->mediumInteger('numbers');`  |  相当于 MEDIUMINT 型态。
`$table->mediumText('description');`  |  相当于 MEDIUMTEXT 型态。
`$table->morphs('taggable');`  |  加入整数 `taggable_id` 与字符串 `taggable_type`。
`$table->nullableMorphs('taggable');`  |  与 `morphs()` 字段相同，但允许为NULL。
`$table->nullableTimestamps();`  |  与 `timestamps()` 相同，但允许为 NULL。
`$table->rememberToken();`  |  加入 `remember_token` 并使用 VARCHAR(100) NULL。
`$table->smallIncrements('id');`  |  递增 ID (主键) ，相当于「UNSIGNED SMALL INTEGER」型态。
`$table->smallInteger('votes');`  |  相当于 SMALLINT 型态。
`$table->softDeletes();`  |  加入 `deleted_at` 字段用于软删除操作。
`$table->string('email');`  |  相当于 VARCHAR 型态。
`$table->string('name', 100);`  |  相当于 VARCHAR 型态，并带有长度。
`$table->text('description');`  |  相当于 TEXT 型态。
`$table->time('sunrise');`  |  相当于 TIME 型态。
`$table->timeTz('sunrise');`  |  相当于 TIME (带时区) 形态。
`$table->tinyInteger('numbers');`  |  相当于 TINYINT 型态。
`$table->timestamp('added_on');`  |  相当于 TIMESTAMP 型态。
`$table->timestampTz('added_on');`  |  相当于 TIMESTAMP (带时区) 形态。
`$table->timestamps();`  |  加入 `created_at` 和 `updated_at` 字段。
`$table->timestampsTz();`  |  加入 `created_at` and `updated_at` (带时区) 字段，并允许为NULL。
`$table->unsignedBigInteger('votes');`  |  相当于 Unsigned BIGINT 型态。
`$table->unsignedInteger('votes');`  |  相当于 Unsigned INT 型态。
`$table->unsignedMediumInteger('votes');`  |  相当于 Unsigned MEDIUMINT 型态。
`$table->unsignedSmallInteger('votes');`  |  相当于 Unsigned SMALLINT 型态。
`$table->unsignedTinyInteger('votes');`  |  相当于 Unsigned TINYINT 型态。
`$table->uuid('id');`  |  相当于 UUID 型态。

<a name="column-modifiers"></a>
### 字段修饰

除了上述的字段类型列表，还有一些其它的字段「修饰」，你可以将它增加到字段中。例如，若要让字段「nullable」，那么你可以使用 `nullable` 方法：

    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->nullable();
    });

以下列表为字段的可用修饰。此列表不包括 [索引修饰](#creating-indexes)：

Modifier  | Description
------------- | -------------
`->after('column')`  |  将此字段放置在其它字段「之后」（仅限 MySQL）
`->comment('my comment')`  |  增加注释
`->default($value)`  |  为此字段指定「默认」值
`->first()`  |  将此字段放置在数据表的「首位」（仅限 MySQL）
`->nullable()`  |  此字段允许写入 NULL 值
`->storedAs($expression)`  |  创建一个存储的生成字段 （仅限 MySQL）
`->unsigned()`  |  设置 `integer` 字段为 `UNSIGNED`
`->virtualAs($expression)`  |  创建一个虚拟的生成字段 （仅限 MySQL）

<a name="changing-columns"></a>
<a name="modifying-columns"></a>
### 修改字段

#### 先决条件

在修改字段之前，请务必在你的 `composer.json` 中增加 `doctrine/dbal` 依赖。Doctrine DBAL 函数库被用于判断当前字段的状态以及创建调整指定字段的 SQL 查询。

    composer require doctrine/dbal

#### 更新字段属性

`change` 方法让你可以修改一些已存在的字段类型，或修改字段属性。比如，你可能想增加字符串字段的长度。想了解 `change` 方法如何使用，让我们来把 `name` 字段的长度从 25 增加到 50 ：

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->change();
    });

我们也能将字段修改为 nullable：

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->nullable()->change();
    });

> {note} 下面的字段类型不能被「修改」: char，double，enum，mediumInteger，timestamp，tinyInteger，ipAddress，json，jsonb，macAddress，mediumIncrements，morphs，nullableMorphs，nullableTimestamps，softDeletes，timeTz，timestampTz，timestamps，timestampsTz，unsignedMediumInteger，unsignedTinyInteger，uuid。

<a name="renaming-columns"></a>
#### 重命名字段

要重命名字段，可使用数据库结构构造器的 `renameColumn` 方法。在重命名字段前，请确保你的 `composer.json` 文件内已经加入 `doctrine/dbal` 依赖：

    Schema::table('users', function (Blueprint $table) {
        $table->renameColumn('from', 'to');
    });

> {note} 数据表的 `enum` 字段暂时不支持修改字段属性。

<a name="dropping-columns"></a>
### 移除字段

要移除字段，可使用数据库结构构造器的 `dropColumn` 方法。在删除 SQLite 数据库的字段前，你需要在 `composer.json` 文件中加入 `doctrine/dbal` 依赖并在终端执行 `composer update` 来安装函数库：

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('votes');
    });

你可以传递多个字段的数组至 `dropCloumn` 方法来移除多个字段：

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });

> {note} SQLite 数据库并不支持在单个迁移中移除或修改多个字段。

<a name="indexes"></a>
## 索引

<a name="creating-indexes"></a>
### 创建索引

数据库结构构造器支持多种类型的索引。首先，让我们先来看一个示例，其指定了字段的值必须是唯一的。你可以简单的在字段定义之后链式调用 `unique` 方法来创建索引：

    $table->string('email')->unique();

此外，你也可以在定义完字段之后创建索引。例如：

    $table->unique('email');

你也可以传递一个字段的数组至索引方法来创建复合索引：

    $table->index(['account_id', 'created_at']);

Laravel 会自动生成一个合理的索引名称，但你也可以使用第二个参数来自定义索引名称:

    $table->index('email', 'my_index_name');

#### 可用的索引类型

Command  | Description
------------- | -------------
`$table->primary('id');`  |  加入主键。
`$table->primary(['first', 'last']);`  |  加入复合键。
`$table->unique('email');`  |  加入唯一索引。
`$table->unique('state', 'my_index_name');`  |  自定义索引名称。
`$table->unique(['first', 'last']);`  | 加入复合唯一键。
`$table->index('state');`  |  加入基本索引。

#### 索引长度 & MySQL / MariaDB

Laravel 默认使用 `utf8mb4` 字符，包括支持在数据库存储「表情」。如果你正在运行的 MySQL release 版本低于5.7.7 或 MariaDB release 版本低于10.2.2 ，为了MySQL为它们创建索引，你可能需要手动配置迁移生成的默认字符串长度，你可以通过调用 `AppServiceProvider` 中的 `Schema::defaultStringLength` 方法来配置它：

    use Illuminate\Support\Facades\Schema;

    /**
     * 引导任何应用程序服务。
     *
     * @return void
     */
    public function boot()
    {
        Schema::defaultStringLength(191);
    }

或者你可以为数据库开启 `innodb_large_prefix` 选项，有关如何正确开启此选项的说明请查阅数据库文档。

<a name="dropping-indexes"></a>
### 移除索引

若要移除索引，则必须指定索引的名称。Laravel 默认会自动给索引分配合理的名称。其将数据表名称、索引的字段名称及索引类型简单地连接在了一起。举例如下：

命令  | 描述
------------- | -------------
`$table->dropPrimary('users_id_primary');`  |  从「users」数据表移除主键。
`$table->dropUnique('users_email_unique');`  |  从「users」数据表移除唯一索引。
`$table->dropIndex('geo_state_index');`  |  从「geo」数据表移除基本索引。

如果你对 `dropIndex` 传参索引数组，默认的约定是索引名称由数据库表名字和键名拼接而成：

    Schema::table('geo', function (Blueprint $table) {
        $table->dropIndex(['state']); // 移除索引 'geo_state_index'
    });

<a name="foreign-key-constraints"></a>
### 外键约束

Laravel 也为创建外键约束提供了支持，用于在数据库层中的强制引用完整性。例如，让我们定义一个有 `user_id` 字段的 `posts` 数据表，`user_id` 引用了 `users` 数据表的 `id` 字段：

    Schema::table('posts', function (Blueprint $table) {
        $table->integer('user_id')->unsigned();

        $table->foreign('user_id')->references('id')->on('users');
    });

你也可以指定约束的「on delete」及「on update」：

    $table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

要移除外键，你可以使用 `dropForeign` 方法。外键约束与索引采用相同的命名方式。所以，我们可以将数据表名称和约束字段连接起来，接着在该名称后面加上「_foreign」后缀：

    $table->dropForeign('posts_user_id_foreign');

你也可以传递一个包含字段的数组，在移除的时候字段会按照惯例被自动转换为对应的外键名称：

    $table->dropForeign(['user_id']);

你可以在迁移文件里使用以下方法来开启和关闭外键约束：

    Schema::enableForeignKeyConstraints();

    Schema::disableForeignKeyConstraints();

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@Kevin](https://github.com/hivenkay)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/14940_1491960022.jpeg?imageView2/1/w/100/h/100">  |  翻译  | 初心易得，始终难守  |



--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org