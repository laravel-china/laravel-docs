# Eloquent：关联

- [简介](#introduction)
- [定义关联](#defining-relationships)
    - [一对一](#one-to-one)
    - [一对多](#one-to-many)
    - [一对多（反向）](#one-to-many-inverse)
    - [多对多](#many-to-many)
    - [远层一对多](#has-many-through)
    - [多态关联](#polymorphic-relations)
    - [多对多多态关联](#many-to-many-polymorphic-relations)
- [查询关联](#querying-relations)
    - [关联方法 Vs. 动态属性](#relationship-methods-vs-dynamic-properties)
    - [基于存在的关联查询](#querying-relationship-existence)
    - [基于不存在的关联查询](#querying-relationship-absence)
    - [关联数据计数](#counting-related-models)
- [预加载](#eager-loading)
    - [为预加载添加约束条件](#constraining-eager-loads)
    - [延迟预加载](#lazy-eager-loading)
- [插入 & 更新关联模型](#inserting-and-updating-related-models)
    - [`save` 方法](#the-save-method)
    - [`create` 方法](#the-create-method)
    - [更新 `belongsTo` 关联](#updating-belongs-to-relationships)
    - [多对多关联](#updating-many-to-many-relationships)
- [更新父级时间戳](#touching-parent-timestamps)

<a name="introduction"></a>
## 简介

数据库表通常相互关联。 例如，一篇博客文章可能有许多评论，或者一个订单对应一个下单用户。Eloquent 让这些关联的管理和使用变得简单，并支持多种类型的关联：

- [一对一](#one-to-one)
- [一对多](#one-to-many)
- [多对多](#many-to-many)
- [远层一对多](#has-many-through)
- [多态关联](#polymorphic-relations)
- [多对多多态关联](#many-to-many-polymorphic-relations)

<a name="defining-relationships"></a>
## 定义关联

Eloquent 关联在 Eloquent 模型类中以方法的形式呈现。如同 Eloquent 模型本身，关联也可以作为强大的 [查询语句构造器](/docs/{{version}}/queries) 使用，提供了强大的链式调用和查询功能。例如，我们可以在 `posts` 关联的链式调用中附加一个约束条件：

    $user->posts()->where('active', 1)->get();

不过，在深入使用关联之前，让我们先学习如何定义每种关联类型。

<a name="one-to-one"></a>
### 一对一

一对一关联是最基本的关联关系。例如，一个 `User` 模型可能关联一个 `Phone` 模型。为了定义这个关联，我们要在 `User` 模型中写一个 `phone` 方法，在`phone` 方法内部调用 `hasOne` 方法并返回其结果：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 获得与用户关联的电话记录。
         */
        public function phone()
        {
            return $this->hasOne('App\Phone');
        }
    }

`hasOne` 方法的第一个参数是关联模型的类名。关联关系定义好后，我们就可以使用 Eloquent 动态属性获得相关的记录。您可以像在访问模型中定义的属性一样，使用动态属性：

    $phone = User::find(1)->phone;

Eloquent 会基于模型名决定外键名称。在当前场景中，Eloquent 假设 `Phone` 模型有一个 `user_id` 外键，如果外键名不是这个，可以通过给 `hasOne` 方法传递第二个参数覆盖默认使用的外键名： 

    return $this->hasOne('App\Phone', 'foreign_key');

此外，Eloquent 假定外键值是与父级 `id`（或自定义 `$primaryKey`）列的值相匹配的。 换句话说，Eloquent 将在 `Phone` 记录的 `user_id` 列中查找与用户表的 `id` 列相匹配的值。 如果您希望该关联使用 `id`以外的自定义键名，则可以给 `hasOne` 方法传递第三个参数：

    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

#### 定义反向关联

我们已经能从 `User` 模型访问到 `Phone` 模型了。现在，再在 `Phone` 模型中定义一个关联，此关联能让我们访问到拥有此电话的 `User` 模型。这时，使用的是与 `hasOne` 方法对应的 `belongsTo` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Phone extends Model
    {
        /**
         * 获得拥有此电话的用户。
         */
        public function user()
        {
            return $this->belongsTo('App\User');
        }
    }

在上面的例子中，Eloquent 会尝试匹配 `Phone` 模型上的 `user_id` 至 `User` 模型上的 `id`。 它是通过检查关系方法的名称并使用 `_id`  作为后缀名来确定默认外键名称的。 但是，如果`Phone`模型的外键不是`user_id`，那么可以将自定义键名作为第二个参数传递给`belongsTo`方法：

    /**
     * 获得拥有此电话的用户。
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key');
    }

如果父级模型没有使用 `id` 作为主键，或者是希望用不同的字段来连接子级模型，则可以通过给 `belongsTo` 方法传递第三个参数的形式指定父级数据表的自定义键：

    /**
     * 获得拥有此电话的用户。
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key', 'other_key');
    }

<a name="default-models"></a>
#### 默认模型

`belongsTo` 关联允许定义默认模型，这适应于当关联结果返回的是 `null` 的情况。这种设计模式通常称为 [空对象模式](https://en.wikipedia.org/wiki/Null_Object_pattern)，为您免去了额外的条件判断代码。在下面的例子中，`user` 关联如果没有找到文章的作者，就会返回一个空的 `App\User` 模型。

    /**
     * 获得此文章的作者。
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault();
    }

您也可以通过传递数组或者使用闭包的形式，填充默认模型的属性：

    /**
     * 获得此文章的作者。
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault([
            'name' => '游客',
        ]);
    }

    /**
     * 获得此文章的作者。
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault(function ($user) {
            $user->name = '游客';
        });
    }

<a name="one-to-many"></a>
### 一对多

「一对多」关联用于定义单个模型拥有任意数量的其它关联模型。例如，一篇博客文章可能会有无限多条评论。就像其它的 Eloquent 关联一样，一对多关联的定义也是在 Eloquent 模型中写一个方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * 获得此博客文章的评论。
         */
        public function comments()
        {
            return $this->hasMany('App\Comment');
        }
    }

记住，Eloquent 会自动确定 `Comment` 模型上正确的外键字段。按照约定，Eloquent 使用父级模型名的「snake case」形式、加上 `_id` 后缀名作为外键字段。对应到上面的场景，就是 Eloquent 假定 `Comment` 模型对应到 `Post` 模型上的那个外键字段是 `post_id`。

关联关系定义好后，我们就可以通过访问 `comments` 属性获得评论集合。记住，因为 Eloquent 提供了「动态属性」，所以我们可以像在访问模型中定义的属性一样，访问关联方法：

    $comments = App\Post::find(1)->comments;

    foreach ($comments as $comment) {
        //
    }

当然，由于所有的关联还可以作为查询语句构造器使用，因此你可以使用链式调用的方式、在 `comments` 方法上添加额外的约束条件：

    $comments = App\Post::find(1)->comments()->where('title', 'foo')->first();

形如 `hasOne` 方法，您也可以在使用 `hasMany` 方法的时候，通过传递额外参数来覆盖默认使用的外键与本地键。

    return $this->hasMany('App\Comment', 'foreign_key');

    return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

<a name="one-to-many-inverse"></a>
### 一对多（反向）

现在，我们已经能获得一篇文章的所有评论，接着再定义一个通过评论获得所属文章的关联。这个关联是 `hasMany` 关联的反向关联，在子级模型中使用 `belongsTo` 方法定义它：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * 获得此评论所属的文章。
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

关联关系定义好后，我们就可以在 `Comment` 模型上使用 `post` 「动态属性」获得 `Post` 模型了。

    $comment = App\Comment::find(1);

    echo $comment->post->title;

在上面的例子中，Eloquent 会尝试用 `Comment` 模型的 `post_id` 与 `Post` 模型的 `id` 进行匹配。默认外键名是 Eloquent 依据关联名、并在关联名后加上 `_id` 后缀确定的。当然，如果 `Comment` 模型的外键不是 `post_id`，那么可以将自定义键名作为第二个参数传递给`belongsTo`方法：

    /**
     * 获得此评论所属的文章。
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key');
    }

如果父级模型没有使用 `id` 作为主键，或者是希望用不同的字段来连接子级模型，则可以通过给 `belongsTo`方法传递第三个参数的形式指定父级数据表的自定义键：

    /**
     * 获得此评论所属的文章。
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
    }

<a name="many-to-many"></a>
### 多对多

多对多关联比 `hasOne` 和 `hasMany` 关联稍微复杂些。这种关联的一个例子就是具有许多角色的用户，而角色也被其他用户共享。例如，许多用户都可以有「管理员」角色。要定义这种关联，需要用到三个数据库表：`users`、`roles` 和 `role_user`。`role_user` 表是以相关联的两个模型数据表、依照字母顺序排列命名的，并且包含 `user_id` 和 `role_id` 字段。

多对多关联是通过写一个方法定义的，在方法内部调用 `belongsToMany` 方法并返回其结果。例如，我们在 `User` 模型中定义一个 `roles` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 获得此用户的角色。
         */
        public function roles()
        {
            return $this->belongsToMany('App\Role');
        }
    }

关联关系定义好后，我们就可以通过 `roles` 动态属性获得用户的角色了：

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        //
    }

当然，如同所有其它的关联类型，您可以调用 `roles` 方法，利用链式调用对查询语句添加约束条件：

    $roles = App\User::find(1)->roles()->orderBy('name')->get();

如前所述，为了确定连接表表名，Eloquent 会按照字母顺序合并两个关联模型的名称。 当然，您可以自由地覆盖这个约定，通过给 `belongsToMany` 方法指定第二个参数实现：

    return $this->belongsToMany('App\Role', 'role_user');

除了自定义连接表表名，您也可以通过给 `belongsToMany` 方法再次传递额外参数来自定义连接表里的键的字段名称。第三个参数是定义此关联的模型在连接表里的键名，第四个参数是另一个模型在连接表里的键名：

    return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');

#### 定义反向关联

定义多对多关联的反向关联，您只要在对方模型里再次调用 `belongsToMany` 方法就可以了。让我们接着以用户角色为例，在 `Role` 模型中定义一个 `users` 方法。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * 获得此角色下的用户。
         */
        public function users()
        {
            return $this->belongsToMany('App\User');
        }
    }

如你所见，除了引入的模型变为 `App\User` 外，其它与在 `User` 模型中定义的完全一样。由于我们重用了 `belongsToMany` 方法，自定义连接表表名和自定义连接表里的键的字段名称在这里同样适用。

#### 获得中间表字段

您已经学到，多对多关联需要有一个中间表支持，Eloquent 提供了一些有用的方法来和这张表进行交互。例如，假设我们的 `User` 对象关联了许多的 `Role` 对象。在获得这些关联对象后，可以使用模型的 `pivot` 属性访问中间表数据：

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

需要注意的是，我们取得的每个 `Role` 模型对象，都会被自动赋予 `pivot` 属性，它代表中间表的一个模型对象，能像其它的 Eloquent 模型一样使用。

默认情况下，`pivot` 对象只包含两个关联模型的键。如果中间表里还有额外字段，则必须在定义关联时明确指出：

    return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');

如果您想让中间表自动维护 `created_at` 和 `updated_at` 时间戳，那么在定义关联时加上 `withTimestamps` 方法即可。

    return $this->belongsToMany('App\Role')->withTimestamps();

#### 通过中间表过滤关联数据

在定义关联时，您可以使用 `wherePivot` 和 `wherePivotIn` 方法过滤 `belongsToMany` 返回的结果：

    return $this->belongsToMany('App\Role')->wherePivot('approved', 1);

    return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);

#### 定义自定义中间表模型

如果您想定义一个自定义模型来表示关联关系中的中间表，可以在定义关联时调用 `using` 方法。所有自定义中间表模型都必须扩展自 `Illuminate\Database\Eloquent\Relations\Pivot` 类。例如，
我们在写 `Role` 模型的关联时，使用自定义中间表模型 `UserRole`：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * 获得此角色下的用户。
         */
        public function users()
        {
            return $this->belongsToMany('App\User')->using('App\UserRole');
        }
    }

当定义 `UserRole` 模型时，我们要扩展自 `Pivot` 类：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Relations\Pivot;

    class UserRole extends Pivot
    {
        //
    }

<a name="has-many-through"></a>
### 远层一对多

「远层一对多」关联提供了方便、简短的方式通过中间的关联来获得远层的关联。例如，一个 `Country` 模型可以通过中间的 `User` 模型获得多个 `Post` 模型。在这个例子中，您可以轻易地收集给定国家的所有博客文章。让我们来看看定义这种关联所需的数据表：

    countries
        id - integer
        name - string

    users
        id - integer
        country_id - integer
        name - string

    posts
        id - integer
        user_id - integer
        title - string

虽然 `posts` 表中不包含 `country_id` 字段，但 `hasManyThrough` 关联能让我们通过 `$country->posts` 访问到一个国家下所有的用户文章。为了完成这个查询，Eloquent 会先检查中间表 `users` 的 `country_id` 字段，找到所有匹配的用户 ID 后，使用这些 ID，在 `posts` 表中完成查找。 

现在，我们已经知道了定义这种关联所需的数据表结构，接下来，让我们在 `Country` 模型中定义它：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Country extends Model
    {
        /**
         * 获得某个国家下所有的用户文章。
         */
        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'App\User');
        }
    }

`hasManyThrough` 方法的第一个参数是我们最终希望访问的模型名称，而第二个参数是中间模型的名称。

当执行关联查询时，通常会使用 Eloquent 约定的外键名。如果您想要自定义关联的键，可以通过给 `hasManyThrough` 方法传递第三个和第四个参数实现，第三个参数表示中间模型的外键名，第四个参数表示最终模型的外键名。第五个参数表示本地键名，而第六个参数表示中间模型的本地键名：

    class Country extends Model
    {
        public function posts()
        {
            return $this->hasManyThrough(
                'App\Post',
                'App\User',
                'country_id', // 用户表外键...
                'user_id', // 文章表外键...
                'id', // 国家表本地键...
                'id' // 用户表本地键...
            );
        }
    }

<a name="polymorphic-relations"></a>
### 多态关联

#### 数据表结构

多态关联允许一个模型在单个关联上属于多个其他模型。例如，想象一下使用您应用的用户可以「评论」文章和视频。使用多态关联，您可以用一个 `comments` 表同时满足这两个使用场景。让我们来看看构建这种关联所需的数据表结构：

    posts
        id - integer
        title - string
        body - text

    videos
        id - integer
        title - string
        url - string

    comments
        id - integer
        body - text
        commentable_id - integer
        commentable_type - string

`comments` 表中有两个需要注意的重要字段 `commentable_id` 和 `commentable_type`。`commentable_id` 用来保存文章或者视频的 ID 值，而 `commentable_type` 用来保存所属模型的类名。`commentable_type` 是在我们访问 `commentable` 关联时， 让 ORM 确定所属的模型是哪个「类型」。

#### 模型结构

接下来，我们来看看创建这种关联所需的模型定义：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * 获得拥有此评论的模型。
         */
        public function commentable()
        {
            return $this->morphTo();
        }
    }

    class Post extends Model
    {
        /**
         * 获得此文章的所有评论。
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

    class Video extends Model
    {
        /**
         * 获得此视频的所有评论。
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

#### 获取多态关联

一旦您的数据库表准备好、模型定义完成后，就可以通过模型来访问关联了。例如，我们只要简单地使用 `comments` 动态属性，就可以获得某篇文章下的所有评论：

    $post = App\Post::find(1);

    foreach ($post->comments as $comment) {
        //
    }

您也可以在多态模型上，通过访问调用了 `morphTo` 的关联方法获得多态关联的拥有者。在当前场景中，就是 `Comment` 模型的 `commentable` 方法。所以，我们可以使用动态属性来访问这个方法：

    $comment = App\Comment::find(1);

    $commentable = $comment->commentable;

`Comment` 模型的 `commentable` 关联会返回 `Post` 或者 `Video` 实例，这取决于评论所属的模型类型。

#### 自定义多态关联的类型字段

默认，Laravel 会使用完全限定类名作为关联模型保存在多态模型上的类型字段值。比如，在上面的例子中，`Comment` 属于 `Post` 或者 `Video`，那么 `commentable_type`的默认值对应地就是 `App\Post` 和 `App\Video`。但是，您可能希望将数据库与程序内部结构解耦。那样的话，你可以定义一个「多态映射表」来指示 Eloquent 使用每个模型自定义类型字段名而不是类名：
 
    use Illuminate\Database\Eloquent\Relations\Relation;

    Relation::morphMap([
        'posts' => 'App\Post',
        'videos' => 'App\Video',
    ]);

您可以在 `AppServiceProvider` 中的 `boot` 函数中使用 `Relation::morphMap` 方法注册「多态映射表」，或者使用一个独立的服务提供者注册。

<a name="many-to-many-polymorphic-relations"></a>
### 多对多多态关联

#### 数据表结构

除了传统的多态关联，您也可以定义「多对多」的多态关联。例如，`Post` 模型和 `Video` 模型可以共享一个多态关联至 `Tag` 模型。 使用多对多多态关联可以让您在文章和视频中共享唯一的标签列表。首先，我们来看看数据表结构：

    posts
        id - integer
        name - string

    videos
        id - integer
        name - string

    tags
        id - integer
        name - string

    taggables
        tag_id - integer
        taggable_id - integer
        taggable_type - string

#### 模型结构

接下来，我们准备在模型上定义关联关系。`Post` 和 `Video` 两个模型都有一个 `tags` 方法，方法内部都调用了 Eloquent 类自身的 `morphToMany` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * 获得此文章的所有标签。
         */
        public function tags()
        {
            return $this->morphToMany('App\Tag', 'taggable');
        }
    }

#### 定义反向关联

接下里，在 `Tag` 模型中，您应该为每个关联模型定义一个方法。在这个例子里，我们要定义一个 `posts` 方法和一个 `videos` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Tag extends Model
    {
        /**
         * 获得此标签下所有的文章。
         */
        public function posts()
        {
            return $this->morphedByMany('App\Post', 'taggable');
        }

        /**
         *  获得此标签下所有的视频。
         */
        public function videos()
        {
            return $this->morphedByMany('App\Video', 'taggable');
        }
    }

#### 获取关联

一旦您的数据库表准备好、模型定义完成后，就可以通过模型来访问关联了。例如，我们只要简单地使用 `tags` 动态属性，就可以获得某篇文章下的所有标签：

    $post = App\Post::find(1);

    foreach ($post->tags as $tag) {
        //
    }

您也可以在多态模型上，通过访问调用了 `morphedByMany` 的关联方法获得多态关联的拥有者。在当前场景中，就是 `Tag` 模型上的 `posts` 方法和 `videos` 方法。所以，我们可以使用动态属性来访问这两个方法：

    $tag = App\Tag::find(1);

    foreach ($tag->videos as $video) {
        //
    }

<a name="querying-relations"></a>
## 查询关联

由于所有类型的关联都通过方法定义，您可以调用这些方法来获取关联实例，而不需要实际运行关联的查询。此外，所有类型的关联都可以作为 [查询语句构造器](/docs/{{version}}/queries) 使用，让你在向数据库执行 SQL 语句前，使用链式调用的方式添加约束条件。

例如，假设一个博客系统，其中 `User` 模型有许多关联的 `Post` 模型：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 获得此用户所有的文章。
         */
        public function posts()
        {
            return $this->hasMany('App\Post');
        }
    }

您也可以像这样在 `posts` 关联上添加额外约束条件：

    $user = App\User::find(1);

    $user->posts()->where('active', 1)->get();

您可以在关联上使用任何 [查询语句构造器](/docs/{{version}}/queries) 的方法，所以，欢迎查阅查询语句构造器的相关文档以便了解您可以使用哪些方法。

<a name="relationship-methods-vs-dynamic-properties"></a>
### 关联方法 Vs. 动态属性

如果您不需要给 Eloquent 关联查询添加额外约束条件，你可以简单的像访问属性一样访问关联。例如，我们刚刚的 `User` 和 `Post` 模型例子中，我们可以这样访问所有用户的文章：

    $user = App\User::find(1);

    foreach ($user->posts as $post) {
        //
    }

动态属性是「懒加载」的，意味着它们的关联数据只在实际被访问时才被加载。因此，开发者经常使用 [预加载](#eager-loading) 提前加载他们之后会用到的关联数据。预加载有效减少了 SQL 语句请求数，避免了重复执行一个模型关联加载数据、发送 SQL 请求带来的性能问题。

<a name="querying-relationship-existence"></a>
### 基于存在的关联查询

当获取模型记录时，您可能希望根据存在的关联对结果进行限制。例如，您想获得至少有一条评论的所有博客文章。为了实现这个功能，您可以给 `has` 方法传递关联名称：

    // 获得所有至少有一条评论的文章...
    $posts = App\Post::has('comments')->get();

您也可以指定一个运算符和数目，进一步自定义查询：

    // 获得所有有三条或三条以上评论的文章...
    $posts = Post::has('comments', '>=', 3)->get();

也可以使用「点」符号构造嵌套的的 `has` 语句。例如，您可以获得所有至少有一条获赞评论的文章：

    // 获得所有至少有一条获赞评论的文章...
    $posts = Post::has('comments.votes')->get();

如果您需要更高级的用法，可以使用 `whereHas`和 `orWhereHas` 方法在 `has` 查询里设置「where」条件。此方法可以让你增加自定义条件至关联约束中，例如对评论内容进行检查：

    // 获得所有至少有一条评论内容满足 foo% 条件的文章
    $posts = Post::whereHas('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

<a name="querying-relationship-absence"></a>
### 基于不存在的关联查询

当获取模型记录时，您可能希望根据不存在的关联对结果进行限制。例如，您想获得 **没有** 任何评论的所有博客文章。为了实现这个功能，您可以给 `doesntHave` 方法传递关联名称：

    $posts = App\Post::doesntHave('comments')->get();

如果您需要更高级的用法，可以使用 `whereDoesntHave` 方法在 `doesntHave` 查询里设置「where」条件。此方法可以让你增加自定义条件至关联约束中，例如对评论内容进行检查：

    $posts = Post::whereDoesntHave('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

<a name="counting-related-models"></a>
### 关联数据计数

如果您只想统计结果数而不需要加载实际数据，那么可以使用 `withCount` 方法，此方法会在您的结果集模型中添加一个 `{关联名}_count` 字段。例如：

    $posts = App\Post::withCount('comments')->get();

    foreach ($posts as $post) {
        echo $post->comments_count;
    }

您可以为多个关联数据「计数」，并为其查询添加约束条件：

    $posts = Post::withCount(['votes', 'comments' => function ($query) {
        $query->where('content', 'like', 'foo%');
    }])->get();

    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;

您也可以为关联数据计数结果起别名，允许在同一个关联上多次计数：

    $posts = Post::withCount([
        'comments',
        'comments as pending_comments_count' => function ($query) {
            $query->where('approved', false);
        }
    ])->get();

    echo $posts[0]->comments_count;

    echo $posts[0]->pending_comments_count;

<a name="eager-loading"></a>
## 预加载

当作为属性访问 Eloquent 关联时，关联数据是「懒加载」的。意味着在你第一次访问该属性时，才会加载关联数据。不过，是当你查询父模型时，Eloquent 可以「预加载」关联数据。预加载避免了 N + 1 查询问题。要说明 N + 1 查询问题，试想一个 `Book` 模型关联到 `Author` 模型：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Book extends Model
    {
        /**
         * 获得此书的作者。
         */
        public function author()
        {
            return $this->belongsTo('App\Author');
        }
    }

现在，让我们来获得所有书籍和作者数据：

    $books = App\Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

这个循环会运行一次查询取回所有数据表上的书籍数据，然后又运行一次查询获得每本书的作者数据。如果我们有 25 本书，则循环就会执行 26 次查询：1 次是获得所有书籍数据，另外 25 条查询用来获得每本书的作者数据。

谢天谢地，我们使用预加载让整个查询减少到 2 次。这是通过指定关联给 `with` 方法办到的：

    $books = App\Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

整个操作，只执行了两条查询：

    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)

#### 预加载多个关联

有时，你需要在一次操作中预加载几个不同的关联。为了实现这个功能，只需在 `with` 方法上传递额外的参数即可：        

    $books = App\Book::with(['author', 'publisher'])->get();

#### 嵌套预加载

预加载嵌套关联，可以使用「点」语法。例如，在一个 Eloquent 语句中，预加载所有书籍作者和这些作者的联系信息：

    $books = App\Book::with('author.contacts')->get();

<a name="constraining-eager-loads"></a>
### 为预加载添加约束条件

有时，你可能希望在预加载关联数据的时候，为查询指定额外的约束条件。这有个例子：

    $users = App\User::with(['posts' => function ($query) {
        $query->where('title', 'like', '%first%');
    }])->get();

在这个例子中，Eloquent 只会预加载标题里包含 `first` 文本的文章。您也可以调用其它的 [查询语句构造器](/docs/{{version}}/queries) 进一步自定义预加载约束条件：

    $users = App\User::with(['posts' => function ($query) {
        $query->orderBy('created_at', 'desc');
    }])->get();

<a name="lazy-eager-loading"></a>
### 延迟预加载

有时，您可能需要在获得父级模型后才去预加载关联数据。例如，当你需要来动态决定是否加载关联模型时，这可能很有帮助：

    $books = App\Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

如果您想设置预加载查询的额外约束条件，可以通过给 `load` 添加数组键的形式达到目的，数组值是接收查询实例的闭包：

    $books->load(['author' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }]);

<a name="inserting-and-updating-related-models"></a>
## 插入 & 更新关联模型

<a name="the-save-method"></a>
### `save` 方法

Eloquent 提供了便捷的方法来将新的模型增加至关联中。例如，也许你需要为一个 `Post` 模型插入一个新的 `Comment`。这是你无须为 `Comment` 手动设置 `posts` 属性，直接在关联上使用 `save` 方法插入 `Comment` 即可：

    $comment = new App\Comment(['message' => '一条新的评论。']);

    $post = App\Post::find(1);

    $post->comments()->save($comment);

需要注意的是，我们没有使用动态属性形式访问 `comments` 关联。相反，我们调用了 `comments` 方法获得关联实例。`save` 方法会自动在新的 `Comment` 模型中添加正确的 `post_id`值。

如果您需要保存多个关联模型，可以使用 `saveMany` 方法：

    $post = App\Post::find(1);

    $post->comments()->saveMany([
        new App\Comment(['message' => '一条新的评论。']),
        new App\Comment(['message' => '另一条评论。']),
    ]);

<a name="the-create-method"></a>
### `create` 方法

除了 `save` 和 `saveMany` 方法，您也可以使用 `create` 方法，它接收一个属性数组、创建模型并插入数据库。还有，`save` 和 `create` 的不同之处在于，`save` 接收的是一个完整的 Eloquent 模型实例，而 `create` 接收的是一个纯 PHP 数组：

    $post = App\Post::find(1);

    $comment = $post->comments()->create([
        'message' => '一条新的评论。',
    ]);

> {tip} 在使用 `create` 方法前，请确认您已经浏览了本文档的 [批量赋值](/docs/{{version}}/eloquent#mass-assignment) 章节。

您可以使用 `createMany` 方法保存多个关联模型：

    $post = App\Post::find(1);

    $post->comments()->createMany([
        [
            'message' => '一条新的评论。',
        ],
        [
            'message' => '另一条新的评论。',
        ],
    ]);

<a name="updating-belongs-to-relationships"></a>
### 更新 `belongsTo` 关联

当更新 `belongsTo` 关联时，可以使用 `associate` 方法。此方法会在子模型中设置外键：

    $account = App\Account::find(10);

    $user->account()->associate($account);

    $user->save();

当删除 `belongsTo` 关联时，可以使用 `dissociate`方法。此方法会设置关联外键为 `null`： 

    $user->account()->dissociate();

    $user->save();

<a name="updating-many-to-many-relationships"></a>
### 多对多关联

#### 附加 / 移除

Eloquent 也提供了几个额外的辅助方法，让操作关联模型更加便捷。例如：我们假设一个用户可以拥有多个角色，并且每个角色都可以被多个用户共享。给某个用户附加一个角色是通过向中间表插入一条记录实现的，使用 `attach` 方法：

    $user = App\User::find(1);

    $user->roles()->attach($roleId);

使用 `attach` 方法时，您也可以通过传递一个数组参数向中间表写入额外数据：

    $user->roles()->attach($roleId, ['expires' => $expires]);

当然，有时也需要移除用户的角色。删除多对多关联记录，使用 `detach` 方法。`detach` 方法会移除掉正确的记录；当然，这两个模型数据依然保存在数据库中：

    // 移除用户的一个角色...
    $user->roles()->detach($roleId);

    // 移除用户的所有角色...
    $user->roles()->detach();

为了方便，`attach` 和 `detach` 都允许传入 ID 数组：

    $user = App\User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([
        1 => ['expires' => $expires],
        2 => ['expires' => $expires]
    ]);

#### 同步关联

您也可以使用 `sync` 方法来构造多对多关联。`sync` 方法可以接收 ID 数组，向中间表插入对应关联数据记录。所有没放在数组里的 IDs 都会从中间表里移除。所以，这步操作完成后，只有在数组里的 IDs 会被保留在中间表中。

    $user->roles()->sync([1, 2, 3]);

您可以通过 ID 传递其他额外的数据到中间表：

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);
 
如果您不想移除现有的 IDs，可以使用 `syncWithoutDetaching` 方法：

    $user->roles()->syncWithoutDetaching([1, 2, 3]);

#### 切换关联

多对多关联也提供了一个 `toggle` 方法用于「切换」给定 IDs 的附加状态。如果给定 ID 已附加，就会被移除。同样的，如果给定 ID 已移除，就会被附加：

    $user->roles()->toggle([1, 2, 3]);

#### 在中间表上保存额外数据

当处理多对多关联时，`save` 方法还可以使用第二个参数，它是一个属性数组，包含插入到中间表的额外字段数据。

    App\User::find(1)->roles()->save($role, ['expires' => $expires]);

#### 更新中间表记录

如果您需要更新中间表中已存在的记录，可以使用 `updateExistingPivot` 方法。此方法接收中间记录的外键和一个属性数组进行更新：

    $user = App\User::find(1);

    $user->roles()->updateExistingPivot($roleId, $attributes);

<a name="touching-parent-timestamps"></a>
## 更新父级时间戳

当一个模型 `belongsTo` 或者 `belongsToMany` 另一个模型，比如一个 `Comment` 属于一个 `Post`，有时更新子模型导致更新父模型时间戳非常有用。例如，当一个 `Comment` 模型更新时，您要自动「触发」父级 `Post` 模型的 `updated_at` 时间戳的更新，Eloquent 让它变得简单。只要在子模型加一个包含关联名称的 `touches` 属性即可：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * 所有会被触发的关联。
         *
         * @var array
         */
        protected $touches = ['post'];

        /**
         * 获得此评论所属的文章。
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }
 
现在，当你更新一个 `Comment` 时，对应父级 `Post` 模型的 `updated_at` 字段也会被同时更新，使其更方便得知何时让一个 `Post` 模型的缓存失效：

    $comment = App\Comment::find(1);

    $comment->text = '编辑了这条评论！';

    $comment->save();

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@baooab](https://laravel-china.org/users/17319)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/images/201708/11/17319/KbHzLBdgHs.png?imageView2/1/w/100/h/100">  |  翻译  | 我在 [这儿](https://github.com/baooab/) |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org