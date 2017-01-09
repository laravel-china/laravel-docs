# 编译资源文件 (Laravel Elixir)

- [简介](#introduction)
- [安装与配置](#installation)
- [运行 Elixir](#running-elixir)
- [使用样式](#working-with-stylesheets)
    - [Less](#less)
    - [Sass](#sass)
    - [Stylus](#stylus)
    - [纯 CSS](#plain-css)
    - [Source Maps](#css-source-maps)
- [使用脚本](#working-with-scripts)
    - [Webpack](#webpack)
    - [Rollup](#rollup)
    - [Scripts](#javascript)
- [复制文件与目录](#copying-files-and-directories)
- [版本与缓存清除](#versioning-and-cache-busting)
- [BrowserSync](#browser-sync)

<a name="introduction"></a>
## 简介

Laravel Elixir 提供了简洁流畅的 API，让你能够在你的 Laravel 应用程序中定义基本的 [Gulp](http://gulpjs.com) 任务。Elixir 支持许多常见的 CSS 与 JavaScript 预处理器，例如 [Sass](http://sass-lang.com) 和 [Webpack](https://webpack.github.io/)。使用链式调用，Elixir 让你流畅地定义开发流程，例如：

```javascript
elixir(function(mix) {
    mix.sass('app.scss')
       .webpack('app.js');
});
```

如果你曾经对于上手 Gulp 及编译资源文件感到困惑，那么你将会爱上 Laravel Elixir，不过 Laravel 并不强迫你使用 Elixir，你可以自由的选用你喜欢的自动化开发流程工具，或者完全不使用此类工具。

<a name="installation"></a>
## 安装及配置

#### 安装 Node

在开始使用 Elixir 之前，你必须先确保你的机器上有安装 Node.js 和 npm。

    node -v
    npm -v

默认情况下，Laravel Homestead 包含了你所需的一切；如果你没有使用 Vagrant，那么你可以从[Node 的官方下载页面](http://nodejs.org/en/download/) 下载可视化安装工具来安装 Node 与 NPM 的最新版本。

#### Gulp
接着，你需要全局安装 [Gulp](http://gulpjs.com) 的 NPM 扩展包：

    npm install --global gulp-cli

#### Laravel Elixir

最后的步骤就是安装 Elixir。在每一份新安装的 Laravel 代码里，你会发现根目录有个名为 `package.json` 的文件。默认的 `package.json` 已经包含了 Elixir 和 Webpack JavaScript 模块。想像它就如同你的 `composer.json` 文件，只不过它定义的是 Node 的依赖扩展包，而不是 PHP 的。你可以使用以下的命令安装依赖扩展包：

    npm install

如果你是在 Windows 系统上或在 Windows 主机系统上运行 VM 进行开发，你需要在运行 `npm install` 命令时将 `--no-bin-links` 开启：

    npm install --no-bin-links

<a name="running-elixir"></a>
## 运行 Elixir

Elixir 是创建于 [Gulp](http://gulpjs.com) 之上，所以要运行你的 Elixir 任务，只需要在命令行运行 `gulp` 命令。在命令增加 `--production` 标示会告知 Elixir 压缩你的 CSS 及 JavaScript 文件：

    // 运行所有任务...
    gulp

    // 运行所有任务并压缩所有 CSS 及 JavaScript...
    gulp --production

一旦运行了上述命令，你会看到一个漂亮的表格，列举出了所有刚刚发生的事件的摘要。

#### 监控资源文件修改

`gulp watch` 会在你的终端里持续运行，监控资源文件是否有发生改变。在 `watch` 命令运行的情况下，一旦资源文件发生变化，Gulp 会自动重新编译：

    gulp watch

<a name="working-with-stylesheets"></a>
## 使用样式

项目根目录的 `gulpfile.js` 包含你所有的 Elixir 任务。Elixir 任务可以使用链式调用的写法来定义你的资源文件该如何进行编译。

<a name="less"></a>
### Less

要将 [Less](http://lesscss.org/) 编译为 CSS，你可以使用 `less` 方法。`less` 方法会假设你的 Less 文件被保存在 `resources/assets/less` 文件夹中。默认情况下，此例子的任务会将编译后的 CSS 放置于 `public/css/app.css`：

```javascript
elixir(function(mix) {
    mix.less('app.less');
});
```

你可能会想合并多个 Less 文件至单个 CSS 文件。同样的，生成的 CSS 会被放置于 `public/css/app.css`:

```javascript
elixir(function(mix) {
    mix.less([
        'app.less',
        'controllers.less'
    ]);
});
```

如果你想自定义编译后的 CSS 输出位置，可以传递第二个参数至 `less` 方法：

```javascript
elixir(function(mix) {
    mix.less('app.less', 'public/stylesheets');
});

// 指定输出的文件名称...
elixir(function(mix) {
    mix.less('app.less', 'public/stylesheets/style.css');
});
```

<a name="sass"></a>
### Sass

`sass` 方法让你能编译 [Sass](http://sass-lang.com/) 至 CSS。Sass 文件的默认读取路径是 `resources/assets/sass`，你可以使用此方法：

```javascript
elixir(function(mix) {
    mix.sass('app.scss');
});
```

同样的，如同 `less` 方法，你可以编译多个 Sass 文件至单个的 CSS 文件，甚至可以自定义生成的 CSS 的输出目录：

```javascript
elixir(function(mix) {
    mix.sass([
        'app.scss',
        'controllers.scss'
    ], 'public/assets/css');
});
```

#### 自定义路径

尽管我们建议你使用 Laravel 默认的资源文件夹，但是如果你想要更换到其他路径，你可以在文件路径前面加上 `./`。这样 Elixir 就会从项目根目录开始查找文件，而不是默认资源文件目录。

举例来说，如果像要编译 `app/assets/sass/app.scss` 文件并且输出到 `public/css/app.css`，可以用下面 `sass` 的命令：

```javascript
elixir(function(mix) {
    mix.sass('./app/assets/sass/app.scss');
});
```

<a name="stylus"></a>
### Stylus

`stylus` 命令可以用来编译 [Stylus](http://stylus-lang.com/) 至 CSS。假设你的 Stylus 文件放在 `resources/assets/stylus`，你可以这样调用：

```javascript
elixir(function(mix) {
    mix.stylus('app.styl');
});
```

> {tip} 这个方法的签名和 `mix.less()` 与 `mix.sass()`是一样的。

<a name="plain-css"></a>
### 纯 CSS

如果你只是想将一些纯 CSS 样式合并成单个的文件，你可以使用 `styles` 方法。此方法的默认路径为 `resources/assets/css` 目录，而生成的 CSS 会被放置于 `public/css/all.css`：

```javascript
elixir(function(mix) {
    mix.styles([
        'normalize.css',
        'main.css'
    ]);
});
```
当然，你也可以通过传递第二个参数至 `styles` 方法，将生成的文件输出至指定的位置：

```javascript
elixir(function(mix) {
    mix.styles([
        'normalize.css',
        'main.css'
    ], 'public/assets/css/site.css');
});
```

<a name="css-source-maps"></a>
### Source Maps

在 Elixir 中，为了更方便的在浏览器中进行调试，source maps 默认是开启的。因此，针对每个被编译的文件，同目录内都会伴随着一个 `*.css.map` 或者 `*.js.map` 文件。

如果你不想为你的应用生成 source maps，你可以使用 `sourcemaps` 配置选项关闭它们：

```javascript
elixir.config.sourcemaps = false;

elixir(function(mix) {
    mix.sass('app.scss');
});
```

<a name="working-with-scripts"></a>
## 使用脚本

Elixir 也提供了一些函数来帮助你使用 JavaScript 文件，像是编译 ECMAScript 6、编译 CoffeeScript、Browserify、压缩、及简单的串联纯 JavaScript 文件。

如果你是使用 ES2015 编写代码，你可以在 [Webpack](http://webpack.github.io) 和 [Rollup](http://rollupjs.org/) 中选择。如果你对这些工具都不熟悉，也没有关系，Elixir 会帮你处理所有复杂的问题。默认情况下，Laravel 的 `gulpfile` 会使用 `webpack` 来编译 Javascript，但是你可以自由选择其他你喜欢的打包工具

<a name="webpack"></a>
### Webpack

`webpack` 命令可以用来编译和打包 [ECMAScript 2015](https://babeljs.io/docs/learn-es2015/) 文件至纯 JavaScript。 这个功能接受相对于 `resources/assets/js` 的文件路径作为参数，并在 `public/js` 文件夹下生成一个打包好的文件：

```javascript
elixir(function(mix) {
    mix.webpack('app.js');
});
```

如果要选择不同的输入和输出目录，只需简单的在路径前面加上 `.`。然后你就可以使用相对应用根目录的路径。比如，想要把 `app/assets/js/app.js` 编译至 `public/dist/app.js`:

```javascript
elixir(function(mix) {
    mix.webpack(
        './app/assets/js/app.js',
        './public/dist'
    );
});
```

如果你想要发挥 Webpack 更多的作用。Elixir会读取你放在应用根目录的 `webpack.config.js` 文件并且在编译过程中[应用这些配置](https://webpack.github.io/docs/configuration.html)。

<a name="rollup"></a>
### Rollup

和 Webpack 类似，Rollup 是新一代的 ES2015 打包工具。这个功能可以接受一组相对于 `resources/assets/js` 的路径为参数，并在 `public/js` 文件夹下生成打包好的文件：

```javascript
elixir(function(mix) {
    mix.rollup('app.js');
});
```

就像 `webpack` 命令一样，你也可以在 `rollup` 命令里自定义输入输出的文件的路径：

    elixir(function(mix) {
        mix.rollup(
            './resources/assets/js/app.js',
            './public/dist'
        );
    });

<a name="javascript"></a>
### Scripts

如果你想将多个 JavaScript 文件合并至单个文件，你可以使用 `scripts` 方法，这个方法可以自动生成 source maps，合并和压缩文件。

`scripts` 方法假设所有的路径都相对于 `resources/assets/js` 目录，且默认会将生成的 JavaScript 放置于 `public/js/all.js`：

```javascript
elixir(function(mix) {
    mix.scripts([
        'order.js',
        'forum.js'
    ]);
});
```

如果你想多个脚本的集合合并成不同文件，你可以使用调用多个 `scripts` 方法。给予该方法的第二个参数会为每个串联决定生成的文件名称：

```javascript
elixir(function(mix) {
    mix.scripts(['app.js', 'controllers.js'], 'public/js/app.js')
       .scripts(['forum.js', 'threads.js'], 'public/js/forum.js');
});
```

如果你想合并指定目录中的所有脚本，你可以使用 `scriptsIn` 方法。生成的 JavaScript 会被放置在 `public/js/all.js`：

```javascript
elixir(function(mix) {
    mix.scriptsIn('public/js/some/directory');
});
```

> {tip} 如果你想合并多个已经编译过的第三方库，比如 jQuery，可以尝试使用 `mix.combine()`。这个命令可以合并文件，但是跳过生成 source map 和压缩文件的步骤。这样一来，编译时间可以大大缩短。

<a name="copying-files-and-directories"></a>
## 复制文件与目录

`copy` 方法可以复制文件与目录至新位置。所有操作路径都相对于项目的根目录：

```javascript
elixir(function(mix) {
	mix.copy('vendor/foo/bar.css', 'public/css/bar.css');
});
```

<a name="versioning-and-cache-busting"></a>
## 版本与缓存清除

许多的开发者会在它们编译后的资源文件中加上时间戳或是唯一的 token，强迫浏览器加载全新的资源文件以取代提供的旧版本代码副本。你可以使用 `version` 方法让 Elixir 处理它们。

`version` 方法接收一个相对于 `public` 目录的文件名称，接着为你的文件名称加上唯一的哈希值，以防止文件被缓存。举例来说，生成出来的文件名称可能像这样：`all-16d570a7.css`：

```javascript
elixir(function(mix) {
    mix.version('css/all.css');
});
```

在为文件生成版本之后，你可以在你的[视图](/docs/{{version}}/views) 中使用 Laravel 的全局 `elixir` PHP 辅助函数来正确加载名称被哈希后的文件。elixir 函数会自动判断被哈希的文件名称：

    <link rel="stylesheet" href="{{ elixir('css/all.css') }}">

#### 为多个文件生成版本

你可以传递一个数组至 `version` 方法来为多个文件生成版本：

```javascript
elixir(function(mix) {
    mix.version(['css/all.css', 'js/app.js']);
});
```

一旦该文件被加上版本，你需要使用 `elixir` 辅助函数来生成哈希文件的正确链接。切记，你只需要传递未哈希文件的名称至 `elixir` 辅助函数。此函数会自动使用未哈希的名称来判断该文件为目前的哈希版本：

    <link rel="stylesheet" href="{{ elixir('css/all.css') }}">

    <script src="{{ elixir('js/app.js') }}"></script>

<a name="browser-sync"></a>
## BrowserSync

当你对前端资源进行修改后，BrowserSync 会自动刷新你的网页浏览器。`browserSync` 接受的参数是包含 `proxy` （一般是应用的本地URL） 属性的 JavaScript 对象。
然后，当你运行了 `gulp watch` 命令，你就可以通过 3000 (`http://project.dev:3000`) 端口来访问你的 web 应用，并且享受浏览器同步的便利。

```javascript
elixir(function(mix) {
    mix.browserSync({
        proxy: 'project.dev'
    });
});
```
