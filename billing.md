# Laravel Cashier

- [简介](#introduction)
- [Stripe 配置](#stripe-configuration)
- [Braintree 配置](#braintree-configuration)
- [订购](#subscriptions)
    - [创建订购](#creating-subscriptions)
    - [确认订购状态](#checking-subscription-status)
    - [改变方案](#changing-plans)
    - [订购数量](#subscription-quantity)
    - [订购税金](#subscription-taxes)
    - [取消订购](#cancelling-subscriptions)
    - [恢复订购](#resuming-subscriptions)
- [处理 Stripe Webhooks](#handling-stripe-webhooks)
    - [订购失败](#handling-failed-subscriptions)
    - [其它 Webhooks](#handling-other-webhooks)
- [处理 Braintree Webhooks](#handling-braintree-webhooks)
    - [订购失败](#handling-braintree-failed-subscriptions)
    - [其它 Webhooks](#handling-braintree-other-webhooks)
- [一次性收费](#single-charges)
- [发票](#invoices)
    - [生成发票的 PDF](#generating-invoice-pdfs)

<a name="introduction"></a>
## 简介

Laravel Cashier 给 [Stripe](https://stripe.com) 和 [Braintree's](https://braintreepayments.com) 的订购交易服务提供了生动流畅的接口。它基本上处理了所有会让人恐惧的订购管理的相关逻辑。除了基本的订购管理外，Cashier 还可以处理优惠券，订购转换，管理订购「数量」、取消宽限期，甚至生成发票的 PDF 文件。

<a name="stripe-configuration"></a>
## Stripe 配置

#### Composer

首先，将 Cashier 扩展包添加至 `composer.json` 文件并运行 `composer update` 命令：

    "laravel/cashier": "~6.0"

> 译者注：请使用 require 来安装扩展包，讨论请见 [正确的 Composer 扩展包安装方法](https://phphub.org/topics/1901)

#### 服务提供者

接着，在 `app` 配置文件中注册 `Laravel\Cashier\CashierServiceProvider` [服务提供者](/docs/{{version}}/providers)。

#### 数据库迁移

我们需要添加新字段到 `users` 用户表中，并且创建 `subscriptions` 表用来存放所有用户的订阅信息：

    Schema::table('users', function ($table) {
        $table->string('stripe_id')->nullable();
        $table->string('card_brand')->nullable();
        $table->string('card_last_four')->nullable();
        $table->timestamp('trial_ends_at')->nullable();
    });

    Schema::create('subscriptions', function ($table) {
        $table->increments('id');
        $table->integer('user_id');
        $table->string('name');
        $table->string('stripe_id');
        $table->string('stripe_plan');
        $table->integer('quantity');
        $table->timestamp('trial_ends_at')->nullable();
        $table->timestamp('ends_at')->nullable();
        $table->timestamps();
    });

在数据库迁移文件创建完毕后请运行 `migrate` Artisan 命令。

#### 模型设置

接着，将 `Billable` trait 和适当的日期访问器添加至你的模型定义中：

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### Stripe 密钥

最后，在你的 `services.php` 配置文件中设置 Stripe 密钥：

    'stripe' => [
        'model'  => App\User::class,
        'secret' => env('STRIPE_SECRET'),
    ],

<a name="braintree-configuration"></a>
## Braintree 配置

#### Braintree 警告

对于大部分的操作，Stripe 和 Braintree 的功能基本上一样。两个服务都支持行用卡支付，不过 Braintree 支持 PayPal。Braintree 也缺少一些 Stripe 的功能，在你选择的时候可以参考一下：

<div class="content-list" markdown="1">
- Braintree 支持 PayPal，Stripe 不支持。
- Braintree 不支持订阅的 `increment` 和 `decrement` 方法。这是 Braintree 的接口限制，不是 Cashier 的。
- Braintree 不支持按照百分比的折扣。这是 Braintree 的接口限制，不是 Cashier 的。
</div>

#### Composer

首先，将 Cashier 扩展包添加至 `composer.json` 文件并运行 `composer update` 命令：

    "laravel/cashier-braintree": "~1.0"

#### 服务提供者

接着，在 `app` 配置文件中注册 `Laravel\Cashier\CashierServiceProvider` [服务提供者](/docs/{{version}}/providers)。

#### 计划积分优惠券

在使用 Cashier 中使用 Braintree 之前，你需要在 Braintree 的控制台上定义一个 `plan-credit` 优惠计划。这个计划会用来合理分配从月度订阅升级到年度订阅的改变，或者年度订阅降级到月度订阅的改变。在 Braintree 的控制台中，你可以为这个计划任意设置一个值，Cashier 会在每一次变更优惠券的时候覆盖此值。

#### 数据库迁移

我们需要添加新字段到 `users` 用户表中，并且创建 `subscriptions` 表用来存放所有用户的订阅信息：

    Schema::table('users', function ($table) {
        $table->string('braintree_id')->nullable();
        $table->string('paypal_email')->nullable();
        $table->string('card_brand')->nullable();
        $table->string('card_last_four')->nullable();
        $table->timestamp('trial_ends_at')->nullable();
    });

    Schema::create('subscriptions', function ($table) {
        $table->increments('id');
        $table->integer('user_id');
        $table->string('name');
        $table->string('braintree_id');
        $table->string('braintree_plan');
        $table->integer('quantity');
        $table->timestamp('trial_ends_at')->nullable();
        $table->timestamp('ends_at')->nullable();
        $table->timestamps();
    });

在数据库迁移文件创建完毕后请运行 `migrate` Artisan 命令。

#### 模型设置

接着，将 `Billable` trait 和适当的日期访问器添加至你的模型定义中：

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### 秘钥设置

在你的 `services.php` 配置文件中设置 Stripe 密钥：

    'braintree' => [
        'model'  => App\User::class,
        'environment' => env('BRAINTREE_ENV'),
        'merchant_id' => env('BRAINTREE_MERCHANT_ID'),
        'public_key' => env('BRAINTREE_PUBLIC_KEY'),
        'private_key' => env('BRAINTREE_PRIVATE_KEY'),
    ],

然后你还需要增加这些 SDK 调用到你的 `AppServiceProvider` 服务提供者的 `boot` 方法内：

    \Braintree_Configuration::environment(env('BRAINTREE_ENV'));
    \Braintree_Configuration::merchantId(env('BRAINTREE_MERCHANT_ID'));
    \Braintree_Configuration::publicKey(env('BRAINTREE_PUBLIC_KEY'));
    \Braintree_Configuration::privateKey(env('BRAINTREE_PRIVATE_KEY'));

<a name="subscriptions"></a>
## 订购

<a name="creating-subscriptions"></a>
### 创建订购

要创建一个订购，首先要获取可交易的模型实例，这通常会是 `App\User` 的实例。一旦你获取了模型实例，你可以使用 `subscription` 方法来管理模型的订购：

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')->create($creditCardToken);

`newSubscription` 方法的第一个参数是订阅的名称，如果你的应用程序只提供一种订阅的话，可以考虑把这个方案命名为 `main` 或者 `primary`。第二个参数是用户订阅的 Stripe / Braintree 计划。这个值应该对应 Stripe 和 Braintree 的后台唯一标识符。

`create` 方法会自动创建订阅并且更新客户 ID 和账单相关的信息。

#### 额外用户详细数据

如果你想自定义额外的顾客详细数据，可以将数据数组作为 `create` 方法的第二个参数传入：

    $user->subscription('monthly')->create($creditCardToken, [
        'email' => $email, 'description' => '我们的第一个客户'
    ]);

想知道更多 Stripe 支持的额外字段，请查看 Stripe 的 [创建顾客文档](https://stripe.com/docs/api#create_customer)。

#### 优惠券

如果你想在创建订购的时候使用优惠券，可以使用 `withCoupon` 方法：

    $user->subscription('monthly')
         ->withCoupon('code')
         ->create($creditCardToken);


<a name="checking-subscription-status"></a>
### 确认订购状态

一旦用户在你的应用程序完成订购，你便可以使用多种方法来检查他们的订购状态。首先，当用户拥有有效订购时，`subscribed` 方法会返回 `true`，即使该订购目前在试用期间：

    if ($user->subscribed('main')) {
        //
    }

`subscribed` 方法很适合用在 [路由中间件](/docs/{{version}}/middleware)，让你可以通过用户的订购状态，过滤访问路由及控制器：

    public function handle($request, Closure $next)
    {
        if ($request->user() && ! $request->user()->subscribed('main')) {
            // This user is not a paying customer...
            return redirect('billing');
        }

        return $next($request);
    }

如果你想确认用户是否还在他们的试用期内，你可以使用 `onTrial` 方法。利用此方法可以在页面的顶部展示正在试用期内的提示：

    if ($user->subscription('main')->onTrial()) {
        //
    }

`subscribedToPlan` 方法允许你使用 Stripe ID / Braintree 计划 ID 来确认用户是否订购某方案。在这个列子中，我们检查用户的 `main` 订阅是否有 `monthly`计划：

    if ($user->subscribedToPlan('monthly', 'main')) {
        //
    }

#### 取消订购状态

若要确认用户是否曾经订购过，但现在已取消订购，你可以使用 `cancelled` 方法：

    if ($user->subscription('main')->cancelled()) {
        //
    }

你可能想确认用户是否已经取消订购，但是订购还在他们完全到期前的「宽限期」内。例如，如果用户在三月五号取消了订购，但是服务会到三月十号才过期。那么用户到三月十号前都是「宽限期」。注意，`subscribed` 方法在这个期间仍然会返回 `true`。

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

<a name="changing-plans"></a>
### 改变方案

当用户在你的应用程序订购之后，他们有时可能想更改自己的订购方案。使用 `swap` 方法可以把用户转换到新的订购方案。举个例子，我们可以简单的将用户切换至 `premium` 订购方案：

    $user = App\User::find(1);

    $user->subscription('main')->swap('provider-plan-id');

如果用户是在试用阶段的话，用户的试用计划会被继续保持。如果订阅有订购数量存在的话，订购数量也会被保持。

    $user->subscription('main')->swap('provider-plan-id');

如果你想切换到一个订阅计划，但是跳过「试用」的话，可以使用 `skipTrial` 方法来实现：
    $user->subscription('main')
            ->skipTrial()
            ->swap('provider-plan-id');

<a name="subscription-quantity"></a>
### 订购数量

> **注意：** 订购数量只有 Stripe 支持。

有时候订购行为会跟「数量」有关。例如，你的应用程序可能会依照帐号的用户人数，**每人** 每月收取 10 元。你可以使用 `incrementQuantity` 和 `decrementQuantity` 方法简单的调整订购数量：

    $user = User::find(1);

    $user->subscription('main')->incrementQuantity();

    // 增加 5 个订购数量...
    $user->subscription('main')->incrementQuantity(5);

    $user->subscription('main')->decrementQuantity();

    // 减少 5 个订购数量...
    $user->subscription('main')->decrementQuantity(5);

另外，你也可以使用 `updateQuantity` 方法来设置指定的数量：

    $user->subscription('main')->updateQuantity(10);

有关订购数量的更多数据，请参阅 [Stripe 文档](https://stripe.com/docs/guides/subscriptions#setting-quantities)。

<a name="subscription-taxes"></a>
### 订购税金

在 Cashier 中，可以很容易的提供发送至 Stripe 的 `tax_percent` 值。要指定一个用户付费订购时的税金比例，请在你的交易模型中实现 `getTaxPercent` 方法，并返回一个介于 0 至 100 间，且不超过两位小数的数值。

    public function getTaxPercent() {
        return 20;
    }

这能让你基于个别模型去运用税率，可能对横跨多个国家的用户群非常有帮助。

<a name="cancelling-subscriptions"></a>
### 取消订购

要取消一个订购，只需要在用户的订购调用 `cancel` 方法：

    $user->subscription('main')->cancel();


当订购被取消时，Cashier 会自动更新数据库的 `ends_at` 字段。这个字段会被用来判断 `subscribed` 方法什么时候该开始返回  `false`。例如，如果顾客在三月一号取消订购，但是服务可以使用到三月五号为止，那么 `subscribed` 方法在三月五号前都会传回 `true`。

你想确认用户是否已经取消他们的订购，但是还在他们的「宽限期」内，可以使用 `onGracePeriod` 方法：

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

<a name="resuming-subscriptions"></a>
### 恢复订购

如果用户已经取消了他们的订购，但你想恢复订购，可以使用 `resume` 方法。用户 **必须** 仍处在宽限期内：

    $user->subscription('main')->resume();

如果客户取消订购后，且接着在服务完全过期前恢复订购，他们将不会在当下被扣款。他们的订购会被重新启动，而付款则会依照平常的周期。

<a name="subscription-trials"></a>
## 订购试用

<a name="with-credit-card-up-front"></a>
### 预先绑定信用卡

你可以使用 `trialDays` 为你的客户提供一定时间的试用期限方案：

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')
                ->trialDays(10)
                ->create($creditCardToken);

此方法会更新数据库中的订阅记录的试用终止时间，同时会指示 Stripe / Braintree 在这个终止时间之前不做计费处理。

> **注意：** 你应该在试用期快终止的时候通知你的用户，因为试用期一到，订阅服务将开始计费。

你可以在用户实例或者用户的订阅实例上使用 `onTrial` 方法来判断当前用户是否正处于试用期内：

    if ($user->onTrial('main')) {
        //
    }

    if ($user->subscription('main')->onTrial()) {
        //
    }

<a name="without-credit-card-up-front"></a>
### 不需要预先绑定信用卡

如果你不要求用户在开始试用期的时候绑定信用卡，你可以直接更新用户表里的 `trial_ends_at` 字段。一般的操作是在注册用户的时候设置 `trial_ends_at` 试用期过期时间：

    $user = User::create([
        // Populate other user properties...
        'trial_ends_at' => Carbon::now()->addDays(10),
    ]);

因为这种试用类型不需要绑定订阅，Cashier 称呼这种类型的试用为「通用试用（generic trial）」。如果 `trial_ends_at` 这个时间还没过期的话，`User` 实例上的 `onTrial` 会返回 `true`：

    if ($user->onTrial()) {
        // 用户是否还在试用期
    }

你可以使用 `onGenericTrial` 来判断某个用户是否还在试用期内，并且使用的是「通用试用」类型：

    if ($user->onGenericTrial()) {
        // 用户是否还在 「通用试用」类型试用期限内
    }

当用户准备好要订阅的时候，你可以使用 `newSubscription` 方法：

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')->create($creditCardToken);

<a name="handling-stripe-webhooks"></a>
## 处理 Stripe Webhooks

<a name="handling-failed-subscriptions"></a>
### 订购失败

如果顾客的信用卡过期了呢？无需担心，Cashier 包含了 Webhook 控制器，可以帮你简单的取消顾客的订单。只要在路由注册控制器：

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );


这样就完成了！失败的交易会经由控制器捕捉并进行处理。控制器在 Stripe 确认订购已经失败后 (通常在三次交易尝试失败后)，才会取消顾客的订单。别忘了：你必须设置 Stripe 控制设置里的 webhook URI。

由于 Stripe Webhooks 必须绕过 Laravel 的 [CSRF 验证](/docs/{{version}}/routing#csrf-protection)，请确定在增加 URI 异常至你的 `VerifyCsrfToken` 中间件，或者在把路由设置于 `web` 路由组外：

    protected $except = [
        'stripe/*',
    ];

<a name="handling-other-webhooks"></a>
### 其它 Webhooks

如果你想要处理额外的 Stripe Webhook 事件，只需要继承 Webhook 控制器。你的方法名称要对应到 Cashier 默认的名称，尤其是方法名称应该使用 `handle` 前缀，并使用「驼峰式」命名法，后面接着你想要处理的 Stripe webhook。例如，如果你想要处理 `invoice.payment_succeeded` webhook，你应该增加一个 `handleInvoicePaymentSucceeded` 方法到控制器。

    <?php

    namespace App\Http\Controllers;

    use Laravel\Cashier\Http\Controllers\WebhookController as BaseController;

    class WebhookController extends BaseController
    {
        /**
         * 处理一个 stripe webhook。
         *
         * @param  array  $payload
         * @return Response
         */
        public function handleInvoicePaymentSucceeded($payload)
        {
            // 处理该事件
        }
    }

<a name="handling-braintree-webhooks"></a>
## 处理 Braintree Webhooks

<a name="handling-braintree-failed-subscriptions"></a>
### 订购失败

如果顾客的信用卡过期了呢？无需担心，Cashier 包含了 Webhook 控制器，可以帮你简单的取消顾客的订单。只要在路由注册控制器：

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

这样就完成了！失败的交易会经由控制器捕捉并进行处理。控制器在 Braintree 确认订购已经失败后 (通常在三次交易尝试失败后)，才会取消顾客的订单。别忘了：你必须设置 Stripe 控制设置里的 webhook URI。

由于 Braintree Webhooks 必须绕过 Laravel 的 [CSRF 验证](/docs/{{version}}/routing#csrf-protection)，请确定在增加 URI 异常至你的 `VerifyCsrfToken` 中间件，或者在把路由设置于 `web` 路由组外：

    protected $except = [
        'braintree/*',
    ];

<a name="handling-braintree-other-webhooks"></a>
### 其他的 Webhooks

如果你需要处理其他的 Braintree Webhook 事件，你可以直接扩展 Webhook 控制器。方法的命名规范是依照 Webhook 来命名的，在 Webhook 前面增加前缀，然后使用驼峰命名法。如：`dispute_opened` 接口，对应的方法名为 `handleDisputeOpened`：

    <?php

    namespace App\Http\Controllers;

    use Braintree\WebhookNotification;
    use Laravel\Cashier\Http\Controllers\WebhookController as BaseController;

    class WebhookController extends BaseController
    {
        /**
         * 处理一个 Braintree webhook
         *
         * @param  WebhookNotification  $webhook
         * @return Response
         */
        public function handleDisputeOpened(WebhookNotification $notification)
        {
            // 处理事件的逻辑
        }
    }

<a name="single-charges"></a>
## 一次性收费

### 简单收费

> **注意：** 当你使用 Stripe 的时候，`charge` 会接受当前应用使用的货币最小分母，而 Braintree 使用的是完整的美元单位。

如果你想对一个已订购客户的信用卡进行「一次性」收费，你可以对一个交易模型实例使用 `charge` 方法。`charge` 方法接受你想收取 **应用程序使用货币的最低单位** 的金额。所以，举例来说，下方的例子将会对用户的信用卡收取 100 美分，或是 1 美元：

    // Stripe 收取的是以美分为单位
    $user->charge(100);

    // Braintree 收取的是以美元为单位
    $user->charge(1);

`charge` 方法接受一个数组作为第二个参数，你可以传递任何你希望的选项至底层的 Stripe 付费创建器：

    $user->charge(100, [
        'custom_option' => $value,
    ]);

当收费失败时 `charge` 方法会抛出异常。完整的 Stripe / Braintree `response` 响应会被返回：

    try {
        $response = $user->charge(100);
    } catch (Exception $e) {
        //
    }

### 收费并生成发票

你可以使用 `invoiceFor` 来一次性收费并生成发票，例如 `一次性收费` 是 $5.00。

    // Stripe 收取的是以美分为单位
    $user->invoiceFor('One Time Fee', 500);

    // Braintree 收取的是以美元为单位
    $user->invoiceFor('One Time Fee', 5);

此次对用户的信用卡收费会在即刻发生。方法 `invoiceFor` 还可以接受第三个参数，此参数以数字形式出现，你可以使用此参数在收费创建时給 Stripe / Braintree 传送信息：

    $user->invoiceFor('One Time Fee', 500, [
        'custom-option' => $value,
    ]);

<a name="invoices"></a>
## 发票

你可以很简单的通过 `invoices` 方法获取交易模型的发票数据数组：

    $invoices = $user->invoices();

当列出发票给客户时，你可以使用发票的辅助函数来显示发票的相关信息。举例来说，你可能希望在表格中列出每个发票的信息，让用户可以简单对其中一个进行下载：

    <table>
        @foreach ($invoices as $invoice)
            <tr>
                <td>{{ $invoice->date()->toFormattedDateString() }}</td>
                <td>{{ $invoice->total() }}</td>
                <td><a href="/user/invoice/{{ $invoice->id }}">Download</a></td>
            </tr>
        @endforeach
    </table>

<a name="generating-invoice-pdfs"></a>
#### 生成发票的 PDF

在生成 PDF 的时候，你需要安装 `dompdf` 扩展包：

    composer require dompdf/dompdf

在路由或是控制器中，使用 `downloadInvoice` 方法可以触发发票的 PDF 下载动作，此方法会自动生成正确的 HTTP 响应并发送下载动作至浏览器：

    Route::get('user/invoice/{invoice}', function ($invoiceId) {
        return Auth::user()->downloadInvoice($invoiceId, [
            'vendor'  => 'Your Company',
            'product' => 'Your Product',
        ]);
    });
