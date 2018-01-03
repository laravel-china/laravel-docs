# Laravel 的前端资源处理 JavaScript＆CSS 构建

- [简介](#introduction)
- [编写 CSS](#writing-css)
- [编写 JavaScript](#writing-javascript)
    - [编写 Vue 组件](#writing-vue-components)
    - [使用 React](#using-react)

<a name="introduction"></a>
## 简介

Laravel 并没有规定你使用哪个 JavaScript 或 CSS 预处理器，不过它还是提供了对大多数应用都很适用的 [Bootstrap](http://getbootstrap.com) 和 [Vue](https://vuejs.org) 来作为默认的起点。默认情况下，Laravel 使用 [NPM](https://npmjs.org) 安装这两个前端依赖。

#### CSS

[Laravel Mix](/docs/{{version}}/mix) 提供了一个内容丰富且简洁的 API 用以编译 SASS 或 Less，这些 CSS 预处理语言扩充了 CSS 语言，增加了诸如变量、mixins 及其他一些功能强大的扩展让编写 CSS 代码变得更加有趣。尽管我们会在这里简要的讨论 CSS 编译的内容，但你应该查阅 [Laravel Mix 文档](/docs/{{version}}/mix) 获取更多关于编译 SASS 或 Less 的信息。

#### JavaScript

Laravel 并不需要你使用特定的 JavsScript 框架或者库来构建应用程序。事实上，你也可以完全不用 JavaScript。不过，Laravel 自带了一些基本的脚手架，能更容易地使用 [Vue](https://vuejs.org) 编写 JavaScript。Vue 提供了健壮的组件化 API 用来构建强大的 JavaScript 应用程序。与 CSS 一样，我们可以使用 Laravel Mix 轻松地将 JavaScript 组件编译成一个浏览器可使用的 JavaScript 文件。

#### 移除前端脚手架

如果要从你的应用程序中移除前端脚手架，可以使用 Artisan 命令 `preset`。该命令加上 `none` 选项时，将从应用程序中删除 Bootstrap 和 Vue 脚手架，只留下一个空白的 SASS 文件和一些常用的 JavaScript 实用程序库:

    php artisan preset none
<a name="writing-css"></a>
## 编写 CSS

 Laravel 的 `package.json` 文件引入了 `bootstrap-sass` 依赖包，可以帮助你使用 Bootstrap 构建应用程序的前端原型。不过，你可以根据自己应用程序的需要在 `package.json` 灵活的添加或者移除依赖包。你不是一定要使用 Bootstrap 框架来构建你的 Laravel 应用程序，它只是给那些想用它的人提供一个很好的起点。

在编译 CSS 代码之前，需要你先使用  [Node 包管理工具(NPM)](https://www.npmjs.org/) 来安装项目的前端依赖：

    npm install

使用 `npm install` 成功安装依赖后，你就可以使用 [Laravel Mix](/docs/{{version}}/mix#working-with-stylesheets) 来将 SASS 文件编译为纯 CSS。`npm run dev` 命令会处理 `webpack.mix.js` 文件中的指令。通常情况下，编译好的 CSS 代码会被放置在 `public/css` 目录：

    npm run dev

Laravel 自带的 `webpack.mix.js` 默认会编译 `resources/assets/sass/app.scss` SASS 文件。`app.scss` 文件导入了一个包含 SASS 变量的文件，并加载 Bootstrap，这对大多数程序来说很方便。你也可以根据自己的需要去定制 `app.scss` 文件的内容，甚至使用完全不同的预处理器，详细配置见 [配置 Laravel Mix](/docs/{{version}}/mix)。

<a name="writing-javascript"></a>
## 编写 JavaScript

在项目根目录中的 `package.json` 可以找到应用程序的所有 JavaScript 依赖。它和 `composer.json` 文件类似，不同的是它指定的是 JavaScript 的依赖而不是 PHP 的依赖。使用 [Node 包管理器 (NPM)](https://npmjs.org) 来安装这些依赖包：

    npm install
> {tip} 默认情况下，Laravel `package.json` 文件默认会包含一些依赖包来帮助你构建 JavaScript 应用程序，例如 `vue` 和 `axios` 。你可以根据需要在 `package.json` 中添加或者移除依赖。

安装依赖之后，就可以使用  `npm run dev`  命令来 [编译资源文件](/docs/{{version}}/mix) 。Webpack 是一个为现代 JavaScript 应用而生的模块构建工具。当你运行 `npm run dev` 命令时，Webpack 会执行 `webpack.mix.js` 文件中的指令：

    npm run dev

默认情况下，Laravel 的 `webpack.mix.js` 会编译 SASS 文件和 `resources/assets/js/app.js` 文件。你可以在 `app.js` 文件中注册你的 Vue 组件，或者如果你更喜欢其他的框架，请配置自己的 JavaScript 应用程序。编译好的 JavaScript 文件通常会放置在 `public/js` 目录。

> {tip} `app.js` 会加载 `resources/assets/js/bootstrap.js` 文件来启动并 配置 Vue、Axios、jQuery 以及其他的 JavaScript 依赖。如果你有其他的 JavaScript 依赖需要去配置，你也可以在这个文件中完成。

<a name="writing-vue-components"></a>
### 编写 Vue 组件

新 Laravel 程序默认会在 `resources/assets/js/components` 中包含一个 `ExampleComponent.vue` 的 Vue 组件。`ExampleComponent.vue` 文件是在同一文件中定义其 JavaScript 和 HTML 模板的 [单文件 Vue 组件](https://vuejs.org/guide/application.html#Single-File-Components) 的示例。它为构建 JavaScript 驱动的应用程序提供了非常方便的方法。这个示例组件已经在 `app.js` 文件中注册：

    Vue.component('example-component', require('./components/ExampleConponent.vue'));

在应用程序中使用组件，你只需要简单的将其放到你的 HTML 模板之中。例如，运行 Artisan 命令 `make:auth` 去生成应用的用户认证和注册的框架页面后，可以把组件放到 `home.blade.php` Blade 模板中：

    @extends('layouts.app')

    @section('content')
        <example-component></example-component>
    @endsection

> {tip} 谨记，每次修改 Vue 组件后都应该运行 `npm run dev` 命令。或者，你可以使用 `npm run watch` 命令来监控并在每次文件被修改时自动重新编译组件。

当然，如果你对编写 Vue 组件的内容感兴趣，你可以读一下 [Vue 文档](http://vuejs.org/guide/)，该文档内容全面又易于阅读。

<a name="using-react"></a>
### 使用 React
如果你喜欢用 React 来构建你的 JavaScript 应用程序，Laravel 很容易就能将 Vue 脚手架替换为 React 脚手架。在任何新建的 Laravel 应用程序下，你可以用 `preset` 命令加 `react` 选项:

    php artisan preset react

该命令将移除 Vue 脚手架并用 React 脚手架替换， 包括 Laravel 自带的示例组件。

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@JokerLinly](https://laravel-china.org/users/5350)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/5350_1481857380.jpg">  | Review | Stay Hungry. Stay Foolish. |

---

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
>
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
>
> 文档永久地址： https://d.laravel-china.org
