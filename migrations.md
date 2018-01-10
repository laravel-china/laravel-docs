# Laravel 的数据库迁移 Migrations

- [简介](#introduction)
- [生成迁移](#generating-migrations)
- [迁移结构](#migration-structure)
- [运行迁移](#running-migrations)
    - [回滚迁移](#rolling-back-migrations)
- [数据表](#tables)
    - [创建数据表](#creating-tables)
    - [重命名/删除数据表](#renaming-and-dropping-tables)
- [字段](#columns)
    - [创建字段](#creating-columns)
    - [字段修饰](#column-modifiers)
    - [修改字段](#modifying-columns)
    - [删除字段](#dropping-columns)
- [索引](#indexes)
    - [创建索引](#creating-indexes)
    - [删除索引](#dropping-indexes)
    - [外键约束](#foreign-key-constraints)

<a name="introduction"></a>
## 简介

数据库迁移就像是数据库的版本控制，可以让你的团队轻松修改并共享应用程序的数据库结构。迁移通常与 Laravel 的数据库结构生成器配合使用，让你轻松地构建数据库结构。如果你曾经试过让同事手动在数据库结构中添加字段，那么数据库迁移可以让你不再需要做这样的事情。

Laravel `Schema` [facade](/docs/{{version}}/facades) 对所有 Laravel 支持的数据库系统提供了创建和操作数据表的相应支持。

<a name="generating-migrations"></a>
## 生成迁移

使用 [Artisan 命令](/docs/{{version}}/artisan)  `make:migration` 来创建迁移：

    php artisan make:migration create_users_table

新的迁移文件会被放置在 `database/migrations` 目录中。每个迁移文件的名称都包含了一个时间戳，以便让 Laravel 确认迁移的顺序。

`--table` 和 `--create` 选项可用来指定数据表的名称，或是该迁移被执行时是否将创建的新数据表。这些选项需在预生成迁移文件时填入指定的数据表：

    php artisan make:migration create_users_table --create=users

    php artisan make:migration add_votes_to_users_table --table=users

如果你想为生成的迁移指定一个自定义输出路径，则可以在运行 `make:migration` 命令时添加 `--path` 选项。给定的路径必须是相对于应用程序的基本路径。

<a name="migration-structure"></a>
## 迁移结构

迁移类通常会包含两个方法：`up` 和 `down`。`up` 方法可为数据库添加新的数据表、字段或索引，而 `down` 方法则是 `up` 方法的逆操作。

你可以在这两个方法中使用 Laravel 数据库结构生成器来创建以及修改数据表。若要了解  `Schema` 生成器中的所有可用方法，[可查阅它的文档](#creating-tables)。以下的迁移实例会创建一张 `flights` 数据表：

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

使用 Artisan 命令 `migrate` 来运行所有未完成的迁移：

    php artisan migrate

> {note} 如果你使用的是 [Homestead 虚拟机](/docs/{{version}}/homestead)，则应该在虚拟机中执行命令。

#### 在生产环境强制执行迁移

一些迁移操作是具有破坏性的，这意味着可能会导致数据丢失。为了防止有人在生产环境中运行这些命令，系统会在这些命令被运行之前与你进行确认。如果要强制忽略系统的提示运行命令，则可以使用 `--force` 标记：

    php artisan migrate --force

<a name="rolling-back-migrations"></a>
### 回滚迁移

若要回滚最后一次迁移，可以使用 `rollback` 命令。此命令将回滚最后一次「迁移」，其中可能包含多个迁移文件：

    php artisan migrate:rollback
在 `rollback` 命令后加上 `step` 参数，你可以限制回滚迁移的个数。例如，以下命令将回滚最近五次迁移：

    php artisan migrate:rollback --step=5

`migrate:reset` 命令可以回滚应用程序中的所有迁移：

    php artisan migrate:reset

#### 使用单个命令来执行回滚或迁移

`migrate:refresh` 命令不仅会回滚数据库的所有迁移还会接着运行 `migrate` 命令。这个命令可以高效地重建整个数据库：

    php artisan migrate:refresh

    // 刷新数据库结构并执行数据填充
    php artisan migrate:refresh --seed

使用 `refresh` 命令并提供 `step` 参数来回滚并再执行最后指定的迁移数。例如，以下命令将回滚并重新执行最后五次迁移：

    php artisan migrate:refresh --step=5
#### 删除所有表 & 迁移

`migrate:fresh` 命令会从数据库中删除所有表，然后执行 `migrate` 命令:

    php artisan migrate:fresh

    php artisan migrate:fresh --seed

<a name="tables"></a>

## 数据表

<a name="creating-tables"></a>
### 创建数据表

可以使用 `Schema` facade 的 `create` 方法来创建新的数据库表。`create` 方法接受两个参数：第一个参数为数据表的名称，第二个参数是 `闭包`，此闭包会接收一个用于定义新数据表的 `Blueprint` 对象：

    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
    });

当然，在创建数据表的时候，可以使用任何数据库结构生成器的 [字段方法](#creating-columns) 来定义数据表的字段。

#### 检查数据表或字段是否存在

可以使用 `hasTable` 和 `hasColumn` 方法来检查数据表或字段是否存在：

    if (Schema::hasTable('users')) {
        //
    }

    if (Schema::hasColumn('users', 'email')) {
        //
    }

#### 数据库连接 & 表选项

如果要对非默认连接的数据库连接执行结构操作，可以使用 `connection` 方法：

    Schema::connection('foo')->create('users', function (Blueprint $table) {
        $table->increments('id');
    });

你可以在数据库结构生成器上使用以下命令来定义表的选项：

| 命令| 描述 |
| ---------------------------------------- | --------------------- |
| `$table->engine = 'InnoDB';`             | 指定表存储引擎 (MySQL) |
| `$table->charset = 'utf8';`              | 指定数据表的默认字符集 (MySQL)|
| `$table->collation = 'utf8_unicode_ci';` | 指定数据表默认的排序规则 (MySQL)|
| `$table->temporary();`                   | 创建临时表(不支持SQL Server). |

<a name="renaming-and-dropping-tables"></a>
### 重命名/删除数据表

若要重命名数据表，可以使用 `rename` 方法：

    Schema::rename($from, $to);

要删除数据表，可使用 `drop` 或 `dropIfExists` 方法：

    Schema::drop('users');

    Schema::dropIfExists('users');

#### 重命名带外键的数据表

在重命名表之前，你应该验证表上的任何外键约束在迁移文件中都有明确的名称，而不是让 Laravel 按照约定来设置一个名称。否则，外键的约束名称将引用旧表名。

<a name="columns"></a>
## 字段

<a name="creating-columns"></a>
### 创建字段

使用 `Schema` facade 的 `table` 方法可以更新现有的数据表。如同 `create` 方法一样，`table` 方法会接受两个参数：一个是数据表的名称，另一个则是接收可以用来向表中添加字段的 `Blueprint` 实例的`闭包`：

    Schema::table('users', function (Blueprint $table) {
        $table->string('email');
    });

#### 可用的字段类型

数据库结构生成器包含构建表时可以指定的各种字段类型：

| 命令 | 描述 |
| ---------------------------------------- | ---------------------------------------- |
| `$table->bigIncrements('id');`           | 递增 ID（主键），相当于「UNSIGNED BIG INTEGER」|
| `$table->bigInteger('votes');`           | 相当于 BIGINT |
| `$table->binary('data');`                | 相当于 BLOB |
| `$table->boolean('confirmed');`          | 相当于 BOOLEAN |
| `$table->char('name', 4);`               | 相当于带有长度的 CHAR |
| `$table->date('created_at');`            | 相当于 DATE |
| `$table->dateTime('created_at');`        | 相当于 DATETIME |
| `$table->dateTimeTz('created_at');`      | 相当于带时区 DATETIME |
| `$table->decimal('amount', 8, 2);`       | 相当于带有精度与基数 DECIMAL |
| `$table->double('column', 8, 2);`        | 相当于带有精度与基数 DOUBLE |
| `$table->enum('level', ['easy', 'hard']);` | 相当于 ENUM |
| `$table->float('amount', 8, 2);`         | 相当于带有精度与基数 FLOAT |
| `$table->geometry('positions');`         | 相当于 GEOMETRY |
| `$table->geometryCollection('positions');` | 相当于 GEOMETRYCOLLECTION |
| `$table->increments('id');`              | 递增的 ID (主键)，相当于「UNSIGNED INTEGER」|
| `$table->integer('votes');`              | 相当于 INTEGER |
| `$table->ipAddress('visitor');`          | 相当于 IP 地址 |
| `$table->json('options');`               | 相当于 JSON |
| `$table->jsonb('options');`              | 相当于 JSONB |
| `$table->lineString('positions');`       | 相当于 LINESTRING |
| `$table->longText('description');`       | 相当于 LONGTEXT |
| `$table->macAddress('device');`          | 相当于 MAC 地址 |
| `$table->mediumIncrements('id');`        | 递增 ID (主键) ，相当于「UNSIGNED MEDIUM INTEGER」 |
| `$table->mediumInteger('votes');`        | 相当于 MEDIUMINT |
| `$table->mediumText('description');`     | 相当于 MEDIUMTEXT |
| `$table->morphs('taggable');`            | 相当于加入递增的 `taggable_id` 与字符串 `taggable_type` |
| `$table->multiLineString('positions');`  | 相当于 MULTILINESTRING |
| `$table->multiPoint('positions');`       | 相当于 MULTIPOINT |
| `$table->multiPolygon('positions');`     | 相当于 MULTIPOLYGON |
| `$table->nullableMorphs('taggable');`    | 相当于可空版本的 `morphs()` 字段 |
| `$table->nullableTimestamps();`          | 相当于可空版本的 `timestamps()` 字段 |
| `$table->point('position');`             | 相当于 POINT |
| `$table->polygon('positions');`          | 相当于 POLYGON |
| `$table->rememberToken();`               | 相当于可空版本的 VARCHAR(100) 的 `remember_token` 字段 |
| `$table->smallIncrements('id');`         | 递增 ID (主键) ，相当于「UNSIGNED SMALL INTEGER」|
| `$table->smallInteger('votes');`         | 相当于 SMALLINT |
| `$table->softDeletes();`                 | 相当于为软删除添加一个可空的 `deleted_at` 字段 |
| `$table->softDeletesTz();`               | 相当于为软删除添加一个可空的 带时区的 `deleted_at` 字段 |
| `$table->string('name', 100);`           | 相当于带长度的 VARCHAR |
| `$table->text('description');`           | 相当于 TEXT |
| `$table->time('sunrise');`               | 相当于 TIME |
| `$table->timeTz('sunrise');`             | 相当于带时区的 TIME |
| `$table->timestamp('added_on');`         | 相当于 TIMESTAMP |
| `$table->timestampTz('added_on');`       | 相当于带时区的 TIMESTAMP |
| `$table->tinyIncrements('id');`          | 相当于自动递增 UNSIGNED TINYINT |
| `$table->tinyInteger('votes');`          | 相当于 TINYINT |
| `$table->unsignedBigInteger('votes');`   | 相当于 Unsigned BIGINT |
| `$table->unsignedDecimal('amount', 8, 2);` | 相当于带有精度和基数的 UNSIGNED DECIMAL |
| `$table->unsignedInteger('votes');`      | 相当于 Unsigned INT |
| `$table->unsignedMediumInteger('votes');` | 相当于 Unsigned MEDIUMINT |
| `$table->unsignedSmallInteger('votes');` | 相当于 Unsigned SMALLINT |
| `$table->unsignedTinyInteger('votes');`  | 相当于 Unsigned TINYINT |
| `$table->uuid('id');`                    | 相当于 UUID |
| `$table->year('birth_year');`            | 相当于 YEAR |

<a name="column-modifiers"></a>
### 字段修饰

除了上述列出的字段类型之外，还有几个可以在添加字段时使用的「修饰符」。例如，如果要把字段设置为「可空」，就使用 `nullable` 方法：

    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->nullable();
    });

以下是所有可用的字段修饰符的列表。此列表不包括 [索引修饰符](#creating-indexes)：

| 修饰符                              | 描述                                       |
| -------------------------------- | ---------------------------------------- |
| `->after('column')`              | 将此字段放置在其它字段「之后」（仅限 MySQL）                |
| `->autoIncrement()`              | 将 INTEGER 类型的字段设置为自动递增的主键                |
| `->charset('utf8')`              | 指定一个字符集（仅限 MySQL）                        |
| `->collation('utf8_unicode_ci')` | 指定列的排序规则（仅限 MySQL/SQL Server）            |
| `->comment('my comment')`        | 为字段增加注释（仅限 MySQL）                        |
| `->default($value)`              | 为字段指定「默认」值                               |
| `->first()`                      | 将此字段放置在数据表的「首位」（仅限 MySQL）                |
| `->nullable($value = true)`      | （默认情况下）此字段允许写入 NULL 值                    |
| `->storedAs($expression)`        | 创建一个存储生成的字段 （仅限 MySQL）                   |
| `->unsigned()`                   | 设置 `integer` 类型的字段为 `UNSIGNED`（仅限 MySQL） |
| `->useCurrent()`                 | 将 TIMESTAMP 类型的字段设置为使用 CURRENT_TIMESTAMP 作为默认值 |
| `->virtualAs($expression)`       | 创建一个虚拟生成的字段 （仅限 MySQL）                   |

<a name="modifying-columns"></a>
### 修改字段

#### 先决条件

在修改字段之前，请确保将 `doctrine/dbal` 依赖添加到 `composer.json` 文件中。Doctrine DBAL 库用于确定字段的当前状态，并创建对该字段进行指定调整所需的 SQL 查询：

    composer require doctrine/dbal

#### 更新字段属性

`change` 方法可以将现有的字段类型修改为新的类型或修改属性。比如，你可能想增加字符串字段的长度，可以使用 `change` 方法把 `name` 字段的长度从 25 增加到 50 ：

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->change();
    });

将字段修改为可空：

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->nullable()->change();
    });

> {note} 只有下面的字段类型不能被「修改」: bigInteger、binary、boolean、date、dateTime、dateTimeTz、 decimal、integer、json、longText、mediumText、smallInteger、string、text、time、unsignedBigInteger、unsignedInteger 和 unsignedSmallInteger。

<a name="renaming-columns"></a>
#### 重命名字段

可以使用结构生成器上的 `renameColumn` 方法来重命名字段。在重命名字段前，请确保你的 `composer.json` 文件内已经加入 `doctrine/dbal` 依赖：

    Schema::table('users', function (Blueprint $table) {
        $table->renameColumn('from', 'to');
    });

> {note} 当前不支持 `enum` 类型的字段重命名。

<a name="dropping-columns"></a>
### 删除字段

可以使用结构生成器上的 `dropColumn` 方法来删除字段。在从 SQLite 数据库删除字段前，你需要在 `composer.json` 文件中加入 `doctrine/dbal` 依赖并在终端执行 `composer update` 来安装该依赖：

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('votes');
    });

你可以传递一个字段数组给 `dropCloumn` 方法来删除多个字段：

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });

> {note} 不支持在使用 SQLite 数据库时在单个迁移中删除或修改多个字段。

#### 可用的命令别名

| 命令 | 描述 |
| ------------------------------ | --------------------------------- |
| `$table->dropRememberToken();` | 删除 `remember_token` 字段 |
| `$table->dropSoftDeletes();`   | 删除 `deleted_at` 字段 |
| `$table->dropSoftDeletesTz();` | `dropSoftDeletes()` 方法的别名 |
| `$table->dropTimestamps();`    | 删除 `created_at` 和 `updated_at` 字段 |
| `$table->dropTimestampsTz();`  | `dropTimestamps()` 方法的别名 |

<a name="indexes"></a>

## 索引

<a name="creating-indexes"></a>
### 创建索引

结构生成器支持多种类型的索引。首先，先指定字段值唯一，即简单地在字段定义之后链式调用 `unique` 方法来创建索引，例如：

    $table->string('email')->unique();

或者，你也可以在定义完字段之后创建索引。例如：

    $table->unique('email');
你甚至可以将数组传递给索引方法来创建一个复合（或合成）索引：

    $table->index(['account_id', 'created_at']);

Laravel 会自动生成一个合理的索引名称，但你也可以传递第二个参数来自定义索引名称:

    $table->index('email', 'unique_email');

#### 可用的索引类型

| 命令 | 描述 |
| --------------------------------------- | ------------------ |
| `$table->primary('id');`                | 添加主键 |
| `$table->primary(['id', 'parent_id']);` | 添加复合键 |
| `$table->unique('email');`              | 添加唯一索引 |
| `$table->index('state');`               | 添加普通索引 |
| `$table->spatialIndex('location');`     | 添加空间索引（ SQLite 除外）|

#### 索引长度 & MySQL / MariaDB

Laravel 默认使用 `utf8mb4` 字符，它支持在数据库中存储「emojis」。如果你是在版本低于5.7.7 的 MySQL release 或者版本低于10.2.2 的 MariaDB release 上创建索引，那就需要你手动配置迁移生成的默认字符串长度。即在 `AppServiceProvider` 中调用 `Schema::defaultStringLength` 方法来配置它：

    use Illuminate\Support\Facades\Schema;

    /**
     * 引导任何应用程序服务
     *
     * @return void
     */
    public function boot()
    {
        Schema::defaultStringLength(191);
    }

或者，你可以开启数据库的 `innodb_large_prefix` 选项，至于如何正确开启，请自行查阅数据库文档。

<a name="dropping-indexes"></a>
### 删除索引

若要移除索引，则必须指定索引的名称。Laravel 默认会自动给索引分配合理的名称。其将数据表名称、索引的字段名称及索引类型简单地连接在了一起。举例如下：

| 命令 | 描述 |
| ---------------------------------------- | ------------------------- |
| `$table->dropPrimary('users_id_primary');` | 从「users」表中删除主键。|
| `$table->dropUnique('users_email_unique');` | 从「users」表中删除唯一索引。|
| `$table->dropIndex('geo_state_index');`  | 从「geo」表中删除基本索引。|
| `$table->dropSpatialIndex('geo_location_spatialindex');` | 从「geo」表中删除空间索引（SQLite除外）。 |

如果将字段数组传递给 `dropIndex` 方法，会删除根据表名、字段和键类型生成的索引名称：

    Schema::table('geo', function (Blueprint $table) {
        $table->dropIndex(['state']); // 移除索引 'geo_state_index'
    });

<a name="foreign-key-constraints"></a>
### 外键约束

Laravel 还支持创建用于在数据库层中的强制引用完整性的外键约束。例如，让我们在 `posts` 表上定义一个引用 `users` 表的 `id` 字段的 `user_id` 字段：

    Schema::table('posts', function (Blueprint $table) {
        $table->integer('user_id')->unsigned();

        $table->foreign('user_id')->references('id')->on('users');
    });

还可以为约束的「on delete」和「on update」属性指定所需的操作：

    $table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

你可以使用 `dropForeign` 方法删除外键。外键约束采用的命名方式与索引相同。即，将数据表名称和约束的字段连接起来，接着加上「_foreign」后缀：

    $table->dropForeign('posts_user_id_foreign');

或者，你也可以传递一个字段数组，在删除的时候会按照约定自动转换为对应的外键名称：

    $table->dropForeign(['user_id']);

你可以在迁移文件里使用以下方法来开启或关闭外键约束：

    Schema::enableForeignKeyConstraints();

    Schema::disableForeignKeyConstraints();

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@Kevin](https://github.com/hivenkay) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/14940_1491960022.jpeg?imageView2/1/w/100/h/100"> | 翻译   | 初心易得，始终难守 |
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | Review | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
