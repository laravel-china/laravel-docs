# 数据库: 数据填充

- [简介](#introduction)
- [编写 Seeders](#writing-seeders)
    - [使用模型工厂](#using-model-factories)
    - [调用其它的 Seeders](#calling-additional-seeders)
- [运行 Seeders](#running-seeders)

<a name="introduction"></a>
## 简介

Laravel 可以简单的使用 seed 类来给数据库填充测试数据。所有的 seed 类都放在 `database/seeds` 目录下。你可以任意地为 Seed 类命名，但是应该遵守某些大小写规范，可用类似 `UserTableSeeder` 之类的命名。 Laravle 默认为你定义了一个 `DatabaseSeeder` 类。你可以在这个类中使用 `call` 方法来运行其它的 seed 类，以借此控制数据填充的顺序。

<a name="writing-seeders"></a>
## 编写数据填充

你可以通过 `make:seeder` [Artisan 命令](/docs/{{version}}/artisan) 来生成一个 Seeder。所有通过框架生成的 Seeder 都将被放置在 `database/seeders` 路径：

    php artisan make:seeder UsersTableSeeder

在 seeder 类里只有一个默认方法：`run`。当运行 `db:seed` [Artisan 命令](/docs/{{version}}/artisan) 时就会调用此方法。你可以在 `run` 方法中给数据库添加任何数据。你可使用 [查询语句构造器](/docs/{{version}}/queries) 或 [Eloquent 模型工厂](/docs/{{version}}/testing#model-factories) 来手动添加数据。

如下所示，我们将修改 Laravel 预先生成好的 `DatabaseSeeder` 类来给 `run` 方法添加一段可在数据库添加数据的语法：

    <?php

    use Illuminate\Database\Seeder;
    use Illuminate\Database\Eloquent\Model;

    class DatabaseSeeder extends Seeder
    {
        /**
         * 运行数据库填充。
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

手动为每一个 seed 模型一一指定属性是很麻烦的一件事。作为替代方案，你可以使用 [模型工厂](/docs/{{version}}/testing#model-factories) 来帮助你更便捷的生成大量数据库数据。首先，阅读 [模型工厂的文档](/docs/{{version}}/testing#model-factories) 来学习如何定义你的工厂。一旦工厂被定义，就能使用 `factory` 这个辅助方法函数来添加数据到数据库。

让我们来创建 50 个用户并为每个用户创建一个关联：

    /**
     * 运行数据库填充。
     *
     * @return void
     */
    public function run()
    {
        factory(App\User::class, 50)->create()->each(function($u) {
            $u->posts()->save(factory(App\Post::class)->make());
        });
    }

<a name="calling-additional-seeders"></a>
### 调用其它的 Seeders

在 `DatabaseSeeder` 类中，你可以使用 `call` 方法来运行其它的 seed 类。为避免发生单个 seeder 类变得太大的情况，可使用 `call`方法来将数据填充拆分成多个文件。只需简单传递你想要运行的 seeder 类名称即可：

    /**
     * 运行数据库填充。
     *
     * @return void
     */
    public function run()
    {
        Model::unguard();

        $this->call(UsersTableSeeder::class);
        $this->call(PostsTableSeeder::class);
        $this->call(CommentsTableSeeder::class);

        Model::reguard();
    }

<a name="running-seeders"></a>
## 运行数据填充

一旦你编写完 seeder 类，则可以使用 `db:seed` Artisan 命令来对数据库进行数据填充。在默认的情况下，`db:seed` 命令将运行 `DatabaseSeeder` 类，并通过它来调用其它的 seed 类。但是，你也可以使用 `--class` 选项来单独运行一个特定的 seeder 类：

    php artisan db:seed

    php artisan db:seed --class=UserTableSeeder

你也可以使用 `migrate:refresh` 命令来对数据库进行数据填充，它会回滚并重新运行所有迁移。这在对数据库进行重构时非常有用：

    php artisan migrate:refresh --seed
