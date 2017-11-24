# Laravel 的 邮件发送功能

- [简介](#introduction)
    - [驱动前提](#driver-prerequisites)
- [生成 Mailable](#generating-mailables)
- [编写 Mailable](#writing-mailables)
    - [配置发送者](#configuring-the-sender)
    - [配置视图](#configuring-the-view)
    - [视图数据](#view-data)
    - [附件](#attachments)
    - [内部附件](#inline-attachments)
    - [自定义 SwiftMailer 消息](#customizing-the-swiftmailer-message)
- [Markdown 格式的邮件](#markdown-mailables)
    - [生成 Markdown 格式的邮件](#generating-markdown-mailables)
    - [编写 Markdown 格式的邮件](#writing-markdown-messages)
    - [自定义组件](#customizing-the-components)
- [在浏览器中预览邮件](#previewing-mailables-in-the-browser)
- [发送邮件](#sending-mail)
    - [队列邮件](#queueing-mail)
- [邮件与本地开发](#mail-and-local-development)
- [事件](#events)

<a name="introduction"></a>
## 简介

Laravel 基于 [SwiftMailer](http://swiftmailer.org) 函数库提供了一套干净、简洁的邮件 API ，Laravel 为 SMTP 、Mailgun 、SparkPost 、 Amazon SES 、 PHP 的 `mail` 函数及 `sendmail` 提供驱动，让你可以快速从本地或云端服务自由地发送邮件。

<a name="driver-prerequisites"></a>
### 驱动前提

基于 API 的驱动，例如 Mailgun 和 SparkPost 通常比 SMTP 服务器更简单快速。如果可能，你应该尽可能使用这些驱动。所有的 API 驱动都需要 Guzzle HTTP 函数库，你可以使用 Composer 包管理器安装它：

    composer require guzzlehttp/guzzle

#### Mailgun 驱动

要使用 Mailgun 驱动，首先必须安装 Guzzle，之后将 `config/mail.php` 配置文件中的  `driver` 选项设置为 `mailgun`。接下来，确认 `config/services.php` 配置文件包含以下选项：

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
    ],

#### SparkPost 驱动

要使用 SparkPost 驱动，首先必须安装 Guzzle，之后将 `config/mail.php` 配置文件中的  `driver` 选项设置为 `sparkpost`。接下来，确认 `config/services.php` 配置文件包含以下选项：

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
    ],

#### SES 驱动

要使用 Amazon SES 驱动，必须安装 PHP 的 Amazon AWS SDK。你可以在 `composer.json` 文件的 `require` 段落加入下面这一行并运行 `composer update` 命令：

    "aws/aws-sdk-php": "~3.0"

接下来，将 `config/mail.php` 配置文件中的 `driver` 设置为 `ses`。然后确认 `config/services.php` 配置文件包含下列选项：

    'ses' => [
        'key' => 'your-ses-key',
        'secret' => 'your-ses-secret',
        'region' => 'ses-region',  // e.g. us-east-1
    ],

<a name="generating-mailables"></a>
## 生成 Mailable

在 Laravel 中，每种类型的邮件都代表一个「Mailable」对象。这些对象存储在 `app/Mail` 目录中。如果在你的应用中没有看见这个目录，别担心，在首次使用 `make:mail` 命令创建 Mailable 类时这个目录会被创建，例如：

    php artisan make:mail OrderShipped

<a name="writing-mailables"></a>
## 编写 Mailable

所有的 「Mailable」类都在其 `build` 方法中完成配置。在这个方法里，你可以调用其他各种方法，如 `from` 、 `subject` 、 `view` 和 `attach` 来配置完成邮件的详情。

<a name="configuring-the-sender"></a>
### 配置发送者

#### 使用 `from` 方法

首先，演示配置邮件的发送者，也就是邮件的参数 「from」，既谁发送了邮件。有两种方法配置发送者。第一种是你可以在 `build` 方法中使用 `from` 方法：

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->view('emails.orders.shipped');
    }

#### 使用一个全局 `from` 地址

然而，如果应用使用相同的 `from` 地址，你每次发送邮件这种设置方式显得笨拙，替代的方法就是在 `config/mail.php` 配置文件中设置一个全局 `from` 地址，这个配置在没有单独指定 「from」时是默认的 `from` ：

    'from' => ['address' => 'example@example.com', 'name' => 'App Name'],

<a name="configuring-the-view"></a>
### 配置视图

在 `build`方法内，你可以使用 `view` 方法指定邮件的模板，以渲染邮件的内容。因为所有邮件都会使用 [Blade 模板](/docs/{{version}}/blade) 渲染内容，你能很容易的使用 Blade 模板引擎构建邮件的 HTML：

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped');
    }

> {tip} 你可以创建一个 `resources/views/emails` 目录来存放所有的邮件模板；然而，这不是强制要求，你可以在有的将邮件模板放在 `resources/views` 目录的任意位置。

#### 纯文本邮件

如果你想要定义一个纯文本邮件，你可以使用 `text` 方法。如同 `view` 方法，`text` 方法接受一个模板名称，这个名称告诉方法使用哪个模板来渲染邮件，可以自由定义邮件 HTML 和纯文本消息：

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->text('emails.orders.shipped_plain');
    }

<a name="view-data"></a>
### 视图数据

#### 通过公共属性

通常，你会希望传递一些数据来渲染你的邮件的 HTML 。那么有两种方法可以让视图获得数据，第一种，Mailable 类任何公共属性都可以在视图中使用。所以，例如你可以传递数据到 Mailable 类的构造函数并且在类中定义数据的公共属性：

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * order 实例。
         *
         * @var Order
         */
        public $order;

        /**
         * 创建一个新消息实例。
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * 构建消息。
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped');
        }
    }

一旦数据被设置为公共属性，它们将自动在视图中加载，所以你可以访问像访问其他 Blade 模板数据一样访问它们：

    <div>
        Price: {{ $order->price }}
    </div>

#### 通过 `with` 方法：

你可以使用 `with` 方法来传递数据给模板。一般情况下，你仍然是使用 Mailable 类的构造函数来接受数据传参。然而你需要为这些数据属性设置 `protected` 或 `private`  声明，否则这些数据会被自动加载到模板中。接下来你可以使用 `with` 方法接受键值数组传参来传递数据给模板，就如控制器里为视图传参一样：

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * order 实例。
         *
         * @var Order
         */
        protected $order;

        /**
         * 创建一个新消息实例。
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * 构建消息。
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->with([
                            'orderName' => $this->order->name,
                            'orderPrice' => $this->order->price,
                        ]);
        }
    }

一旦数据已经用 `with` 方法传递，它们将自动在视图中加载，所以你可以访问像访问其他 Blade 模板数据一样访问它们：

    <div>
        Price: {{ $orderPrice }}
    </div>

<a name="attachments"></a>
### 附件

要在邮件中加入附件，在 `build` 方法中使用 `attach` 方法。`attach` 方法接受文件的绝对路径作为它的第一个参数：

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file');
        }

附加文件到消息时，你也可以传递 `数组` 给 `attache` 方法作为第二个参数，以指定显示名称和 / 或是 MIME 类型：

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file', [
                            'as' => 'name.pdf',
                            'mime' => 'application/pdf',
                        ]);
        }

#### 原始数据附件

`attachData` 可以使用字节数据作为附件。例如，你可以使用这个方法将内存中生成而没有保存到磁盘中的 PDF 附加到邮件中。`attachData` 方法第一个参数接收原始字节数据，第二个参数为文件名，第三个参数接受一个数组以指定其他参数：

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attachData($this->pdf, 'name.pdf', [
                            'mime' => 'application/pdf',
                        ]);
        }

<a name="inline-attachments"></a>
### 内部附件

在邮件中嵌入内部图标通常是件麻烦事；然而 Laravel 提供了一个便利的方法，让你在邮件中附件图片并获取适当的 CID 。要嵌入内部图片，在邮件模板的 `$message` 变量上调用 `embed` 方法，Laravel 会自动让你所有邮件模板都能够获取到 `$message` 变量，所以不必担心如何手动传递它：

    <body>
        这是一张图片：

        <img src="{{ $message->embed($pathToFile) }}">
    </body>

#### 嵌入原始数据附件

如果你已经有想嵌入原始数据，希望嵌入邮件模板，你可以调用 `$message` 变量的 `embedData` 方法：

    <body>
        这是一张原始数据图片：

        <img src="{{ $message->embedData($data, $name) }}">
    </body>

<a name="customizing-the-swiftmailer-message"></a>
### 自定义 SwiftMailer 消息

`Mailable` 基类的 `withSwiftMessage` 方法允许你注册一个回调，该回调将在邮件发送之前调用，参数是原始 SwiftMailer 消息实例。这让你有机会在邮件发送之前自定义消息：

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            $this->view('emails.orders.shipped');

            $this->withSwiftMessage(function ($message) {
                $message->getHeaders()
                        ->addTextHeader('Custom-Header', 'HeaderValue');
            });
        }

<a name="markdown-mailables"></a>
## Markdown 格式的 Mailable 类

Markdown 格式的 Mailable 消息允许你从预编译的模板和你的 Mailable 类中的邮件提醒组件中受益。因为消息是用 Markdown 格式写的， Laravel 能为消息体渲染出漂亮、响应式的 HTML 模板，也能自动生成一个纯文本的副本。

<a name="generating-markdown-mailables"></a>
### 生成 Markdown 格式的 Mailable

要生成一个包含友好的 Markdown 模板的 Mailable 类，你在使用  `make:mail` 这个 Artisan 命令时，要加上 `--markdown` 选项：

    php artisan make:mail OrderShipped --markdown=emails.orders.shipped

然后，在使用 `build` 方法配置 Mailable 时，用 `markdown` 方法来换掉 `view` 方法， `markdown` 方法接受一个 Markdown 模板的名称和一个将在模板中可用的选项数组：

    /**
     * 构建消息。
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->markdown('emails.orders.shipped');
    }

<a name="writing-markdown-messages"></a>
### 编写 Markdown 格式的消息

Markdown Mailable 使用 Blade 组件和 Markdown 语法的组合，允许你轻松地构建邮件消息，同时利用 Laravel 的预制组件。

    @component('mail::message')
    # Order Shipped

    Your order has been shipped!

    @component('mail::button', ['url' => $url])
    View Order
    @endcomponent

    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent

#### 按钮组件

按钮组件渲染一个居中的连接按钮，组件接受两个参数，一个 `url` 和一个可选的 `color` 。支持的颜色有 `blue` 、 `green` 、 和 `red` 。你可以在邮件消息体中加入随便多个你想要的按钮。

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    View Order
    @endcomponent

#### 面板组件

面板组件将面板中给定的一块文字的背景渲染成跟其他的消息体背景稍微不同。这样可以让这块文字引起人们的注意。

    @component('mail::panel')
    This is the panel content.
    @endcomponent

#### 表格组件

表格组件允许你将一个 Markdown 格式的表格转换成一个 HTML 格式的表格。组件接受 Markdown 格式的表格作为它的内容。表格列对齐方式按照默认的 Markdown 对齐风格而定。

    @component('mail::table')
    | Laravel       | Table         | Example  |
    | ------------- |:-------------:| --------:|
    | Col 2 is      | Centered      | $10      |
    | Col 3 is      | Right-Aligned | $20      |
    @endcomponent

<a name="customizing-the-components"></a>
### 自定义组件

你或许会为了自定义而导出你应用中所有的 Markdown 邮件组件。要导出这些组件，使用 `vendor:publish` 这个 Artisan 命令来发布资源文件标签。

    php artisan vendor:publish --tag=laravel-mail

这个命令会发布 Markdown 邮件组件到 `resources/views/vendor/mail` 文件夹。而 `mail` 文件夹会包含一个 `html` 文件夹和一个 `markdown` 文件夹，每个文件夹都包含他们的可用组件的描述。你可以按照你的意愿自定义这些组件。

#### 自定义样式表 CSS

在导出组件之后，`resources/views/vendor/mail/html/themes` 文件夹将包含一个默认的 `default.css` 文件。你可以在这个文件中自定义 CSS ，你定义的这些样式将会在 Markdown 格式消息体转换成 HTML 格式时自动得到应用。

> {tip} 如果你想为 Markdown 组件构建一个全新的主题，只要写一个新的 CSS 文件，放在 `html/themes` 文件夹，然后在你的 `mail` 配置文件中改变 `theme` 选项就可以了。

<a name="previewing-mailables-in-the-browser"></a>
## 在浏览器中预览邮件

当你设计一个邮件模板时，可以很方便地在浏览器中快速预览渲染出来的邮件，就像典型的 Blade 模板一样。为此， Laravel 允许你直接从路由闭包或者控制器中返回任何的邮件。当邮件被返回时，他会被渲染并显示在浏览器中，允许你快速预览其设计，而不必发送到实际的电子邮件地址。

    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

<a name="sending-mail"></a>
## 发送邮件

要发送邮件，使用 `Mail` [facade](/docs/{{version}}/facades) 的 `to` 方法。 `to` 方法接受一个邮件地址，一个 user 实现或一个 users 集合。如果传递一个对象或集合，mailer 将自动使用 `email` 和 `name` 属性来设置邮件收件人，所以确保你的对象里有这些属性。一旦指定收件人，你可以传递一个实现到 Mailable 类的 `send` 方法：

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Mail\OrderShipped;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Mail;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  Request  $request
         * @param  int  $orderId
         * @return Response
         */
        public function ship(Request $request, $orderId)
        {
            $order = Order::findOrFail($orderId);

            // Ship order...

            Mail::to($request->user())->send(new OrderShipped($order));
        }
    }

当然，不局限于只使用「to」给收件人发送邮件，你可以通过一个单一的链式调用来自由的设置 「to」，「cc」和 「bcc」接收者：

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="queueing-mail"></a>
### 队列 Mail

#### 将邮件消息加入队列

由于发送消息会大幅延迟应用响应时间，许多开发者选择将邮件消息加入队列在后台进行发送。Laravel 使用内置的 [统一的队列 API](/docs/{{version}}/queues) 来轻松完成此工作。将邮件消息加入队列，使用 `Mail` facade 的 `queue` 方法：

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

这个方法会自动将工作加入队列，以便后台发送邮件。当然，在使用这个功能前，你需要 [设置你的队列](/docs/{{version}}/queues) 。

#### 延迟邮件消息队列

如果你希望延迟发送已加入队列的邮件消息，你可以使用 `later` 方法。`later` 第一个参数接受一个 `DateTime` 实现以告知这个消息应该何时发送：

    $when = Carbon\Carbon::now()->addMinutes(10);

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later($when, new OrderShipped($order));

#### 推送到特定队列

因为所有 Mailable 类是通过 `make:mail` 命令生成并使用 `Illuminate\Bus\Queueable` trait ，你可以在任何 Mailable 类实现中调用 `onQueue` 来指定队列名称，还有 `onConnection` 方法来指定队列链接名称：

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

#### 默认队列

如果你的 Mailable 类想要默认使用队列，你可以在类中实现 `ShouldQueue` 接口契约。现在，即便你调用 `send` 方法来发送邮件， Mailable 类仍将邮件放入队列中发送。

    use Illuminate\Contracts\Queue\ShouldQueue;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        //
    }

<a name="mail-and-local-development"></a>
## 邮件和本地开发

当应用处于开发阶段在发送邮件时，你或许不想真实的发送邮件到真实的邮件地址。Laravel 提供几种方法以在本地开发时真实发送 「失去能力」。

#### 日志驱动

代替真实发送，`log` 邮件驱动将所有邮件消息写入日志文件以供检查，需要更多根据环境来设置应用程序的信息，可参考 [配置文件](/docs/{{version}}/configuration#environment-configuration).

#### 通用收件者

另一个由 Laravel 提供的解决方案是设置一个通用邮件接收者，由框架发送邮件。这个方法将所有邮件发送到一个邮件地址，而不是发送给实际收件人。这可以通过 `config/mail.php` 配置文件的 `to` 选项来完成：

    'to' => [
        'address' => 'example@example.com',
        'name' => 'Example'
    ],

#### Mailtrap

最后，你可以使用像 [Mailtrap](https://mailtrap.io) 和 `smtp` 驱动来将你的邮件消息发送到一个 「虚假的」邮箱中，你却可以在一个真实的邮件客户端中查看它们。这个方法的好处是让你可以在 Mailtrap 的消息阅读器中查看最终的实际邮件。

<a name="events"></a>
## 事件

Laravel 会在发送邮件消息之前触发一个事件。切记，这个事件只会在邮件 *发送* 时触发，在加入队列时不触发。你可以在你的 `EventServiceProvider` 注册一个事件监听器：

    /**
     * 应用事件监听映射。
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Mail\Events\MessageSending' => [
            'App\Listeners\LogSentMessage',
        ],
    ];

## 译者署名

| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@qufo](https://github.com/qufo)  | <img class="avatar-66 rm-style" src="https://avatars1.githubusercontent.com/u/2526883?v=3&s=460?imageView2/1/w/100/h/100">  |  翻译  | 欢迎共同探讨。[@Qufo](https://github.com/qufo) |
| [@limxx](https://github.com/limxx)  | <img class="avatar-66 rm-style" src="https://avatars0.githubusercontent.com/u/16585030?v=4&s=400">  |  翻译  | Winter is coming. |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org
