# 视图

- [创建视图](#creating-views)
- [传递数据到视图](#passing-data-to-views)
    - [把数据共享给所有视图](#sharing-data-with-all-views)
- [视图合成器](#view-composers)

<a name="creating-views"></a>
## 创建视图

视图的用途是用来存放应用程序中 HTML 内容，并且能够将你的控制器层（或应用逻辑层）与展现层分开。视图文件目录为 `resources/views` ，示例视图如下：

    <!-- 此视图文件位置：resources/views/greeting.php -->

    <html>
        <body>
            <h1>Hello, {{ $name }}</h1>
        </body>
    </html>

上述视图文件位置为 `resources/views/greeting.php` ，我们可以通过全局函数 `view` 来使用这个视图，如下：

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

如你所见，`view` 函数中，第一个参数即 `resources/views` 目录中视图文件的文件名，第二个参数是一个数组，数组中的数据可以直接在视图文件中使用。在上面示例中，我们将 `name` 变量传递到了视图中，并在视图中使用 [Blade 模板语言](/docs/{{version}}/blade) 打印出来。

当然，视图文件也可能存放在 `resources/views` 的子目录中，你可以使用英文句点 `.` 来引用深层子目录中的视图文件。例如，一个视图的位置为 `resources/views/admin/profile.php` ，使用示例如下：

    return view('admin.profile', $data);

#### 判断视图文件是否存在

如果需要测试一个视图文件是否存在，你可以使用 `View` Facade 上的 `exists` 方法来判定，如果测试的视图文件存在，则返回值为 `true` ：

    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        //
    }

<a name="passing-data-to-views"></a>
## 传递数据到视图

如上述例子中，你可以使用数组将数据传递到视图文件：

    return view('greetings', ['name' => 'Victoria']);

当使用上面方式传递数据时，第二个参数（ `$data` ）必须是键值对数组。在视图文件中，你可以通过对应的关键字（ `$key` ）取用相应的数据值，例如 `<?php echo $key; ?>`。如果只需要传递特定数据而非一个臃肿的数组到视图文件，可以使用 `with` 辅助函数，示例如下：

    return view('greeting')->with('name', 'Victoria');

<a name="sharing-data-with-all-views"></a>
#### 把数据共享给所有视图

有时候可能需要共享特定的数据给应用程序中所有的视图，那这时候你需要 `View` Facade 的 `share` 方法。通常需要将所有 `share` 方法的调用代码放到 [服务提供者](/docs/{{version}}/providers) 的 `boot` 方法中，此时你可以选择使用 `AppServiceProvider` 或创建独立的 [服务提供者](/docs/{{version}}/providers) 。示例代码如下：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            View::share('key', 'value');
        }

        /**
         * Register the service provider.
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

视图合成器是在视图渲染时调用的一些回调或者类方法。如果你需要在某些视图渲染时绑定一些数据上去，那么视图合成器就是你的的不二之选，另外他还可以帮你将这些绑定逻辑整理到特定的位置。

下面例子中，我们会在一个 [服务提供者](/docs/{{version}}/providers) 中注册一些视图合成器。同时使用 `View` Facade 来访问 `Illuminate\Contracts\View\Factory` contract 的底层实现。注意：Laravel 没有存放视图合成器的默认目录，但你可以根据自己的喜好来重新组织，例如：`App\Http\ViewComposers`。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function boot()
        {
            // Using class based composers...
            View::composer(
                'profile', 'App\Http\ViewComposers\ProfileComposer'
            );

            // Using Closure based composers...
            View::composer('dashboard', function ($view) {
                //
            });
        }

        /**
         * Register the service provider.
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
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new profile composer.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            // Dependencies automatically resolved by service container...
            $this->users = $users;
        }

        /**
         * Bind data to the view.
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

#### 视图塑造器

视图 **塑造器** 和视图合成器非常相似。不同之处在于：视图塑造器在视图实例化时执行，而视图合成器在视图渲染时执行。如下，可以使用 `creator` 方法来注册一个视图塑造器：

    View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');
