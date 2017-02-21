# Laravel 的消息通知系统

- [简介](#introduction)
- [创建通知](#creating-notifications)
- [发送通知](#sending-notifications)
    - [使用 Notifiable Trait](#using-the-notifiable-trait)
    - [使用 Notification Facade](#using-the-notification-facade)
    - [指定发送频道](#specifying-delivery-channels)
    - [队列化通知](#queueing-notifications)
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

除了 [发送邮件](/docs/{{version}}/mail)，Laravel 还支持通过多种频道发送通知，包括邮件、短信（通过 [Nexmo](https://www.nexmo.com/)）以及 [Slack](https://slack.com)。通知还能存到数据库，这样就能在网页界面上显示了。

通常情况下，通知应该是简短、有信息量的消息来通知用户你的应用发生了什么。举例来说，如果你在编写一个在线交易应用，你应该会通过邮件和短信频道来给用户发送一条「账单已付」的通知。

<a name="creating-notifications"></a>
## 创建通知

Laravel 中一条通知就是一个类（通常存在 `app/Notifications` 文件夹里）。看不到的话不要担心，运行一下 `make:notification` 命令就能创建了：

    php artisan make:notification InvoicePaid

这个命令会在 `app/Notifications` 目录下生成一个新的通知类。这个类包含 `via` 方法和几个消息构建方法（比如 `toMail` 或 `toDatabase`），它们会针对指定的渠道把通知转换过为对应的消息。

<a name="sending-notifications"></a>
## 发送通知

<a name="using-the-notifiable-trait"></a>
### 使用 Notifiable Trait

通知可以通过两种方法发送： `Notifiable` trait 的 `notify` 方法或 `Notification` [facade](/docs/{{version}}/facades)。首先，让我们探索使用 trait ：

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
                    ->greeting('Hello!')
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
        return new Mailable($this->invoice)->to($this->user->email);
    }

<a name="error-messages"></a>
#### 错误消息

有些通知是给用户提示错误，比如账单支付失败的提示。你可以通过调用 `error` 方法来指定这条邮件消息被当做一个错误提示。当邮件消息使用了 `error` 方法后，引导链接按钮会变成红色而非蓝色：

    /**
     * 获取通知的邮件展示方式
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Message
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
         * 邮件频道的路由。
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
Markdown mail notifications allow you to take advantage of the pre-built templates of mail notifications, while giving you more freedom to write longer, customized messages. Since the messages are written in Markdown, Laravel is able to render beautiful, responsive HTML templates for the messages while also automatically generating a plain-text counterpart.

<a name="generating-the-message"></a>
### 生成消息

要使用相应的 Markdown 模板生成通知，您可以使用 `make：notification` Artisan命令的 `--markdown` 选项：

    php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
    
与所有其他邮件通知一样，使用 Markdown 模板的通知应在其通知类上定义一个 `toMail` 方法。 但是，不使用 `line` 和 `action` 方法来构造通知，而是使用 `markdown` 方法来指定应该使用的 Markdown 模板的名称：

    /**
     * Get the mail representation of the notification.
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
### Writing The Message

Markdown mail notifications use a combination of Blade components and Markdown syntax which allow you to easily construct notifications while leveraging Laravel's pre-crafted notification components:

    @component('mail::message')
    # Invoice Paid

    Your invoice has been paid!

    @component('mail::button', ['url' => $url])
    View Invoice
    @endcomponent

    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent

#### Button Component

The button component renders a centered button link. The component accepts two arguments, a `url` and an optional `color`. Supported colors are `blue`, `green`, and `red`. You may add as many button components to a notification as you wish:

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    View Invoice
    @endcomponent

#### Panel Component

The panel component renders the given block of text in a panel that has a slightly different background color than the rest of the notification. This allows you to draw attention to a given block of text:

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

You may export all of the Markdown notification components to your own application for customization. To export the components, use the `vendor:publish` Artisan command to publish the `laravel-mail` asset tag:

    php artisan vendor:publish --tag=laravel-mail

This command will publish the Markdown mail components to the `resources/views/vendor/mail` directory. The `mail` directory will contain a `html` and a `markdown` directory, each containing their respective representations of every available component. You are free to customize these components however you like.

#### Customizing The CSS

After exporting the components, the `resources/views/vendor/mail/html/themes` directory will contain a `default.css` file. You may customize the CSS in this file and your styles will automatically be in-lined within the HTML representations of your Markdown notifications.

> {tip} If you would like to build an entirely new theme for the Markdown components, simply write a new CSS file within the `html/themes` directory and change the `theme` option of your `mail` configuration file.

<a name="database-notifications"></a>
## Database Notifications

<a name="database-prerequisites"></a>
### Prerequisites

The `database` notification channel stores the notification information in a database table. This table will contain information such as the notification type as well as custom JSON data that describes the notification.

You can query the table to display the notifications in your application's user interface. But, before you can do that, you will need to create a database table to hold your notifications. You may use the `notifications:table` command to generate a migration with the proper table schema:

    php artisan notifications:table

    php artisan migrate

<a name="formatting-database-notifications"></a>
### Formatting Database Notifications

If a notification supports being stored in a database table, you should define a `toDatabase` or `toArray` method on the notification class. This method will receive a `$notifiable` entity and should return a plain PHP array. The returned array will be encoded as JSON and stored in the `data` column of your `notifications` table. Let's take a look at an example `toArray` method:

    /**
     * Get the array representation of the notification.
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

#### `toDatabase` Vs. `toArray`

The `toArray` method is also used by the `broadcast` channel to determine which data to broadcast to your JavaScript client. If you would like to have two different array representations for the `database` and `broadcast` channels, you should define a `toDatabase` method instead of a `toArray` method.

<a name="accessing-the-notifications"></a>
### Accessing The Notifications

Once notifications are stored in the database, you need a convenient way to access them from your notifiable entities. The `Illuminate\Notifications\Notifiable` trait, which is included on Laravel's default `App\User` model, includes a `notifications` Eloquent relationship that returns the notifications for the entity. To fetch notifications, you may access this method like any other Eloquent relationship. By default, notifications will be sorted by the `created_at` timestamp:

    $user = App\User::find(1);

    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }

If you want to retrieve only the "unread" notifications, you may use the `unreadNotifications` relationship. Again, these notifications will be sorted by the `created_at` timestamp:

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }

> {tip} To access your notifications from your JavaScript client, you should define a notification controller for your application which returns the notifications for a notifiable entity, such as the current user. You may then make an HTTP request to that controller's URI from your JavaScript client.

<a name="marking-notifications-as-read"></a>
### Marking Notifications As Read

Typically, you will want to mark a notification as "read" when a user views it. The `Illuminate\Notifications\Notifiable` trait provides a `markAsRead` method, which updates the `read_at` column on the notification's database record:

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }

However, instead of looping through each notification, you may use the `markAsRead` method directly on a collection of notifications:

    $user->unreadNotifications->markAsRead();

You may also use a mass-update query to mark all of the notifications as read without retrieving them from the database:

    $user = App\User::find(1);

    $user->unreadNotifications()->update(['read_at' => Carbon::now()]);

Of course, you may `delete` the notifications to remove them from the table entirely:

    $user->notifications()->delete();

<a name="broadcast-notifications"></a>
## Broadcast Notifications

<a name="broadcast-prerequisites"></a>
### Prerequisites

Before broadcasting notifications, you should configure and be familiar with Laravel's [event broadcasting](/docs/{{version}}/broadcasting) services. Event broadcasting provides a way to react to server-side fired Laravel events from your JavaScript client.

<a name="formatting-broadcast-notifications"></a>
### Formatting Broadcast Notifications

The `broadcast` channel broadcasts notifications using Laravel's [event broadcasting](/docs/{{version}}/broadcasting) services, allowing your JavaScript client to catch notifications in realtime. If a notification supports broadcasting, you should define a `toBroadcast` method on the notification class. This method will receive a `$notifiable` entity and should return a `BroadcastMessage` instance. The returned data will be encoded as JSON and broadcast to your JavaScript client. Let's take a look at an example `toBroadcast` method:

    use Illuminate\Notifications\Messages\BroadcastMessage;

    /**
     * Get the broadcastable representation of the notification.
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

#### Broadcast Queue Configuration

All broadcast notifications are queued for broadcasting. If you would like to configure the queue connection or queue name that is used to the queue the broadcast operation, you may use the `onConnection` and `onQueue` methods of the `BroadcastMessage`:

    return new BroadcastMessage($data)
                    ->onConnection('sqs')
                    ->onQueue('broadcasts');

> {tip} In addition to the data you specify, broadcast notifications will also contain a `type` field containing the class name of the notification.

<a name="listening-for-notifications"></a>
### Listening For Notifications

Notifications will broadcast on a private channel formatted using a `{notifiable}.{id}` convention. So, if you are sending a notification to a `App\User` instance with an ID of `1`, the notification will be broadcast on the `App.User.1` private channel. When using [Laravel Echo](/docs/{{version}}/broadcasting), you may easily listen for notifications on a channel using the `notification` helper method:

    Echo.private('App.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });

#### Customizing The Notification Channel

If you would like to customize which channels a notifiable entity receives its broadcast notifications on, you may define a `receivesBroadcastNotificationsOn` method on the notifiable entity:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * The channels the user receives notification broadcasts on.
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
## SMS Notifications

<a name="sms-prerequisites"></a>
### Prerequisites

Sending SMS notifications in Laravel is powered by [Nexmo](https://www.nexmo.com/). Before you can send notifications via Nexmo, you need to install the `nexmo/client` Composer package and add a few configuration options to your `config/services.php` configuration file. You may copy the example configuration below to get started:

    'nexmo' => [
        'key' => env('NEXMO_KEY'),
        'secret' => env('NEXMO_SECRET'),
        'sms_from' => '15556666666',
    ],

The `sms_from` option is the phone number that your SMS messages will be sent from. You should generate a phone number for your application in the Nexmo control panel.

<a name="formatting-sms-notifications"></a>
### Formatting SMS Notifications

If a notification supports being sent as a SMS, you should define a `toNexmo` method on the notification class. This method will receive a `$notifiable` entity and should return a `Illuminate\Notifications\Messages\NexmoMessage` instance:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content');
    }

#### Unicode Content

If your SMS message will contain unicode characters, you should call the `unicode` method when constructing the `NexmoMessage` instance:

    /**
     * Get the Nexmo / SMS representation of the notification.
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
### Customizing The "From" Number

If you would like to send some notifications from a phone number that is different from the phone number specified in your `config/services.php` file, you may use the `from` method on a `NexmoMessage` instance:

    /**
     * Get the Nexmo / SMS representation of the notification.
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
### Routing SMS Notifications

When sending notifications via the `nexmo` channel, the notification system will automatically look for a `phone_number` attribute on the notifiable entity. If you would like to customize the phone number the notification is delivered to, define a `routeNotificationForNexmo` method on the entity:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Nexmo channel.
         *
         * @return string
         */
        public function routeNotificationForNexmo()
        {
            return $this->phone;
        }
    }

<a name="slack-notifications"></a>
## Slack Notifications

<a name="slack-prerequisites"></a>
### Prerequisites

Before you can send notifications via Slack, you must install the Guzzle HTTP library via Composer:

    composer require guzzlehttp/guzzle

You will also need to configure an "Incoming Webhook" integration for your Slack team. This integration will provide you with a URL you may use when [routing Slack notifications](#routing-slack-notifications).

<a name="formatting-slack-notifications"></a>
### Formatting Slack Notifications

If a notification supports being sent as a Slack message, you should define a `toSlack` method on the notification class. This method will receive a `$notifiable` entity and should return a `Illuminate\Notifications\Messages\SlackMessage` instance. Slack messages may contain text content as well as an "attachment" that formats additional text or an array of fields. Let's take a look at a basic `toSlack` example:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->content('One of your invoices has been paid!');
    }

In this example we are just sending a single line of text to Slack, which will create a message that looks like the following:

<img src="https://laravel.com/assets/img/basic-slack-notification.png">

#### Customizing The Sender & Recipient

You may use the `from` and `to` methods to customize the sender and recipient. The `from` method accepts a username and emoji identifier, while the `to` method accepts a channel or username:

    /**
     * Get the Slack representation of the notification.
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

<a name="slack-attachments"></a>
### Slack Attachments

You may also add "attachments" to Slack messages. Attachments provide richer formatting options than simple text messages. In this example, we will send an error notification about an exception that occurred in an application, including a link to view more details about the exception:

    /**
     * Get the Slack representation of the notification.
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

The example above will generate a Slack message that looks like the following:

<img src="https://laravel.com/assets/img/basic-slack-attachment.png">

Attachments also allow you to specify an array of data that should be presented to the user. The given data will be presented in a table-style format for easy reading:

    /**
     * Get the Slack representation of the notification.
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

The example above will create a Slack message that looks like the following:

<img src="https://laravel.com/assets/img/slack-fields-attachment.png">

#### Markdown Attachment Content

If some of your attachment fields contain Markdown, you may use the `markdown` method to instruct Slack to parse and display the given attachment fields as Markdown formatted text:

    /**
     * Get the Slack representation of the notification.
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
### Routing Slack Notifications

To route Slack notifications to the proper location, define a `routeNotificationForSlack` method on your notifiable entity. This should return the webhook URL to which the notification should be delivered. Webhook URLs may be generated by adding an "Incoming Webhook" service to your Slack team:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Slack channel.
         *
         * @return string
         */
        public function routeNotificationForSlack()
        {
            return $this->slack_webhook_url;
        }
    }

<a name="notification-events"></a>
## Notification Events

When a notification is sent, the `Illuminate\Notifications\Events\NotificationSent` event is fired by the notification system. This contains the "notifiable" entity and the notification instance itself. You may register listeners for this event in your `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Notifications\Events\NotificationSent' => [
            'App\Listeners\LogNotification',
        ],
    ];

> {tip} After registering listeners in your `EventServiceProvider`, use the `event:generate` Artisan command to quickly generate listener classes.

Within an event listener, you may access the `notifiable`, `notification`, and `channel` properties on the event to learn more about the notification recipient or the notification itself:

    /**
     * Handle the event.
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
## Custom Channels

Laravel ships with a handful of notification channels, but you may want to write your own drivers to deliver notifications via other channels. Laravel makes it simple. To get started, define a class that contains a `send` method. The method should receive two arguments: a `$notifiable` and a `$notification`:

    <?php

    namespace App\Channels;

    use Illuminate\Notifications\Notification;

    class VoiceChannel
    {
        /**
         * Send the given notification.
         *
         * @param  mixed  $notifiable
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return void
         */
        public function send($notifiable, Notification $notification)
        {
            $message = $notification->toVoice($notifiable);

            // Send notification to the $notifiable instance...
        }
    }

Once your notification channel class has been defined, you may simply return the class name from the `via` method of any of your notifications:

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
         * Get the notification channels.
         *
         * @param  mixed  $notifiable
         * @return array|string
         */
        public function via($notifiable)
        {
            return [VoiceChannel::class];
        }

        /**
         * Get the voice representation of the notification.
         *
         * @param  mixed  $notifiable
         * @return VoiceMessage
         */
        public function toVoice($notifiable)
        {
            // ...
        }
    }
