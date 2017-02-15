# Laravel 的资源任务编译器 Laravel Mix

- [简介](#introduction)
- [安装与配置](#installation)
- [运行 Mix](#running-mix)
- [使用样式](#working-with-stylesheets)
    - [Less](#less)
    - [Sass](#sass)
    - [纯 CSS](#plain-css)
    - [资源地图](#css-source-maps)
- [使用脚本](#working-with-scripts)
   - [代码分割](#code-splitting)
   - [自定义 Webpack 配置](#custom-webpack-configuration)
- [复制文件与目录](#copying-files-and-directories)
- [版本与缓存清除](#versioning-and-cache-busting)
- [通知](#notifications)

<a name="introduction"></a>
## 简介

Laravel Mix 提供了流畅的 API，让你能够在你的 Laravel 应用程序中定义 Webpack 的构建步骤。而且可以让你的 Laravel 应用使用一系列常见的 CSS 与 JavaScript 预处理器。使用简单的链式调用，你能够流畅地定义开发流程，例如：

	mix.js('resources/assets/js/app.js', 'public/js')
		.sass('resources/assets/sass/app.scss', 'public/css');

如果你曾经对于上手 Webpack 及编译资源文件感到困惑，那么你将会爱上 Laravel Mix，不过 Laravel 并不强迫你使用 Mix，你可以自由的选用你喜欢的自动化开发流程工具，或者完全不使用此类工具。

<a name="installation"></a>
## 安装及配置

#### 安装 Node

在开始使用 Mix 之前，你必须先确保你的机器上有安装 Node.js 和 npm。

    node -v
    npm -v

默认情况下，Laravel Homestead 包含了你所需的一切；如果你没有使用 Vagrant，那么你可以从 [Node 的官方下载页面](http://nodejs.org/en/download/) 下载可视化安装工具来安装 Node 与 NPM 的最新版本。

#### Laravel Mix

最后的步骤就是安装 Laravel Mix。在每一份新安装的 Laravel 代码里，你会发现根目录有个名为 `package.json` 的文件。默认的 `package.json` 已经包含了你需要开始工程的所有文件。想像它就如同你的 `composer.json` 文件，只不过它定义的是 Node 的依赖扩展包，而不是 PHP 的。你可以使用以下的命令安装依赖扩展包：

    npm install

如果你是在 Windows 系统上或在 Windows 主机系统上运行 VM 进行开发，你需要在运行 `npm install` 命令时将 `--no-bin-links` 开启：

    npm install --no-bin-links

<a name="running-mix"></a>
## 运行 Mix

Mix 是一个 [Webpack](https://webpack.js.org) 配置层之上，所以要运行你的 Mix 任务，只需执行 NPM 脚本，包括默认的 Laravel `package.json` 文件。

Mix 是创建于 [Gulp](http://gulpjs.com) 之上，所以要运行你的 Mix 任务，只需要在命令行运行 `gulp` 命令。在命令增加 `--production` 标示会告知 Mix 压缩你的 CSS 及 JavaScript 文件：

    // 运行所有 Mix 任务...
    npm run dev

    // 运行所有 Mix 任务和压缩资源输出
    npm run production

#### 监控资源文件修改

`npm run watch` 会在你的终端里持续运行，监控资源文件是否有发生改变。在 `watch` 命令运行的情况下，一旦资源文件发生变化，Webpack 会自动重新编译：

    npm run watch

<a name="working-with-stylesheets"></a>
## 使用样式

`webpack.mix.js` 文件是编译所有资源文件的入口点。而且这是一个简单的配置去使用 Webpack 。Mix 任务可以使用链式调用的写法来定义你的资源文件该如何进行编译。

<a name="less"></a>
### Less

要将 [Less](http://lesscss.org/) 编译为 CSS，你可以使用 `less` 方法。我们可以编译我们的主文件 `app.less` 放置到 `public/css/app.css`。

	mix.less('resources/assets/less/app.less', 'public/css');

多次调用 `less` 方法可用于编译多个文件：

    mix.less('resources/assets/less/app.less', 'public/css')
       .less('resources/assets/less/admin.less', 'public/css');

如果你想自定义编译后的 CSS 文件名，可以传递一个完整的文件路径作为 `less` 方法 的第二个参数：

    mix.less('resources/assets/less/app.less', 'public/stylesheets/styles.css');

<a name="sass"></a>
### Sass

`sass` 方法让你能编译 [Sass](http://sass-lang.com/) 至 CSS。你可以使用此方法：

	mix.sass('resources/assets/sass/app.scss', 'public/css');

同样的，如同 `less` 方法，你可以编译多个 Sass 文件至各自的 CSS 文件，甚至可以自定义生成的 CSS 的输出目录：

	mix.sass('resources/assets/sass/app.sass', 'public/css')
       .sass('resources/assets/sass/admin.sass', 'public/css/admin');

<a name="plain-css"></a>
### 纯 CSS

如果你只是想将一些纯 CSS 样式合并成单个的文件，你可以使用 `combine` 方法。此方法也支持连接 JavaScript 文件

    mix.combine([
        'public/css/vendor/normalize.css',
        'public/css/vendor/videojs.css'
    ], 'public/css/all.css');

<a name="css-source-maps"></a>
### 资源地图

尽管 Source Maps 默认被关闭，但可以在 `webpack.mix.js` 文件中通过调用 `mix.sourceMaps()` 方法来激活。尽管它伴随着编译/性能的开销，但当使用编译资源时将为你的浏览器的开发工具提供额外的调试信息。

    mix.js('resources/assets/js/app.js', 'public/js')
       .sourceMaps();

<a name="working-with-scripts"></a>
## 使用 JavaScript

Mix 也提供了一些函数来帮助你使用 JavaScript 文件，像是编译 ECMAScript 2015、模块绑定、压缩、及简单的串联纯 JavaScript 文件。更好的是这一切工作无缝衔接，并不需要很多的自定义配置：

    mix.js('resources/assets/js/app.js', 'public/js');

有了这一行代码，你现在可以利用：
<div class="content-list" markdown="1">
- ECMAScript 2015 语法.
- 编译 `.vue` 文件.
- 针对生产环境压缩代码.
</div>

<a name="code-splitting"></a>
### 代码分割

与你的供应商库绑定所有指定应用的 JavaScript，其中一个潜在的缺点是它使长期缓存更难。比如，对应用程序代码的一次更新将迫使浏览器重新下载所有的供应商库，即使它们没有更改。

如果你想要频繁升级你应用的 JavaScript，你应该考虑将所有供应商库提取到它们的文件中。通过这种方式改变的应用程序代码不会影响 `vendor.js` 的文件缓存。Mix 的 `extract` 方法让这变得轻而易举：

    mix.js('resources/assets/js/app.js', 'public/js')
       .extract(['vue'])

`extract` 方法接受数组中所有的库或模块，提取到一个 `vendor.js` 文件。使用上面的代码段为例，Mix 会生成接下来的文件：

<div class="content-list" markdown="1">
- `public/js/manifest.js`: *Webpack 显示运行时*
- `public/js/vendor.js`: *依赖库*
- `public/js/app.js`: *应用代码*
</div>

为了避免 JavaScript 错误，一定要按照正确的顺序加载这些文件：

    <script src="/js/manifest.js"></script>
    <script src="/js/vendor.js"></script>
    <script src="/js/app.js"></script>

<a name="custom-webpack-configuration"></a>
### 自定义 Webpack 配置

在后台，Laravel Mix 引用了一个预配置的 `webpack.config.js` 文件让你尽可能快地运行起来。有时，你可能需要手动地修改这个文件。你可能有一个特殊的加载程序或插件需要被引用，或者你也许更喜欢使用 Stylus 来代替 Sass。这种情况下，有两个选择：

#### 合并

Mix 提供了 `webpackConfig` 方法允许你合并任何短的 Webpack 来覆盖配置。这是一个特别有吸引力的选择，因为它不需要你去复制和维护自己的 `webpack.config.js` 文件副本。`webpackConfig` 方法可以接受你希望添加的 [Webpack 特定配置](https://webpack.js.org/configuration/) 任何对象。

    mix.webpackConfig({
        resolve: {
            modules: [
                path.resolve(__dirname, 'vendor/laravel/spark/resources/assets/js')
            ]
        }
    });

#### 自定义配置说明

第二种选择就是可以复制 Mix 的配置文件 `webpack.config.js` 到工程的根目录。

    cp node_modules/laravel-mix/setup/webpack.config.js ./

接着你需要在 `package.json` 文件中升级 NPM 脚本，确定它们不再直接引用 Mix 的配置文件。只需从命令中删除 `--config="node_modules/laravel-mix/setup/webpack.config.js"` 条目。完成之后你就可以根据需求随意修改配置文件了。

<a name="copying-files-and-directories"></a>
## 复制文件与目录

`copy` 方法可用于复制文件和目录到新的位置。当一个在 `node_modules` 目录中的指定资源需要重定位到你的 `public` 文件夹时，这将会变得非常有用：

    mix.copy('node_modules/foo/bar.css', 'public/css/bar.css');

<a name="versioning-and-cache-busting"></a>
## 版本与缓存清除

许多的开发者会在它们编译后的资源文件中加上时间戳或是唯一的 token，强迫浏览器加载全新的资源文件以取代提供的旧版本代码副本。你可以使用 `version` 方法让 Mix 处理它们。

`version` 方法会自动为所有编译过的文件名称添加唯一的哈希值，允许更方便的缓存清除：

    mix.js('resources/assets/js/app.js', 'public/js')
       .version();

在为文件生成版本之后，你并不知道确切的文件名。所以你应该在 [views](/docs/{{version}}/views) 中使用 Laravel 的全局函数  `mix` 去加载这个合适的哈希资源。`mix` 函数会自动确定当前哈希文件的名称：

    <link rel="stylesheet" href="{{ mix('/css/app.css') }}">

由于监听文件版本变动通常在开发中是不必要的，所以你可能希望版本处理的命令只运行在 `npm run production` 中：

    mix.js('resources/assets/js/app.js', 'public/js');

    if (mix.config.inProduction) {
        mix.version();
    }

<a name="notifications"></a>
## 通知

可用时，Mix 会自动为每个 Bundle 显示系统通知。这会直接反馈该编译 Bundle 是否成功。然而有些情况我们可能希望禁用这些通知。比如一个这样的例子，可能会在您的生产服务器上触发 Mix。通过 `disablenotifications` 方法可以停用通知：

    mix.disableNotifications();
