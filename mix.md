# Laravel 的资源任务编译器 Laravel Mix

- [简介](#introduction)
- [安装与配置](#installation)
- [运行 Mix](#running-mix)
- [使用样式](#working-with-stylesheets)
    - [Less](#less)
    - [Sass](#sass)
    - [Stylus](#stylus)
    - [PostCSS](#postcss)
    - [纯 CSS](#plain-css)
	- [URL Processing](#url-processing)
    - [资源地图](#css-source-maps)
- [使用脚本](#working-with-scripts)
    - [Vendor Extraction](#vendor-extraction)
    - [React](#react-support)
    - [Vanilla JS](#vanilla-js)
    - [自定义 Webpack 配置](#custom-webpack-configuration)
- [复制文件与目录](#copying-files-and-directories)
- [版本与缓存清除](#versioning-and-cache-busting)
- [Browsersync Reloading](#browsersync-reloading)
- [通知](#notifications)

<a name="introduction"></a>
## 简介

Laravel Mix 提供了简洁流畅的 API，让你能够为你的 Laravel 应用定义 Webpack 的编译任务。Mix 支持许多常见的 CSS 与 JavaScrtip 预处理器，通过简单的方法，你可以轻松的管理资源。例如：

	mix.js('resources/assets/js/app.js', 'public/js')
		.sass('resources/assets/sass/app.scss', 'public/css');

如果你曾经对于使用 Webpack 及编译资源感到困惑，那么你绝对会爱上 Laravel Mix。当然，在 Laravel 应用开发中使用 Mix 并不是必须的，你也可以选择任何你喜欢的资源编译工具，或者不使用任何工具。

<a name="installation"></a>
## 安装

#### 安装 Node

在开始使用 Mix 之前，你必须先确定你的开发环境上有安装 Node.js 和 NPM。

    node -v
    npm -v


默认情况下, Laravel Homestead 会包含你所需的一切。当然，如果你没有使用 Vagrant，那么你可以浏览 [nodejs](https://nodejs.org/en/download/) 下载可视化的安装工具来安装最新版的 Node 和 NPM.

#### Laravel Mix

剩下的只需要安装 Laravel Mix！随着新安装的 Laravel, 你会发现根目录下有个名为 `package.json` 的文件。就如同 `composer.json` 文件, 只不过 `package.json` 文件定义的是 Node 的依赖，而不是 PHP。你可以使用以下的命令安装依赖扩展包：

    npm install

如果你使用的是 Windows 系统或运行在 Windows 系统上的 VM, 你需要在运行 `npm install` 命令时将 `--no-bin-links` 开启：

    npm install --no-bin-links

<a name="running-mix"></a>
## 使用

Mix 基于 [Webpack](https://webpack.js.org) 的配置， 所以运行定义于 `package.json` 文件中的 NPM 脚本即可执行 Mix 的编译任务:

    // 运行所有 Mix 任务...
    npm run dev

    // 运行所有 Mix 任务和压缩资源输出
    npm run production

#### 监控资源文件修改

`npm run watch` 会在你的终端里持续运行，监控资源文件是否有发生改变。在 watch 命令运行的情况下，一旦资源文件发生变化，Webpack 会自动重新编译：


    npm run watch

You may find that in certain environments Webpack isn't updating when your files change. If this is the case on your system, consider using the `watch-poll` command:

    npm run watch-poll

<a name="working-with-stylesheets"></a>
## 使用样式

项目根目录的 `webpack.mix.js` 文件是资源编译的入口。可以把它看作是 Webpack 的配置文件。Mix 任务可以使用链式调用的写法来定义你的资源文件该如何进行编译。

<a name="less"></a>
### Less

`less` 方法可以让你将 [Less](http://lesscss.org/) 编译为 CSS。下面的命令可以把 `app.less` 编译为 `public/css/app.css`。

	mix.less('resources/assets/less/app.less', 'public/css');

多次调用 `less` 方法可以编译多个文件:

    mix.less('resources/assets/less/app.less', 'public/css')
       .less('resources/assets/less/admin.less', 'public/css');

如果你想自定义编译后的 CSS 文件名, 你可以传递一个完整的路径到 `less` 方法的第二个参数:

    mix.less('resources/assets/less/app.less', 'public/stylesheets/styles.css');

If you need to override the [underlying Less plug-in options](https://github.com/webpack-contrib/less-loader#options), you may pass an object as the third argument to `mix.less()`:

    mix.less('resources/assets/less/app.less', 'public/css', {
        strictMath: true
    });

<a name="sass"></a>
### Sass

`sass` 方法可以让你将 [Sass](http://sass-lang.com/) 便以为 CSS。你可以使用此方法：

	mix.sass('resources/assets/sass/app.scss', 'public/css');

同样的，如同 `less` 方法, 你可以将多个 Sass 文件编译为多个 CSS 文件，甚至可以自定义生成的 CSS 的输出目录：

	mix.sass('resources/assets/sass/app.sass', 'public/css')
       .sass('resources/assets/sass/admin.sass', 'public/css/admin');

Additional [Node-Sass plug-in options](https://github.com/sass/node-sass#options) may be provided as the third argument:

    mix.sass('resources/assets/less/app.less', 'public/css', {
        precision: 5
    });

<a name="stylus"></a>
### Stylus

Similar to Less and Sass, the `stylus` method allows you to compile [Stylus](http://stylus-lang.com/) into CSS:

    mix.stylus('resources/assets/stylus/app.styl', 'public/css');

You may also install additional Stylus plug-ins, such as [Rupture](https://github.com/jescalan/rupture). First, install the plug-in in question through NPM (`npm install rupture`) and then require it in your call to `mix.stylus()`:

    mix.stylus('resources/assets/stylus/app.styl', 'public/css', {
        use: [
            require('rupture')()
        ]
    });

<a name="postcss"></a>
### PostCSS

[PostCSS](http://postcss.org/), a powerful tool for transforming your CSS, is included with Laravel Mix out of the box. By default, Mix leverages the popular [Autoprefixer](https://github.com/postcss/autoprefixer) plug-in to automatically apply all necessary CSS3 vendor prefixes. However, you're free to add any additional plug-ins that are appropriate for your application. First, install the desired plug-in through NPM and then reference it in your `webpack.mix.js` file:

    mix.sass('resources/assets/sass/app.scss', 'public/css')
       .options({
            postCss: [
                require('postcss-css-variables')()
            ]
       });

<a name="plain-css"></a>
### 纯 CSS

如果你只是想将一些纯 CSS 样式合并成单个的文件, 你可以使用 `styles` 方法。

    mix.styles([
        'public/css/vendor/normalize.css',
        'public/css/vendor/videojs.css'
    ], 'public/css/all.css');

<a name="url-processing"></a>
### URL Processing

Because Laravel Mix is built on top of Webpack, it's important to understand a few Webpack concepts. For CSS compilation, Webpack will rewrite and optimize any `url()` calls within your stylesheets. While this might initially sound strange, it's an incredibly powerful piece of functionality. Imagine that we want to compile Sass that includes a relative URL to an image:

    .example {
        background: url('../images/example.png');
    }

> {note} Absolute paths for `url()`s will be excluded from URL-rewriting. For example, `url('/images/thing.png')` or `url('http://example.com/images/thing.png')` won't be modified.

By default, Laravel Mix and Webpack will find `example.png`, copy it to your `public/images` folder, and then rewrite the `url()` within your generated stylesheet. As such, your compiled CSS will be:

    .example {
      background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
    }

As useful as this feature may be, it's possible that your existing folder structure is already configured in a way you like. If this is the case, you may disable `url()` rewriting like so:

    mix.sass('resources/assets/app/app.scss', 'public/css')
       .options({
          processCssUrls: false
       });

With this addition to your `webpack.mix.js` file, Mix will no longer match `url()`s or copy assets to your public directory. In other words, the compiled CSS will look just like how you originally typed it:

    .example {
        background: url("../images/thing.png");
    }

<a name="css-source-maps"></a>
### 资源地图

source maps 默认状态下是禁用的，你可以通过在 `webpack.mix.js` 文件中调用 `mix.sourceMaps()` 方法来开启。它会带来一些编译成本，但在使用编译后的资源文件时可以更方便的在浏览器中进行调试：

    mix.js('resources/assets/js/app.js', 'public/js')
       .sourceMaps();

<a name="working-with-scripts"></a>
## 使用脚本

Mix 也提供了一些函数来帮助你使用 JavaScript 文件，像是编译 ECMAScript 2015、模块编译、压缩、及简单的串联纯 JavaScript 文件。更棒的是，这些都不需要自定义的配置：

    mix.js('resources/assets/js/app.js', 'public/js');

这一行简单的代码，支持：

<div class="content-list" markdown="1">
- ECMAScript 2015 语法.
- Modules
- 编译 `.vue` 文件.
- 针对生产环境压缩代码.
</div>

<a name="vendor-extraction"></a>
### Vendor Extraction

将应用程序的 JavaScript 与依赖库捆绑在一起的一个潜在缺点是，使得长期缓存更加困难。如，对应用程序代码的单独更新将强制浏览器重新下载所有依赖库，即使它们没有更改。

如果你打算频繁更新应用程序的 JavaScript，应该考虑将所有的依赖库提取到单独文件中。这样，对应用程序代码的更改不会影响 vendor.js 文件的缓存。Mix 的 `extract` 方法可以轻松做到：

    mix.js('resources/assets/js/app.js', 'public/js')
       .extract(['vue'])

`extract` 方法接受你希望提取到 `vendor.js` 文件中的所有的依赖库或模块的数组。使用以上代码片段作为示例，Mix 将生成以下文件：

<div class="content-list" markdown="1">
- `public/js/manifest.js`: *Webpack 显示运行时*
- `public/js/vendor.js`: *依赖库*
- `public/js/app.js`: *应用代码*
</div>

为了避免 `JavaScript` 错误，请务必按正确的顺序加载这些文件：

    <script src="/js/manifest.js"></script>
    <script src="/js/vendor.js"></script>
    <script src="/js/app.js"></script>

<a name="react"></a>
### React

Mix can automatically install the Babel plug-ins necessary for React support. To get started, replace your `mix.js()` call with `mix.react()`:

    mix.react('resources/assets/js/app.jsx', 'public/js');

Behind the scenes, React will download and include the appropriate `babel-preset-react` Babel plug-in.

<a name="vanilla-js"></a>
### Vanilla JS

Similar to combining stylesheets with `mix.styles()`, you may also combine and minify any number of JavaScript files with the `scripts()` method:

    mix.scripts([
        'public/js/admin.js',
        'public/js/dashboard.js'
    ], 'public/js/all.js');

This option is particularly useful for legacy projects where you don't require Webpack compilation for your JavaScript.

> {tip} A slight variation of `mix.scripts()` is `mix.babel()`. Its method signature is identical to `scripts`; however, the concatenated file will receive Babel compilation, which translates any ES2015 code to vanilla JavaScript that all browsers will understand.

<a name="custom-webpack-configuration"></a>
### 自定义 Webpack 配置

Laravel Mix 默认引用了一个预先配置的 `webpack.config.js` 文件，以便尽快启动和运行。有时，你可能需要手动修改此文件。例如，你可能有一个特殊的加载器或插件需要被引用，或者也许你喜欢使用 Stylus 而不是 Sass。在这种情况下，你有两个选择：

#### 合并

Mix 提供了一个有用的 `webpackConfig` 方法，允许合并任何 `Webpack` 配置以覆盖默认配置。这是一个非常好的选择，你不需要复制和维护 `webpack.config.js` 文件。webpackConfig 方法接受一个对象，该对象应包含要应用的任何 [Webpack 配置项](https://webpack.js.org/configuration/)：

    mix.webpackConfig({
        resolve: {
            modules: [
                path.resolve(__dirname, 'vendor/laravel/spark/resources/assets/js')
            ]
        }
    });

<a name="copying-files-and-directories"></a>
## 复制文件与目录

`copy` 方法可以复制文件与目录至新位置。当 `node_modules` 目录中的特定资源需要复制到 `public` 文件夹时会很有用。

    mix.copy('node_modules/foo/bar.css', 'public/css/bar.css');

<a name="versioning-and-cache-busting"></a>
## 版本与缓存清除

许多的开发者会在它们编译后的资源文件中加上时间戳或是唯一的 token，强迫浏览器加载全新的资源文件以取代提供的旧版本代码副本。你可以使用 version 方法让 Mix 处理它们。

`version` 方法为你的文件名称加上唯一的哈希值，以防止文件被缓存：

    mix.js('resources/assets/js/app.js', 'public/js')
       .version();

在为文件生成版本之后，你将不知道确切的文件名。因此，你应该在你的视图 中使用 Laravel 的全局 `mix` PHP 辅助函数来正确加载名称被哈希后的文件。 `mix` 函数会自动判断被哈希的文件名称：

    <link rel="stylesheet" href="{{ mix('/css/app.css') }}">

在开发中通常是不需要版本化，你可能希望仅在运行 `npm run production` 的时候进行版本化：

    mix.js('resources/assets/js/app.js', 'public/js');

    if (mix.config.inProduction) {
        mix.version();
    }

<a name="browsersync-reloading"></a>
## Browsersync Reloading

[BrowserSync](https://browsersync.io/) can automatically monitor your files for changes, and inject your changes into the browser without requiring a manual refresh. You may enable support by calling the `mix.browserSync()` method:

    mix.browserSync('my-domain.dev');

    // Or...

    // https://browsersync.io/docs/options
    mix.browserSync({
        proxy: 'my-domain.dev'
    });

You may pass either a string (proxy) or object (BrowserSync settings) to this method. Next, start Webpack's dev server using the `npm run watch` command. Now, when you modify a script or PHP file, watch as the browser instantly refreshes the page to reflect your changes.

<a name="notifications"></a>
## 通知

在可用的时候，Mix 会将每个包的编译是否成功以系统通知的方式反馈给你。如果你希望停用这些通知，可以通过 `disableNotifications` 方法实现：
    
    mix.disableNotifications();

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@zyxcba](https://github.com/cmzz)  | <img class="avatar-66 rm-style" src="https://avatars3.githubusercontent.com/u/6111715?v=3&s=100">  |  翻译  | [考拉客](http://kaolake.net) - 考拉微商店主加盟立返100元！ |

