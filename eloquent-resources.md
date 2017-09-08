# Eloquent: API 资源

- [简介](#introduction)
- [生成资源](#generating-resources)
- [概念综述](#concept-overview)
- [编写资源](#writing-resources)
    - [数据包裹](#data-wrapping)
    - [分页](#pagination)
    - [条件属性](#conditional-attributes)
    - [条件关联](#conditional-relationships)
    - [添加元数据](#adding-meta-data)
- [资源响应](#resource-responses)

<a name="introduction"></a>
## 简介

当构建 API 时，你往往需要一个转换层来联结你的 Eloquent 模型和实际返回给用户的 JSON 响应。 Laravel 的资源类能够让你以更直观简便的方式将模型和模型集合转化成 JSON 。

<a name="generating-resources"></a>
## 生成资源

你可以使用 `make:resource` Artisan 命令来生成资源类。默认情况下生成的资源都会被放置在框架 `app/Http/Resources` 文件夹下。资源继承 `Illuminate\Http\Resources\Json\Resource` 类：

    php artisan make:resource User

#### 资源集合

除了生成资源转换单个模型外，你还可以生成资源集合用来转换模型的集合。这允许你在响应中包含与给定资源相关的链接与其他元信息。

你需要在生成资源时添加 `--collection` 标志以生成一个资源集合。你也可以直接在资源的名称中包含 `Collection` 向 Laravel 表示应该生成一个资源集合。资源集合继承 `Illuminate\Http\Resources\Json\ResourceCollection` 类：

    php artisan make:resource Users --collection

    php artisan make:resource UserCollection

<a name="concept-overview"></a>
## 概念综述

> {tip} 这是对资源和资源集合的概述。强烈建议你阅读本文档的其他部分，以深入了解如何更好地自定义和使用资源。

在深入了解如何定制化编写你的资源之前，让我们先来看看在 Laravel 中如何使用资源。一个资源类表示一个单一模型需要被转换成 JSON 格式。例如，现在我们有一个简单的 `User` 资源类：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class User extends Resource
    {
        /**
         * 将资源转换成数组。
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

每一个资源类都定义了一个 `toArray` 方法，在发送响应时它会返回应该被转化成 JSON 的属性数组。注意在这里我们可以直接使用 `$this` 变量来访问模型属性。这是因为资源类将自动代理属性和方法到底层模型以方便访问。你可以在路由或控制器中返回已定义的资源：

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

### 资源集合

你可以在路由或控制器中使用 `collection` 方法来创建资源实例，以返回多个资源的集合或分页响应：

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

使用如上方法，你将不能添加任何附加的元数据和集合一起返回。如果你需要自定义资源集合响应，你需要创建一个资源集合来返回多个模型的集合：

    php artisan make:resource UserCollection

你可以轻松的在已生成的资源集合类中定义任何你想在响应中返回的元数据：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * 将资源集合转换成数组。
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

你可以在路由或控制器中返回已定义的资源集合：

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="writing-resources"></a>
## 编写资源

> {tip} 如果你还没有阅读过 [概念综述](#concept-overview) , 那么强烈建议你在继续使用本文档前阅读这个部分。

从本质上来说，资源的作用很简单。它们只需要将一个给定的模型转换成一个数组。所以每一个资源都包含一个 `toArray` 方法用来将你的模型属性转换成可以返回给用户的 API 友好的数组：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class User extends Resource
    {
        /**
         * 将资源转换成数组。
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

你可以在路由或控制器中返回已定义的资源：

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

#### 关联

如果你希望在响应中包含关联资源，你只需要简单的将它们添加到 `toArray` 方法返回的数组中。在下面这个例子里，我们将使用 `Post` 资源的 `collection` 方法将用户的文章添加到资源响应中：

    /**
     * 将资源转换成数组。
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => Post::collection($this->posts),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

> {tip} 如果你只想在关联已经加载时添加关联资源，请查看文档 [条件关联](#conditional-relationships) 。

#### 资源集合

资源是将单个模型转换成数组，而资源集合是将多个模型的集合转换成数组。所有的资源都提供了一个 `collection` 方法来生成一个 「临时」 资源集合，所以你没有必要为每一个模型类型都编写一个资源集合类：

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

如果你需要自定义返回集合的元数据，则需要定义一个资源集合：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * 将资源集合转换成数组。
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

和单个资源一样，你可以在路由或控制器中直接返回资源集合：

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="data-wrapping"></a>
### 数据包裹

默认情况下，当资源响应被转换成 JSON 时，顶层资源将会被包裹在 `data` 键中。因此一个典型的资源集合响应如下所示：

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ]
    }

你可以使用资源基类的 `withoutWrapping` 方法来禁用顶层资源的包裹。你应该在 `AppServiceProvider` 或其他在程序每一个请求中都会被加载的 [服务提供者](/docs/{{version}}/providers) 中调用此方法：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Http\Resources\Json\Resource;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 在注册后进行服务的启动。
         *
         * @return void
         */
        public function boot()
        {
            Resource::withoutWrapping();
        }

        /**
         * 在容器中注册绑定。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

> {note}  `withoutWrapping` 方法只会禁用顶层资源的包裹，不会删除你手动添加到资源集合中的 `data` 键。

### 包裹嵌套资源

你有绝对的自由决定你的资源关联如何被包裹。如果你希望无论怎样嵌套，都将所有资源集合包裹在 `data` 键中，那么你需要为每个资源都定义一个资源集合类，并将返回的集合包裹在 `data` 键中。

当然，你可能会担心这样顶层资源将会被包裹在两个 `data` 键中。请放心， Laravel 将永远不会让你的资源被双层包裹，因此你不必担心被转换的资源集合会被多重嵌套：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class CommentsCollection extends ResourceCollection
    {
        /**
         * 将资源集合转换成数组。
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return ['data' => $this->collection];
        }
    }

### 数据包裹和分页

当在资源响应中返回分页集合时，即使你调用了 `withoutWrapping` 方法， Laravel 也会将你的资源数据包裹在 `data` 键中。这是因为分页响应中总会有 `meta` 和 `links` 键包含着分页状态信息：

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }

<a name="pagination"></a>
### 分页

你可以将分页实例传递给资源的 `collection` 方法或者自定义的资源集合：

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::paginate());
    });

分页响应中总有 `meta` 和 `links` 键包含着分页状态信息：

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }

<a name="conditional-attributes"></a>
### 条件属性

有些时候，你可能希望在给定条件满足时添加属性到资源响应里。例如，你可能希望如果当前用户是 「管理员」 时添加某个值到资源响应中。在这种情况下 Laravel 提供了一些辅助方法来帮助你解决问题。 `when` 方法可以被用来有条件地向资源响应添加属性：

    /**
     * 将资源转换成数组。
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'secret' => $this->when($this->isAdmin(), 'secret-value'),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

在上面这个例子中，只有当 `$this->isAdmin()` 方法返回 `true` 时， `secret` 键才会最终在资源响应中被返回。如果该方法返回 `false` ，则 `secret` 键将会在资源响应被发送给客户端之前被删除。 `when` 方法可以使你避免使用条件语句拼接数组，转而用更优雅的方式来编写你的资源。

 `when` 方法也接受闭包作为其第二个参数，只有在给定条件为 `true` 时，才从闭包中计算返回的值：

    'secret' => $this->when($this->isAdmin(), function () {
        return 'secret-value';
    }),

> {tip} 记住，你在资源上调用的方法将被代理到底层模型实例。所以在这种情况下，你调用的 `isAdmin` 方法实际上是调用最初传递给资源的 Eloquent 模型上的方法。

#### 有条件地合并数据

有些时候，你可能希望在给定条件满足时添加多个属性到资源响应里。在这种情况下，你可以使用 `mergeWhen` 方法在给定的条件为 `true` 时将多个属性添加到响应中：

    /**
     * 将资源转换成数组。
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            $this->mergeWhen($this->isAdmin(), [
                'first-secret' => 'value',
                'second-secret' => 'value',
            ]),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

同理，如果给定的条件为 `false` 时，则这些属性将会在资源响应被发送给客户端之前被删除。

> {note} 在混合字符串和数字键的数组中你不应该使用 `mergeWhen` 方法。此外，它不应该在不按顺序排列的数字键的数组中被使用。

<a name="conditional-relationships"></a>
### 条件关联

除了有条件地添加属性之外，你还可以根据模型关联是否已加载来有条件的在你的资源响应中包含关联。这允许你在控制器中决定加载哪些模型关联，这样你的资源可以在模型关联被加载后才添加它们。

这样做可以避免在你的资源中出现 「N+1」 查询问题。你应该使用 `whenLoaded` 方法来有条件的加载关联。为了避免加载不必要的关联，此方法接受关联的名称而不是关联本身作为其参数：

    /**
     * 将资源转换成数组。
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => Post::collection($this->whenLoaded('posts')),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

在上面这个例子中，如果关联没有被加载，则 `posts` 键将会在资源响应被发送给客户端之前被删除。

#### 条件中间表信息

除了在你的资源响应中有条件地包含关联外，你还可以使用 `whenPivotLoaded` 方法有条件地从多对多关联的中间表中添加数据。 `whenPivotLoaded` 方法第一个参数为中间表的名称。第二个参数应为当模型中间表信息可用时返回要添加数据的闭包：

    /**
     * 将资源转换成数组。
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoaded('role_users', function () {
                return $this->pivot->expires_at;
            }),
        ];
    }

<a name="adding-meta-data"></a>
### 添加元数据

一些 JSON API 标准需要你在资源和资源集合响应中添加元数据。这通常包括资源或相关资源的 `links` ，或一些关于资源本身的元数据。如果你需要返回有关资源的其他元数据，只需要将它们包含在 `toArray` 方法中即可。例如在转换资源集合时你可能需要添加 `links` 信息：

    /**
     * 将资源转换成数组。
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }

当添加额外元数据到你的资源中时，你不必担心会覆盖 Laravel 在返回分页响应时自动添加的 `links` 或 `meta` 键。你添加的任何其他 `links` 会与分页响应添加的 `links` 相合并。

#### 顶层元数据

有时候你可能希望当资源被作为顶层资源返回时添加某些元数据到资源响应中。这通常包括整个响应的元信息。你可以在资源类中添加 `with` 方法来定义元数据。此方法应返回一个元数据数组，当资源被作为顶层资源渲染时，这个数组将会被包含在资源响应中：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * 将资源集合转换成数组。
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return parent::toArray($request);
        }

        /**
         * 返回应该和资源一起返回的其他数据数组。
         *
         * @param \Illuminate\Http\Request  $request
         * @return array
         */
        public function with($request)
        {
            return [
                'meta' => [
                    'key' => 'value',
                ],
            ];
        }
    }

#### 构造资源时添加元数据

你还可以在路由或控制器中构造资源实例时添加顶层数据。所有资源都可以使用 `additional` 方法来接受应该被添加到资源响应中的数据数组：

    return (new UserCollection(User::all()->load('roles')))
                    ->additional(['meta' => [
                        'key' => 'value',
                    ]]);

<a name="resource-responses"></a>
## 资源响应

就像你已经知道那样，资源可以直接在路由和控制器中被返回：

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

但是有些时候你可能需要自定义发送给客户端的 HTTP 响应。你有两种选择。第一，你可以在资源上链式调用 `response` 方法。此方法将返回 `Illuminate\Http\Response` 实例，允许你自定义响应头信息：

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return (new UserResource(User::find(1)))
                    ->response()
                    ->header('X-Value', 'True');
    });

或者你也可以在资源中定义一个 `withResponse` 方法。此方法将会在资源被作为顶层资源在响应中返回时被调用：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class User extends Resource
    {
        /**
         * 将资源转换成数组。
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
            ];
        }

        /**
         * 自定义资源响应。
         *
         * @param  \Illuminate\Http\Request
         * @param  \Illuminate\Http\Response
         * @return void
         */
        public function withResponse($request, $response)
        {
            $response->header('X-Value', 'True');
        }
    }

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@young](https://laravel-china.org/users/4236)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/4236_1461228940.png?imageView2/1/w/200/h/200">  |  翻译  | 18届应届生 [求工作](https://xiayang.me/hire-me)，Laravel、PHP、web后端开发方向 |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org