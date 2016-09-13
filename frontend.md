# JavaScript 与 CSS

- [简介](#introduction)
- [编写 CSS](#writing-css)
- [编写 JavaScript](#writing-javascript)
    - [编写 Vue 组件](#writing-vue-components)

<a name="introduction"></a>
## 简介

Laravel 并没有规定你使用哪个 JavaScript 或 CSS 预处理器，不过默认提供了对大多数应用都很适用的 [Bootstrap](http://getbootstrap.com) 和 [Vue](https://vuejs.org) 来作为起点。 Laravel 默认使用 [NPM](https://npmjs.org) 安装这些前端的依赖。

#### CSS

[Laravel Elixir](/docs/{{version}}/elixir) 提供了一个简洁、富有表现力的 API 来编译 SASS 或 Less，这些 CSS 预处理语言扩充了 CSS 语言，增加了诸如变量、混合（mixins）及其他一些强大的功能让编写 CSS 代码变得更加有趣。

尽管我们会在这里简要的讨论 CSS 编译的内容，但是，你应该访问 [Laravel Elixir 文档](/docs/{{version}}/elixir) 获取更多关于编译 SASS 或 Less 的信息。

#### JavaScript

Laravel 并不需要你使用特定的 JavsScript 框架或者库来构建应用程序。事实上，你也可以完全不用 JavaScript。不过，Laravel 自带了用 [Vue](https://vuejs.org) 实现的基本脚手架代码来帮你更轻松的开始现代化 JavaScript 编码。Vue 提供了强大的组件化 API 用来构建健壮的 JavaScript 应用程序。

<a name="writing-css"></a>
## 编写 CSS

在 Laravel 的根目录中的 `Package.json` 文件引入了 `bootstrap-sass` 依赖包可以帮助你使用 Bootstrap 制作应用程序的前端原型。不过，你可以根据自己应用程序的需要在 `package.json` 灵活的添加或者移除依赖包。使用 Bootstrap 框架来构建你的 Laravel 应用程序并不是必选项，它只是给那些想用它的人提供一个很好的起点。

编译 CSS 代码之前，需要你先使用 NPM 来安装前端的依赖：

    npm install

使用 `npm install` 成功安装依赖之后，你就可以使用 [Gulp](http://gulpjs.com/) 来将 SASS 文件编译为纯 CSS。`gulp` 命令会执行处理 `gulpfile.js` 文件中的指令。通常情况下，编译好的 CSS 代码会被放置在 `public/css` 目录：

    gulp

默认情况下，`gulpfile.js` 会编译 `resources/assets/sass/app.scss` SASS 文件。`app.scss` 文件导入了一个包含 SASS 变量的文件，加载了 Bootstrap 框架，这对大多数程序来说是一个很好的出发点。你也可以根据自己的需要去定制 `app.scss` 文件的内容，甚至使用完全不同的预处理器，详细配置见 [配置 Laravel Elixir](/docs/{{version}}/elixir)。

<a name="writing-javascript"></a>
## 编写 JavaScript

在项目根目录中的 `package.json` 可以找到应用程序的所有 JavaScript 依赖。它和 `composer.json` 文件类似，不同的是它指定的是 JavaScript 的依赖而不是 PHP 的依赖。使用 [Node 包管理器 (NPM)](https://npmjs.org) 来安装这些依赖包：

    npm install

Laravel `package.json` 文件默认会包含一些依赖包来帮助你开始建立 JavaScript 应用程序，例如 `vue` 和 `vue-resource` 。你可以根据自己程序的需要在 `package.json` 中添加或者移除依赖。

成功安装依赖之后，你就可以使用  `gulp`  命令来 [编译资源文件](/docs/{{version}}/elixir) 。Gulp 是一个 JavaScript 的命令行构建系统。当你运行 `Gulp` 命令时，Gulp 会执行 `gulpfile.js` 文件中的指令：

    gulp

默认情况下，`gulpfile.js` 会编译 SASS 文件和 `resources/assets/js/app.js` 文件。你可以在 `app.js` 文件中注册你的 Vue 组件，或者如果你更喜欢其他的框架，也可以在这里进行配置。编译好的 JavaScript 文件通常会放置在 `public/js` 目录。

> {tip} `app.js` 会加载 `resources/assets/js/bootstrap.js` 文件来启动、 配置 Vue，Vue Resource，jQuery，以及其他的 JavaScript 依赖。如果你有额外的 JavaScript 依赖需要去配置，你也可以在这个文件中完成。

<a name="writing-vue-components"></a>
### 编写 Vue 组件

全新安装的 Laravel 程序默认会在 `resources/assets/js/components` 中包含一个 `Example.vue` 的 Vue 组件。`Example.vue` 文件是一个 [单文件 Vue 组件](https://vuejs.org/guide/application.html#Single-File-Components) 的示例，单文件 Vue 组件允许我们在同一个文件中编写 JavaScript 和 HTML 模板，它提供了一种非常方便的方式去构建 JavaScript 驱动的应用程序。这个示例组件注册在 `app.js` 文件。

    Vue.component('example', require('./components/Example.vue'));

在应用程序中使用示例组件，你只需要简单的将其放到你的 HTML 模板之中。例如，在你运行 `make:auth` Artisan 命令去生成应用的用户认证和注册的脚手架页面后，你可以把组件放到 `homde.blade.php` Blade 模板：

    @extends('layouts.app')

    @section('content')
        <example></example>
    @endsection

> {tip} 谨记，你需要在每次修改 Vue 组件后都需要运行 `gulp` 命令。或者，你可以使用 `gulp watch` 命令来监控并在每次文件被修改时自动重新编译组件。

当然，如果你对学习更多编写 Vue 组件的内容感兴趣，你可以读一下 [Vue 官方引导文档](http://vuejs.org/guide/)，它提供了一个透彻、易懂的文档让你一览 Vue 框架的概貌。

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@JobsLong](https://phphub.org/users/56)  | <img class="avatar-66 rm-style" src="http://i4.buimg.com/567571/a3dc28a55fdb2b7a.png">  |  翻译  | 个人主页：[http://jobslong.com](http://jobslong.com)  |
