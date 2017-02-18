# Laravel 的 邮箱发送功能

- [简介](#introduction)
    - [准备驱动](#driver-prerequisites)
- [生成 mailables 类](#generating-mailables)
- [编写 mailables 类](#writing-mailables)
    - [配置邮件发送人](#configuring-the-sender)
    - [配置视图](#configuring-the-view)
    - [向视图传递数据](#view-data)
    - [附件](#attachments)
    - [Inline Attachments](#inline-attachments)
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

要使用 Amazon SES驱动，首先要安装 Amazon AWS SDK for PHP ，要安装这个库，你可以在你的 `composer.json` 文件的 `require` 节里加入以下行，然后运行 `composer update` 命令：

    "aws/aws-sdk-php": "~3.0"

然后，在你的 `config/mail.php` 配置文件里指定 `driver` 为 `ses` 。并确保你的 `config/services.php` 配置文件包含以下选项：

    'ses' => [
        'key' => 'your-ses-key',
        'secret' => 'your-ses-secret',
        'region' => 'ses-region',  // e.g. us-east-1
    ],

<a name="generating-mailables"></a>
## 生成 Mailables 类

在 Laravel 中，每种类型的邮件发送程序被描述成一个「mailables」类。这些类保存在 `app/Mail` 文件夹。如果在你的应用中没有看到这个文件夹也别担心，当你用 `make:mail` 命令创建第一个「mailables」类时，这个目录会自动创建：

    php artisan make:mail OrderShipped

<a name="writing-mailables"></a>
## 编写 Mailables 类

所有「mailables」类都在 `build` 方法中进行配置。在这个方法中，你可以调其他的方法，比如 `from`, `subject`, `view`, 和 `attach` 来配置邮件的详情和发送方式。

<a name="configuring-the-sender"></a>
### 配置邮件发送人

#### 使用 `from` 方法

首先，让我们解释下如何配置邮件发件人。或者说，邮件的「from」是谁。有两种方法指定发件人，第一种，你可以在你的 mailable 类的 `build` 方法中调用 `from` 方法：

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

#### 使用一个全局的 `from` 地址

然而，如果你整个应用使用相同的发件人「from」地址，在你生成的每个 mailables 类中调用 `from` 方法就显得太麻烦了。作为替代，你可以在你的 `config/mail.php` 配置文件中指定一个全局的「from」地址。在 mailable 类中没有另外指定「from」地址的时候，就会使用这个默认的地址：

    'from' => ['address' => 'example@example.com', 'name' => 'App Name'],

<a name="configuring-the-view"></a>
### 配置视图

在 mailable 类的 `build` 方法中，你可以使用 `view` 方法来指定使用哪个模板来渲染邮件的内容，因为通常情况下每个邮件都使用 [Blade 模板](/docs/{{version}}/blade) 来渲染内容，你可以全权控制 Blade 模板来构建你邮件的 HTML :

    /**
     * Build the message.
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
### 向视图传递数据

#### 通过公共属性

Typically, you will want to pass some data to your view that you can utilize when rendering the email's HTML. There are two ways you may make data available to your view. First, any public property defined on your mailable class will automatically be made available to the view. So, for example, you may pass data into your mailable class' constructor and set that data to public properties defined on the class:

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
         * The order instance.
         *
         * @var Order
         */
        public $order;

        /**
         * Create a new message instance.
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped');
        }
    }

Once the data has been set to a public property, it will automatically be available in your view, so you may access it like you would access any other data in your Blade templates:

    <div>
        Price: {{ $order->price }}
    </div>

#### Via The `with` Method:

If you would like to customize the format of your email's data before it is sent to the template, you may manually pass your data to the view via the `with` method. Typically, you will still pass data via the mailable class' constructor; however, you should set this data to `protected` or `private` properties so the data is not automatically made available to the template. Then, when calling the `with` method, pass an array of data that you wish to make available to the template:

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
         * The order instance.
         *
         * @var Order
         */
        protected $order;

        /**
         * Create a new message instance.
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Build the message.
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

Once the data has been passed to the `with` method, it will automatically be available in your view, so you may access it like you would access any other data in your Blade templates:

    <div>
        Price: {{ $orderPrice }}
    </div>

<a name="attachments"></a>
### Attachments

To add attachments to an email, use the `attach` method within the mailable class' `build` method. The `attach` method accepts the full path to the file as its first argument:

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

When attaching files to a message, you may also specify the display name and / or MIME type by passing an `array` as the second argument to the `attach` method:

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

#### Raw Data Attachments

The `attachData` method may be used to attach a raw string of bytes as an attachment. For example, you might use this method if you have generated a PDF in memory and want to attach it to the email without writing it to disk. The `attachData` method accepts the raw data bytes as its first argument, the name of the file as its second argument, and an array of options as its third argument:

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
### Inline Attachments

Embedding inline images into your emails is typically cumbersome; however, Laravel provides a convenient way to attach images to your emails and retrieving the appropriate CID. To embed an inline image, use the `embed` method on the `$message` variable within your email template. Laravel automatically makes the `$message` variable available to all of your email templates, so you don't need to worry about passing it in manually:

    <body>
        Here is an image:

        <img src="{{ $message->embed($pathToFile) }}">
    </body>

#### Embedding Raw Data Attachments

If you already have a raw data string you wish to embed into an email template, you may use the `embedData` method on the `$message` variable:

    <body>
        Here is an image from raw data:

        <img src="{{ $message->embedData($data, $name) }}">
    </body>

<a name="markdown-mailables"></a>
## Markdown Mailables

Markdown mailable messages allow you to take advantage of the pre-built templates and components of mail notifications in your mailables. Since the messages are written in Markdown, Laravel is able to render beautiful, responsive HTML templates for the messages while also automatically generating a plain-text counterpart.

<a name="generating-markdown-mailables"></a>
### Generating Markdown Mailables

To generate a mailable with a corresponding Markdown template, you may use the `--markdown` option of the `make:mail` Artisan command:

    php artisan make:mail OrderShipped --markdown=emails.orders.shipped

Then, when configuring the mailable within its `build` method, call the `markdown` method instead of the `view` method. The `markdown` methods accepts the name of the Markdown template and an optional array of data to make available to the template:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->markdown('emails.orders.shipped');
    }

<a name="writing-markdown-messages"></a>
### Writing Markdown Messages

Markdown mailables use a combination of Blade components and Markdown syntax which allow you to easily construct mail messages while leveraging Laravel's pre-crafted components:

    @component('mail::message')
    # Order Shipped

    Your order has been shipped!

    @component('mail::button', ['url' => $url])
    View Order
    @endcomponent

    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent

#### Button Component

The button component renders a centered button link. The component accepts two arguments, a `url` and an optional `color`. Supported colors are `blue`, `green`, and `red`. You may add as many button components to a message as you wish:

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    View Order
    @endcomponent

#### Panel Component

The panel component renders the given block of text in a panel that has a slightly different background color than the rest of the message. This allows you to draw attention to a given block of text:

    @component('mail::panel')
    This is the panel content.
    @endcomponent

#### Table Component

The table component allows you to transform a Markdown table into an HTML table. The component accepts the Markdown table as its content. Table column alignment is supported using the default Markdown table alignment syntax:

    @component('mail::table')
    | Laravel       | Table         | Example  |
    | ------------- |:-------------:| --------:|
    | Col 2 is      | Centered      | $10      |
    | Col 3 is      | Right-Aligned | $20      |
    @endcomponent

<a name="customizing-the-components"></a>
### Customizing The Components

You may export all of the Markdown mail components to your own application for customization. To export the components, use the `vendor:publish` Artisan command to publish the `laravel-mail` asset tag:

    php artisan vendor:publish --tag=laravel-mail

This command will publish the Markdown mail components to the `resources/views/vendor/mail` directory. The `mail` directory will contain a `html` and a `markdown` directory, each containing their respective representations of every available component. You are free to customize these components however you like.

#### Customizing The CSS

After exporting the components, the `resources/views/vendor/mail/html/themes` directory will contain a `default.css` file. You may customize the CSS in this file and your styles will automatically be in-lined within the HTML representations of your Markdown mail messages.

> {tip} If you would like to build an entirely new theme for the Markdown components, simply write a new CSS file within the `html/themes` directory and change the `theme` option of your `mail` configuration file.

<a name="sending-mail"></a>
## Sending Mail

To send a message, use the `to` method on the `Mail` [facade](/docs/{{version}}/facades). The `to` method accepts an email address, a user instance, or a collection of users. If you pass an object or collection of objects, the mailer will automatically use their `email` and `name` properties when setting the email recipients, so make sure these attributes are available on your objects. Once you have specified your recipients, you may pass an instance of your mailable class to the `send` method:

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

Of course, you are not limited to just specifying the "to" recipients when sending a message. You are free to set "to", "cc", and "bcc" recipients all within a single, chained method call:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="queueing-mail"></a>
### Queueing Mail

#### Queueing A Mail Message

Since sending email messages can drastically lengthen the response time of your application, many developers choose to queue email messages for background sending. Laravel makes this easy using its built-in [unified queue API](/docs/{{version}}/queues). To queue a mail message, use the `queue` method on the `Mail` facade after specifying the message's recipients:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

This method will automatically take care of pushing a job onto the queue so the message is sent in the background. Of course, you will need to [configure your queues](/docs/{{version}}/queues) before using this feature.

#### Delayed Message Queueing

If you wish to delay the delivery of a queued email message, you may use the `later` method. As its first argument, the `later` method accepts a `DateTime` instance indicating when the message should be sent:

    $when = Carbon\Carbon::now()->addMinutes(10);

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later($when, new OrderShipped($order));

#### Pushing To Specific Queues

Since all mailable classes generated using the `make:mail` command make use of the `Illuminate\Bus\Queueable` trait, you may call the `onQueue` and `onConnection` methods on any mailable class instance, allowing you to specify the connection and queue name for the message:

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

#### Queueing By Default

If you have mailable classes that you want to always be queued, you may implement the `ShouldQueue` contract on the class. Now, even if you call the `send` method when mailing, the mailable will still be queued since it implements the contract:

    use Illuminate\Contracts\Queue\ShouldQueue;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        //
    }

<a name="mail-and-local-development"></a>
## Mail & Local Development

When developing an application that sends email, you probably don't want to actually send emails to live email addresses. Laravel provides several ways to "disable" the actual sending of emails during local development.

#### Log Driver

Instead of sending your emails, the `log` mail driver will write all email messages to your log files for inspection. For more information on configuring your application per environment, check out the [configuration documentation](/docs/{{version}}/configuration#environment-configuration).

#### Universal To

Another solution provided by Laravel is to set a universal recipient of all emails sent by the framework. This way, all the emails generated by your application will be sent to a specific address, instead of the address actually specified when sending the message. This can be done via the `to` option in your `config/mail.php` configuration file:

    'to' => [
        'address' => 'example@example.com',
        'name' => 'Example'
    ],

#### Mailtrap

Finally, you may use a service like [Mailtrap](https://mailtrap.io) and the `smtp` driver to send your email messages to a "dummy" mailbox where you may view them in a true email client. This approach has the benefit of allowing you to actually inspect the final emails in Mailtrap's message viewer.

<a name="events"></a>
## Events

Laravel fires an event just before sending mail messages. Remember, this event is fired when the mail is *sent*, not when it is queued. You may register an event listener for this event in your `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Mail\Events\MessageSending' => [
            'App\Listeners\LogSentMessage',
        ],
    ];

