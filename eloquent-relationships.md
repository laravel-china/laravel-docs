# Eloquent：关联

- [简介](#introduction)
- [定义关联](#defining-relationships)
    - [一对一](#one-to-one)
    - [一对多](#one-to-many)
    - [多对多](#many-to-many)
    - [远层一对多](#has-many-through)
    - [多态关联](#polymorphic-relations)
    - [多态多对多关联](#many-to-many-polymorphic-relations)
- [关联查找](#querying-relations)
    - [预加载](#eager-loading)
    - [预加载条件限制](#constraining-eager-loads)
    - [延迟预加载](#lazy-eager-loading)
- [写入关联模型](#inserting-related-models)
    - [多对多关联](#inserting-many-to-many-relationships)
    - [连动上层时间戳](#touching-parent-timestamps)

<a name="introduction"></a>
## 简介

数据表通常会互相关联。例如，一篇博客文章可能有很多评论，或是一张订单与下单的客户相关联。Eloquent 让管理和处理这些关联变得很容易，并支持多种类型的关联：

- [一对一](#one-to-one)
- [一对多](#one-to-many)
- [多对多](#many-to-many)
- [远层一对多](#has-many-through)
- [多态关联](#polymorphic-relations)
- [多态多对多关联](#many-to-many-polymorphic-relations)

<a name="defining-relationships"></a>
## 定义关联

你会在你的 Eloquent 模型类内将 Eloquent 关联定义为函数。因为关联像 Eloquent 模型一样也可以作为强大的 [查询语句构造器](/docs/{{version}}/queries)，定义关联为函数提供了强而有力的链式调用及查找功能。例如：

    $user->posts()->where('active', 1)->get();

不过，在深入了解使用关联之前，先让我们学习如何定义每个类型：

<a name="one-to-one"></a>
### 一对一

一对一关联是很基本的关联。例如一个 `User` 模型也许会对应到一个 `Phone`。要定义这种关联，我们必须将 `phone` 方法放置于 `User` 模型上。`phone` 方法应该要返回基类 Eloquent 上 `hasOne` 方法的结果：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 获取与指定用户互相关联的电话纪录。
         */
        public function phone()
        {
            return $this->hasOne('App\Phone');
        }
    }

传到 hasOne 方法里的第一个参数是关联模型的类名称。定义好关联之后，我们就可以使用 Eloquent 的动态属性来获取关联纪录。动态属性让你能够访问关联函数，就像他们是在模型中定义的属性：

    $phone = User::find(1)->phone;

Eloquent 会假设对应的关联的外键名称是基于模型名称。在这个例子里，它会自动假设 `Phone` 模型拥有 `user_id` 外键。如果你想要重写这个惯例，可以传入第二个参数到 `hasOne` 方法里。

    return $this->hasOne('App\Phone', 'foreign_key');

此外，Eloquent 默认外键会在上层模型的 `id` 字段有个对应值。换句话说，Eloquent 会寻找用户的 `id` 字段与 `Phone` 模型的 `user_id` 字段的值相同的纪录。如果你想让关联使用 `id` 以外的值，你可以传递第三个参数至 `hasOne` 方法来指定你自定的键：

    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

#### 定义相对的关联

所以，我们可以从我们的 `User` 访问 `Phone` 模型。现在，让我们在 `Phone` 模型定义一个关联，此关联能够让我们访问拥有此电话的 `User`。我们可以定义与 `hasOne` 关联相对的 `belongsTo` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Phone extends Model
    {
        /**
         * 获取拥有此电话的用户。
         */
        public function user()
        {
            return $this->belongsTo('App\User');
        }
    }

在上述例子中，Eloquent 会尝试匹配 `Phone` 模型的 `user_id` 至 `User` 模型的 `id`。Eloquent 判断的默认外键名称参考于关联模型的方法，并在方法名称后面加上 `_id`。当然，如果 `Phone` 模型的外键不是 `user_id`，你可以传递自定的键名至 `belongsTo` 方法的第二个参数：

    /**
     * 获取拥有此电话的用户。
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key');
    }

如果你的上层模型不是使用 `id` 作为主键，或是你希望以不同的字段 join 下层模型，你可以传递第三参数至 `belongsTo` 方法指定上层数据表的自定键：

    /**
     * 获取拥有此电话的用户。
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key', 'other_key');
    }

<a name="one-to-many"></a>
### 一对多

一个「一对多」关联使用于定义单一模型拥有任意数量的其他关联模型。例如，一篇博客文章可能有无限多个评论。就像其他的 Eloquent 关联一样，可以借由放置一个函数至你的 Eloquent 模型来定义一对多关联：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * 获取博客文章的评论。
         */
        public function comments()
        {
            return $this->hasMany('App\Comment');
        }
    }

切记，Eloquent 会自动判断 `Comment` 模型上正确的外键字段。以惯例来说，Eloquent 会取用自身模型的「蛇底式命名」后的名称，并在后方加上 `_id`。所以，以此例来说，Eloquent 会假设 `Comment` 模型的外键是 `post_id`。

一旦关联被定义之后，我们可以通过 `comments` 属性访问评论的集合。切记，因为 Eloquent 提供了「动态属性」，我们可以访问关联函数，就像他们是在模型中定义的属性：

    $comments = App\Post::find(1)->comments;

    foreach ($comments as $comment) {
        //
    }

当然，因为所有的关联也提供查询语句构造器的功能，你可以对获取到的评论进一步增加条件，通过调用 `comments` 方法，接着在该查找的后方链式调用查找条件：

    $comments = App\Post::find(1)->comments()->where('title', 'foo')->first();

就像 `hasOne` 方法，你也可以通过传递额外的参数至 `hasMany` 方法重写外键与本地键：

    return $this->hasMany('App\Comment', 'foreign_key');

    return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

#### 定义相对的关联

现在我们能访问所有文章的评论，让我们定义一个通过评论访问上层文章的关联。若要定义相对于 `hasMany` 的关联，在下层模型定义一个叫做 `belongsTo` 方法的关联函数：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * 获取拥有此评论的文章。
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

一旦关联被定义之后，我们可以通过 `post`「动态属性」获取 `Comment` 的 `Post` 模型：

    $comment = App\Comment::find(1);

    echo $comment->post->title;

在上述例子中，Eloquent 会尝试匹配 `Comment` 模型的 `post_id` 至 `Post` 模型的 `id`。Eloquent 判断的默认外键名称参考于关联模型的方法，并在方法名称后面加上 `_id`。当然，如果 `Comment` 模型的外键不是 `post_id`，你可以传递自定的键名至 `belongsTo` 方法的第二个参数：

    /**
     * 获取拥有此评论的文章。
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key');
    }

如果你的上层模型不是使用 `id` 作为主键，或是你希望以不同的字段 join 下层模型，你可以传递第三参数至 `belongsTo` 方法指定上层数据表的自定键：

    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
    }

<a name="many-to-many"></a>
### 多对多

多对多关联稍微比 `hasOne` 及 `hasMany` 关联还复杂。这种关联的例子如，一个用户可能用有很多身份，而一种身份可能很多用户都有。举例来说，很多用户都拥有「管理者」的身份。要定义这种关联，需要使用三个数据表：`users`、`roles` 和 `role_user`。`role_user` 表命名是以相关联的两个模型数据表，依照字母顺序命名，并包含了 `user_id` 和 `role_id` 字段。

多对多关联的定义是通过编写一个在自身 Eloquent 类调用 `belongsToMany` 的方法。举个例子，让我们在 `User` 模型定义 `roles` 方法：

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

一旦关联被定义，你可以使用 `roles` 动态属性访问用户的身份：

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        //
    }

当然，就像所有的其他关联类型，你可以调用 `roles` 方法，接着在该关联之后链式调用上查找的条件：

    $roles = App\User::find(1)->roles()->orderBy('name')->get();

如前文所提，若要判断关联合并的数据表名称，Eloquent 会合并两个关联模型的名称并依照字母顺序命名。当然你可以自由的重写这个惯例。你可以通过传递第二个参数至 `belongsToMany` 方法来达成：

    return $this->belongsToMany('App\Role', 'user_roles');

除了自定合并数据表的名称，你也可以通过传递额外参数至 `belongsToMany` 方法来自定数据表里键的字段名称。第三个参数是你定义在关联中的模型的外键名称，而第四个参数则是你要合并的模型中的外键名称：

    return $this->belongsToMany('App\Role', 'user_roles', 'user_id', 'role_id');

#### 定义相对的关联

要定义相对于多对多的关联，你只需要简单的放置另一个名为 `belongsToMany` 至你关联的模型。继续我们的用户身份例子，让我们在 `Role` 模型定义 `users` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * 属于该身份的用户
         */
        public function users()
        {
            return $this->belongsToMany('App\User');
        }
    }

如你所见，此定义除了简单的参考 `App\User` 模型外，与 `User` 的对应完全相同。因为我们重复使用了 `belongsToMany` 方法，当定义相对于多对多的关联时，所有常用的自定数据表与键的选项都是可用的。

#### 获取中介表字段

如你所知，要操作多对多关联需要一个中介的数据表。Eloquent 提供了一些有用的方法和这张表交互。例如，假设 `User` 对象关联到很多 `Role` 对象。访问这些关联对象时，我们可以在模型使用 `pivot` 属性访问中介数据表的数据：

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

注意我们取出的每个 `Role` 模型对象，会自动被赋予 `pivot` 属性。此属性是代表中介表的模型，且可以像其它的 Eloquent 模型一样被使用。

默认情况下，`pivot` 对象只提供模型的键。如果你的 pivot 数据表包含了其他的属性，可以在定义关联方法时指定那些字段：

    return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');

如果你想要枢纽表自动维护 `created_at` 和 `updated_at` 时间戳，在定义关联方法时加上 `withTimestamps` 方法：

    return $this->belongsToMany('App\Role')->withTimestamps();

<a name="has-many-through"></a>
### 远层一对多

「远层一对多」提供了方便简短的方法，可以通过中介的关联获取远层的关联。例如，一个 `Country` 模型可能通过中介的 `Users` 模型关联到多个 `Posts` 模型。让我们看看定义此种关联的数据表：

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

虽然 `posts` 本身不包含 `country_id` 字段，但 `hasManyThrough` 关联通过 `$country->posts` 来提供我们访问一个国家的文章。要运行此查找，Eloquent 会检查中介表 `users` 的 `country_id`。在找到匹配的用户 IDs 后，就会在 `posts` 数据表使用它们来查找。

现在我们已经检查了关联的数据表结构，让我们将它定义在 `Country` 模型：

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

`hasManyThrough` 方法的第一个参数为我们希望最终访问的模型名称，而第二个参数为中介模型的名称。

当运行关联的查找时，通常将会使用 Eloquent 的外键惯例。如果你想要自定关联的键，你可以传递它们至 `hasManyThrough` 方法的第三与第四个参数。第三个参数为中介模型的外键名称，而第四个参数为最终模型的外键名称。

    class Country extends Model
    {
        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'App\User', 'country_id', 'user_id');
        }
    }

<a name="polymorphic-relations"></a>
### 多态关联

#### 数据表结构

多态关联允许一个模型在单一的关联从属一个以上的其他模型。举个例子，假设你想保存照片给你的工作人员及你的产品。使用多态关联，你可以对这两种情况使用单一的 `photos` 数据表。首先，让我们查看创建这种关联所需的数据表结构：

    staff
        id - integer
        name - string

    products
        id - integer
        price - integer

    photos
        id - integer
        path - string
        imageable_id - integer
        imageable_type - string

有两个要注意的重要字段是 `photos` 数据表的 `imageable_id` 和 `imageable_type` 字段。`imageable_id` 字段会包含所属的工作人员或产品的 ID 值，而 `imageable_type` 字段会包含所属的模型类名称。当访问 `imageable` 关联时，`imageable_type` 字段会被 ORM 用于判断所属的模型是哪个「类型」。

#### 模型结构

接着，让我们查看创建这种关联所需的模型定义：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Photo extends Model
    {
        /**
         * 获取所有所属的可拥有图片的模型。
         */
        public function imageable()
        {
            return $this->morphTo();
        }
    }

    class Staff extends Model
    {
        /**
         * 获取所有工作人员的照片。
         */
        public function photos()
        {
            return $this->morphMany('App\Photo', 'imageable');
        }
    }

    class Product extends Model
    {
        /**
         * 获取所有产品的照片。
         */
        public function photos()
        {
            return $this->morphMany('App\Photo', 'imageable');
        }
    }

#### 获取多态关联

一旦你的数据表及模型被定义后，你可以通过你的模型访问关联。例如，若要访问工作人员的所有照片，我们可以简单的使用 `photos` 动态属性：

    $staff = App\Staff::find(1);

    foreach ($staff->photos as $photo) {
        //
    }

你也可以从多态模型的多态关联里，通过访问运行调用 `morphTo` 的方法名称获取拥有者。在我们例子中，就是 `Phone` 模型中的 `imageable` 方法。所以，我们可以访问使用动态属性访问这个方法：

    $photo = App\Photo::find(1);

    $imageable = $photo->imageable;

`Photo` 模型的 `imageable` 关联会返回 `Staff` 或 `Product` 实例，这取决于照片所属模型的类型。

#### 自定义多态关联的类型字段

默认情况下，Laravel 会使用「包含命名空间的类名」作为多态表的类型区分，例如，`Post` 和 `Comment` 可以被 `Like`，`likable_type` 的值会是 `App\Post` 或 `App\Comment`。

然而，你也可以选择自定义自己的「多态对照表」：

    Relation::morphMap([
        App\Post::class,
        App\Comment::class,
    ]);

或者定义对应字段：

    Relation::morphMap([
        'posts' => App\Post::class,
        'likes' => App\Like::class,
    ]);

> [Summer](http://github.com/summerblue): 可以使用 class_basename(App\Post::class) 来得到 `Post`

你可以在 `AppServiceProvider` 中注册你的「多态对照表」，或者创建一个单独的提供者文件。

<a name="many-to-many-polymorphic-relations"></a>
### 多态多对多关联

#### 数据表结构

除了一般的多态关联，你也可以定义「多对多」的多态关联。例如，博客的 `Post` 和 `Video` 模型可以共用多态关联至 `Tag` 模型。使用多对多的多态关联能够让你的博客文章及影片共用独立标签的单一列表。首先，让我们查看数据表结构：

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

接着，我们已经准备好定义模型的关联。`Post` 及 `Video` 模型会都拥有 `tags` 方法，并在该方法内调用自身 Eloquent 类的 `morphToMany` 方法：

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

然后，在 `Tag` 模型上，你必须对每个要关联的模型定义一个方法。所以，在这个例子里，我们需要定义一个 `posts` 方法及一个 `videos` 方法：

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
         * 获取所有被赋予该标签的影片。
         */
        public function videos()
        {
            return $this->morphedByMany('App\Video', 'taggable');
        }
    }

#### 获取关联

一旦你的数据表及模型被定义后，你可以通过你的模型访问关联。例如，若要访问文章的所有标签，你可以简单的使用 `tags` 动态属性：

    $post = App\Post::find(1);

    foreach ($post->tags as $tag) {
        //
    }

你也可以从多态模型的多态关联里，通过访问运行调用 `morphedByMany` 的方法名称获取拥有者。在我们例子中，就是 `Tag` 模型中的 `posts` 或 `videos` 方法。所以，你可以访问使用动态属性访问这个方法：

    $tag = App\Tag::find(1);

    foreach ($tag->videos as $video) {
        //
    }

<a name="querying-relations"></a>
## 查找关联

因为所有类型的 Eloquent 关联都是通过函数定义，你可以调用这些函数获得关联的一个实例，并不需要实际运行关联的查找。此外，所有类型的 Eloquent 关联也提供了[查询语句构造器](/docs/{{version}}/queries)的功能，让你能够在你对你的数据库运行该 SQL 前，在关联查找接着链式调用上条件。

例如，假设有一个博客系统，其中 `User` 模型拥有许多关联的 `Post` 模型：

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

你可以查找 `posts` 关联并增加额外的条件至关联，像是：

    $user = App\User::find(1);

    $user->posts()->where('active', 1)->get();

请注意，你可以在关联使用 [查询语句构造器](/docs/{{version}}/queries) 的任何方法！

#### 关联方法与动态属性

如果你不需要增加额外的条件至 Eloquent 的关联查找，你可以简单的访问关联就如同属性一样。例如，延续我们刚刚的 `User` 及 `Post` 例子模型，我们可以访问所有用户的文章，像是：

    $user = App\User::find(1);

    foreach ($user->posts as $post) {
        //
    }

动态属性是「延迟加载」，意指它们只会在你需要访问它们的时候加载关联数据。正因为如此，开发者通常使用[预加载](#eager-loading)来预先加载在加载模型后将会被访问的关联数据。预加载能有效减少你的 SQL 查询语句。

#### 查找关联是否存在

当访问模型的纪录时，你可能希望根据关联的存在限制你的结果。例如，假设你想获取博客的所有至少有一篇评论的文章。要达成这项功能，你可以传递名称至关联的 `has` 方法：

    // 获取所有至少有一篇评论的文章...
    $posts = App\Post::has('comments')->get();

你也可以制定运算符及数量来进一步的自定该查找：

    // 获取所有至少有三篇评论的文章...
    $posts = Post::has('comments', '>=', 3)->get();

也可以使用「点」符号建构嵌套的 `has` 语句。例如，你可能想获取所有至少有一篇评论被评分的文章：

    // 获取所有至少有一篇评论被评分的文章...
    $posts = Post::has('comments.votes')->get();

如果你想要更高端的用法，可以使用 `whereHas` 和 `orWhereHas` 方法，在 `has` 查找里设置「where」条件。此方法可以让你增加自定义的条件至关联条件中，像是检查评论的内容：

    // 获取所有至少有一篇评论相似于 foo% 的文章
    $posts = Post::whereHas('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

<a name="eager-loading"></a>
### 预加载

当通过属性访问 Eloquent 关联时，该关联数据会被「延迟加载」。意指该关联数据直到你第一次以属性访问前，实际上并没有被加载。不过，Eloquent 可以在你查找上层模型时「预加载」关联数据。预加载避免了 N + 1 查找的问题。要说明 N + 1 查找的问题，试想一个 `Book` 模型会关联至 `Author`：

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

现在，让我们获取所有的书籍及其作者：

    $books = App\Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

上方的循环会运行一次查找并取回所有数据表上的书籍，然后每本书都会运行一次查找获取作者。所以，若有 25 本书，循环就会进行 26 次查找：1 次是原本的书籍，及对每本书查找并获取作者的额外 25 次。

很幸运地，我们可以使用预加载将查找的操作减少至 2 次。当查找时，使用 `with` 方法指定想要预加载的关联数据：

    $books = App\Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

对于该操作，只会运行两次查找：

    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)

#### 预加载多种关联

有时你可能想要在单次操作中预加载多种不同的关联。要这么做，只需要传递额外的参数至 `方法`：

    $books = App\Book::with('author', 'publisher')->get();

#### 嵌套预加载

若要预加载嵌套关联，你可以使用「点」语法。例如，让我们在一个 Eloquent 语法中，预加载所有书籍的作者，及所有作者的个人联系方式：

    $books = App\Book::with('author.contacts')->get();

<a name="constraining-eager-loads"></a>
### 预加载条件限制

有时你可能想要预加载关联，并且指定预加载额外的查找条件。下面有一个例子：

    $users = App\User::with(['posts' => function ($query) {
        $query->where('title', 'like', '%first%');

    }])->get();

在这个例子里，Eloquent 只会预加载文章标题字段包含 `first` 的文章。当然，你也可以调用其他的[查询语句构造器](/docs/{{version}}/queries)来进一步自定预加载的操作：

    $users = App\User::with(['posts' => function ($query) {
        $query->orderBy('created_at', 'desc');

    }])->get();

<a name="lazy-eager-loading"></a>
### 延迟预加载

有时你可能需要在上层模型已经被获取后才预加载关联。例如，当你需要动态决定是否加载关联模型时相当有帮助：

    $books = App\Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

如果你想设置预加载查找的额外条件，你可以传递一个 `闭包` 至 `load` 方法：

    $books->load(['author' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }]);

<a name="inserting-related-models"></a>
## 写入关联模型

#### Save 方法

Eloquent 提供了方便的方法来增加新的模型至关联中。例如，也许你需要写入新的 `Commnet` 至 `Post` 模型中。除了手动设置 `Commnet` 的 `post_id` 属性外，你也可以直接使用关联的 `save` 方法写入 `Comment`：

    $comment = new App\Comment(['message' => 'A new comment.']);

    $post = App\Post::find(1);

    $post->comments()->save($comment);

注意我们并没有使用动态属性访问 `comments` 关联。相反地，我们调用 `comments` 方法获取关联的实例。`save` 方法会自动在新的 `Comment` 模型增加正确的 `post_id` 值。

如果你需要保存多个关联模型，你可以使用 `saveMany` 方法：

    $post = App\Post::find(1);

    $post->comments()->saveMany([
        new App\Comment(['message' => 'A new comment.']),
        new App\Comment(['message' => 'Another comment.']),
    ]);

#### Save 与多对多关联

当使用多对多关联时，`save` 方法允许传入一个额外的中介表属性数组作为第二个参数：

    App\User::find(1)->roles()->save($role, ['expires' => $expires]);

#### Create 方法

除了 `save` 与 `saveMany` 方法，你也可以使用 `create` 方法，该方法允许传入属性的数组来立模型，并写入至数据库。同样的，`save` 与 `create` 不同的地方是，`save` 允许传入一个完整的 Eloquent 模型实例，但 `create` 只允许原始的 PHP `数组`：

    $post = App\Post::find(1);

    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

在使用 `create` 方法前，请确定浏览了文档的[批量赋值](/docs/{{version}}/eloquent#mass-assignment)章节。

<a name="updating-belongs-to-relationships"></a>
#### 更新「从属」关联

当更新一个 `belongsTo` 关联时，你可以使用 `associate` 方法。此方法会设置外键至下层模型：

    $account = App\Account::find(10);

    $user->account()->associate($account);

    $user->save();

当删除一个 `belongsTo` 关联时，你可以使用 `dissociate` 方法。此方法会重置关联至此下层模型的外键：

    $user->account()->dissociate();

    $user->save();

<a name="inserting-many-to-many-relationships"></a>
### 多对多关联

#### 附加与卸除

当使用多对多关联时，Eloquent 提供一些额外的辅助方法让操作关联模型时更加方便。例如，让我们假设一个用户可以拥有多个身份，且每个身份可以被多个用户拥有。要附加一个规则至一个用户，并 join 模型及写入记录至中介表，可以使用 `attach` 方法：

    $user = App\User::find(1);

    $user->roles()->attach($roleId);

当附加一个关联至模型时，你也可以传递一个需被写入至中介表的额外数据数组：

    $user->roles()->attach($roleId, ['expires' => $expires]);

当然，有些时候也需要移除用户的一个身份。要移除一个多对多的纪录，使用 `detach` 方法。`detach` 方法会从中介表中移除正确的纪录；当然，两个模型依然会存在于数据库中：

    // 从用户上移除单一身份...
    $user->roles()->detach($roleId);

    // 从用户上移除所有身份...
    $user->roles()->detach();

为了方便，`attach` 与 `detach` 都允许传入 IDs 的数组：

    $user = App\User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([1 => ['expires' => $expires], 2, 3]);

#### 为了方便的同步

你也可以使用 `sync` 方法建构多对多关联。`sync` 允许传入放置于中介表的 IDs 数组。任何不在指定数组中的 IDs 将会从中介表中被删除。所以，在此操作结束后，只会有数组中的 IDs 存在于中介表中：

    $user->roles()->sync([1, 2, 3]);

你也可以传递中介表上该 IDs 额外的值：

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);

<a name="touching-parent-timestamps"></a>
### 连动上层时间戳

当一个模型 `belongsTo` 或 `belongsToMany` 另一个模型时，像是一个 `Comment` 属于一个 `Post`，对于下层模型被更新时，欲更新上层的时间戳相当有帮助。举例来说，当一个 `Commnet` 模型被更新，你可能想要自动的「连动」所属 `Post` 的 `updated_at` 时间戳。Eloquent 使得此事相当容易。只要在关联的下层模型增加一个包含名称的 `touches` 属性即可：

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

现在，当你更新一个 `Comment`，它所属的 `Post` 拥有的 `updated_at` 字段也会同时更新：

    $comment = App\Comment::find(1);

    $comment->text = '编辑此评论！';

    $comment->save();
