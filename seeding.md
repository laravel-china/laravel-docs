# Laravel 数据库之：数据填充

- [简介](#introduction)
- [编写 Seeders](#writing-seeders)
    - [使用模型工厂](#using-model-factories)
    - [调用其他 Seeders](#calling-additional-seeders)
- [ 运行 Seeders](#running-seeders)

<a name="introduction"></a>
## 简介

Laravel 可以用 seed 类轻松地为数据库填充测试数据。所有的 seed 类都存放在 `database/seeds` 目录下。你可以任意为 seed 类命名，但是应该遵守类似 `UsersTableSeeder` 的命名规范。Laravel 默认定义了一个 `DatabaseSeeder` 类。可以在这个类中使用 `call` 方法来运行其它的 seed 类来控制数据填充的顺序。

<a name="writing-seeders"></a>
## 编写 Seeders

可以通过运行 `make:seeder` [Artisan 命令](/docs/{{version}}/artisan) 来生成一个 Seeder。所有由框架生成的 seeders 都将被放置在 `database/seeds` 目录下：

    php artisan make:seeder UsersTableSeeder

一个 seeder 类只包含一个默认方法：`run`。这个方法在 `db:seed` [Artisan 命令](/docs/{{version}}/artisan) 被调用时执行。在 `run` 方法里你可以为数据库添加任何数据。你也可以用 [查询语句构造器](/docs/{{version}}/queries) 或 [Eloquent 模型工厂](/docs/{{version}}/database-testing#writing-factories) 来手动添加数据。

如下所示，我们来修改默认的 `DatabaseSeeder` 类并为 `run` 方法添加一条数据库插入语句：

    <?php

    use Illuminate\Database\Seeder;
    use Illuminate\Database\Eloquent\Model;

    class DatabaseSeeder extends Seeder
    {
        /**
         * 运行数据库填充
         *
         * @return void
         */
        public function run()
        {
            DB::table('users')->insert([
                'name' => str_random(10),
                'email' => str_random(10).'@gmail.com',
                'password' => bcrypt('secret'),
            ]);
        }
    }

<a name="using-model-factories"></a>
### 使用模型工厂

当然，手动为每个模型填充指定属性很麻烦。作为替代方案，你可以使用 [模型工厂](/docs/{{version}}/database-testing#writing-factories) 来轻松地生成大量数据库记录。首先，阅读 [模型工厂文档](/docs/{{version}}/database-testing#writing-factories) 来学习如何定义工厂。一旦定义了工厂，就可以使用 `factory` 这个辅助函数来向数据库中添加记录。

如下所示，我们来创建 50 个用户并为每个用户创建关联：

    /**
     * 运行数据库填充
     *
     * @return void
     */
    public function run()
    {
        factory(App\User::class, 50)->create()->each(function ($u) {
            $u->posts()->save(factory(App\Post::class)->make());
        });
    }

<a name="calling-additional-seeders"></a>
### 调用其他 Seeders

在 `DatabaseSeeder` 类中，你可以使用 `call` 方法来运行其他的 seed 类。为避免单个 seeder 类过大，可使用 `call` 方法将数据填充拆分成多个文件。只需简单传递要运行的 seeder 类名称即可：

    /**
     * 运行数据库填充
     *
     * @return void
     */
    public function run()
    {
        $this->call(UsersTableSeeder::class);
        $this->call(PostsTableSeeder::class);
        $this->call(CommentsTableSeeder::class);
    }

<a name="running-seeders"></a>
## 运行 Seeders

一旦完成了 seeder 类的编写，就可以使用 `db:seed` 这个 Artisan 命令来填充数据库。在默认情况下，`db:seed` 命令将运行可以用来调用其他填充类的 `DatabaseSeeder` 类。但是可以用 `--class` 选项来单独运行一个特定的 seeder 类：

    php artisan db:seed

    php artisan db:seed --class=UsersTableSeeder

也可以使用会先回滚再重新运行所有迁移的 `migrate:refresh` 命令来填充数据库。这个命令在彻底重构数据库时非常有用：

    php artisan migrate:refresh --seed

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@wqer1019](https://laravel-china.org/users/5435)  | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/9254545?v=4&s=100">  |  翻译  | laravel是世界上最优雅的框架，[@wqer1019](https://github.com/wqer1019) at Github  |

--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org