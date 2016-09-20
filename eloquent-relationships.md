# Eloquent: 关联

- [简介](#introduction)
- [定义关联](#defining-relationships)
    - [一对一](#one-to-one)
    - [一对多](#one-to-many)
    - [一对多 (反向关联)](#one-to-many-inverse)
    - [多对多](#many-to-many)
    - [远层一对多](#has-many-through)
    - [多态关联](#polymorphic-relations)
    - [多态多对多关联](#many-to-many-polymorphic-relations)
- [查找关联](#querying-relations)
    - [关联方法和动态属性](#relationship-methods-vs-dynamic-properties)
    - [查找关联是否存在](#querying-relationship-existence)
    - [关联数据计数](#counting-related-models)
- [预加载](#eager-loading)
    - [预加载条件限制](#constraining-eager-loads)
    - [延迟预加载](#lazy-eager-loading)
- [插入 & 更新关联模型](#inserting-and-updating-related-models)
    - [`save` 方法](#the-save-method)
    - [`create` 方法](#the-create-method)
    - [更新「从属」关联](#updating-belongs-to-relationships)
    - [多对多关联](#updating-many-to-many-relationships)
- [连动上层时间戳](#touching-parent-timestamps)

<a name="introduction"></a>
## 简介

数据表之间经常会互相进行关联。例如，一篇博客文章可能会有多条评论，或是一张订单可能对应一个下单客户。Eloquent 让管理和处理这些关联变得很容易，同时也支持多种类型的关联：

- [一对一](#one-to-one)
- [一对多](#one-to-many)
- [多对多](#many-to-many)
- [远层一对多](#has-many-through)
- [多态关联](#polymorphic-relations)
- [多态多对多关联](#many-to-many-polymorphic-relations)

<a name="defining-relationships"></a>
## 定义关联

你可在 Eloquent 模型类内中，把 Eloquent 关联定义成函数(functions)。因为，关联就像 Eloquent 模型一样，也可以作为强大的 [查询语句构造器](/docs/{{version}}/queries)，定义关联为函数，因为其提供了强而有力的链式调用及查找功能。例如，我们可以在 `posts` 关联的链式调用中附加一个约束条件：

    $user->posts()->where('active', 1)->get();

不过，在深入了解使用关联之前，先让我们来学习如何定义每个类型：

<a name="one-to-one"></a>
### 一对一

「一对一」关联是一个非常基本的关联关系。举个例子，一个 `User` 模型会关联一个 `Phone` 模型。为了定义这种关联关系，我们需要在 `User` 模型中写一个 `phone` 方法。且 `phone` 方法应该调用 `hasOne` 方法并返回其结果：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 获取与用户关联的电话号码
         */
        public function phone()
        {
            return $this->hasOne('App\Phone');
        }
    }

第一个传到 `hasOne` 方法里的参数是关联模型的类名。一旦定义好两者之间关联，我们就可以通过使用 Eloquent 的动态属性来获取关联纪录。动态属性允许你访问关联函数，如同他们是定义在模型中的属性：

    $phone = User::find(1)->phone;

Eloquent 会假设对应关联的外键名称是基于模型名称的。在这个例子里，它会自动假设 `Phone` 模型拥有 `user_id` 外键。如果你想要重写这个约定，则可以传入第二个参数到 `hasOne` 方法里。

    return $this->hasOne('App\Phone', 'foreign_key');

此外，Eloquent 假设外键会和上层模型的 `id` 字段(或者自定义的 `$primaryKey`)的值相匹配。换句话说，Eloquent 会寻找用户的 `id` 字段与 `Phone` 模型的 `user_id` 字段的值相同的纪录。如果你想让关联使用 `id` 以外的值，则可以传递第三个参数至 `hasOne` 方法来指定你自定义的键：

    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

#### 定义反向关联

所以，我们可以从 `User` 模型访问到 `Phone` 模型。现在，让我们在 `Phone` 模型上定义一个关联，此关联能够让我们访问拥有此电话的 `User` 模型。我们可以定义与 `hasOne` 关联相对应的 `belongsTo` 方法

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Phone extends Model
    {
        /**
         * 获取拥有该电话的用户模型。
         */
        public function user()
        {
            return $this->belongsTo('App\User');
        }
    }

在上述例子中，Eloquent 会尝试匹配 `Phone` 模型的 `user_id` 至 `User` 模型的 `id`。Eloquent 判断的默认外键名称参考自关联模型的方法名称，并会在方法名称后面加上 `_id`。当然，如果 `Phone` 模型的外键不是 `user_id`，则可以传递自定义键名作为 `belongsTo` 方法的第二个参数：

    /**
     * 获取拥有该电话的用户模型。
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key');
    }

如果你的父级模型不是使用 `id` 作为主键，或是希望以不同的字段来连接下层模型，则可以传递第三个参数至 `belongsTo` 方法来指定父级数据表的自定义键：

    /**
     * 获取拥有该电话的用户模型。
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key', 'other_key');
    }

<a name="one-to-many"></a>
### 一对多

一个「一对多」关联用于定义单个模型拥有任意数量的其它关联模型。例如，一篇博客文章可能会有无限多个评论。就像其它的 Eloquent 关联一样，可以通过在 Eloquent 模型中写一个函数来定义一对多关联：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * 获取这篇博文下的所有评论。
         */
        public function comments()
        {
            return $this->hasMany('App\Comment');
        }
    }

切记，Eloquent 会自动判断 `Comment` 模型上正确的外键字段。按约定来说，Eloquent 会取用自身模型的名称的「Snake Case」，并在后方加上 `_id`。所以，以此例来说，Eloquent 会假设 `Comment` 模型的外键是 `post_id`。

一旦关联被定义，则可以通过 `comments` 属性来访问评论的集合。切记，因为 Eloquent 提供了「动态属性」，因此我们可以对关联函数进行访问，就像他们是在模型中定义的属性一样：

    $comments = App\Post::find(1)->comments;

    foreach ($comments as $comment) {
        //
    }

当然，因为所有的关联也都提供了查询语句构造器的功能，因此你可以对获取到的评论进一步增加条件，通过调用 `comments` 方法然后在该方法后面链式调用查询条件:

    $comments = App\Post::find(1)->comments()->where('title', 'foo')->first();

就像 `hasOne` 方法，你也可以通过传递额外的参数至 `hasMany` 方法来重写外键与本地键：

    return $this->hasMany('App\Comment', 'foreign_key');

    return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

<a name="one-to-many-inverse"></a>
### 一对多 (反向关联)

现在我们已经能访问到所有文章的评论，让我们来接着定义一个通过评论访问所属文章的关联。若要定义相对于 `hasMany` 的关联，可在子级模型定义一个叫做 `belongsTo` 方法的关联函数：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * 获取该评论所属的文章模型。
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

一旦关联被定义之后，则可以通过 `post` 的「动态属性」来获取 `Comment` 模型相对应的 `Post` 模型：

    $comment = App\Comment::find(1);

    echo $comment->post->title;

在上述例子中，Eloquent 会尝试将 `Comment` 模型的 `post_id` 与 `Post` 模型的 `id` 进行匹配。Eloquent 判断的默认外键名称参考自关联模型的方法，并在方法名称后面加上 `_id`。当然，如果 `Comment` 模型的外键不是 `post_id`，则可以传递自定义键名作为 `belongsTo` 方法的第二个参数：

    /**
     * 获取该评论所属的文章模型。
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key');
    }

如果你的父级模型不是使用 `id` 作为主键，或是你希望以不同的字段来连接下层模型，则可以传递第三个参数给 `belongsTo` 方法来指定上层数据表的自定义键：

    /**
     * 获取该评论所属的文章模型。
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
    }

<a name="many-to-many"></a>
### 多对多

「多对多」关联要稍微比 `hasOne` 及 `hasMany` 关联复杂。如，一个用户可能拥有多种身份，而一种身份能同时被多个用户拥有。举例来说，很多用户都拥有「管理员」的身份。要定义这种关联，需要使用三个数据表：`users`、`roles` 和 `role_user`。`role_user` 表命名是以相关联的两个模型数据表来依照字母顺序命名，并包含了 `user_id` 和 `role_id` 字段。

多对多关联通过编写一个在自身 Eloquent 类调用的 `belongsToMany` 的方法来定义。举个例子，让我们在 `User` 模型中定义 `roles` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 属于该用户的身份。
         */
        public function roles()
        {
            return $this->belongsToMany('App\Role');
        }
    }

一旦关联被定义，则可以使用 `roles` 动态属性来访问用户的身份：

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        //
    }

当然，就如所有其它的关联类型一样，你也可以调用 `roles` 方法并在该关联之后链式调用查询条件：

    $roles = App\User::find(1)->roles()->orderBy('name')->get();

如前文提到那样，Eloquent 会合并两个关联模型的名称并依照字母顺序命名。当然你也可以随意重写这个约定。可通过传递第二个参数至 `belongsToMany` 方法来实现：

    return $this->belongsToMany('App\Role', 'role_user');

除了自定义合并数据表的名称，你也可以通过传递额外参数至 `belongsToMany` 方法来自定义数据表里的键的字段名称。第三个参数是你定义在关联中的模型外键名称，而第四个参数则是你要合并的模型外键名称：

    return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');

#### 定义相对的关联

要定义相对于多对多的关联，只需简单的放置另一个名为 `belongsToMany` 的方法到你关联的模型上。让我们接着以用户身份为例，在 `Role` 模型中定义 `users` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * 属于该身份的用户。
         */
        public function users()
        {
            return $this->belongsToMany('App\User');
        }
    }

如你所见，此定义除了简单的参考 `App\User` 模型外，与 `User` 的对应完全相同。因为我们重复使用了 `belongsToMany` 方法，当定义相对于多对多的关联时，所有常用的自定义数据表与键的选项都是可用的。

#### 获取中间表字段

正如你所知，要操作多对多关联需要一个中间数据表。Eloquent 提供了一些有用的方法来和这张表进行交互。例如，假设 `User` 对象关联到很多的 `Role` 对象。访问这些关联对象时，我们可以在模型中使用 `pivot` 属性来访问中间数据表的数据：

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

注意我们取出的每个 `Role` 模型对象，都会被自动赋予 `pivot` 属性。此属性代表中间表的模型，它可以像其它的 Eloquent 模型一样被使用。

默认情况下，`pivot` 对象只提供模型的键。如果你的 pivot 数据表包含了其它的属性，则可以在定义关联方法时指定那些字段：

    return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');

如果你想要中间表自动维护 `created_at` 和 `updated_at` 时间戳，可在定义关联方法时加上 `withTimestamps` 方法：

    return $this->belongsToMany('App\Role')->withTimestamps();

#### 使用中间表来过滤关联数据

你可以使用 `wherePivot` 和 `wherePivotIn` 来增加中间件表过滤条件：

    return $this->belongsToMany('App\Role')->wherePivot('approved', 1);

    return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);

<a name="has-many-through"></a>
### 远层一对多

「远层一对多」提供了方便简短的方法来通过中间的关联获取远层的关联。例如，一个 `Country` 模型可能通过中间的 `Users` 模型关联到多个 `Posts` 模型。让我们来看看定义此种关联的数据表：

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

虽然 `posts` 本身不包含 `country_id` 字段，但 `hasManyThrough` 关联通过 `$country->posts` 来让我们可以访问一个国家的文章。若运行此查找，则 Eloquent 会检查中间表 `users` 的 `country_id`。在找到匹配的用户 ID 后，就会在 `posts` 数据表中使用它们来进行查找。

现在我们已经检查完了关联的数据表结构，让我们来接着在 `Country` 模型中定义它：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Country extends Model
    {
        /**
         * 获取该国家的所有文章。
         */
        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'App\User');
        }
    }

`hasManyThrough` 方法的第一个参数为我们希望最终访问的模型名称，而第二个参数为中间模型的名称。

当运行关联查找时，通常会使用 Eloquent 的外键约定。如果你想要自定义关联的键，则可以将它们传递至 `hasManyThrough` 方法的第三与第四个参数。第三个参数为中间模型的外键名称，而第四个参数为最终模型的外键名称，第五个参数则为本地键。

    class Country extends Model
    {
        public function posts()
        {
            return $this->hasManyThrough(
                'App\Post', 'App\User',
                'country_id', 'user_id', 'id'
            );
        }
    }

<a name="polymorphic-relations"></a>
### 多态关联

#### 数据表结构

多态关联允许一个模型在单个关联中从属一个以上其它模型。举个例子，想象一下使用你应用的用户可以「评论」文章和视频。使用多态关联关系，您可以使用一个 `comments` 数据表就可以同时满足两个使用场景。首先，让我们观察一下用来创建这关联的数据表结构：

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

有两个需要注意的字段是 `comments` 表中的 `commentable_id` 和 `commentable_type`。其中， `commentable_id` 用于存放文章或者视频的 id ，而 `commentable_type` 用于存放所属模型的类名。注意的是， `commentable_type` 是当我们访问 `commentable` 关联时， ORM 用于判断所属的模型是哪个「类型」。

#### 模型结构

接着，让我们来查看创建这种关联所需的模型定义：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * 获取所有拥有的 commentable 模型。
         */
        public function commentable()
        {
            return $this->morphTo();
        }
    }

    class Post extends Model
    {
        /**
         * 获取所有文章的评论。
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

    class Video extends Model
    {
        /**
         * 获取所有视频的评论。
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

#### 获取多态关联

一旦你的数据表及模型被定义，则可以通过模型来访问关联。例如，若要访问谋篇文章的所有评论，则可以简单的使用 `comments` 动态属性：

    $post = App\Post::find(1);

    foreach ($post->comments as $comment) {
        //
    }

你也可以从多态模型的多态关联中，通过访问调用 `morphTo` 的方法名称来获取拥有者，也就是此例子中 `Comment` 模型的 `commentable` 方法。所以，我们可以使用动态属性来访问这个方法：

    $comment = App\Comment::find(1);

    $commentable = $comment->commentable;

`Comment` 模型的 `commentable` 关联会返回 `Post` 或 `Video` 实例，这取决于评论所属模型的类型。

#### 自定义多态关联的类型字段

默认情况下，Laravel 会使用「包含命名空间的类名」作为多态表的类型区分，例如，`Comment` 属于 `Post` 或者 `Video` , `commentable_type ` 的默认值可以分别是 `App\Post` 或者 `App\Video` 。然而，你也许会想使用应用中的内部结构来对数据库进行解耦。在这个例子中，你可以定义一个「多态对照表」来指引 Eloquent 对各个模型使用自定义名称而非类名：

    use Illuminate\Database\Eloquent\Relations\Relation;

    Relation::morphMap([
        'posts' => App\Post::class,
        'videos' => App\Video::class,
    ]);

你需要在你自己的 `AppServiceProvider ` 中的 `boot` 函数注册这个 `morphMap` ，或者创建一个独立且满足你要求的服务提供者。

<a name="many-to-many-polymorphic-relations"></a>
### 多态多对多关联

#### 数据表结构

除了一般的多态关联，你也可以定义「多对多」的多态关联。例如，博客的 `Post` 和 `Video` 模型可以共用多态关联至 `Tag` 模型。使用多对多的多态关联能够让你的博客文章及图片共用独立标签的单个列表。首先，让我们先来查看数据表结构：

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

接着，我们已经准备好定义模型的关联。`Post` 及 `Video` 模型都会拥有 `tags` 方法，并在该方法内调用自身 Eloquent 类的 `morphToMany` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * 获取该文章的所有标签。
         */
        public function tags()
        {
            return $this->morphToMany('App\Tag', 'taggable');
        }
    }

#### 定义相对的关联

然后，在 `Tag` 模型上，你必须为每个要关联的模型定义一个方法。因此，在这个例子中，我们需要定义一个 `posts` 方法及一个 `videos` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Tag extends Model
    {
        /**
         * 获取所有被赋予该标签的文章。
         */
        public function posts()
        {
            return $this->morphedByMany('App\Post', 'taggable');
        }

        /**
         * 获取所有被赋予该标签的图片。
         */
        public function videos()
        {
            return $this->morphedByMany('App\Video', 'taggable');
        }
    }

#### 获取关联

一旦你的数据表及模型被定义，则可以通过你的模型来访问关联。例如，你可以简单的使用 `tags` 动态属性来访问文章的所有标签：

    $post = App\Post::find(1);

    foreach ($post->tags as $tag) {
        //
    }

你也可以从多态模型的多态关联中，通过访问运行调用 `morphedByMany` 的方法名称来获取拥有者。在此例子中，就是 `Tag` 模型的 `posts` 或 `videos` 方法。因此，你可以通过访问使用动态属性来访问这个方法：

    $tag = App\Tag::find(1);

    foreach ($tag->videos as $video) {
        //
    }

<a name="querying-relations"></a>
## 查找关联

所有类型的 Eloquent 关联都是通过函数来定义的，你可以通过调用这些函数来获得关联的一个实例，而不需要实际运行关联的查找。此外，所有类型的 Eloquent 关联也提供了 [查询语句构造器](/docs/{{version}}/queries) 的功能，让你能够在数据库运行该 SQL 前，在关联查找后面链式调用条件。

例如，假设有一个博客系统，其中 `User` 模型拥有许多关联的 `Post` 模型:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 获取该用户的所有文章。
         */
        public function posts()
        {
            return $this->hasMany('App\Post');
        }
    }

你可以查找 `posts` 关联并增加额外的条件至关联，像这样：

    $user = App\User::find(1);

    $user->posts()->where('active', 1)->get();


你可以在关联中使用 [查询语句构造器](/docs/{{version}}/queries)  中的任意方法，所以，欢迎查阅查询语句构造器的相关文档以便了解那些有助于你实现的方法。

<a name="relationship-methods-vs-dynamic-properties"></a>
### 关联方法与动态属性

如果你不需要增加额外的条件至 Eloquent 的关联查找，则可以简单的像访问属性一样来访问关联。例如我们刚刚的 `User` 及 `Post` 模型示例，我们可以像这样来访问所有用户的文章：

    $user = App\User::find(1);

    foreach ($user->posts as $post) {
        //
    }

动态属性是「延迟加载」的，意味着它们只会在被访问的时候才加载关联数据。正因为如此，开发者通常需要使用 [预加载](#eager-loading) 来预先加载关联数据，关联数据将会在模型加载后被访问。预加载能有效减少你的 SQL 查询语句。

<a name="querying-relationship-existence"></a>
### 查找关联是否存在

当访问模型的纪录时，你可能希望根据关联的存在来对结果进行限制。比方说你想获取博客中那些至少拥有一条评论的文章。则可以通过传递名称至关联的 `has` 方法来实现：

    // 获取那些至少拥有一条评论的文章...
    $posts = App\Post::has('comments')->get();

你也可以制定运算符及数量来进一步自定义查找：

    // 获取所有至少有三条评论的文章...
    $posts = Post::has('comments', '>=', 3)->get();

也可以使用「点」符号来构造嵌套的 has 语句。例如，你可能想获取那些至少有一条评论被投票的文章：

    // 获取所有至少有一条评论被评分的文章...
    $posts = Post::has('comments.votes')->get();

如果你想要更高级的用法，则可以使用 `whereHas` 和 `orWhereHas` 方法，在 has 查找里设置「where」条件。此方法可以让你增加自定义条件至关联条件中，例如对评论内容进行检查：

    //  获取那些至少有一条评论包含 foo 的文章
    $posts = Post::whereHas('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

<a name="counting-related-models"></a>
### 关联数据计数

如果你想对关联数据进行计数但又不想再发起单独的 SQL 请求，你可以使用 `withCount` 方法，此方法会在你的结果集中增加一个 `{relation}_count` 字段：

    $posts = App\Post::withCount('comments')->get();

    foreach ($posts as $post) {
        echo $post->comments_count;
    }

你还可以像在查询语句中添加约束一样，获取多重关联的「计数」。

    $posts = Post::withCount(['votes', 'comments' => function ($query) {
        $query->where('content', 'like', 'foo%');
    }])->get();

    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;

<a name="eager-loading"></a>
## 预加载

当通过属性访问 Eloquent 关联时，该关联数据会被「延迟加载」。意味着该关联数据只有在你使用属性访问它时才会被加载。不过，Eloquent 可以在你查找上层模型时「预加载」关联数据。预加载避免了 N + 1 查找的问题。要说明 N + 1 查找的问题，可试想一个关联到 `Author` 的 `Book` 模型，如下所示：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Book extends Model
    {
        /**
         * 获取编写该书的作者。
         */
        public function author()
        {
            return $this->belongsTo('App\Author');
        }
    }

现在，让我们来获取所有书籍及其作者的数据：

    $books = App\Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

上方的循环会运行一次查找并取回所有数据表上的书籍，接着每本书会运行一次查找作者的操作。因此，若存在着 25 本书，则循环就会执行 26 次查找：1 次是查找所有书籍，其它 25 次则是在查找每本书的作者。

很幸运地，我们可以使用预加载来将查找的操作减少至 2 次。可在查找时使用 with 方法来指定想要预加载的关联数据：

    $books = App\Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

对于该操作则只会执行两条 SQL 语句：

    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)

#### 预加载多种关联

有时你可能想要在单次操作中预加载多种不同的关联。要这么做，只需传递额外的参数至 `方法` 即可：

    $books = App\Book::with('author', 'publisher')->get();

#### 嵌套预加载

若要预加载嵌套关联，则可以使用「点」语法。例如，让我们在一个 Eloquent 语法中，预加载所有书籍的作者，及所有作者的个人联系方式：

    $books = App\Book::with('author.contacts')->get();

<a name="constraining-eager-loads"></a>
### 预加载条件限制

有时你可能想要预加载关联，并且指定预加载查询的额外条件。如下所示：

    $users = App\User::with(['posts' => function ($query) {
        $query->where('title', 'like', '%first%');
    }])->get();

在这个例子里，Eloquent 只会预加载标题包含 `first` 的文章。当然，你也可以调用其它的 [查询语句构造器](/docs/{{version}}/queries) 来进一步自定义预加载的操作：

    $users = App\User::with(['posts' => function ($query) {
        $query->orderBy('created_at', 'desc');
    }])->get();

<a name="lazy-eager-loading"></a>
### 延迟预加载

有时你可能需要在上层模型被获取后才预加载关联。当你需要来动态决定是否加载关联模型时会很有帮助，如下所示：

    $books = App\Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

如果你想设置预加载查询的额外条件，则可以传递一个键值为你想要的关联的数组至 `load` 方法。这个数组的值应是用于接收查询 `闭包` 实例：

    $books->load(['author' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }]);

<a name="inserting-and-updating-related-models"></a>
## 写入关联模型

<a name="the-save-method"></a>
### Save 方法

Eloquent 提供了便捷的方法来将新的模型增加至关联中。例如，将新的 `Commnet` 写入至 `Post` 模型。除了手动设置 `Commnet` 的 `post_id` 属性外，你也可以直接使用关联的 `save` 方法来写入 `Comment`：

    $comment = new App\Comment(['message' => 'A new comment.']);

    $post = App\Post::find(1);

    $post->comments()->save($comment);

注意我们并没有使用动态属性来访问 `comments` 关联。相反地，我们调用了 comments 方法来获取关联的实例。save 方法会自动在新的 `Comment` 模型中增加正确的 `post_id` 值。

如果你需要保存多个关联模型，则可以使用 `saveMany` 方法：

    $post = App\Post::find(1);

    $post->comments()->saveMany([
        new App\Comment(['message' => 'A new comment.']),
        new App\Comment(['message' => 'Another comment.']),
    ]);

<a name="the-create-method"></a>
### Create 方法

除了 `save` 与 `saveMany` 方法外，你也可以使用 `create` 方法，该方法允许传入属性的数组来建立模型并写入数据库。`save` 与 `create` 的不同之处在于，`save` 允许传入一个完整的 Eloquent 模型实例，但 `create` 只允许传入原始的 PHP 数组：

    $post = App\Post::find(1);

    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

在使用 create 方法前，请确认你已浏览了文档的 [批量赋值](/docs/{{version}}/eloquent#mass-assignment) 章节。

<a name="updating-belongs-to-relationships"></a>
### 更新「从属」关联

当更新一个 `belongsTo` 关联时，可以使用 `associate` 方法。此方法会将外键设置到下层模型：

    $account = App\Account::find(10);

    $user->account()->associate($account);

    $user->save();

当删除一个 `belongsTo` 关联时，你可以使用 `dissociate` 方法。此方法会置该关联的外键为空(null)：

    $user->account()->dissociate();

    $user->save();

<a name="updating-many-to-many-relationships"></a>
### 多对多关联

#### 附加与卸除

当使用多对多关联时，Eloquent 提供了一些额外的辅助函数让操作关联模型更加方便。例如，让我们假设一个用户可以拥有多个身份，且每个身份都可以被多个用户拥有。要附加一个规则至一个用户，并连接模型以及将记录写入至中间表，则可以使用 `attach` 方法：

    $user = App\User::find(1);

    $user->roles()->attach($roleId);

当附加一个关联至模型时，你也可以传递一个需被写入至中间表的额外数据数组：

    $user->roles()->attach($roleId, ['expires' => $expires]);

当然，我们有时也需要来移除用户的身份。要移除一个多对多的纪录，可使用 `detach` 方法。`detach` 方法会从中间表中移除正确的纪录；当然，这两个模型依然会存在于数据库中：

    // 移除用户身上某一身份...
    $user->roles()->detach($roleId);

    // 移除用户身上所有身份...
    $user->roles()->detach();

为了方便，attach 与 detach 都允许传入 ID 数组：

    $user = App\User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([1 => ['expires' => $expires], 2, 3]);

#### 更新关联

你也可以使用 `sync` 方法去创建一个多对多的关联。 `sync` 方法可以用数组形式的 IDs 插入中间的数据表。任何一个不存在于给定数组的 IDs 将会在中间表内被删除。所以，操作完成之后，只有那些在给定数组内的 IDs 会被保留在中间表中。

    $user->roles()->sync([1, 2, 3]);

You may also pass additional intermediate table values with the IDs:

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);

If you do not want to detach existing IDs, you may use the `syncWithoutDetaching` method:

    $user->roles()->syncWithoutDetaching([1, 2, 3]);

#### Saving Additional Data On A Pivot Table

When working with a many-to-many relationship, the `save` method accepts an array of additional intermediate table attributes as its second argument:

    App\User::find(1)->roles()->save($role, ['expires' => $expires]);

#### Updating A Record On A Pivot Table

If you need to update an existing row in your pivot table, you may use `updateExistingPivot` method. This method accepts the pivot record foreign key and an array of attributes to update:

    $user = App\User::find(1);

	$user->roles()->updateExistingPivot($roleId, $attributes);

<a name="touching-parent-timestamps"></a>
## 连动父级时间戳

当一个模型 `belongsTo` 或 `belongsToMany` 另一个模型时，像是一个 `Comment` 属于一个 `Post`。这对于子级模型被更新时，要更新父级的时间戳相当有帮助。举例来说，当一个 `Commnet` 模型被更新时，你可能想要「连动」更新 `Post` 所属的 `updated_at` 时间戳。Eloquent 使得此事相当容易。只要在关联的下层模型中增加一个包含名称的 `touches` 属性即可：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * 所有的关联将会被连动。
         *
         * @var array
         */
        protected $touches = ['post'];

        /**
         * 获取拥有此评论的文章。
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

现在，当你更新一个 Comment 时，它所属的 Post 拥有的 `updated_at` 字段也会被同时更新，使其更方便的得知何时让一个 `Post` 模型的缓存失效：

    $comment = App\Comment::find(1);

    $comment->text = 'Edit to this comment!';

    $comment->save();
