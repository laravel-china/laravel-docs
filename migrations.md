# Database: Migrations

- [介绍](#introduction)
- [创建迁移](#generating-migrations)
- [迁移结构](#migration-structure)
- [运行迁移](#running-migrations)
    - [回滚迁移](#rolling-back-migrations)
- [编写迁移](#writing-migrations)
    - [创建表](#creating-tables)
    - [重命名/删除表](#renaming-and-dropping-tables)
    - [创建列](#creating-columns)
    - [修改列](#modifying-columns)
    - [移除列](#dropping-columns)
    - [创建索引](#creating-indexes)
    - [移除索引](#dropping-indexes)
    - [外键约束](#foreign-key-constraints)

<a name="introduction"></a>
## 介绍

迁移就像是数据库的版本控制,可以方便团队的修改和共享数据库结构.迁移通常会和[结构生成器](/docs/{{version}}/schema)一起使用，可以很简单的为你的应用创建数据库结构.

Laravel `Schema` [facade](/docs/{{version}}/facades)提供一个与数据库无关的数据表创建和操作方法,它可以很好的处理 Laravel 支持的各种数据库类型，并且在不同系统间提供一致性的 API 操作。

<a name="generating-migrations"></a>
## 创建迁移

使用 [Artisan CLI](/docs/{{version}}/artisan) 的 `make:migrate` 命令建立迁移文件：

    php artisan make:migration create_users_table

迁移文件会建立在 database/migrations 目录下，文件名会包含时间戳记，在执行迁移时用来确定顺序。

`--table` 和 `--create` 参数可以用来指定数据表名称，以及迁移文件是否要建立新的数据表。这些选项可以简单预填充到为指定的表生成的迁移存根文件:

    php artisan make:migration add_votes_to_users_table --table=users

    php artisan make:migration create_users_table --create=users

如果你想指定生成迁移文件的自定义导出路径,可以在执行`make:migration`命令使用`--path`参数,提供的路径应该是相对于应用程序的基本路径.

<a name="migration-structure"></a>
## 迁移结构

一个迁移类应包含两个方法:`up`和`down`,`up`方法用于为数据库添加新的表,列或索引,而`down`方法应简单的反转有`up`方法执行的操作.
在这两种方法内,你可以使用Laravel的结构生成器意味深长的创建和修改数据表.要了解结构生成器所有的可用方法,[点击这里](#creating-tables).
For example, let's look at a sample migration that creates a `flights` table:
下面我们举一个简单的例子,创建`flights`表迁移:

    <?php

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;

    class CreateFlightsTable extends Migration
    {
        /**
         * Run the migrations.
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
         * Reverse the migrations.
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

通过Artisan命令`migrate`来运行所有未执行的迁移,如果你用的是[Homestead](/docs/{{version}}/homestead)虚拟机开发环境,你应该在你的VM里运行下面的命令:

    php artisan migrate

果运行命令报错`class not found`,可以试试这条命令`composer dump-autoload`,然后再重新执行上面的迁移命令.

#### 在生产环境中强制迁移

有些迁移操作是破坏性的,可能导致你丢失数据,为了保护你的生产数据,当你对生产数据库执行这些命令之前,系统会提示您进行确认,如果不想系统提示确认,可以用参数`--force`:

    php artisan migrate --force

<a name="rolling-back-migrations"></a>
### 回滚迁移

用`rollback`命令可以回滚最近的一次迁移操作,注意，执行过的最后一“批”迁移可以包括多个迁移文件:

    php artisan migrate:rollback

`migrate:reset`命令可以回滚应用的所有迁移:

    php artisan migrate:reset

#### 回滚/迁移单个命令

`migrate:refresh`命令会先回滚所有数据库迁移,然后再执行`migrate`命令,该命令会重建你的整个数据库:

    php artisan migrate:refresh

    php artisan migrate:refresh --seed

<a name="writing-migrations"></a>
## 编写迁移

<a name="creating-tables"></a>
### 创建表

用结构生成器(`Schema` facade)的`create`方法可以创建新的数据表.`create`方法有两个参数,第一个参数是表名,第二个参数是`Closure`(闭包函数),它接收一个`Blueprint`(蓝图)对象,用于创建新表:

    Schema::create('users', function ($table) {
        $table->increments('id');
    });

当然，在创建表时，你可以使用结构生成器的任何的[列方法](http://laravel-china.org/docs/5.1/migrations#creating-columns)来定义表的列。

#### 检查表/列是否存在

通过`hasTable`和`hasColumn`方法,你可以很容易的检查表或列是否存在:

    if (Schema::hasTable('users')) {
        //
    }

    if (Schema::hasColumn('users', 'email')) {
        //
    }

#### 连接和存储引擎

如果你要进行模式操作的数据库连接不是你的默认连接,可以使用`connection`方法:

    Schema::connection('foo')->create('users', function ($table) {
        $table->increments('id');
    });

通过设置结构生成器的`engine`参数,为一张表设置数据存储引擎:

    Schema::create('users', function ($table) {
        $table->engine = 'InnoDB';

        $table->increments('id');
    });

<a name="renaming-and-dropping-tables"></a>
### 重命名/删除表

用`rename`方法可以重命名一个存在的数据表:

    Schema::rename($from, $to);

用`drop`或`dropIfExists`方法可以删除一个存在的数据表:

    Schema::drop('users');

    Schema::dropIfExists('users');

<a name="creating-columns"></a>
### 创建列

我们用结构生成器的`table`方法更新现有的表,和`create`方法类似,`table`方法接受两个参数:表名和一个`Closure`(闭包,它接收一个`Blueprint`(蓝图)实例,用于添加列到数据表):

    Schema::table('users', function ($table) {
        $table->string('email');
    });

#### 可用列类型

数据表产生器提供多种字段类型可使用，在您建立数据表时也许会用到:

命令  | 功能描述
------------- | -------------
`$table->bigIncrements('id');`  |  ID 自动增量，使用相当于「big integer」类型
`$table->bigInteger('votes');`  |  相当于 BIGINT 类型
`$table->binary('data');`  |  相当于 BLOB 类型
`$table->boolean('confirmed');`  |  相当于 BOOLEAN 类型
`$table->char('name', 4);`  |  相当于 CHAR 类型，并带有长度
`$table->date('created_at');`  |  相当于 DATE 类型
`$table->dateTime('created_at');`  |  相当于 DATETIME 类型
`$table->decimal('amount', 5, 2);`  |  相当于 DECIMAL 类型，并带有精度与基数
`$table->double('column', 15, 8);`  |  相当于 DOUBLE 类型，总共有 15 位数，在小数点后面有 8 位数
`$table->enum('choices', array('foo', 'bar'));` | 相当于 ENUM 类型
`$table->float('amount');`  |  相当于 FLOAT 类型
`$table->increments('id');`  |  相当于 Incrementing 类型 (数据表主键)
`$table->integer('votes');`  |  相当于 INTEGER 类型
`$table->json('options');`  |  相当于 JSON 类型
`$table->jsonb('options');`  |  JSONB equivalent to the table
`$table->longText('description');`  |  相当于 LONGTEXT 类型
`$table->mediumInteger('numbers');`  |  相当于 MEDIUMINT 类型
`$table->mediumText('description');`  |  相当于 MEDIUMTEXT 类型
`$table->morphs('taggable');`  |  加入整数 `taggable_id` 与字串 `taggable_type`
`$table->nullableTimestamps();`  |  与 `timestamps()` 相同，但允许 NULL
`$table->smallInteger('votes');`  |  相当于 SMALLINT 类型
`$table->tinyInteger('numbers');`  |  相当于 TINYINT 类型
`$table->softDeletes();`  |  加入 **deleted\_at** 字段于软删除使用
`$table->string('email');`  |  相当于 VARCHAR 类型
`$table->string('name', 100);`  |  相当于 VARCHAR 类型，并指定长度
`$table->text('description');`  |  相当于 TEXT 类型
`$table->time('sunrise');`  |  相当于 TIME 类型
`$table->timestamp('added_on');`  |  相当于 TIMESTAMP 类型
`$table->timestamps();`  |  加入 **created\_at** 和 **updated\_at** 字段
`$table->rememberToken();`  |  加入 `remember_token` 使用 VARCHAR(100) NULL

#### 列修饰符

除了以上列出的列类型,还有几个列修饰符,可以在加入列的同时使用,例如,让某列"可以为空",你可能会用到`nullable`方法:

    Schema::table('users', function ($table) {
        $table->string('email')->nullable();
    });

下面是所有可用列修饰符列表,这个列表并不被包含在[索引修饰符](#creating-indexes)里:

修饰符  | 描述
------------- | -------------
->first() |	将列调整为表的第一列 (MySQL Only)
->after('column') |	将列调整到'column'列的后面 (MySQL Only)
->nullable() |	该列允许列插入空值
->default($value) |	指定该列的默认值为$value
->unsigned() |	指定 `integer(整型)` 列为 `UNSIGNED(无符号)`列

<a name="changing-columns"></a>
<a name="modifying-columns"></a>
### 修改列

#### 先决条件(Prerequisites)

修改列之前,请务必确定你的`composer.json`文件已添加依赖项`doctrine/dbal`,Doctrine DBAL库用于确定列的当前状态，并创建指定调整到列的SQL查询。

#### 修改列属性

`change`能让你修改一个已经存在的列的类型或属性.例如:你像增加一个string类型列的字段长度,让我们看看`change`方法的行动力,假设我们想要将字段 `name` 的长度从 25 增加到 50:

    Schema::table('users', function ($table) {
        $table->string('name', 50)->change();
    });

我们还可以设定,让这个字段可以为空:

    Schema::table('users', function ($table) {
        $table->string('name', 50)->nullable()->change();
    });

<a name="renaming-columns"></a>
#### 修改列名称

要修改列名称,你可能会在结构生成器中用到`renameColumn`方法,请确认在修改前 `composer.json` 文件内已经加入 `doctrine/dbal`.:

    Schema::table('users', function ($table) {
        $table->renameColumn('from', 'to');
    });

> **注意:** `enum`字段类型不支持字段名称修改

<a name="dropping-columns"></a>
### 移除列

要移除列，可在结构生成器内使用 `dropColumn` 方法.:

    Schema::table('users', function ($table) {
        $table->dropColumn('votes');
    });

如果你给`dropcolumn`方法传递包含多个列名的数组,则可以移除多个列:

    Schema::table('users', function ($table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });

> **注意:** 从SQLite数据库删除列前,你需要将`composer.json` 文件内已经加入 `doctrine/dbal`,并在终端运行`composer update`来安装库.

<a name="creating-indexes"></a>
### 创建索引

结构生成器支持多种索引类型.首先，让我们来看一个例子，它指定列的值应该是唯一的。要创建索引,我们可以在定义列的时候,用`unique`方法链上去:

    $table->string('email')->unique();

或者，您可以先定义列,之后再创建索引,例如:

    $table->unique('email');

你甚至可以将一个数组传递给一个索引方法来创建一个复合索引:

    $table->index(['account_id', 'created_at']);

#### 可用的索引类型

命令  | 功能描述
------------- | -------------
$table->primary('id'); |	添加主键 (primary key)
$table->primary(['first', 'last']); |	添加组合键(composite keys)
$table->unique('email'); |	添加唯一索引 (unique index)
$table->index('state'); |	添加基本索引 (index)

<a name="dropping-indexes"></a>
### 移除索引

要移除索引就必须指定索引名称,默认情况下,Laravel默认有脉络可循的索引名称.简单地链接这些数据表与索引的字段名称和类型.举例如下:

命令  | 功能描述
------------- | -------------
`$table->dropPrimary('users_id_primary');`  |  从「users」数据表移除主键
`$table->dropUnique('users_email_unique');`  |  从「users」数据表移除唯一索引
`$table->dropIndex('geo_state_index');`  |  从「geo」数据表移除基本索引

### 移除时间戳记和软删除

要移除 `timestamps`、`nullableTimestamps` 或 `softDeletes` 字段类型，您可以使用以下方法：

命令  | 功能描述
------------- | -------------
`$table->dropTimestamps();`  |  移除 **created\_at** 和 **updated\_at** 字段
`$table->dropSoftDeletes();`  |  移除 **deleted\_at** 字段

### 保存引擎

要配置数据表的保存引擎，可在结构生成器配置 `engine` 属性：

  	Schema::create('users', function($table)
  	{
  		$table->engine = 'InnoDB';

  		$table->string('email');
  	});

<a name="foreign-key-constraints"></a>
### 外键约束

Laravel 也支持数据表的外键约束,这是用来在数据库级别强制引用完整性,例如:我们在`posts`表定义的`user_id`列是引用了`user`表的`id`列:

    Schema::table('posts', function ($table) {
        $table->integer('user_id')->unsigned();

        $table->foreign('user_id')->references('id')->on('users');
    });

您也可以指定在「on delete」和「on update」进行约束动作:

    $table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

要移除外键，可使用 `dropForeign` 方法。外键的命名约定雷同其他索引,因此,我们可以连接约束中的表名和列,后缀为`_foreign`:

    $table->dropForeign('posts_user_id_foreign');


