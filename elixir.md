# Laravel Elixir

- [简介](#introduction)
- [安装与配置](#installation)
- [运行 Elixir](#running-elixir)
- [使用样式](#working-with-stylesheets)
    - [Less](#less)
    - [Sass](#sass)
    - [纯 CSS](#plain-css)
    - [Source Maps](#css-source-maps)
- [使用脚本](#working-with-scripts)
    - [CoffeeScript](#coffeescript)
    - [Browserify](#browserify)
    - [Babel](#babel)
    - [Scripts](#javascript)
- [复制文件与目录](#copying-files-and-directories)
- [版本与缓存清除](#versioning-and-cache-busting)
- [BrowserSync](#browser-sync)
- [调用既有的 Gulp 任务](#calling-existing-gulp-tasks)
- [编写 Elixir 扩展功能](#writing-elixir-extensions)

<a name="introduction"></a>
## 简介

Laravel Elixir 提供了简洁流畅的 API，让你能够在你的 Laravel 应用程序中定义基本的 [Gulp](http://gulpjs.com) 任务。Elixir 支持许多常见的 CSS 与 JavaScrtip 预处理器，甚至包含了测试工具。使用链式调用，Elixir 让你流畅的定义你的开发流程，例如：

```javascript
elixir(function(mix) {
    mix.sass('app.scss')
       .coffee('app.coffee');
});
```

如果你曾经对于上手 Gulp 及编译资源文件感到困惑，那么你将会爱上 Laravel Elixir，不过 Laravel 并不强迫你使用 Elixir，你可以自由的选用你喜欢的自动化开发流程工具。

> TODO: 这里需要增加 Elixir 相关介绍文章链接，尤其说明在生产环境中使用的好处，如合并、压缩 CSS 和 JS 文件等。

<a name="installation"></a>
## 安装及配置

### 安装 Node

在开始使用 Elixir 之前，你必须先确定你的机器上有安装 Node.js。

    node -v

默认情况下，Laravel Homestead 会包含你所需的一切；但是，如果你没有使用 Vagrant，那么你可以简单的浏览 [Node 的下载页面](http://nodejs.org/download/)进行安装。

### Gulp

接着，你需要全局安装 [Gulp](http://gulpjs.com) 的 NPM 扩展包：

    npm install --global gulp

### Laravel Elixir

最后的步骤就是安装 Elixir！在每一份新安装的 Laravel 代码里，你会发现根目录有个名为 `package.json` 的文件。想像它就如同你的 `composer.json` 文件，只是它定义的是 Node 的依赖扩展包，而不是 PHP 的。你可以使用以下的命令安装依赖扩展包：

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

#### 监控资源文件修改

因为每次修改你的资源文件之后在命令行运行 `gulp` 命令会相当不便，因此你可以使用 `gulp watch` 命令。此命令会在你的命令行继续运行并监控资源文件的任何修改。当发生修改时，新文件将会自动被编译：

    gulp watch

<a name="working-with-stylesheets"></a>
## 使用样式

项目根目录的 `gulpfile.js` 包含你所有的 Elixir 任务。Elixir 任务可以被链式调用起来，以定义你的资源文件该如何进行编译。

<a name="less"></a>
### Less

要将 [Less](http://lesscss.org/) 编译为 CSS，你可以使用 `less` 方法。`less` 方法会假设你的 Less 文件被保存在 `resources/assets/less` 中。默认情况下，此例子的任务会将编译后的 CSS 放置于 `public/css/app.css`：

```javascript
elixir(function(mix) {
    mix.less('app.less');
});
```

你可能会想合并多个 Less 文件至单个 CSS 文件。同样的，产生的 CSS 会被放置于 `public/css/app.css`：

```javascript
elixir(function(mix) {
    mix.less([
        'app.less',
        'controllers.less'
    ]);
});
```

如果你想自定义编译后的 CSS 输出位置，你可以传递第二个参数至 `less` 方法：

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

`sass` 方法让你能编译 [Sass](http://sass-lang.com/) 至 CSS。你的 Sass 文件默认会被保存在 `resources/assets/sass`，你可以像这样使用此方法：

```javascript
elixir(function(mix) {
    mix.sass('app.scss');
});
```

同样的，如同 `less` 方法，你可以编译多个 Sass 文件至单一的 CSS 文件，甚至可以自定义产生的 CSS 的输出目录：

```javascript
elixir(function(mix) {
    mix.sass([
        'app.scss',
        'controllers.scss'
    ], 'public/assets/css');
});
```

<a name="plain-css"></a>
### 纯 CSS

如果你只是想将一些纯 CSS 样式合并成单一的文件，你可以使用 `styles` 方法。传递给此方法的路径相对于 `resources/assets/css` 目录，而产生的 CSS 会被放置于 `public/css/all.css`：

```javascript
elixir(function(mix) {
    mix.styles([
        'normalize.css',
        'main.css'
    ]);
});
```

当然，你也可以通过传递第二个参数至 `styles` 方法，将产生的文件输出至自定的位置：

```javascript
elixir(function(mix) {
    mix.styles([
        'normalize.css',
        'main.css'
    ], 'public/assets/css');
});
```

<a name="css-source-maps"></a>
### Source Maps

Source maps 在默认情况下是开启的。因此，针对每个被编译的文件，同目录内都会伴随着一个 `*.css.map` 文件。这个文件能够让你在浏览器调试时，可以追踪编译后的样式选择器至原始的 Sass 或 Less 位置。

如果你不想为你的 CSS 产生 source maps，你可以使用一个简单的配置选项关闭它们：

```javascript
elixir.config.sourcemaps = false;

elixir(function(mix) {
    mix.sass('app.scss');
});
```

<a name="working-with-scripts"></a>
## 使用脚本

Elixir 也提供了一些函数来帮助你使用 JavaScript 文件，像是编译 ECMAScript 6、编译 CoffeeScript、Browserify、压缩、及简单的串联纯 JavaScript 文件。

<a name="coffeescript"></a>
### CoffeeScript

`coffee` 方法可以用于编译 [CoffeeScript](http://coffeescript.org/) 至纯 JavaScript。`coffee` 函数接收一个相对于 `resources/assets/coffee` 目录的 CoffeeScript 文件名字符串或数组，接着在 `public/js` 目录产生单一的 `app.js` 文件：

```javascript
elixir(function(mix) {
    mix.coffee(['app.coffee', 'controllers.coffee']);
});
```

<a name="browserify"></a>
### Browserify

Elixir 还附带了一个 `browserify` 方法，给予你在浏览器引入模块及 ECMAScript 6 的有用的特性。

此任务假设你的脚本都保存在 `resources/assets/js`，并会将产生的文件放置于 `public/js/main.js`：

```javascript
elixir(function(mix) {
    mix.browserify('main.js');
});
```

虽然 Browserify 附带了 Partialify 及 Babelify 转换器，但是只要你愿意，你可以随意安装并增加更多的转换器：

    npm install aliasify --save-dev

```javascript
elixir.config.js.browserify.transformers.push({
    name: 'aliasify',
    options: {}
});

elixir(function(mix) {
    mix.browserify('main.js');
});
```

<a name="babel"></a>
### Babel

`babel` 方法可被用于编译 [ECMAScript 6 与 7](https://babeljs.io/docs/learn-es2015/) 至纯 JavaScript。此函数接收一个相对于 `resources/assets/js` 目录的文件数组，接着在 `public/js` 目录产生单一的 `all.js` 文件：

```javascript
elixir(function(mix) {
    mix.babel([
        'order.js',
        'product.js'
    ]);
});
```

若要选择不同的输出位置，只需简单的指定你希望的路径作为第二个参数。该方法除了 Babel 的编译外，特色与功能同等于 `mix.scripts()`。


<a name="javascript"></a>
### Scripts

如果你想将多个 JavaScript 文件合并至单一文件，你可以使用 `scripts` 方法。

`scripts` 方法假设所有的路径都相对于 `resources/assets/js` 目录，且默认会将产生的 JavaScript 放置于 `public/js/all.js`：

```javascript
elixir(function(mix) {
    mix.scripts([
        'jquery.js',
        'app.js'
    ]);
});
```

如果你想多个脚本的集合合并成不同文件，你可以使用调用多个 `scripts` 方法。给予该方法的第二个参数会为每个串联决定产生的文件名称：

```javascript
elixir(function(mix) {
    mix.scripts(['app.js', 'controllers.js'], 'public/js/app.js')
       .scripts(['forum.js', 'threads.js'], 'public/js/forum.js');
});
```

如果你想合并指定目录中的所有脚本，你可以使用 `scriptsIn` 方法。产生的 JavaScript 会被放置在 `public/js/all.js`：

```javascript
elixir(function(mix) {
    mix.scriptsIn('public/js/some/directory');
});
```

<a name="copying-files-and-directories"></a>
## 复制文件与目录

`copy` 方法可以复制文件与目录至新位置。所有操作路径都相对于项目的根目录：

```javascript
elixir(function(mix) {
	mix.copy('vendor/foo/bar.css', 'public/css/bar.css');
});

elixir(function(mix) {
	mix.copy('vendor/package/views', 'resources/views');
});
```

<a name="versioning-and-cache-busting"></a>
## 版本与缓存清除

许多的开发者会在它们编译后的资源文件中加上时间戳或是唯一的 token，强迫浏览器加载全新的资源文件以取代提供的旧版本代码副本。你可以使用 `version` 方法让 Elixir 处理它们。

`version` 方法接收一个相对于 `public` 目录的文件名称，接着为你的文件名称加上唯一的哈希值，以防止文件被缓存。举例来说，产生出来的文件名称可能像这样：`all-16d570a7.css`：

```javascript
elixir(function(mix) {
    mix.version('css/all.css');
});
```

在为文件产生版本之后，你可以在你的[视图](/docs/{{version}}/views)中使用 Laravel 的全局 `elixir` PHP 辅助函数来正确加载名称被哈希后的文件。`elixir` 函数会自动判断被哈希的文件名称：

    <link rel="stylesheet" href="{{ elixir('css/all.css') }}">

#### 为多个文件产生版本

你可以传递一个数组至 `version` 方法来为多个文件产生版本：

```javascript
elixir(function(mix) {
    mix.version(['css/all.css', 'js/app.js']);
});
```

一旦该文件被加上版本，你需要使用 `elixir` 辅助函数来产生哈希文件的正确链接。切记，你只需要传递未哈希文件的名称至 `elixir` 辅助函数。此函数会自动使用未哈希的名称来判断该文件为目前的哈希版本：

    <link rel="stylesheet" href="{{ elixir('css/all.css') }}">

    <script src="{{ elixir('js/app.js') }}"></script>

<a name="browser-sync"></a>
## BrowserSync

当你对前端资源进行修改后，BrowserSync 会自动刷新你的网页浏览器。你可以使用 `browserSync` 方法来告知 Elixir，当你运行 `gulp watch` 命令时启动 BrowserSync 服务器：

```javascript
elixir(function(mix) {
    mix.browserSync();
});
```

一旦你运行 `gulp watch`，就能使用连接端口 3000 启用浏览器同步并访问你的网页应用程序：`http://homestead.app:3000`。如果你在本机开发所使用的域名不是 `homestead.app`，那么你可以传递一个[选项](http://www.browsersync.io/docs/options/)的数组作为 `browserSync` 方法的第一个参数：

```javascript
elixir(function(mix) {
    mix.browserSync({
    	proxy: 'project.app'
    });
});
```

<a name="calling-existing-gulp-tasks"></a>
## 调用既有的 Gulp 任务

如果你需要在 Elixir 调用一个既有的 Gulp 任务，你可以使用 `task` 方法。举个例子，假设你有一个 Gulp 任务，当你调用时会输出一些简单的文本：

```javascript
gulp.task('speak', function() {
    var message = 'Tea...Earl Grey...Hot';

    gulp.src('').pipe(shell('say ' + message));
});
```

如果你希望在 Elixir 中调用这个任务，使用 `mix.task` 方法并传递该任务的名称作为该方法唯一的参数：

```javascript
elixir(function(mix) {
    mix.task('speak');
});
```

#### 自定义监控器

如果你想注册一个监控器让你的自定义任务能在每次文件改变时就运行，只需传递一个正则表达式作为 `task` 方法的第二个参数：

```javascript
elixir(function(mix) {
    mix.task('speak', 'app/**/*.php');
});
```

<a name="writing-elixir-extensions"></a>
## 编写 Elixir 扩展功能

如果你需要比 Elixir 的 `task` 方法更灵活的方案，你可以创建自定义的 Elixir 扩展功能。Elixir 扩展功能允许你传递参数至你的自定义任务。举例来说，你可以编写一个扩展功能，像是：

```javascript
// 文件：elixir-extensions.js

var gulp = require('gulp');
var shell = require('gulp-shell');
var Elixir = require('laravel-elixir');

var Task = Elixir.Task;

Elixir.extend('speak', function(message) {

    new Task('speak', function() {
        return gulp.src('').pipe(shell('say ' + message));
    });

});

// mix.speak('Hello World');
```

就是这样！注意，你的 Gulp 具体的逻辑必须被放置在 `Task` 第二个参数传递的构造器函数里面。你可以将此扩展功能放置在 Gulpfile 的上方，取而代之也可以导出至一个自定义任务的文件。举个例子，如果你将你的扩展功能放置在 `elixir-extensions.js` 文件中，那么你可以在 `Gulpfile` 中像这样引入该文件：

```javascript
// 文件：Gulpfile.js

var elixir = require('laravel-elixir');

require('./elixir-extensions')

elixir(function(mix) {
    mix.speak('Tea, Earl Grey, Hot');
});
```

#### 自定义监控器

如果你想在运行 `gulp watch` 时能够重新触发你的自定义任务，你可以注册一个监控器：

```javascript
new Task('speak', function() {
    return gulp.src('').pipe(shell('say ' + message));
})
.watch('./app/**');
```
