# Laravel 的视图功能

- [创建视图](#creating-views)
- [传递数据到视图](#passing-data-to-views)
    - [共享数据给所有视图](#sharing-data-with-all-views)
- [视图合成器](#view-composers)

<a name="creating-views"></a>
## 创建视图

视图包含你应用程序的HTML内容，并且能够将你的控制器层逻辑（或应用层逻辑）与展现层逻辑分开。视图文件存放于 `resources/views`目录下 ，一个简单的视图示例如下：

    <!-- 此视图文件位置：resources/views/greeting.blade.php -->

    <html>
        <body>
            <h1>Hello, {{ $name }}</h1>
        </body>
    </html>

该视图文件位于 `resources/views/greeting.blade.php` ，我们可以通过全局函数 `view` 来使用这个它，比如：

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

如你所见，`view` 函数中，传入的第一个参数对应着 `resources/views` 目录中视图文件的文件名，第二个参数是一个数组，数组中的数据用于在视图文件中使用。在示例中，我们将 `name` 变量传递到视图中，并在视图中使用 [Blade 模板语言](/docs/{{version}}/blade) 打印出来。

当然，视图文件也嵌套在 `resources/views` 目录的子目录中，英文句点 `.` 可以用来引用嵌套的视图。例如，一个位于 `resources/views/admin/profile.blade.php`的视图 ，可以这样引用它：

    return view('admin.profile', $data);

#### 判断一个视图文件是否存在

如果需要判断一个视图文件是否存在，你可以使用 `View` Facade 上的 `exists` 方法来判定，如果视图文件存在，该方法会返回 `true` ：

    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        //
    }

<a name="passing-data-to-views"></a>
## 传递数据至视图

如上述例子所示，你可以使用数组将数据传递到视图：

    return view('greetings', ['name' => 'Victoria']);

当用这种方式传递数据时，第二个参数（ `$data` ）必须是键\值对数组。在视图文件中，你可以通过对应的关键字（ `$key` ）取用相应的数据值，例如 `<?php echo $key; ?>`。如果只需要传递单个数据片段而非整个数组到视图，您可以使用 `with` 方法：

    return view('greeting')->with('name', 'Victoria');

<a name="sharing-data-with-all-views"></a>
#### 把数据共享给所有视图

有时，您可能需要共享一段数据给应用程序的所有视图，您可以使用`View` Facade 的 `share` 方法。通常需要将所有 `share` 方法的调用代码放到 [服务提供者](/docs/{{version}}/providers) 的 `boot` 方法中，此时你可以选择使用 `AppServiceProvider` 或创建独立的 [服务提供者](/docs/{{version}}/providers) 。示例代码如下：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 启动任意应用服务
         *
         * @return void
         */
        public function boot()
        {
            View::share('key', 'value');
        }

        /**
         * 注册服务提供者
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="view-composers"></a>
## 视图合成器

视图合成器是在一个视图被渲染时调用的一些回调或者类方法。如果你需要在某些视图被渲染时绑定一些数据上去，那么视图合成器就是你的的不二之选，另外他还可以帮你将这些绑定逻辑整理到特定的位置。

在下面这个例子中，我们会在一个 [服务提供者](/docs/{{version}}/providers) 中注册一些视图合成器。同时使用 `View` Facade 来访问 `Illuminate\Contracts\View\Factory` contract 的底层实现。注意：Laravel 没有存放视图合成器的默认目录，但你可以根据自己的喜好来重新组织，例如：`App\Http\ViewComposers`。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * 在容器中注册绑定
         *
         * @return void
         */
        public function boot()
        {
            // 使用基于类的合成器...
            View::composer(
                'profile', 'App\Http\ViewComposers\ProfileComposer'
            );

            // 使用基于闭包的合成器...
            View::composer('dashboard', function ($view) {
                //
            });
        }

        /**
         * 注册服务器提供者
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

> {note} 注意，如果你创建了新的一个 [服务提供者](/docs/{{version}}/providers) 来存放你视图合成器的注册项，那么你需要将这个 [服务提供者](/docs/{{version}}/providers) 添加到配置文件 `config/app.php` 的 `providers` 数组中。

到此我们已经注册了上面的视图合成器，效果是每次 `profile` 视图渲染时，都会执行 `ProfileComposer@compose` 方法。那么下面我们来定义这个合成器类吧。

    <?php

    namespace App\Http\ViewComposers;

    use Illuminate\View\View;
    use App\Repositories\UserRepository;

    class ProfileComposer
    {
        /**
         * 实现用户仓库
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * 创建一个新的配置文件合成器
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            // 依赖关系由服务容器自动解析...
            $this->users = $users;
        }

        /**
         * 将数据绑定到视图。
         *
         * @param  View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }

每当视图渲染时，该合成器的 `compose` 方法都会被调用，并且传入一个 `Illuminate\View\View` 实例作为参数，在这个过程中，你可以使用 `with` 方法绑定数据到目标视图。

> {tip} 所有的视图合成器都会由 [服务容器](/docs/{{version}}/container) 来解析，所以你可以在合成器的构造函数中使用类型提示来注入需要的任何依赖。

#### 将视图合成器附加到多个视图

通过将目标视图文件数组作为第一个参数传入 `composer` 方法，你可以把一个视图合成器同时附加到多个视图。

    View::composer(
        ['profile', 'dashboard'],
        'App\Http\ViewComposers\MyViewComposer'
    );

`composer` 方法同时也接受通配符 `*` ，可以让你将一个视图合成器一次性绑定到所有的视图：

    View::composer('*', function ($view) {
        //
    });

#### 视图构造器

视图 **构造器** 和视图合成器非常相似。不同之处在于：视图构造器在视图实例化时执行，而视图合成器在视图渲染时执行。要注册一个视图塑造器，可以使用 `creator`方法，如下：

    View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');



--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org