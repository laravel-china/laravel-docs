# Laravel 的 邮件发送功能

- [简介](#introduction)
    - [准备驱动](#driver-prerequisites)
- [生成 mailables 类](#generating-mailables)
- [编写 mailables 类](#writing-mailables)
    - [配置邮件发件人](#configuring-the-sender)
    - [配置视图](#configuring-the-view)
    - [向视图传递数据](#view-data)
    - [附件](#attachments)
    - [行内附件](#inline-attachments)
- [Markdown 格式的邮件](#markdown-mailables)
    - [生成 Markdown 格式的邮件](#generating-markdown-mailables)
    - [编写 Markdown 格式的邮件](#writing-markdown-messages)
    - [自定义组件](#customizing-the-components)
- [发送邮件](#sending-mail)
    - [邮件队列](#queueing-mail)
- [邮件与本地开发](#mail-and-local-development)
- [事件](#events)

<a name="introduction"></a>
## 简介

Laravel 基于 [SwiftMailer](http://swiftmailer.org) 函数库提供了一套干净，简洁的邮件 API ， Laravel 为 SMTP，Mailgun，SparkPost， Amazon SES，PHP 的 `mail` 函数及 `sendmail` 提供驱动，让你可以快速从本地或云端服务自由地发送邮件。

<a name="driver-prerequisites"></a>
### 准备驱动

基于 API 驱动的邮件系统，例如 Mailgun 和 SparkPost 通常比 SMTP 服务来得简单和快速。如果有可能，你应该选择基于 API 驱动的邮件系统。所有的基于 API 驱动的邮件系统都需要 Guzzle HTTP 库，这个库可以通过 Composer 包管理器来安装：

    composer require guzzlehttp/guzzle

#### Mailgun 驱动

要使用 Mailgun 驱动，首先要安装 Guzzle ，再在你的 `config/mail.php` 配置文件里指定 `driver` 为 `mailgun` 。然后，确保你的 `config/services.php` 配置文件包含以下选项：

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
    ],

#### SparkPost 驱动

要使用 SparkPost 驱动，首先要安装 Guzzle ，再在你的 `config/mail.php` 配置文件里指定 `driver` 为 `sparkpost` 。然后，确保你的 `config/services.php` 配置文件包含以下选项：

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
    ],

#### SES 驱动

要使用 Amazon SES 驱动，首先要安装 Amazon AWS SDK for PHP ，要安装这个库，你可以在你的 `composer.json` 文件的 `require` 节里加入以下行，然后运行 `composer update` 命令：

    "aws/aws-sdk-php": "~3.0"

然后，在你的 `config/mail.php` 配置文件里指定 `driver` 为 `ses` 。并确保你的 `config/services.php` 配置文件包含以下选项：

    'ses' => [
        'key' => 'your-ses-key',
        'secret' => 'your-ses-secret',
        'region' => 'ses-region',  // e.g. us-east-1
    ],

<a name="generating-mailables"></a>
## 生成 Mailables 类

在 Laravel 中，每种类型的邮件被描述成一个「mailables」类。这些类保存在 `app/Mail` 文件夹。如果在你的应用中没有看到这个文件夹也别担心，当你用 `make:mail` 命令创建第一个「mailables」类时，这个目录会自动创建：

    php artisan make:mail OrderShipped

<a name="writing-mailables"></a>
## 编写 Mailables 类

所有「mailables」类都在 `build` 方法中进行配置。在这个方法中，你可以调其他的方法，比如 `from`, `subject`, `view`, 和 `attach` 来配置邮件的详情和发送方式。

<a name="configuring-the-sender"></a>
### 配置邮件发送人

#### 使用 `from` 方法

首先，让我们解释下如何配置邮件发件人。或者说，邮件的「from」是谁。有两种方法指定发件人，第一种，你可以在你的 mailable 类的 `build` 方法中调用 `from` 方法：

    /**
     * 构建消息。
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->view('emails.orders.shipped');
    }

#### 使用一个全局的 `from` 地址

然而，如果你整个应用使用相同的发件人「from」地址，在你生成的每个 mailables 类中调用 `from` 方法就显得太麻烦了。作为替代，你可以在你的 `config/mail.php` 配置文件中指定一个全局的「from」地址。在 mailable 类中没有另外指定「from」地址的时候，就会使用这个默认的地址：

    'from' => ['address' => 'example@example.com', 'name' => 'App Name'],

<a name="configuring-the-view"></a>
### 配置视图

在 mailable 类的 `build` 方法中，你可以使用 `view` 方法来指定使用哪个模板来渲染邮件的内容，因为通常情况下每个邮件都使用 [Blade 模板](/docs/{{version}}/blade) 来渲染内容，你可以全权控制 Blade 模板来构建你邮件的 HTML :

    /**
     * 构建消息。
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped');
    }

> {tip} 你可以创建一个 `resources/views/emails` 目录来存放你所有的邮件模板；或者，放在 `resources/views` 目录下的任何位置都可以。

#### 纯文本邮件

如果你想要定义一个纯文本版的邮件，你可以使用 `text` 方法。和 `view` 方法一样，`text` 方法接受一个模板名称，程序将用这个模板来渲染邮件内容。这样你就可以方便地发送纯文本邮件或是 HTML 格式的邮件：

    /**
     * 构建消息。
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->text('emails.orders.shipped_plain');
    }

<a name="view-data"></a>
### 向视图传递数据

#### 通过公共属性

通常，你会希望传递一些数据到你的视图中，这样你就可以利用这些数据来渲染邮件的 HTML 内容。有两种方法可以办到。首先，所有在你的 mailable 中定义的公共属性将会自动地应用到你的视图中。比如说，你可以通过传递一些数据到构造函数中并且在构造函数中把他们设置为公共属性：

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
         * order 实例.
         *
         * @var Order
         */
        public $order;

        /**
         * 创建一个新的消息实例。
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

一旦数据被设置为公共属性，在视图文件中就可以用了，你可以在 Blade 模板中象访问其他数据一样访问他们：

    <div>
        Price: {{ $order->price }}
    </div>

#### 通过 `with` 方法：

如果你想在数据发到模板之前自定义他的格式，你可以手工地通过 `with` 方法把你的数据传到视图中去。一般而言，你仍通过 mailable 类的构造函数传递数据，但当你要把这些数据设置为 `protected` 或 `private` 属性时，数据不会自动传递到模板了。那时，你可以调用 `with` 方法，把你希望在视图中使用的数据放在数组中传递到模板去。

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
         * 创建新的消息实例。
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

一旦数据被通过 `with` 方法传递进视图，在视图文件中就可以用了，你可以在 Blade 模板中象访问其他数据一样访问他们：

    <div>
        Price: {{ $orderPrice }}
    </div>

<a name="attachments"></a>
### 附件

要在邮件中加入附件，可以使用 mailable 类的 `build` 方法，在这个方法里通过 `attach` 方法来附加， `attach` 方法的第1个参数是附件的绝对路径。

        /**
         * 构建消息。
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file');
        }

当附加一个附件到邮件消息体时，你可以通过传递一个数组作为 `attach` 的第2个参数，来指定显示名称或附件文件的 MIME 类型。

        /**
         * 构建消息。
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

`attachData` 方法可以使用字节数据当附件。比如，你在内存里构建了一个 pdf 文件，不想保存到硬盘上，却想附加在邮件中当附件。 `attachData` 方法接受字节数据为他的第1个参数，文件名作为其第2个参数，一个包含选项值的数组作为其第3个参数：

        /**
         * 构建消息。
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
### 行内附件

要在邮件中嵌入行内图片通常是个麻烦事；然而 Laravel 提供了一种简单的方法让图片附加到你的邮件中并获得相应的 CID 。要嵌入一个行内图片，在邮件模板的 `$message` 变量上调用 `embed` 方法。 Laravel 已经让 `$message` 变量在所有的邮件模板中都自动可用，所以你不用担心如何手工传递它：

    <body>
        Here is an image:

        <img src="{{ $message->embed($pathToFile) }}">
    </body>

#### 嵌入原始数据附件

如果你已有了原始数据并且想嵌入到邮件模板中，你可以调用 `$message` 变量的 `embedData` 方法：

    <body>
        Here is an image from raw data:

        <img src="{{ $message->embedData($data, $name) }}">
    </body>

<a name="markdown-mailables"></a>
## Markdown 格式的 Mailables 类

Markdown 格式的 mailable 消息允许你从预编译的模板和你的 mailables 类中的邮件提醒组件中受益。因为消息是用 Markdown 格式写的， Laravel 能为消息体渲染出漂亮、响应式的 HTML 模板，也能自动生成一个纯文本的副本。

<a name="generating-markdown-mailables"></a>
### 生成 Markdown 格式的 Mailables

要生成一个包含友好的 Markdown 模板的 mailable 类，你在使用  `make:mail` 这个 Artisan 命令时，要加上 `--markdown` 选项：

    php artisan make:mail OrderShipped --markdown=emails.orders.shipped

然后，在使用 `build` 方法配置 mailable 时，用 `markdown` 方法来换掉 `view` 方法， `view` 方法接受一个 Markdown 模板的名称和一个将在模板中可用的选项数组：

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

Markdown mailables 使用 Blade 组件和 Markdown 语法的组合，允许你轻松地构建邮件消息，同时利用 Laravel 的预制组件。

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

按钮连组件渲染一个居中的连接按钮，组件接受两个参数，一个 `url` 和一个可选的 `color` 。支持的颜色有 `blue`， `green`， 和 `red` 。你可以在邮件消息体中加入随便多个你想要的按钮。

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

<a name="sending-mail"></a>
## 发送邮件

要发送一个邮件，使用 `Mail` [facade](/docs/{{version}}/facades) 的 `to` 方法。 `to` 方法接受一个邮件地址，一个 user 实例，或者 users 集合。如果你传进一个对象或者对象的集合， mailer 将自动使用他们的 `email` 和 `name` 属性来设定为邮件的收件人，所以要确定传进去的对象这两个属性可用。一旦指定了收件人，你可以传递一个你的 mailable 类的实例给 `send` 方法。

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
         * 订单发货。
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

当然，发邮件时不只限定于使用「to」来指定收件人，你可以链式地使用「to」,「cc」和「bcc」来指定相应的收件人。

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="queueing-mail"></a>
### 邮件队列

#### 将邮件消息加入队列

因为发邮件会拖长应用程序的响应时间，许多开发者选择将邮件放在队列中在后台进行发送。 Laravel 使用内置的 [统一队列API](/docs/{{version}}/queues) 来轻松完成此工作。要把邮件消息加入到队列，在指定收件人后使用 `mail` facade 的 `queue` 方法：

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

这个方法会自动把发邮件的任务推送到任务队列中以便后台发送，当然在使用这个功能之前，你得先 [配置你的队列](/docs/{{version}}/queues) 。

#### 延迟邮件消息队列

如果你希望延迟发送队列里的邮件消息，你可以使用 `later` 方法。 `later` 方法的第1个参数接受一个 `DateTime` 实例指定邮件消息要在什么时候发送。

    $when = Carbon\Carbon::now()->addMinutes(10);

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later($when, new OrderShipped($order));

#### 推送到指定的队列

因为所有的 mailable 类都是通过 `make:mail` 命令生成的，他们都使用 `Illuminate\Bus\Queueable` trait ，你可以在任何的 mailable 实例中调用 `onQueue` 方法来指定队列名称，调用 `onConnection` 方法来指定连接名称。

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

#### 默认队列

如果你的 mailable 类想要默认使用队列，你可以在类中实现 `ShouldQueue` 接口契约。现在，即便你调用 `send` 方法来发送邮件， mailable 类仍将邮件放入队列中发送。

    use Illuminate\Contracts\Queue\ShouldQueue;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        //
    }

<a name="mail-and-local-development"></a>
## 邮件和本地开发

当开发一个发送邮件的应用时，你或许不想将邮件发送到一个真的邮件地址上。 Laravel 提供了几种方法在本地开发时「停用」真实的邮件发送过程。

#### 日志驱动

作为替代， `log` 邮件驱动会把所有的邮件信息写入到日志文件以供检查。想知道在每个环境中如何配置，可参考  [配置文档](/docs/{{version}}/configuration#environment-configuration).

#### 通用收件人

Laravel 提供的另一个解决方案是在整个框架中指定一个通用的收件人。使用这种方法，应用中生成的所有邮件将会发送到这个指定的地址上，而不是发送给你在邮件程序中指定的真正收件人。这可以通过在 `config/mail.php` 配置文件中指定 `to` 选项来完成。

    'to' => [
        'address' => 'example@example.com',
        'name' => 'Example'
    ],

#### Mailtrap

最后，你可以使用象 [Mailtrap](https://mailtrap.io) 和 `smtp` 驱动来将你的邮件发送到一个 「虚假的」 邮箱，你却可以在一个真实的邮件客户端中查看它们。这个方法的好处是可以让你在 Mailtrap 的邮件查看器中看到最终的邮件。

<a name="events"></a>
## 事件

Laravel 会在发送邮件消息之前触发一个事件。记住，这个事件是在邮件 *发送* 的时候触发，而不是在邮件消息推送到队列时触发。你可以在你的 `EventServiceProvider` 中注册一个事件监听器来监听这个事件：

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
