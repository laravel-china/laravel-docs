# Eloquent: 关联

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

此外，Eloquent 假定外键值是与父级 `id`（或自定义 `$primaryKey`）列的值相匹配的。 换句话说，Eloquent 将在 `Phone` 记录的 `user_id` 列中查找与用户表的 `id` 列相匹配的值。 如果您希望该关联使用 `id`以外的自定义键名，则可以通过给 `hasOne` 方法传递第三个参数的形式指定：

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

在上面的例子中，Eloquent 会尝试匹配 `Phone` 模型上的 `user_id` 至 `User` 模型上的 `id`。 它是通过检查关系方法的名称并使用 `_id`  作为后缀名来确定默认的外键名称。 但是，如果`Phone`模型的外键不是`user_id`，那么可以将自定义键名作为第二个参数传递给`belongsTo`方法：

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

`belongsTo` 关联允许定义默认模型，这适应于当关联结果返回的是 `null` 的情况。这种设计模式通常称为 [空对象模式](https://en.wikipedia.org/wiki/Null_Object_pattern)，为您免去了额外的条件判断代码。在下面的例子中，`user` 关联如果没有找到博文的作者，就会返回一个空的 `App\User` 模型。

    /**
     * 获得此博文的作者。
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault();
    }

您也可以通过传递数组或者使用闭包的方式，填充默认模型的属性：

    /**
     * 获得此博文的作者。
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault([
            'name' => '游客',
        ]);
    }

    /**
     * 获得此博文的作者。
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

当然，由于所有的关联还可以作为查询语句构造器使用，因此你可以使用链式调用的方式、在 `comments` 方法上再添加额外的约束条件，获得评论集合：

    $comments = App\Post::find(1)->comments()->where('title', 'foo')->first();

形如 `hasOne` 方法，您也可以在使用 `hasMany` 方法的时候，通过传递额外参数来覆盖默认的外键与本地键。

    return $this->hasMany('App\Comment', 'foreign_key');

    return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

<a name="one-to-many-inverse"></a>
### 一对多（反向）

现在，我们已经能获得一篇博文的所有评论，接着再定义一个通过评论获得所属博文的关联。这个关联是 `hasMany` 关联的反向关联，在子级模型中使用 `belongsTo` 方法定义它：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * 获得此评论所属的博文。
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
     * 获得此评论所属的博文。
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key');
    }

如果父级模型没有使用 `id` 作为主键，或者是希望用不同的字段来连接子级模型，则可以通过给 `belongsTo`方法传递第三个参数的形式指定父级数据表的自定义键：

    /**
     * 获得此评论所属的博文。
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
    }

<a name="many-to-many"></a>
### 多对多

Many-to-many relations are slightly more complicated than `hasOne` and `hasMany` relationships. An example of such a relationship is a user with many roles, where the roles are also shared by other users. For example, many users may have the role of "Admin". To define this relationship, three database tables are needed: `users`, `roles`, and `role_user`. The `role_user` table is derived from the alphabetical order of the related model names, and contains the `user_id` and `role_id` columns.

Many-to-many relationships are defined by writing a method that returns the result of the `belongsToMany` method. For example, let's define the `roles` method on our `User` model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The roles that belong to the user.
         */
        public function roles()
        {
            return $this->belongsToMany('App\Role');
        }
    }

Once the relationship is defined, you may access the user's roles using the `roles` dynamic property:

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        //
    }

Of course, like all other relationship types, you may call the `roles` method to continue chaining query constraints onto the relationship:

    $roles = App\User::find(1)->roles()->orderBy('name')->get();

As mentioned previously, to determine the table name of the relationship's joining table, Eloquent will join the two related model names in alphabetical order. However, you are free to override this convention. You may do so by passing a second argument to the `belongsToMany` method:

    return $this->belongsToMany('App\Role', 'role_user');

In addition to customizing the name of the joining table, you may also customize the column names of the keys on the table by passing additional arguments to the `belongsToMany` method. The third argument is the foreign key name of the model on which you are defining the relationship, while the fourth argument is the foreign key name of the model that you are joining to:

    return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');

#### Defining The Inverse Of The Relationship

To define the inverse of a many-to-many relationship, you simply place another call to `belongsToMany` on your related model. To continue our user roles example, let's define the `users` method on the `Role` model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\User');
        }
    }

As you can see, the relationship is defined exactly the same as its `User` counterpart, with the exception of simply referencing the `App\User` model. Since we're reusing the `belongsToMany` method, all of the usual table and key customization options are available when defining the inverse of many-to-many relationships.

#### Retrieving Intermediate Table Columns

As you have already learned, working with many-to-many relations requires the presence of an intermediate table. Eloquent provides some very helpful ways of interacting with this table. For example, let's assume our `User` object has many `Role` objects that it is related to. After accessing this relationship, we may access the intermediate table using the `pivot` attribute on the models:

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

Notice that each `Role` model we retrieve is automatically assigned a `pivot` attribute. This attribute contains a model representing the intermediate table, and may be used like any other Eloquent model.

By default, only the model keys will be present on the `pivot` object. If your pivot table contains extra attributes, you must specify them when defining the relationship:

    return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');

If you want your pivot table to have automatically maintained `created_at` and `updated_at` timestamps, use the `withTimestamps` method on the relationship definition:

    return $this->belongsToMany('App\Role')->withTimestamps();

#### Filtering Relationships Via Intermediate Table Columns

You can also filter the results returned by `belongsToMany` using the `wherePivot` and `wherePivotIn` methods when defining the relationship:

    return $this->belongsToMany('App\Role')->wherePivot('approved', 1);

    return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);

#### Defining Custom Intermediate Table Models

If you would like to define a custom model to represent the intermediate table of your relationship, you may call the `using` method when defining the relationship. All custom models used to represent intermediate tables of relationships must extend the `Illuminate\Database\Eloquent\Relations\Pivot` class. For example, we may define a `Role` which uses a custom `UserRole` pivot model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\User')->using('App\UserRole');
        }
    }

When defining the `UserRole` model, we will extend the `Pivot` class:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Relations\Pivot;

    class UserRole extends Pivot
    {
        //
    }

<a name="has-many-through"></a>
### Has Many Through

The "has-many-through" relationship provides a convenient shortcut for accessing distant relations via an intermediate relation. For example, a `Country` model might have many `Post` models through an intermediate `User` model. In this example, you could easily gather all blog posts for a given country. Let's look at the tables required to define this relationship:

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

Though `posts` does not contain a `country_id` column, the `hasManyThrough` relation provides access to a country's posts via `$country->posts`. To perform this query, Eloquent inspects the `country_id` on the intermediate `users` table. After finding the matching user IDs, they are used to query the `posts` table.

Now that we have examined the table structure for the relationship, let's define it on the `Country` model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Country extends Model
    {
        /**
         * Get all of the posts for the country.
         */
        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'App\User');
        }
    }

The first argument passed to the `hasManyThrough` method is the name of the final model we wish to access, while the second argument is the name of the intermediate model.

Typical Eloquent foreign key conventions will be used when performing the relationship's queries. If you would like to customize the keys of the relationship, you may pass them as the third and fourth arguments to the `hasManyThrough` method. The third argument is the name of the foreign key on the intermediate model. The fourth argument is the name of the foreign key on the final model. The fifth argument is the local key, while the sixth argument is the local key of the intermediate model:

    class Country extends Model
    {
        public function posts()
        {
            return $this->hasManyThrough(
                'App\Post',
                'App\User',
                'country_id', // Foreign key on users table...
                'user_id', // Foreign key on posts table...
                'id', // Local key on countries table...
                'id' // Local key on users table...
            );
        }
    }

<a name="polymorphic-relations"></a>
### Polymorphic Relations

#### Table Structure

Polymorphic relations allow a model to belong to more than one other model on a single association. For example, imagine users of your application can "comment" both posts and videos. Using polymorphic relationships, you can use a single `comments` table for both of these scenarios. First, let's examine the table structure required to build this relationship:

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

Two important columns to note are the `commentable_id` and `commentable_type` columns on the `comments` table. The `commentable_id` column will contain the ID value of the post or video, while the `commentable_type` column will contain the class name of the owning model. The `commentable_type` column is how the ORM determines which "type" of owning model to return when accessing the `commentable` relation.

#### Model Structure

Next, let's examine the model definitions needed to build this relationship:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Get all of the owning commentable models.
         */
        public function commentable()
        {
            return $this->morphTo();
        }
    }

    class Post extends Model
    {
        /**
         * Get all of the post's comments.
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

    class Video extends Model
    {
        /**
         * Get all of the video's comments.
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

#### Retrieving Polymorphic Relations

Once your database table and models are defined, you may access the relationships via your models. For example, to access all of the comments for a post, we can simply use the `comments` dynamic property:

    $post = App\Post::find(1);

    foreach ($post->comments as $comment) {
        //
    }

You may also retrieve the owner of a polymorphic relation from the polymorphic model by accessing the name of the method that performs the call to `morphTo`. In our case, that is the `commentable` method on the `Comment` model. So, we will access that method as a dynamic property:

    $comment = App\Comment::find(1);

    $commentable = $comment->commentable;

The `commentable` relation on the `Comment` model will return either a `Post` or `Video` instance, depending on which type of model owns the comment.

#### Custom Polymorphic Types

By default, Laravel will use the fully qualified class name to store the type of the related model. For instance, given the example above where a `Comment` may belong to a `Post` or a `Video`, the default `commentable_type` would be either `App\Post` or `App\Video`, respectively. However, you may wish to decouple your database from your application's internal structure. In that case, you may define a relationship "morph map" to instruct Eloquent to use a custom name for each model instead of the class name:

    use Illuminate\Database\Eloquent\Relations\Relation;

    Relation::morphMap([
        'posts' => 'App\Post',
        'videos' => 'App\Video',
    ]);

You may register the `morphMap` in the `boot` function of your `AppServiceProvider` or create a separate service provider if you wish.

<a name="many-to-many-polymorphic-relations"></a>
### Many To Many Polymorphic Relations

#### Table Structure

In addition to traditional polymorphic relations, you may also define "many-to-many" polymorphic relations. For example, a blog `Post` and `Video` model could share a polymorphic relation to a `Tag` model. Using a many-to-many polymorphic relation allows you to have a single list of unique tags that are shared across blog posts and videos. First, let's examine the table structure:

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

#### Model Structure

Next, we're ready to define the relationships on the model. The `Post` and `Video` models will both have a `tags` method that calls the `morphToMany` method on the base Eloquent class:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * Get all of the tags for the post.
         */
        public function tags()
        {
            return $this->morphToMany('App\Tag', 'taggable');
        }
    }

#### Defining The Inverse Of The Relationship

Next, on the `Tag` model, you should define a method for each of its related models. So, for this example, we will define a `posts` method and a `videos` method:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Tag extends Model
    {
        /**
         * Get all of the posts that are assigned this tag.
         */
        public function posts()
        {
            return $this->morphedByMany('App\Post', 'taggable');
        }

        /**
         * Get all of the videos that are assigned this tag.
         */
        public function videos()
        {
            return $this->morphedByMany('App\Video', 'taggable');
        }
    }

#### Retrieving The Relationship

Once your database table and models are defined, you may access the relationships via your models. For example, to access all of the tags for a post, you can simply use the `tags` dynamic property:

    $post = App\Post::find(1);

    foreach ($post->tags as $tag) {
        //
    }

You may also retrieve the owner of a polymorphic relation from the polymorphic model by accessing the name of the method that performs the call to `morphedByMany`. In our case, that is the `posts` or `videos` methods on the `Tag` model. So, you will access those methods as dynamic properties:

    $tag = App\Tag::find(1);

    foreach ($tag->videos as $video) {
        //
    }

<a name="querying-relations"></a>
## Querying Relations

Since all types of Eloquent relationships are defined via methods, you may call those methods to obtain an instance of the relationship without actually executing the relationship queries. In addition, all types of Eloquent relationships also serve as [query builders](/docs/{{version}}/queries), allowing you to continue to chain constraints onto the relationship query before finally executing the SQL against your database.

For example, imagine a blog system in which a `User` model has many associated `Post` models:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get all of the posts for the user.
         */
        public function posts()
        {
            return $this->hasMany('App\Post');
        }
    }

You may query the `posts` relationship and add additional constraints to the relationship like so:

    $user = App\User::find(1);

    $user->posts()->where('active', 1)->get();

You are able to use any of the [query builder](/docs/{{version}}/queries) methods on the relationship, so be sure to explore the query builder documentation to learn about all of the methods that are available to you.

<a name="relationship-methods-vs-dynamic-properties"></a>
### Relationship Methods Vs. Dynamic Properties

If you do not need to add additional constraints to an Eloquent relationship query, you may simply access the relationship as if it were a property. For example, continuing to use our `User` and `Post` example models, we may access all of a user's posts like so:

    $user = App\User::find(1);

    foreach ($user->posts as $post) {
        //
    }

Dynamic properties are "lazy loading", meaning they will only load their relationship data when you actually access them. Because of this, developers often use [eager loading](#eager-loading) to pre-load relationships they know will be accessed after loading the model. Eager loading provides a significant reduction in SQL queries that must be executed to load a model's relations.

<a name="querying-relationship-existence"></a>
### Querying Relationship Existence

When accessing the records for a model, you may wish to limit your results based on the existence of a relationship. For example, imagine you want to retrieve all blog posts that have at least one comment. To do so, you may pass the name of the relationship to the `has` method:

    // Retrieve all posts that have at least one comment...
    $posts = App\Post::has('comments')->get();

You may also specify an operator and count to further customize the query:

    // Retrieve all posts that have three or more comments...
    $posts = Post::has('comments', '>=', 3)->get();

Nested `has` statements may also be constructed using "dot" notation. For example, you may retrieve all posts that have at least one comment and vote:

    // Retrieve all posts that have at least one comment with votes...
    $posts = Post::has('comments.votes')->get();

If you need even more power, you may use the `whereHas` and `orWhereHas` methods to put "where" conditions on your `has` queries. These methods allow you to add customized constraints to a relationship constraint, such as checking the content of a comment:

    // Retrieve all posts with at least one comment containing words like foo%
    $posts = Post::whereHas('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

<a name="querying-relationship-absence"></a>
### Querying Relationship Absence

When accessing the records for a model, you may wish to limit your results based on the absence of a relationship. For example, imagine you want to retrieve all blog posts that **don't** have any comments. To do so, you may pass the name of the relationship to the `doesntHave` method:

    $posts = App\Post::doesntHave('comments')->get();

If you need even more power, you may use the `whereDoesntHave` method to put "where" conditions on your `doesntHave` queries. This method allows you to add customized constraints to a relationship constraint, such as checking the content of a comment:

    $posts = Post::whereDoesntHave('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

<a name="counting-related-models"></a>
### Counting Related Models

If you want to count the number of results from a relationship without actually loading them you may use the `withCount` method, which will place a `{relation}_count` column on your resulting models. For example:

    $posts = App\Post::withCount('comments')->get();

    foreach ($posts as $post) {
        echo $post->comments_count;
    }

You may add the "counts" for multiple relations as well as add constraints to the queries:

    $posts = Post::withCount(['votes', 'comments' => function ($query) {
        $query->where('content', 'like', 'foo%');
    }])->get();

    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;

You may also alias the relationship count result, allowing multiple counts on the same relationship:

    $posts = Post::withCount([
        'comments',
        'comments as pending_comments_count' => function ($query) {
            $query->where('approved', false);
        }
    ])->get();

    echo $posts[0]->comments_count;

    echo $posts[0]->pending_comments_count;

<a name="eager-loading"></a>
## Eager Loading

When accessing Eloquent relationships as properties, the relationship data is "lazy loaded". This means the relationship data is not actually loaded until you first access the property. However, Eloquent can "eager load" relationships at the time you query the parent model. Eager loading alleviates the N + 1 query problem. To illustrate the N + 1 query problem, consider a `Book` model that is related to `Author`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Book extends Model
    {
        /**
         * Get the author that wrote the book.
         */
        public function author()
        {
            return $this->belongsTo('App\Author');
        }
    }

Now, let's retrieve all books and their authors:

    $books = App\Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

This loop will execute 1 query to retrieve all of the books on the table, then another query for each book to retrieve the author. So, if we have 25 books, this loop would run 26 queries: 1 for the original book, and 25 additional queries to retrieve the author of each book.

Thankfully, we can use eager loading to reduce this operation to just 2 queries. When querying, you may specify which relationships should be eager loaded using the `with` method:

    $books = App\Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

For this operation, only two queries will be executed:

    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)

#### Eager Loading Multiple Relationships

Sometimes you may need to eager load several different relationships in a single operation. To do so, just pass additional arguments to the `with` method:

    $books = App\Book::with(['author', 'publisher'])->get();

#### Nested Eager Loading

To eager load nested relationships, you may use "dot" syntax. For example, let's eager load all of the book's authors and all of the author's personal contacts in one Eloquent statement:

    $books = App\Book::with('author.contacts')->get();

<a name="constraining-eager-loads"></a>
### Constraining Eager Loads

Sometimes you may wish to eager load a relationship, but also specify additional query constraints for the eager loading query. Here's an example:

    $users = App\User::with(['posts' => function ($query) {
        $query->where('title', 'like', '%first%');
    }])->get();

In this example, Eloquent will only eager load posts where the post's `title` column contains the word `first`. Of course, you may call other [query builder](/docs/{{version}}/queries) methods to further customize the eager loading operation:

    $users = App\User::with(['posts' => function ($query) {
        $query->orderBy('created_at', 'desc');
    }])->get();

<a name="lazy-eager-loading"></a>
### Lazy Eager Loading

Sometimes you may need to eager load a relationship after the parent model has already been retrieved. For example, this may be useful if you need to dynamically decide whether to load related models:

    $books = App\Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

If you need to set additional query constraints on the eager loading query, you may pass an array keyed by the relationships you wish to load. The array values should be `Closure` instances which receive the query instance:

    $books->load(['author' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }]);

<a name="inserting-and-updating-related-models"></a>
## Inserting & Updating Related Models

<a name="the-save-method"></a>
### The Save Method

Eloquent provides convenient methods for adding new models to relationships. For example, perhaps you need to insert a new `Comment` for a `Post` model. Instead of manually setting the `post_id` attribute on the `Comment`, you may insert the `Comment` directly from the relationship's `save` method:

    $comment = new App\Comment(['message' => 'A new comment.']);

    $post = App\Post::find(1);

    $post->comments()->save($comment);

Notice that we did not access the `comments` relationship as a dynamic property. Instead, we called the `comments` method to obtain an instance of the relationship. The `save` method will automatically add the appropriate `post_id` value to the new `Comment` model.

If you need to save multiple related models, you may use the `saveMany` method:

    $post = App\Post::find(1);

    $post->comments()->saveMany([
        new App\Comment(['message' => 'A new comment.']),
        new App\Comment(['message' => 'Another comment.']),
    ]);

<a name="the-create-method"></a>
### The Create Method

In addition to the `save` and `saveMany` methods, you may also use the `create` method, which accepts an array of attributes, creates a model, and inserts it into the database. Again, the difference between `save` and `create` is that `save` accepts a full Eloquent model instance while `create` accepts a plain PHP `array`:

    $post = App\Post::find(1);

    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

> {tip} Before using the `create` method, be sure to review the documentation on attribute [mass assignment](/docs/{{version}}/eloquent#mass-assignment).

You may use the `createMany` method to create multiple related models:

    $post = App\Post::find(1);

    $post->comments()->createMany([
        [
            'message' => 'A new comment.',
        ],
        [
            'message' => 'Another new comment.',
        ],
    ]);

<a name="updating-belongs-to-relationships"></a>
### Belongs To Relationships

When updating a `belongsTo` relationship, you may use the `associate` method. This method will set the foreign key on the child model:

    $account = App\Account::find(10);

    $user->account()->associate($account);

    $user->save();

When removing a `belongsTo` relationship, you may use the `dissociate` method. This method will set the relationship's foreign key to `null`:

    $user->account()->dissociate();

    $user->save();

<a name="updating-many-to-many-relationships"></a>
### Many To Many Relationships

#### Attaching / Detaching

Eloquent also provides a few additional helper methods to make working with related models more convenient. For example, let's imagine a user can have many roles and a role can have many users. To attach a role to a user by inserting a record in the intermediate table that joins the models, use the `attach` method:

    $user = App\User::find(1);

    $user->roles()->attach($roleId);

When attaching a relationship to a model, you may also pass an array of additional data to be inserted into the intermediate table:

    $user->roles()->attach($roleId, ['expires' => $expires]);

Of course, sometimes it may be necessary to remove a role from a user. To remove a many-to-many relationship record, use the `detach` method. The `detach` method will remove the appropriate record out of the intermediate table; however, both models will remain in the database:

    // Detach a single role from the user...
    $user->roles()->detach($roleId);

    // Detach all roles from the user...
    $user->roles()->detach();

For convenience, `attach` and `detach` also accept arrays of IDs as input:

    $user = App\User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([
        1 => ['expires' => $expires],
        2 => ['expires' => $expires]
    ]);

#### Syncing Associations

You may also use the `sync` method to construct many-to-many associations. The `sync` method accepts an array of IDs to place on the intermediate table. Any IDs that are not in the given array will be removed from the intermediate table. So, after this operation is complete, only the IDs in the given array will exist in the intermediate table:

    $user->roles()->sync([1, 2, 3]);

You may also pass additional intermediate table values with the IDs:

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);

If you do not want to detach existing IDs, you may use the `syncWithoutDetaching` method:

    $user->roles()->syncWithoutDetaching([1, 2, 3]);

#### Toggling Associations

The many-to-many relationship also provides a `toggle` method which "toggles" the attachment status of the given IDs. If the given ID is currently attached, it will be detached. Likewise, if it is currently detached, it will be attached:

    $user->roles()->toggle([1, 2, 3]);

#### Saving Additional Data On A Pivot Table

When working with a many-to-many relationship, the `save` method accepts an array of additional intermediate table attributes as its second argument:

    App\User::find(1)->roles()->save($role, ['expires' => $expires]);

#### Updating A Record On A Pivot Table

If you need to update an existing row in your pivot table, you may use `updateExistingPivot` method. This method accepts the pivot record foreign key and an array of attributes to update:

    $user = App\User::find(1);

    $user->roles()->updateExistingPivot($roleId, $attributes);

<a name="touching-parent-timestamps"></a>
## Touching Parent Timestamps

When a model `belongsTo` or `belongsToMany` another model, such as a `Comment` which belongs to a `Post`, it is sometimes helpful to update the parent's timestamp when the child model is updated. For example, when a `Comment` model is updated, you may want to automatically "touch" the `updated_at` timestamp of the owning `Post`. Eloquent makes it easy. Just add a `touches` property containing the names of the relationships to the child model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * All of the relationships to be touched.
         *
         * @var array
         */
        protected $touches = ['post'];

        /**
         * Get the post that the comment belongs to.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

Now, when you update a `Comment`, the owning `Post` will have its `updated_at` column updated as well, making it more convenient to know when to invalidate a cache of the `Post` model:

    $comment = App\Comment::find(1);

    $comment->text = 'Edit to this comment!';

    $comment->save();
