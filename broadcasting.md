# Laravel 的事件广播系统

- [简介](#introduction)
    - [配置](#configuration)
    - [对驱动器的要求](#driver-prerequisites)
- [概念综述](#concept-overview)
    - [使用示例程序](#using-example-application)
- [定义广播事件](#defining-broadcast-events)
    - [广播名称](#broadcast-name)
    - [广播数据](#broadcast-data)
    - [广播队列](#broadcast-queue)
    - [广播条件](#broadcast-conditions)
- [频道授权](#authorizing-channels)
    - [定义授权路由](#defining-authorization-routes)
    - [定义授权回调](#defining-authorization-callbacks)
- [对事件进行广播](#broadcasting-events)
    - [只广播给他人](#only-to-others)
- [接受广播](#receiving-broadcasts)
    - [安装 Laravel Echo](#installing-laravel-echo)
    - [对事件进行监听](#listening-for-events)
    - [退出频道](#leaving-a-channel)
    - [命名空间](#namespaces)
- [Presence 频道](#presence-channels)
    - [授权 Presence 频道](#authorizing-presence-channels)
    - [加入 Presence 频道](#joining-presence-channels)
    - [广播到 Presence 频道](#broadcasting-to-presence-channels)
- [客户端事件](#client-events)
- [消息通知](#notifications)

<a name="introduction"></a>
## 简介

在现代的 web 应用程序中，WebSockets 被用来实现需要实时、即时更新的接口。当服务器上的数据被更新后，更新信息将通过 WebSocket 连接发送到客户端等待处理。相比于不停地轮询应用程序，WebSocket 是一种更加可靠和高效的选择。

为了帮助你建立这类应用, Laravel 将通过 WebSocket 连接来使「广播」[事件](/docs/{{version}}/events) 变得更加轻松。广播事件允许你在服务端代码和客户端 JavaScript 应用之间共享相同的事件名。

> {tip} 在深入了解事件广播之前，请确认你已阅读所有关于 Laravel [事件和侦听器](/docs/{{version}}/events) 的文档。

<a name="configuration"></a>
### 配置

所有关于事件广播的配置都被保存在 `config/broadcasting.php` 文件中。 Laravel 自带了几个广播驱动器：[Pusher](https://pusher.com), [Redis](/docs/{{version}}/redis), 和一个用于本地开发与调试的 `log` 驱动器。另外，还有一个 `null` 驱动器可以让你完全关闭广播功能。每一个驱动的示例配置都可以在 `config/broadcasting.php` 文件中被找到。

#### 广播服务提供者

在对事件进行广播之前，你必须先注册 `App\Providers\BroadcastServiceProvider`。对于一个全新安装的 Laravel 应用程序，你只需在 `config/app.php` 配置文件的 `providers`  数组中取消对该提供者的注释即可。该提供者将允许你注册广播授权路由和回调。

#### CSRF 令牌

[Laravel Echo](#installing-laravel-echo) 会需要访问当前会话的 CSRF 令牌。如果可用，Echo 会从 `Laravel.csrfToken` JavaScript 对象中获取该令牌。如果你运行了 `make:auth` Artisan 命令，该对象会在 `resources/views/layouts/app.blade.php` 布局文件中被定义。如果你未使用该布局文件，可以在应用程序的 `head` HTML 元素中定义一个 `meta` 标签：

    <meta name="csrf-token" content="{{ csrf_token() }}">

<a name="driver-prerequisites"></a>
### 对驱动器的要求

#### Pusher

如果你使用 [Pusher](https://pusher.com) 对事件进行广播，请用 Composer 包管理器来安装 Pusher PHP SDK：

    composer require pusher/pusher-php-server "~3.0"

然后，你需要在 `config/broadcasting.php` 配置文件中填写你的 Pusher 证书。该文件中已经包含了一个 Pusher 示例配置，你只需指定 Pusher key、secret 和 application ID 即可。`config/broadcasting.php` 中的 `pusher` 配置项同时也允许你指定 Pusher 支持的 `options` ，例如 cluster：

    'options' => [
        'cluster' => 'eu',
        'encrypted' => true
    ],

当把 Pusher 和 [Laravel Echo](#installing-laravel-echo) 一起使用时，你应该在 `resources/assets/js/bootstrap.js` 文件中实例化 Echo 对象时指定 `pusher` 作为所需要的 broadcaster :


    import Echo from "laravel-echo"
    
    window.Pusher = require('pusher-js');
    
    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key'
    });

#### Redis

如果你使用 Redis 广播器，请安装 Predis 库：

    composer require predis/predis

Redis 广播器会使用 Redis 的「生产者/消费者」特性来广播消息；尽管如此，你仍需将它与 WebSocket 服务器一起使用。WebSocket 服务器会从 Redis 接收消息，然后再将消息广播到你的 WebSocket 频道上去。

当 Redis 广播器发布一个事件时，该事件会被发布到它指定的频道上去，传输的数据是一个采用 JSON 编码的字符串。该字符串包含了事件名、 `data` 数据和生成该事件套接字 ID 的用户（如果可用的话）。

#### Socket.IO

如果你想把 Redis 广播器和 Socket.IO 服务器一起使用，你需要将 Socket.IO JavaScript 客户端库文件包含到应用程序的 `head` HTML 元素中。当 Socket.IO 服务启动的时候，它会自动把 Socket.IO JavaScript 客户端库暴露在一个标准的 URL中。例如，如果你的应用和 Socket.IO 服务器运行在同域名下，你可以像下面这样来访问你的 Socket.IO JavaScript 客户端库：

    <script src="//{{ Request::getHost() }}:6001/socket.io/socket.io.js"></script>

接着，你需要在实例化 Echo 时指定 `socket.io` 连接器和 `host`。


    import Echo from "laravel-echo"
    
    window.Echo = new Echo({
        broadcaster: 'socket.io',
        host: window.location.hostname + ':6001'
    });

最后，你需要运行一个与 Laravel 兼容的 Socket.IO 服务器。Laravel 官方并没有实现 Socket.IO 服务器；不过，可以选择一个由社区驱动维护的项目 [tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server) ，目前托管在 GitHub。

#### 对队列的要求

在开始广播事件之前，你还需要配置和运行 [队列侦听器](/docs/{{version}}/queues) 。所有的事件广播都是通过队列任务来完成的，因此应用程序的响应时间不会受到明显影响。

<a name="concept-overview"></a>
## 概念综述

Laravel 的事件广播允许你使用基于驱动的 WebSockets 将服务端的 Larevel 事件广播到客户端的 JavaScript 应用程序。当前的 Laravel 自带了 [Pusher](https://pusher.com) 和 Redis 驱动。通过使用 [Laravel Echo](#installing-laravel-echo) 的 Javascript 包，我们可以很方便地在客户端消费事件。

事件通过「频道」来广播，这些频道可以被指定为公开的或私有的。任何访客都可以订阅一个不需要认证和授权的公开频道；然而，如果想订阅一个私有频道，那么该用户必须通过认证，并获得该频道的授权。

<a name="using-example-application"></a>
### 使用示例程序

让我们先用一个电子商务网站作为例子来概览一下事件广播。我们不会讨论如何配置 [Pusher](https://pusher.com) 或者 [Laravel Echo](#installing-laravel-echo) 的细节，因为这些会在本文档的其他章节被详细介绍。

在我们的应用程序中，让我们假设有一个允许用户查看订单配送状态的页面。有一个 `ShippingStatusUpdated` 事件会在配送状态更新时被触发：

    event(new ShippingStatusUpdated($update));

#### `ShouldBroadcast` 接口

当用户在查看自己的订单时，我们不希望他们必须通过刷新页面才能看到状态更新。我们希望一旦有更新时就主动将更新信息广播到客户端。所以，我们必须让 `ShippingStatusUpdated` 事件实现 `ShouldBroadcast` 接口。这会让 Laravel 在事件被触发时广播该事件：


    <?php
    
    namespace App\Events;
    
    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    
    class ShippingStatusUpdated implements ShouldBroadcast
    {
        /**
         * Information about the shipping status update.
         *
         * @var string
         */
        public $update;
    }

`ShouldBroadcast` 接口要求事件实现 `broadcastOn` 方法。该方法负责指定事件被广播到哪些频道。在通过 Artisan 命令生成的事件类中，一个空的 `broadcastOn` 方法已经被预定义好了，所以我们要做的仅仅是指定频道。我们希望只有订单的创建者能够看到状态更新，所以我们要把该事件广播到与这个订单绑定的私有频道上去：

    /**
     * Get the channels the event should broadcast on.
     *
     * @return array
     */
    public function broadcastOn()
    {
        return new PrivateChannel('order.'.$this->update->order_id);
    }

#### 频道授权

记住，用户只有在被授权后才能监听私有频道。我们可以在 `routes/channels.php` 文件中定义频道的授权规则。在本例中，我们需要对试图监听私有 `order.1` 频道的所有用户进行验证，确保只有订单的创建者才能进行监听：

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

`channel` 方法接收两个参数：频道名称和一个回调函数，该回调通过返回 `true` 或者 `false` 来表示用户是否被授权监听该频道。

所有的授权回调接收当前被认证的用户作为第一个参数，任何额外的通配符参数作为后续参数。在本例中，我们使用  `{orderId}` 占位符来表示频道名称的「ID」部分是通配符。

#### 对事件广播进行监听

接下来，就只剩下在 JavaScript 应用程序中监听事件了。我们可以使用 Laravel Echo 来实现。首先，使用 `private` 方法来订阅私有频道。然后，使用 `listen` 方法来监听 `ShippingStatusUpdated` 事件。默认情况下，事件的所有公有属性会被包括在广播事件中：

    Echo.private(`order.${orderId}`)
        .listen('ShippingStatusUpdated', (e) => {
            console.log(e.update);
        });

<a name="defining-broadcast-events"></a>
## 定义广播事件

要告知 Laravel 一个给定的事件是广播类型，只需在事件类中实现 `Illuminate\Contracts\Broadcasting\ShouldBroadcast` 接口即可。该接口已经被导入到所有由框架生成的事件类中，所以你可以很方便地将它添加到你自己的事件中。

`ShouldBroadcast` 接口要求你实现一个方法：`broadcastOn`. `broadcastOn` 方法返回一个频道或一个频道数组，事件会被广播到这些频道。频道必须是 `Channel`、`PrivateChannel` 或 `PresenceChannel` 的实例。`Channel` 实例表示任何用户都可以订阅的公开频道，而 `PrivateChannels` 和 `PresenceChannels` 则表示需要 [频道授权](#authorizing-channels) 的私有频道：


    <?php
    
    namespace App\Events;
    
    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    
    class ServerCreated implements ShouldBroadcast
    {
        use SerializesModels;
    
        public $user;
    
        /**
         * 创建一个新的事件实例
         *
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }
    
        /**
         * 指定事件在哪些频道上进行广播
         *
         * @return Channel|array
         */
        public function broadcastOn()
        {
            return new PrivateChannel('user.'.$this->user->id);
        }
    }

然后，你只需要像你平时那样 [触发事件](/docs/{{version}}/events) 。一旦事件被触发，一个 [队列任务](/docs/{{version}}/queues) 会自动广播事件到你指定的广播驱动器上。

<a name="broadcast-name"></a>
### 广播名称

Laravel 默认会使用事件的类名作为广播名称来广播事件。不过，你也可以在事件类中通过定义一个 `broadcastAs` 方法来自定义广播名称:

    /**
     * 事件的广播名称。
     *
     * @return string
     */
    public function broadcastAs()
    {
        return 'server.created';
    }

如果您使用 `broadcastAs` 方法自定义广播名称，你需要在你使用订阅事件的时候为事件类加上 `.` 前缀。这将指示 Echo 不要将应用程序的命名空间添加到事件中：

    .listen('.server.created', function (e) {
        ....
    });


<a name="broadcast-data"></a>
### 广播数据

当一个事件被广播时，它所有的 `public` 属性会自动被序列化为广播数据，这允许你在你的 JavaScript 应用中访问事件的公有数据。因此，举个例子，如果你的事件有一个公有的 `$user` 属性，它包含了一个 Elouqent 模型，那么事件的广播数据会是：

    {
        "user": {
            "id": 1,
            "name": "Patrick Stewart"
            ...
        }
    }

然而，如果你想更细粒度地控制你的广播数据，你可以添加一个 `broadcastWith` 方法到你的事件中。这个方法应该返回一个数组，该数组中的数据会被添加到广播数据中：

    /**
     * 指定广播数据
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['id' => $this->user->id];
    }

<a name="broadcast-queue"></a>
### 广播队列

默认情况下，每一个广播事件都被添加到默认的队列上，默认的队列连接在 `queue.php` 配置文件中指定。你可以通过在事件类中定义一个 `broadcastQueue` 属性来自定义广播器使用的队列。该属性用于指定广播使用的队列名称：

    /**
     * 指定事件被放置在哪个队列上
     *
     * @var string
     */
    public $broadcastQueue = 'your-queue-name';

如果要使用 `sync` 队列而不是默认队列驱动程序广播你的事件，你可以实现 `ShouldBroadcastNow` 接口而不是 `ShouldBroadcast`:


    <?php
    
    use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;
    
    class ShippingStatusUpdated implements ShouldBroadcastNow
    {
        //
    }
<a name="broadcast-conditions"></a>
### 广播条件

有时，你想在给定条件为 true ，才广播你的事件。你可以通过在事件类中添加一个 `broadcastWhen` 方法来定义这些条件：

    /**
     * Determine if this event should broadcast.
     *
     * @return bool
     */
    public function broadcastWhen()
    {
        return $this->value > 100;
    }

<a name="authorizing-channels"></a>
## 频道授权

对于私有频道，用户只有被授权后才能监听。实现过程是用户向你的 Laravel 应用程序发起一个携带频道名称的 HTTP 请求，你的应用程序判断该用户是否能够监听该频道。在使用 [Laravel Echo](#installing-laravel-echo) 时，上述 HTTP 请求会被自动发送；尽管如此，你仍然需要定义适当的路由来响应这些请求。

<a name="defining-authorization-routes"></a>
### 定义授权路由

幸运的是，我们可以在 Laravel 里很容易地定义路由来响应频道授权请求。在 `BroadcastServiceProvider` 中，你会看到一个对  `Broadcast::routes` 方法的调用。该方法会注册 `/broadcasting/auth` 路由来处理授权请求：

    Broadcast::routes();

`Broadcast::routes` 方法会自动把它的路由放进 `web` 中间件组中；另外，如果你想对一些属性自定义，可以向该方法传递一个包含路由属性的数组：

    Broadcast::routes($attributes);

<a name="defining-authorization-callbacks"></a>
### 定义授权回调

接下来，我们需要定义真正用于处理频道授权的逻辑。这是在 `routes/channels.php` 文件中完成。在该文件中，你可以用 `Broadcast::channel` 方法来注册频道授权回调函数：

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

`channel` 方法接收两个参数：频道名称和一个回调函数，该回调通过返回 `true` 或 `false` 来表示用户是否被授权监听该频道。

所有的授权回调接收当前被认证的用户作为第一个参数，任何额外的通配符参数作为后续参数。在本例中，我们使用  `{orderId}` 占位符来表示频道名称的「ID」部分是通配符。

#### 授权回调模型绑定

就像 HTTP 路由一样，频道路由也可以利用显式或隐式 [路由模型绑定](/docs/{{version}}/routing#route-model-binding)。例如，相比于接收一个字符串或数字类型的 order ID，你也可以请求一个真正的 `Order` 模型实例:


    use App\Order;
    
    Broadcast::channel('order.{order}', function ($user, Order $order) {
        return $user->id === $order->user_id;
    });

<a name="broadcasting-events"></a>
## 对事件进行广播

一旦你已经定义好了一个事件并实现了 `ShouldBroadcast` 接口，剩下的就是使用 `event` 函数来触发该事件。事件分发器会识别出实现了 `ShouldBroadcast` 接口的事件并将它们加入到队列进行广播：

    event(new ShippingStatusUpdated($update));

<a name="only-to-others"></a>
### 只广播给他人

当创建一个使用到事件广播的应用程序时，你可以用 `broadcast` 函数来替代 `event` 函数。和 `event` 函数一样，`broadcast` 函数将事件分发到服务端侦听器：

    broadcast(new ShippingStatusUpdated($update));

不同的是 `broadcast` 函数有一个 `toOthers` 方法允许你将当前用户从广播接收者中排除：

    broadcast(new ShippingStatusUpdated($update))->toOthers();

为了更好地理解什么时候使用 `toOthers` 方法，让我们假设有一个任务列表的应用程序，用户可以通过输入任务名称来新建任务。为了新建任务，你的应用程序需要发起一个请求到 `/task` 路由，该路由在接收到请求并成功创建新任务后会触发一个任务被新建的事件广播，并返回新任务的 JSON 响应。当你的 JavaScript 应用程序接收到来自该路由的响应时，它会直接将新任务插入到任务列表，就像这样：

    axios.post('/task', task)
        .then((response) => {
            this.tasks.push(response.data);
        });

然而，别忘了我们还将接收到一个事件广播。如果你的 JavaScript 应用程序同时监听该事件以便添加新任务到任务列表，你将会在你的列表中看到重复的任务：一份来自路由响应，另一份来自广播。

你可以通过使用 `toOthers` 方法让广播器只广播事件到其他用户来解决该问题。

#### 配置

当你实例化 Laravel Echo 实例时，一个套接字 ID 会被指定到该连接。如果你使用 [Vue](https://vuejs.org) 和 [Axios](https://github.com/mzabriskie/axios)，套接字 ID 会自动被添加到每一个请求的 `X-Socket-ID` 头中。然后，当你调用 `toOthers` 方法时，Laravel 会从头中提取出套接字 ID，并告诉广播器不要广播任何消息到该套接字 ID 的连接上。

如果你没有使用 Vue 和 Axios，则需要手动配置 JavaScript 应用程序来发送 `X-Socket-ID` 头。你可以用 `Echo.socketId` 方法来获取套接字 ID：

    var socketId = Echo.socketId();

<a name="receiving-broadcasts"></a>
## 接收广播

<a name="installing-laravel-echo"></a>
### 安装 Laravel Echo

Laravel Echo 是一个 JavaScript 库，它使得订阅频道和监听由 Laravel 广播的事件变得非常容易。你可以通过 NPM 包管理器来安装 Echo。在本例中，因为我们将使用 Pusher 广播器，请安装 `pusher-js` 包：

    npm install --save laravel-echo pusher-js

一旦 Echo 被安装好，你就可以在你应用程序的 JavaScript 中创建一个全新的 Echo 实例。做这件事的一个理想地方是在  `resources/assets/js/bootstrap.js` 文件的底部，Laravel 框架自带了该文件：


    import Echo from "laravel-echo"
    
    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key'
    });

当你使用 `pusher` 连接器来创建一个 Echo 实例的时候，你需要指定 `cluster` 以及指定连接是否需要加密：

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key',
        cluster: 'eu',
        encrypted: true
    });

<a name="listening-for-events"></a>
### 对事件进行监听

一旦你安装好并实例化了 Echo，你就可以开始监听事件广播了。首先，使用 `channel` 方法来获取一个频道实例，然后调用  `listen` 方法来监听指定的事件：

    Echo.channel('orders')
        .listen('OrderShipped', (e) => {
            console.log(e.order.name);
        });

如果你想监听私有频道上的事件，请使用 `private` 方法。你可以通过链式调用 `listen` 方法来监听一个频道上的多个事件：

    Echo.private('orders')
        .listen(...)
        .listen(...)
        .listen(...);

<a name="leaving-a-channel"></a>
### 退出频道

如果想退出频道，你需要在你的 Echo 实例上调用 `leave` 方法：

    Echo.leave('orders');

<a name="namespaces"></a>
### 命名空间

你可能注意到了在上面的例子中我们没有为事件类指定完整的命名空间。这是因为 Echo 会自动认为事件在 `App\Events` 命名空间下。你可以在实例化 Echo 的时候传递一个 `namespace` 配置选项来指定根命名空间：

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key',
        namespace: 'App.Other.Namespace'
    });

另外，你也可以在使用 Echo 订阅事件的时候为事件类加上 `.` 前缀。这时需要填写完全限定名称的类名：

    Echo.channel('orders')
        .listen('.Namespace.Event.Class', (e) => {
            //
        });

<a name="presence-channels"></a>
## Presence 频道

Presence 频道是在私有频道的安全性基础上，额外暴露出有哪些人订阅了该频道。这使得它可以很容易地建立强大的、协同的应用，如当有一个用户在浏览页面时，通知其他正在浏览相同页面的用户。

<a name="joining-a-presence-channel"></a>
### 授权 Presence 频道

Presence 频道也是私有频道；因此，用户必须 [获取授权后才能访问他们](#authorizing-channels)。与私有频道不同的是，在给 presence 频道定义授权回调函数时，如果一个用户已经加入了该频道，那么不应该返回 `true`，而应该返回一个关于该用户信息的数组。

由授权回调函数返回的数据能够在你的 JavaScript 应用程序中被 presence 频道事件侦听器所使用。如果用户没有获得加入该 presence 频道的授权，那么你应该返回 `false` 或 `null`：

    Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
        if ($user->canJoinRoom($roomId)) {
            return ['id' => $user->id, 'name' => $user->name];
        }
    });

<a name="joining-presence-channels"></a>
### 加入 Presence 频道

你可以用 Echo 的 `join` 方法来加入 presence 频道。`join` 方法会返回一个实现了 `PresenceChannel` 的对象，它通过暴露  `listen` 方法，允许你订阅 `here`、`joining` 和 `leaving` 事件。

    Echo.join(`chat.${roomId}`)
        .here((users) => {
            //
        })
        .joining((user) => {
            console.log(user.name);
        })
        .leaving((user) => {
            console.log(user.name);
        });

`here` 回调函数会在你成功加入频道后被立即执行，它接收一个包含用户信息的数组，用来告知当前订阅在该频道上的其他用户。`joining` 方法会在其他新用户加入到频道时被执行，`leaving` 会在其他用户退出频道时被执行。

<a name="broadcasting-to-presence-channels"></a>
### 广播到 Presence 频道

Presence 频道可以像公开和私有频道一样接收事件。使用一个聊天室的例子，我们要把 `NewMessage` 事件广播到聊天室的 presence 频道。要实现它，我们将从事件的 `broadcastOn` 方法中返回一个 `PresenceChannel` 实例：

    /**
     * 指定事件在哪些频道上进行广播
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PresenceChannel('room.'.$this->message->room_id);
    }

和公开或私有事件一样，presence 频道事件也能使用 `broadcast` 函数来广播。同样的，你还能用 `toOthers` 方法将当前用户从广播接收者中排除：

    broadcast(new NewMessage($message));

    broadcast(new NewMessage($message))->toOthers();

你可以通过 Echo 的 `listen` 方法来监听 join 事件：

    Echo.join(`chat.${roomId}`)
        .here(...)
        .joining(...)
        .leaving(...)
        .listen('NewMessage', (e) => {
            //
        });

<a name="client-events"></a>
## 客户端事件

有时候你可能希望广播一个事件给其他已经连接的客户端，但并不需要通知你的 Laravel 程序。试想一下当你想提醒用户，另外一个用户正在输入中的时候，显而易见客服端事件对于「输入中」之类的通知就显得非常有用了。你可以使用 Echo 的 `whisper` 方法来广播客户端事件：

    Echo.channel('chat')
        .whisper('typing', {
            name: this.user.name
        });

你可以使用 `listenForWhisper` 方法来监听客户端事件：

    Echo.channel('chat')
        .listenForWhisper('typing', (e) => {
            console.log(e.name);
        });

<a name="notifications"></a>
## 消息通知

将 [消息通知](/docs/{{version}}/notifications) 和事件广播一同使用，你的 JavaScript 应用程序可以在不刷新页面的情况下接收新的消息通知。首先，请先阅读关于如何使用 [the broadcast notification channel](/docs/{{version}}/notifications#broadcast-notifications) 的文档。

一旦你将一个消息通知配置为使用广播频道，你需要使用 Echo 的 `notification` 方法来监听广播事件。谨记，频道名称应该和接收消息通知的实体类名相匹配：

    Echo.private(`App.User.${userId}`)
        .notification((notification) => {
            console.log(notification.type);
        });

在本例中，所有通过 `broadcast` 频道发送到 `App\User` 实例的消息通知都会被该回调接收到。一个针对 `App.User.{id}` 频道的授权回调函数已经被包含在 `Laravel` 的 `BroadcastServiceProvider` 中了。

## 译者署名
| 用户名                                      | 头像                                       | 职能   | 签名                                       |
| ---------------------------------------- | ---------------------------------------- | ---- | ---------------------------------------- |
| [@沈益飞](https://laravel-china.org/users/13655) | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/13655_1490162781.png?imageView2/1/w/100/h/100"> | 翻译   | [@m809745357](https://github.com/m809745357) at Github |


--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org