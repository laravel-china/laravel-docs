# 通知

- [简介](#introduction)
- [创建通知](#creating-notifications)
- [发送通知](#sending-notifications)
    - [使用 Notifiable Trait](#using-the-notifiable-trait)
    - [使用 Notification Facade](#using-the-notification-facade)
    - [指定发送渠道](#specifying-delivery-channels)
    - [队列化通知](#queueing-notifications)
- [邮件通知](#mail-notifications)
    - [格式化邮件消息](#formatting-mail-messages)
    - [自定义接受者](#customizing-the-recipient)
    - [自定义主题](#customizing-the-subject)
    - [自定义模板](#customizing-the-templates)
    - [错误消息](#error-messages)
- [数据库通知](#database-notifications)
    - [数据库依赖](#database-prerequisites)
    - [格式化数据库通知](#formatting-database-notifications)
    - [访问通知](#accessing-the-notifications)
    - [标为已读](#marking-notifications-as-read)
- [广播通知](#broadcast-notifications)
    - [依赖](#broadcast-prerequisites)
    - [格式化广播通知](#formatting-broadcast-notifications)
    - [监听通知](#listening-for-notifications)
- [短信通知](#sms-notifications)
    - [依赖](#sms-prerequisites)
    - [格式化短信通知](#formatting-sms-notifications)
    - [自定义 `From`](#customizing-the-from-number)
    - [路由短信通知](#routing-sms-notifications)
- [Slack 通知](#slack-notifications)
    - [依赖](#slack-prerequisites)
    - [格式化 Slack 通知](#formatting-slack-notifications)
    - [路由 Slack 通知](#routing-slack-notifications)
- [通知事件](#notification-events)
- [自定义发送渠道](#custom-channels)

<a name="introduction"></a>
## 简介

除了 [发送邮件](/docs/{{version}}/mail)，Laravel 还支持通过多种渠道发送通知，包括邮件，短信（通过 [Nexmo](https://www.nexmo.com/)），以及 [Slack](https://slack.com)。通知还能存到数据库，这样就能在网页界面上显示了。

通常情况下，通知应该是简短、有信息量的消息来通知用户你的应用发生了什么。举例来说，如果你在编写一个在线交易应用，你应该会通过邮件和短信渠道来给用户发送一条 “账单已付” 的通知。

<a name="creating-notifications"></a>
## 创建通知

Laravel 中一条通知就是一个类（通常存在 `app/Notifications` 文件夹里）。看不到的话不要担心，运行一下 `make:notification` 命令就能创建了：

    php artisan make:notification InvoicePaid

这个命令会在 `app/Notifications` 目录下生成一个新的通知类。每个通知类包含一个 `via` 方法和好几个构造消息的方法（比如 `toMail` 或 `toDatabase`），这几个构造消息的方法是专门用来针对特定渠道将通知转换成消息的。

<a name="sending-notifications"></a>
## 发送通知

<a name="using-the-notifiable-trait"></a>
### 使用 Notifiable Trait

通知可以通过两种方法发送： `Notifiable` trait 的 `notify` 方法或 `Notification` [facade](/docs/{{version}}/facades)。首先，我们检查下 `Notifiable` trait 。默认的 `App\User` 模型中使用了这个 trait，它包含着一个可以用来发通知的方法：`notify`。它需要一个通知实例做参数：

    use App\Notifications\InvoicePaid;

    $user->notify(new InvoicePaid($invoice));

> {tip} 记住，你可以在任意模型中使用 `Illuminate\Notifications\Notifiable` trait，而不仅仅是在 `User` 模型中。

<a name="using-the-notification-facade"></a>
### 使用 Notification Facade

另外，你可以通过 `Notification` [facade](/docs/{{version}}facades) 来发送通知。它主要用在当你给多个可接收通知的实体发送通知的时候，比如给用户集合发通知。要用 facade 发送通知的话，要把可接收通知的实体和通知的实例传递给 `send` 方法：

    Notification::send($users, new InvoicePaid($invoice));

<a name="specifying-delivery-channels"></a>
### 指定发送渠道

每个通知类都有个 `via` 方法，它决定了通知在哪个渠道上发送。开箱即用的通知渠道有 `mail`, `database`, `broadcast`, `nexmo`, 和 `slack`。

> {tip} 如果你想用其他的渠道比如电报或者推送服务，可以去看下社区驱动的 [Laravel 通知渠道网站](http://laravel-notification-channels.com)

`via` 方法受到一个`$notifiable` 实例，它是接收通知的类实例。你可以用 `$notifiable` 来决定通知用哪个渠道来发送：

    /**
     * 获取通知发送渠道
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

> {note} 在队列化通知前你需要配置队列，并 [启动处理器](/docs/{{version}}/queues)。

发送通知可能会花很长时间，尤其是发送渠道需要调用外部 API 的时候。要加速应用响应的话，可以通过添加 `ShouldQueue` 接口和 `Queueable` trait 把通知加入队列。它们两个在使用 `make:notification` 命令来生成通知文件的时候就已经被导入了，所以你只需要添加到你的通知类就行了：

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

如果你想延迟发送，你可以通过 `delay` 方法来链式操作你的通知实例。

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
                    ->line('One of your invoices has been paid!')
                    ->action('View Invoice', $url)
                    ->line('Thank you for using our application!');
    }

> {tip} 注意我们在方法中用了 `$this->invoice->id` ，其实你可以传递应用所需要的任何数据来传递给通知的构造器。

在这个例子中，我们注册了一行文本，引导链接 ，然后又是一行文本。`MailMessage` 提供的这些方法简化了对小的XXX邮件的格式化操作。邮件渠道将会把这些消息组件转换成漂亮的响应式的 HTML 邮件模板并附上文本。下面是个 `mail` 渠道生成的邮件示例：

<img src="https://laravel.com/assets/img/notification-example.png" width="551" height="596">

<a name="customizing-the-recipient"></a>
### 自定义接收者

当通过 `mail` 渠道来发送通知的时候，通知系统将会自动寻找你的 notifiable 实体中的邮件属性。你可以通过在实体中定义 `routeNotificationForMail` 方法来自定义邮件地址。

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * 邮件渠道的路由
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

默认情况下，邮件主题是格式化成了标题格式的通知类的类名。所以如果你对通知类名为 `InvoicePaid`，邮件主题将会是 `Invoice Paid`。如果你想显式指定消息的主题，你可以在构建消息时调用 `subject` 方法：

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

<a name="customizing-the-template"></a>
### 自定义模板

你可以通过发布通知包的资源来修改 HTML 模板和纯文本模板。运行这个命令后，邮件通知模板就被放在了 `resources/views/vendor/notifications` 文件夹下。

    php artisan vendor:publish --tag laravel-notifications

<a name="error-messages"></a>
### 错误消息

有些通知是给用户提示错误，比如账单支付失败的提示。你可以通过调用 `error` 方法来指定这条邮件消息被当做一个错误提示。当邮件消息使用了 `error` 方法后，引导链接按钮会变成红色而非蓝色。

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

<a name="database-notifications"></a>
## 数据库通知

<a name="database-prerequisites"></a>
### 依赖

数据库通知渠道在一张数据表里存储通知信息。这张表包含了比如通知类型、JSON 格式数据等描述通知的信息。

你可以查询这张表的内容在应用界面上展示通知。但是在这之前，你需要先创建一个数据表来保存通知。你可以用 `notifications:table` 命令来生成迁移表。

    php artisan notifications:table

    php artisan migrate

<a name="formatting-database-notifications"></a>
### 格式化数据库通知

如果通知支持被存储到数据表中，你应该在通知类中定义一个 `toDatabase` 或 `toArray` 方法。这个方法接收 `$notifiable` 实体参数并返回一个 PHP 一维数组。这个返回的数组将被转成 JSON 格式并存储到通知数据表的 `data` 列。我们来看一个 `toArray` 的例子：

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

#### `toDatabase` Vs. `toArray`

`toArray` 方法在 `broadcast` 渠道也用到了，它用来决定广播给 JavaScript 客户端的数据。如果你想在 `database` 和 `broadcast` 渠道中采用两种不同的数组展示方式，你应该定义 `toDatabase` 方法而非 `toArray` 方法。

<a name="accessing-the-notifications"></a>
### 访问通知

一旦通知被存到数据库中，你需要一种方便的方式来从通知实体中访问它们。 `Illuminate\Notifications\Notifiable` trait 
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

The `broadcast` channel broadcasts notifications using Laravel's [event broadcasting](/docs/{{version}}/broadcasting) services, allowing your JavaScript client to catch notifications in realtime. If a notification supports broadcasting, you should define a `toBroadcast` or `toArray` method on the notification class. This method will receive a `$notifiable` entity and should return a plain PHP array. The returned array will be encoded as JSON and broadcast to your JavaScript client. Let's take a look at an example `toArray` method:

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

> {tip} In addition to the data you specify, broadcast notifications will also contain a `type` field containing the class name of the notification.

#### `toBroadcast` Vs. `toArray`

The `toArray` method is also used by the `database` channel to determine which data to store in your database table. If you would like to have two different array representations for the `database` and `broadcast` channels, you should define a `toBroadcast` method instead of a `toArray` method.

<a name="listening-for-notifications"></a>
### Listening For Notifications

Notifications will broadcast on a private channel formatted using a `{notifiable}.{id}` convention. So, if you are sending a notification to a `App\User` instance with an ID of `1`, the notification will be broadcast on the `App.User.1` private channel. When using [Laravel Echo](/docs/{{version}}/broadcasting), you may easily listen for notifications on a channel using the `notification` helper method:

    Echo.private('App.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });

<a name="sms-notifications"></a>
## 短信通知

<a name="sms-prerequisites"></a>
### 依赖

在 Laravel 中发送短信通知是基于 [Nexmo](https://www.nexmo.com/)服务的。在通过 Nexmo 发送短信通知前，你需要安装 `nexmo/client` Composer 包并在 `config/services.php` 配置文件中添加几个配置选项。你可以复制下面的配置示例来开始使用：

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
     * 获取通知的短信展示方式
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content');
    }

<a name="customizing-the-from-number"></a>
### 自定义 “From” 数字

如果你想通过一个手机号来发送某些通知，而这个手机号不同于配置文件中指定的话，你可以在 `NexmoMessage` 实例中使用 `from`：

    /**
     * 获取通知的短信展示方式
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

当通过 `nexmo` 渠道来发送通知的时候，通知系统会自动寻找通知实体的 `phone_number` 属性。如果你想自定义通知被发送的手机号码，你要在通知实体里定义一个 `routeNotificationForNexmo`
方法。

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Nexmo 渠道的路由通知
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
### 依赖

通过 Slack 发送通知前，你必须通过 Composer 安装 Guzzle HTTP 库：

    composer require guzzlehttp/guzzle

你还需要给 Slack 组配置 "Incoming Webhook" 。它将提供你使用 [路由 Slack 通知](#routing-slack-notifications)时用到的 URL。

<a name="formatting-slack-notifications"></a>
### 格式化 Slack 通知

如果通知支持被当做 Slack 消息发送，你应该在通知类里定义一个 `toSlack` 方法。这个方法将收到一个 `$notifiable` 实体并且返回一个 `Illuminate\Notifications\Messages\SlackMessage` 实例。Slack 消息可以包含文本内容以及一个 "attachment" 用来格式化额外文本或者字段数组。我们来看个基本的 `toSlack` 例子：

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

#### Slack 附件

你可以给 Slack 消息添加 "attachments"。XXX（Attachments）提供了比简单文本消息更丰富的格式化选项。在这个例子中，我们将发送一条有关应用中异常的错误通知，它包含了一个可以查看这个异常更多详情的链接：

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

XXX 也允许你指定一个应该被展示给用户的数据的数组。给定的数据将会以表格样式展示出来，这能方便阅读：

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
         * Slack 渠道的通知路由
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

当通知发送后，通知系统就会触发 `Illuminate\Notifications\Events\NotificationSent` 事件。它包含了 "notifiable" 实体和通知实例本身。你应该在 `EventServiceProvider` 中注册监听器：

    /**
     * 应用中的事件监听映射。
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
     * 处理事件。
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
## 自定义渠道

Laravel 提供了开箱即用的通知渠道，但是你可能会想编写自己的驱动来通过其他渠道发送通知。Laravel 很容易实现。首先，定义一个包含 `send` 方法的类。这个方法应该收到两个参数：`$notifiable` 和 `$notification`:

    <?php

    namespace App\Channels;

    use Illuminate\Notifications\Notification;

    class VoiceChannel
    {
        /**
         * 发送给定通知。
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

一旦定义了通知渠道类，你应该在所有通知里通过 `via` 方法来简单地返回这个渠道的类名。

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
         * 获取通知渠道
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

         * @param  mixed  $notifiable
         * @return VoiceMessage
         */
        public function toVoice($notifiable)
        {
            // ...
        }
    }
