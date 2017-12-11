# Laravel 的消息通知系统

- [简介](#introduction)
- [创建通知](#creating-notifications)
- [发送通知](#sending-notifications)
    - [使用 Notifiable Trait](#using-the-notifiable-trait)
    - [使用 Notification Facade](#using-the-notification-facade)
    - [指定发送频道](#specifying-delivery-channels)
    - [队列化通知](#queueing-notifications)
    - [按需通知](#on-demand-notifications)
- [邮件通知](#mail-notifications)
    - [格式化邮件消息](#formatting-mail-messages)
    - [自定义接受者](#customizing-the-recipient)
    - [自定义主题](#customizing-the-subject)
    - [自定义模板](#customizing-the-templates)
- [Markdown 邮件通知](#markdown-mail-notifications)
    - [生成消息](#generating-the-message)
    - [写消息](#writing-the-message)
    - [自定义组件](#customizing-the-components)
- [数据库通知](#database-notifications)
    - [先决条件](#database-prerequisites)
    - [格式化数据库通知](#formatting-database-notifications)
    - [访问通知](#accessing-the-notifications)
    - [标为已读](#marking-notifications-as-read)
- [广播通知](#broadcast-notifications)
    - [先决条件](#broadcast-prerequisites)
    - [格式化广播通知](#formatting-broadcast-notifications)
    - [监听通知](#listening-for-notifications)
- [短信通知](#sms-notifications)
    - [先决条件](#sms-prerequisites)
    - [格式化短信通知](#formatting-sms-notifications)
    - [自定义 `From` 号码](#customizing-the-from-number)
    - [路由短信通知](#routing-sms-notifications)
- [Slack 通知](#slack-notifications)
    - [先决条件](#slack-prerequisites)
    - [格式化 Slack 通知](#formatting-slack-notifications)
    - [Slack Attachments](#slack-attachments)
    - [路由 Slack 通知](#routing-slack-notifications)
- [通知事件](#notification-events)
- [自定义发送频道](#custom-channels)

<a name="introduction"></a>
## 简介

除了 [发送邮件](/docs/{{version}}/mail)，Laravel 还支持通过多种频道发送通知，包括邮件、短信（通过 [Nexmo](https://www.nexmo.com/)）以及 [Slack](https://slack.com) 。通知还能存到数据库，这样就能在网页界面上显示了。

通常情况下，通知应该是简短、有信息量的消息来通知用户你的应用发生了什么。举例来说，如果你在编写一个在线交易应用，你应该会通过邮件和短信频道来给用户发送一条 「账单已付」 的通知。

<a name="creating-notifications"></a>
## 创建通知

Laravel 中一条通知就是一个类（通常存在 `app/Notifications` 文件夹里）。看不到的话不要担心，运行一下 `make:notification` 命令就能创建了：

    php artisan make:notification InvoicePaid

这个命令会在 `app/Notifications` 目录下生成一个新的通知类。这个类包含 `via` 方法和几个消息构建方法（比如 `toMail` 或 `toDatabase`），它们会针对指定的渠道把通知转换过为对应的消息。

<a name="sending-notifications"></a>
## 发送通知

<a name="using-the-notifiable-trait"></a>
### 使用 Notifiable Trait

通知可以通过两种方法发送： `Notifiable` trait 的 `notify` 方法或 `Notification` [facade](/docs/{{version}}/facades) 。首先，让我们探索使用 trait ：

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;
    }

默认的 `App\User` 模型中使用了这个 trait，它包含着一个可以用来发通知的方法：`notify` 。 `notify` 方法需要一个通知实例做参数：

    use App\Notifications\InvoicePaid;

    $user->notify(new InvoicePaid($invoice));

> {tip} 记住，你可以在任意模型中使用 `Illuminate\Notifications\Notifiable` trait，而不仅仅是在 `User` 模型中。

<a name="using-the-notification-facade"></a>
### 使用 Notification Facade

另外，你可以通过 `Notification` [facade](/docs/{{version}}/facades) 来发送通知。它主要用在当你给多个可接收通知的实体发送通知的时候，比如给用户集合发通知。要用 facade 发送通知的话，要把可接收通知的实体和通知的实例传递给 `send` 方法：

    Notification::send($users, new InvoicePaid($invoice));

<a name="specifying-delivery-channels"></a>
### 指定发送频道

每个通知类都有个 `via` 方法，它决定了通知在哪个频道上发送。开箱即用的通知频道有 `mail`, `database`, `broadcast`, `nexmo`, 和 `slack` 。

> {tip} 如果你想用其他的频道比如 Telegram 或者 Pusher ，可以去看下社区驱动的 [Laravel 通知频道网站](http://laravel-notification-channels.com) 。

`via` 方法受到一个 `$notifiable` 实例，它是接收通知的类实例。你可以用 `$notifiable` 来决定通知用哪个频道来发送：

    /**
     * 获取通知发送频道
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return $notifiable->prefers_sms ? ['nexmo'] : ['mail', 'database'];
    }

<a name="queueing-notifications"></a>
### 队列化通知

> {note} 在队列化通知前你需要配置队列，并 [运行队列处理器](/docs/{{version}}/queues)。

发送通知可能会花很长时间，尤其是发送频道需要调用外部 API 的时候。要加速应用响应的话，可以通过添加 `ShouldQueue` 接口和 `Queueable` trait 把通知加入队列。它们两个在使用 `make:notification` 命令来生成通知文件的时候就已经被导入了，所以你只需要添加到你的通知类就行了：

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        // ...
    }

一旦加入 `ShouldQueue` 接口，你就能像平常那样发送通知了。Laravel 会检测 `ShouldQueue` 接口并自动将通知的发送放入队列中。

    $user->notify(new InvoicePaid($invoice));

如果你想延迟发送，你可以通过 `delay` 方法来链式操作你的通知实例：

    $when = Carbon::now()->addMinutes(10);

    $user->notify((new InvoicePaid($invoice))->delay($when));
    
<a name="on-demand-notifications"></a>
### 按需通知

有时候你可能需要将通知发送给某个不是以”用户”身份存储在你的应用中的人。使用`Notification::route` 方法，你可以在发送通知之前指定 ad-hoc 通知路由信息：

    Notification::route('mail', 'taylor@laravel.com')
                ->route('nexmo', '5555555555')
                ->notify(new InvoicePaid($invoice));

<a name="mail-notifications"></a>
## 邮件通知

<a name="formatting-mail-messages"></a>
### 格式化邮件消息

如果一条通知支持以邮件发送，你应该在通知类里定义一个 `toMail` 方法。这个方法将收到一个 `$notifiable` 实体并返回一个 `Illuminate\Notifications\Messages\MailMessage` 实例。邮件消息可以包含多行文本也可以是引导链接。我们来看一个 `toMail` 方法的例子：

    /**
     * 获取通知的邮件展示方式
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->line('One of your invoices has been paid!')
                    ->action('View Invoice', $url)
                    ->line('Thank you for using our application!');
    }

> {tip} 注意我们在方法中用了 `$this->invoice->id` ，其实你可以传递应用所需要的任何数据来传递给通知的构造器。

在这个例子中，我们注册了一行文本，引导链接 ，然后又是一行文本。 `MailMessage` 提供的这些方法简化了对小的事务性的邮件进行格式化操作。邮件频道将会把这些消息组件转换成漂亮的响应式的 HTML 邮件模板并附上文本。下面是个 `mail` 频道生成的邮件示例：

<img src="https://laravel.com/assets/img/notification-example.png" width="551" height="596">

> {tip} 发送邮件通知前，请在 `config/app.php` 中设置 `name` 值。 这将会在邮件通知消息的头部和尾部中被使用。

#### 其他通知格式选项

你可以使用 `view` 方法来指定一个应用于渲染通知电子邮件的自定义模板，而不是在通知类中定义文本的「模板」：

    /**
     * 获取通知的邮件展示方式
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)->view(
            'emails.name', ['invoice' => $this->invoice]
        );
    }

另外，你可以从 `toMail` 方法中返回一个 [mailable 对象](/docs/{{version}}/mail) ：

    use App\Mail\InvoicePaid as Mailable;

    /**
     * 获取通知的邮件展示方式
     *
     * @param  mixed  $notifiable
     * @return Mailable
     */
    public function toMail($notifiable)
    {
        return (new Mailable($this->invoice))->to($this->user->email);
    }

<a name="error-messages"></a>
#### 错误消息

有些通知是给用户提示错误，比如账单支付失败的提示。你可以通过调用 `error` 方法来指定这条邮件消息被当做一个错误提示。当邮件消息使用了 `error` 方法后，引导链接按钮会变成红色而非蓝色：

    /**
     * 获取通知的邮件展示方式
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
       return (new MailMessage)
                    ->error()
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-recipient"></a>
### 自定义接收者

当通过 `mail` 频道来发送通知的时候，通知系统将会自动寻找你的 notifiable 实体中的 `email` 属性。你可以通过在实体中定义 `routeNotificationForMail` 方法来自定义邮件地址。

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * 邮件频道的路由
         *
         * @return string
         */
        public function routeNotificationForMail()
        {
            return $this->email_address;
        }
    }

<a name="customizing-the-subject"></a>
### 自定义主题

默认情况下，邮件主题是格式化成了标题格式的通知类的类名。所以如果你对通知类名为 `InvoicePaid` ，邮件主题将会是 `Invoice Paid` 。如果你想显式指定消息的主题，你可以在构建消息时调用 `subject` 方法：

    /**
     * 获取通知的邮件展示方式
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-templates"></a>
### 自定义模板

你可以通过发布通知包的资源来修改 HTML 模板和纯文本模板。运行这个命令后，邮件通知模板就被放在了 `resources/views/vendor/notifications` 文件夹下：

    php artisan vendor:publish --tag=laravel-notifications

<a name="markdown-mail-notifications"></a>
## Markdown 邮件通知

Markdown 邮件通知可让您利用邮件通知的预先构建的模板，同时给予您更多自由地撰写更长的自定义邮件。 由于消息是用 Markdown 编写的， Laravel 能够为消息呈现漂亮，响应的 HTML 模板，同时还自动生成纯文本对应。

<a name="generating-the-message"></a>
### 生成消息

要使用相应的 Markdown 模板生成通知，您可以使用 `make：notification` Artisan命令的 `--markdown` 选项：

    php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
    
与所有其他邮件通知一样，使用 Markdown 模板的通知应在其通知类上定义一个 `toMail` 方法。 但是，不使用 `line` 和 `action` 方法来构造通知，而是使用 `markdown` 方法来指定应该使用的 Markdown 模板的名称：

    /**
     * 获取通知的邮件展示方式
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="writing-the-message"></a>
### 写消息

Markdown 邮件通知使用 Blade 组件和Markdown语法的组合，允许您轻松构建通知，同时利用Laravel的预制通知组件：

    @component('mail::message')
    # Invoice Paid

    Your invoice has been paid!

    @component('mail::button', ['url' => $url])
    View Invoice
    @endcomponent

    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent

#### 按钮组件

按钮组件渲染了一个居中的链接按钮。组件有两个参数， `url` 和一个可选的参数 `color` 。支持的颜色有 `blue` ， `green` ， 和 `red` 。你可以根据你的需要向通知中添加任意数量的按钮组件：

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    View Invoice
    @endcomponent

#### 面板组件

面板部件在面板中呈现给定文本块，该面板具有与通知的其余部分稍微不同的背景颜色。 这将会允许你将给定的文本块引起注意：

    @component('mail::panel')
    This is the panel content.
    @endcomponent

#### 表格组件

表格组件允许你将一个 Markdown 表格转化成一个 HTML 表格。表格组件接受 Markdown 表格作为内容。支持使用默认的 Markdown 表格对齐语法使表列对齐：

    @component('mail::table')
    | Laravel       | Table         | Example  |
    | ------------- |:-------------:| --------:|
    | Col 2 is      | Centered      | $10      |
    | Col 3 is      | Right-Aligned | $20      |
    @endcomponent

<a name="customizing-the-components"></a>
### 自定义组件

您可以将所有 Markdown 通知组件导出到您自己的应用程序中进行自定义。 要导出组件，请使用 `vendor：publish` Artisan命令来发布 `laravel-mail` 标签：

    php artisan vendor:publish --tag=laravel-mail

此命令将 Markdown 邮件组件发布到 `resources/views/vendor/mail` 目录。 `mail` 目录将包含一个 `html` 和一个 `markdown` 目录，每个目录包含它们各自可用组件的展示方式。 您可以随意自定义这些组件。

#### 自定义CSS

导出组件后， `resources/views/vendor/mail/html/themes` 目录将包含一个 `default.css` 文件。 您可以在此文件中自定义 CSS ，并且您的样式将自动内嵌在 Markdown 通知的HTML展示中。

> {tip} 如果您想为 Markdown 组件构建一个全新的主题，只需在 `html/themes` 目录中写一个新的 CSS 文件，并更改 `mail` 配置文件中的 `theme` 选项。

<a name="database-notifications"></a>
## 数据库通知

<a name="database-prerequisites"></a>
### 先决条件

`database` 通知频道在一张数据表里存储通知信息。这张表包含了比如通知类型、JSON 格式数据等描述通知的信息。

你可以查询这张表的内容在应用界面上展示通知。但是在这之前，你需要先创建一个数据表来保存通知。你可以用 `notifications:table` 命令来生成迁移表：

    php artisan notifications:table

    php artisan migrate

<a name="formatting-database-notifications"></a>
### 格式化数据库通知

如果通知支持被存储到数据表中，你应该在通知类中定义一个 `toDatabase` 或 `toArray` 方法。这个方法接收 `$notifiable` 实体参数并返回一个普通的 PHP 数组。这个返回的数组将被转成 JSON 格式并存储到通知数据表的 `data` 列。我们来看一个 `toArray` 的例子：

    /**
     * 获取通知的数组展示方式
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toArray($notifiable)
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ];
    }

#### `toDatabase` Vs `toArray`

`toArray` 方法在 `broadcast` 频道也用到了，它用来决定广播给 JavaScript 客户端的数据。如果你想在 `database` 和 `broadcast` 频道中采用两种不同的数组展示方式，你应该定义 `toDatabase` 方法而非 `toArray` 方法。

<a name="accessing-the-notifications"></a>
### 访问通知

一旦通知被存到数据库中，你需要一种方便的方式来从通知实体中访问它们。 `Illuminate\Notifications\Notifiable` trait 包含一个可以返回这个实体所有通知的 `notifications` Eloquent 关联。要获取这些通知，你可以像用其他 Eloquent 关联一样来使用这个方法。默认情况下，通知将会以 `created_at` 时间戳来排序：

    $user = App\User::find(1);

    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }

如果你只想检索未读通知，你可以使用 `unreadNotifications` 关联。检索出来的通知也是以 `created_at` 时间戳来排序的：

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }

> {tip} 要从 JavaScript 客户端来访问通知的话，你应该定义一个通知控制器来给可通知的实体返回通知，比如给当前用户返回通知。然后你就可以在 JavaScript 客户端来发起对应 URI 的 HTTP 请求了。

<a name="marking-notifications-as-read"></a>
### 标为已读

通常情况下，当用户查看了通知时，你就希望把通知标为已读。`Illuminate\Notifications\Notifiable` trait 提供了一个 `markAsRead` 方法，它能在对应的数据库记录里更新 `read_at` 列：

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }

你可以使用 `markAsRead` 方法直接操作一个通知集合，而不是一条条遍历每个通知：

    $user->unreadNotifications->markAsRead();

你可以用批量更新的方式来把所有通知标为已读，而不用在数据库里检索：

    $user = App\User::find(1);

    $user->unreadNotifications()->update(['read_at' => Carbon::now()]);

当然，你可以通过 `delete` 通知来把它们从数据库删除：

    $user->notifications()->delete();

<a name="broadcast-notifications"></a>
## 广播通知

<a name="broadcast-prerequisites"></a>
### 先决条件

在广播通知前，你应该配置并熟悉 Laravel [事件广播](/docs/{{version}}/broadcasting) 服务。事件广播提供了一种 JavaScript 客户端响应服务端 Laravel 事件的机制。

<a name="formatting-broadcast-notifications"></a>
### 格式化广播通知

`broadcast` 频道使用 Laravel  [事件广播](/docs/{{version}}/broadcasting) 服务来广播通知，它使得 JavaScript 客户端可以实时捕捉通知。如果一条通知支持广播，你应该在通知类里定义一个 `toBroadcast` 或 `toArray` 方法。这个方法将收到一个 `$notifiable` 实体并返回一个普通的 PHP 数组。返回的数组会被编码成 JSON 格式并广播给你的 JavaScript 客户端。我们来看个 `toArray` 方法的例子：

    use Illuminate\Notifications\Messages\BroadcastMessage;

    /**
     * 获取通知的数组展示方式
     *
     * @param  mixed  $notifiable
     * @return BroadcastMessage
     */
    public function toBroadcast($notifiable)
    {
        return new BroadcastMessage([
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ]);
    }

#### 广播队列配置
 
所有广播通知都排队等待广播。 如果要配置用于广播队列操作的队列连接或队列名称，你可以使用 `BroadcastMessage` 的 `onConnection` 和 `onQueue` 方法：

    return new BroadcastMessage($data)
                    ->onConnection('sqs')
                    ->onQueue('broadcasts');

> {tip} 除了你指定的数据外，广播通知也包含一个 `type` 字段，这个字段包含了通知类的类名。

<a name="listening-for-notifications"></a>
### 监听通知

通知将会在一个私有频道里进行广播，频道格式为 `{notifiable}.{id}`。所以，如果你给 ID 为 `1` 的 `App\User` 实例发送通知，这个通知就在 `App.User.1` 私有频道里被发送。当你使用 [Laravel Echo](/docs/{{version}}/broadcasting) 的时候，你可以很简单地使用 `notification` 辅助函数来监听一个频道的通知：

    Echo.private('App.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });

#### 自定义通知通道

如果您想自定义应通知实体接收其广播通知的渠道，您可以定义一个 `receiveBroadcastNotificationsOn` 方法：

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * 用户接收的通知广播
         *
         * @return array
         */
        public function receivesBroadcastNotificationsOn()
        {
            return [
                new PrivateChannel('users.'.$this->id),
            ];
        }
    }

<a name="sms-notifications"></a>
## 短信通知

<a name="sms-prerequisites"></a>
### 先决条件

在 Laravel 中发送短信通知是基于 [Nexmo](https://www.nexmo.com/) 服务的。在通过 Nexmo 发送短信通知前，你需要安装 `nexmo/client` Composer 包并在 `config/services.php` 配置文件中添加几个配置选项。你可以复制下面的配置示例来开始使用：

    'nexmo' => [
        'key' => env('NEXMO_KEY'),
        'secret' => env('NEXMO_SECRET'),
        'sms_from' => '15556666666',
    ],

`sms_from` 选项是短信消息发送者的手机号码。你可以在 Nexmo 控制面板中生成一个手机号码。

<a name="formatting-sms-notifications"></a>
### 格式化短信通知

如果通知支持以短信方式发送，你应该在通知类里定义一个 `toNexmo` 方法。这个方法将会收到一个  `$notifiable` 实体并返回一个 `Illuminate\Notifications\Messages\NexmoMessage` 实例：

    /**
     * 获取通知的 Nexmo / 短信展示方式
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content');
    }

#### Unicode 内容

如果您的 SMS 消息包含 unicode 字符，您应该在构建 `NexmoMessage` 实例时调用 `unicode` 方法：

    /**
     * 获取通知的 Nexmo / 短信展示方式
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your unicode message')
                    ->unicode();
    }

<a name="customizing-the-from-number"></a>
### 自定义 `From` 号码

如果你想通过一个手机号来发送某些通知，而这个手机号不同于你的 `config/services.php` 配置文件中指定的话，你可以在 `NexmoMessage` 实例中使用 `from`：

    /**
     * 获取通知的 Nexmo / 短信展示方式
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content')
                    ->from('15554443333');
    }

<a name="routing-sms-notifications"></a>
### 路由短信通知

当通过 `nexmo` 频道来发送通知的时候，通知系统会自动寻找通知实体的 `phone_number` 属性。如果你想自定义通知被发送的手机号码，你要在通知实体里定义一个 `routeNotificationForNexmo` 方法。

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Nexmo 频道的路由通知
         *
         * @return string
         */
        public function routeNotificationForNexmo()
        {
            return $this->phone;
        }
    }

<a name="slack-notifications"></a>
## Slack 通知

<a name="slack-prerequisites"></a>
### 先决条件

通过 Slack 发送通知前，你必须通过 Composer 安装 Guzzle HTTP 库：

    composer require guzzlehttp/guzzle

你还需要给 Slack 组配置 「Incoming Webhook」 。它将提供你使用 [路由 Slack 通知](#routing-slack-notifications) 时用到的 URL。

<a name="formatting-slack-notifications"></a>
### 格式化 Slack 通知

如果通知支持被当做 Slack 消息发送，你应该在通知类里定义一个 `toSlack` 方法。这个方法将收到一个 `$notifiable` 实体并且返回一个 `Illuminate\Notifications\Messages\SlackMessage` 实例。Slack 消息可以包含文本内容以及一个 「attachment」 用来格式化额外文本或者字段数组。我们来看个基本的 `toSlack` 例子：

    /**
     * 获取通知的 Slack 展示方式
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->content('One of your invoices has been paid!');
    }

这个例子中，我们只发送了一行文本给 Slack，这会创建类似下面的一条消息：

<img src="https://laravel.com/assets/img/basic-slack-notification.png">

#### 自定义发件人和收件人

您可以使用 `from` 和 `to` 方法来自定义发件人和收件人。 `from` 方法接受用户名和表情符号标识符，而 `to` 方法接受一个频道或用户名：

    /**
     * 获取通知的 Slack 展示方式
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Ghost', ':ghost:')
                    ->to('#other')
                    ->content('This will be sent to #other');
    }

你也可以不使用表情符号，改为使用图片作为你的 logo：

    /**
     * 获取通知的 Slack 展示方式
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Laravel')
                    ->image('https://laravel.com/favicon.png')
                    ->content('This will display the Laravel logo next to the message');
    }

<a name="slack-attachments"></a>
### Slack 附加项 (Attachments)

你可以给 Slack 消息添加 "附加项"。附加项提供了比简单文本消息更丰富的格式化选项。在这个例子中，我们将发送一条有关应用中异常的错误通知，它里面有个可以查看这个异常更多详情的链接：

    /**
     * 获取通知的 Slack 展示方式。
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was not found.');
                    });
    }

上面这个例子将会生成一条类似下面的 Slack 消息：

<img src="https://laravel.com/assets/img/basic-slack-attachment.png">

附加项也允许你指定一个应该被展示给用户的数据的数组。给定的数据将会以表格样式展示出来，这能方便阅读：

    /**
     * 获取通知的 Slack 展示方式
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/invoices/'.$this->invoice->id);

        return (new SlackMessage)
                    ->success()
                    ->content('One of your invoices has been paid!')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Invoice 1322', $url)
                                   ->fields([
                                        'Title' => 'Server Expenses',
                                        'Amount' => '$1,234',
                                        'Via' => 'American Express',
                                        'Was Overdue' => ':-1:',
                                    ]);
                    });
    }

上面的例子将会生成一条类似下面的 Slack 消息：

<img src="https://laravel.com/assets/img/slack-fields-attachment.png">

#### Markdown 附件内容

如果一些附件字段包含 Markdown ，您可以使用 `markdown` 方法指示 Slack 解析并将给定的附件字段显示为 Markdown 格式的文本：

    /**
     * 获取通知的 Slack 展示方式
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was **not found**.')
                                   ->markdown(['title', 'text']);
                    });
    }

<a name="routing-slack-notifications"></a>
### 路由 Slack 通知

要把 Slack 通知路由到正确的位置，你要在通知实体中定义 `routeNotificationForSlack` 方法。它返回通知要被发往的 URL 回调地址。URL 可以通过在 Slack 组里面添加 "Incoming Webhook" 服务来生成。

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Slack 频道的通知路由
         *
         * @return string
         */
        public function routeNotificationForSlack()
        {
            return $this->slack_webhook_url;
        }
    }

<a name="notification-events"></a>
## 通知事件

当通知发送后，通知系统就会触发 `Illuminate\Notifications\Events\NotificationSent` 事件。它包含了 「notifiable」 实体和通知实例本身。你应该在 `EventServiceProvider` 中注册监听器：

    /**
     * 应用中的事件监听映射
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Notifications\Events\NotificationSent' => [
            'App\Listeners\LogNotification',
        ],
    ];

> {tip} 注册完监听器后，使用 `event:generate` Artisan 命令来快速生成监听器类。

在事件监听器中，你可以访问事件中的 `notifiable`, `notification`, 和 `channel` 属性，来了解通知接收者和通知本身：

    /**
     * 处理事件
     *
     * @param  NotificationSent  $event
     * @return void
     */
    public function handle(NotificationSent $event)
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
    }

<a name="custom-channels"></a>
## 自定义频道

Laravel 提供了开箱即用的通知频道，但是你可能会想编写自己的驱动来通过其他频道发送通知。Laravel 很容易实现。首先，定义一个包含 `send` 方法的类。这个方法应该收到两个参数：`$notifiable` 和 `$notification`:

    <?php

    namespace App\Channels;

    use Illuminate\Notifications\Notification;

    class VoiceChannel
    {
        /**
         * 发送给定通知
         *
         * @param  mixed  $notifiable
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return void
         */
        public function send($notifiable, Notification $notification)
        {
            $message = $notification->toVoice($notifiable);

            // 将通知发送给 $notifiable 实例
        }
    }

一旦定义了通知频道类，你应该在所有通知里通过 `via` 方法来简单地返回这个频道的类名。

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use App\Channels\VoiceChannel;
    use App\Channels\Messages\VoiceMessage;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class InvoicePaid extends Notification
    {
        use Queueable;

        /**
         * 获取通知频道
         *
         * @param  mixed  $notifiable
         * @return array|string
         */
        public function via($notifiable)
        {
            return [VoiceChannel::class];
        }

        /**
         * 获取通知的声音展示方式
         *
         * @param  mixed  $notifiable
         * @return VoiceMessage
         */
        public function toVoice($notifiable)
        {
            // ...
        }
    }

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@carlclone](https://laravel-china.org/users/7283)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/7283_1481302280.jpeg?imageView2/1/w/200/h/200">  |  翻译  |  [@carlclone](https://github.com/carlclone) at Github  |



--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org
