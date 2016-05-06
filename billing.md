# Laravel Cashier

- [简介](#introduction)
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
- [一次性收费](#single-charges)
- [发票](#invoices)
    - [生成发票的 PDF](#generating-invoice-pdfs)

<a name="introduction"></a>
## 简介

Laravel Cashier 给 [Stripe](https://stripe.com) 的订购交易服务提供了生动流畅的接口。它基本上处理了所有会让人恐惧的订购管理的相关逻辑。除了基本的订购管理外，Cashier 还可以处理优惠券，订购转换，管理订购「数量」、取消宽限期，甚至生成发票的 PDF 文件。

<a name="configuration"></a>
### 配置

#### Composer 安装

首先，将 Cashier 扩展包添加至 `composer.json` 文件并运行 `composer update` 命令：

    "laravel/cashier": "~5.0" (For Stripe SDK ~2.0, and Stripe APIs on 2015-02-18 version and later)
    "laravel/cashier": "~4.0" (For Stripe APIs on 2015-02-18 version and later)
    "laravel/cashier": "~3.0" (For Stripe APIs up to and including 2015-02-16 version)

> **[Summer](http://github.com/summerblue)：**请使用 require 来安装扩展包，讨论请见 [正确的 Composer 扩展包安装方法](https://phphub.org/topics/1901)

#### 服务提供者

接着，在 `app` 配置文件中注册 `Laravel\Cashier\CashierServiceProvider` [服务提供者](/docs/{{version}}/providers)。

#### 数据库迁移

使用 Cashier 前，我们需要增加几个字段到数据库，你可以使用 `cashier:table` Artisan 命令，创建迁移文件来添加必要字段。举个例子，若要增加字段到 users 数据表，使用命令：`php artisan cashier:table users`。

创建完迁移文件后，只需运行 `migrate` 命令即可。

#### 模型设置

接着，将 `Billable` trait 和适当的日期访问器添加至你的模型定义中：

    use Laravel\Cashier\Billable;
    use Laravel\Cashier\Contracts\Billable as BillableContract;

    class User extends Model implements BillableContract
    {
        use Billable;

        protected $dates = ['trial_ends_at', 'subscription_ends_at'];
    }

你的模型中 `$dates` 属性里添加的字段会指定 Eloquent 必须将该字段返回为 Carbon 或 DateTime 实例，而不是原始字符串。

#### Stripe 密钥

最后，在你的 `services.php` 配置文件中设置 Stripe 密钥：

    'stripe' => [
        'model'  => 'User',
        'secret' => env('STRIPE_API_SECRET'),
    ],

<a name="subscriptions"></a>
## 订购

<a name="creating-subscriptions"></a>
### 创建订购

要创建一个订购，首先要获取可交易的模型实例，这通常会是 `App\User` 的实例。一旦你获取了模型实例，你可以使用 `subscription` 方法来管理模型的订购：

    $user = User::find(1);

    $user->subscription('monthly')->create($creditCardToken);

`create` 方法会自动创建与 Stripe 的交易，以及将 Stripe 客户 ID 和其它相关帐款信息更新到数据库。如果你的方案有在 Stripe 设置试用期，试用的截止日期也会自动保存至用户的记录。

如果你想要实现试用期，但是你想完全用应用程序来管理试用期，而不是在 Stripe 里设置，那么你必须手动设置试用截止日期：

    $user->trial_ends_at = Carbon::now()->addDays(14);

    $user->save();

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

    if ($user->subscribed()) {
        //
    }

`subscribed` 方法很适合用在 [路由中间件](/docs/{{version}}/middleware)，让你可以通过用户的订购状态，过滤访问路由及控制器：

    public function handle($request, Closure $next)
    {
        if ($request->user() && ! $request->user()->subscribed()) {
            // 此用户不是付费用户...
            return redirect('billing');
        }

        return $next($request);
    }

如果你想确认用户是否还在他们的试用期内，你可以使用 `onTrial` 方法。利用此方法可以在页面的顶部展示正在试用期内的提示：

    if ($user->onTrial()) {
        //
    }

`onPlan` 方法可以用 Stripe ID 来确认用户是否订购某方案：

    if ($user->onPlan('monthly')) {
        //
    }

#### 取消订购状态

若要确认用户是否曾经订购过，但现在已取消订购，你可以使用 `cancelled` 方法：

    if ($user->cancelled()) {
        //
    }

你可能想确认用户是否已经取消订购，但是订购还在他们完全到期前的「宽限期」内。例如，如果用户在三月五号取消了订购，但是服务会到三月十号才过期。那么用户到三月十号前都是「宽限期」。注意，`subscribed` 方法在这个期间仍然会返回 `true`。

    if ($user->onGracePeriod()) {
        //
    }

`everSubscribed` 方法可以用来确认用户是否订购过应用程序里的方案：

    if ($user->everSubscribed()) {
        //
    }

<a name="changing-plans"></a>
### 改变方案

当用户在你的应用程序订购之后，他们有时可能想更改自己的订购方案。使用 `swap` 方法可以把用户转换到新的订购方案。举个例子，我们可以简单的将用户切换至 `premium` 订购方案：

    $user = App\User::find(1);

    $user->subscription('premium')->swap();

如果用户还在试用期间，试用服务会跟之前一样可用。如果订单有「数量」，也会和之前一样。当改变方案时，你也可以使用 `prorate` 方法以表示该费用是按照比例计算。此外，你可以使用 `swapAndInvoice` 方法马上开发票给改变方案的用户：

    $user->subscription('premium')
                ->prorate()
                ->swapAndInvoice();

<a name="subscription-quantity"></a>
### 订购数量

有时候订购行为会跟「数量」有关。例如，你的应用程序可能会依照帐号的用户人数，**每人** 每月收取 10 元。你可以使用 `increment` 和 `decrement` 方法简单的调整订购数量：

    $user = User::find(1);

    $user->subscription()->increment();

    // 增加 5 个订购数量...
    $user->subscription()->increment(5);

    $user->subscription()->decrement();

    // 减少 5 个订购数量...
    $user->subscription()->decrement(5);

另外，你也可以使用 `updateQuantity` 方法来设置指定的数量：

    $user->subscription()->updateQuantity(10);

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

    $user->subscription()->cancel();

当订购被取消时，Cashier 会自动更新数据库的 `subscription_ends_at` 字段。这个字段会被用来判断 `subscribed` 方法什么时候该开始返回  `false`。例如，如果顾客在三月一号取消订购，但是服务可以使用到三月五号为止，那么 `subscribed` 方法在三月五号前都会传回 `true`。

你想确认用户是否已经取消他们的订购，但是还在他们的「宽限期」内，可以使用 `onGracePeriod` 方法：

    if ($user->onGracePeriod()) {
        //
    }

<a name="resuming-subscriptions"></a>
### 恢复订购

如果用户已经取消了他们的订购，但你想恢复订购，可以使用 `resume` 方法：

    $user->subscription('monthly')->resume($creditCardToken);

如果客户取消订购后，且接着在服务完全过期前恢复订购，他们将不会在当下被扣款。他们的订购会被重新启动，而付款则会依照平常的周期。

<a name="handling-stripe-webhooks"></a>
## 处理 Stripe Webhooks

<a name="handling-failed-subscriptions"></a>
### 订购失败

如果顾客的信用卡过期了呢？无需担心，Cashier 包含了 Webhook 控制器，可以帮你简单的取消顾客的订单。只要在路由注册控制器：

    Route::post('stripe/webhook', '\Laravel\Cashier\WebhookController@handleWebhook');

这样就完成了！失败的交易会经由控制器捕捉并进行处理。控制器在 Stripe 确认订购已经失败后 (通常在三次交易尝试失败后)，才会取消顾客的订单。别忘了：你必须设置 Stripe 控制设置里的 webhook URI。

由于 Stripe Webhooks 必须绕过 Laravel 的 [CSRF 验证](/docs/{{version}}/routing#csrf-protection)，请确定在增加 URI 异常至你的 `VerifyCsrfToken` 中间件：

    protected $except = [
        'stripe/*',
    ];

<a name="handling-other-webhooks"></a>
### 其它 Webhooks

如果你想要处理额外的 Stripe Webhook 事件，只需要继承 Webhook 控制器。你的方法名称要对应到 Cashier 默认的名称，尤其是方法名称应该使用 `handle` 前缀，并使用「驼峰式」命名法，后面接着你想要处理的 Stripe webhook。例如，如果你想要处理 `invoice.payment_succeeded` webhook，你应该增加一个 `handleInvoicePaymentSucceeded` 方法到控制器。

    <?php

    namespace App\Http\Controllers;

    use Laravel\Cashier\WebhookController as BaseController;

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

<a name="single-charges"></a>
## 一次性收费

如果你想对一个已订购客户的信用卡进行「一次性」收费，你可以对一个交易模型实例使用 `charge` 方法。`charge` 方法接受你想收取 **应用程序使用货币的最低单位** 的金额。所以，举例来说，下方的例子将会对用户的信用卡收取 100 美分，或是 1 美元：

    $user->charge(100);

`charge` 方法接受一个数组作为第二个参数，你可以传递任何你希望的选项至底层的 Stripe 付费创建器：

    $user->charge(100, [
        'source' => $token,
        'receipt_email' => $user->email,
    ]);

当收费失败时 `charge` 方法会返回 `false`。这通常表示收费被拒绝：

    if ( ! $user->charge(100)) {
        // 该收费被拒绝...
    }

如果收费成功，该方法会返回完整的 Stripe 响应。

<a name="invoices"></a>
## 发票

你可以很简单的通过 `invoices` 方法获取交易模型的发票数据数组：

    $invoices = $user->invoices();

当列出发票给客户时，你可以使用发票的辅助函数来显示发票的相关信息。举例来说，你可能希望在表格中列出每个发票的信息，让用户可以简单对其中一个进行下载：

    <table>
        @foreach ($invoices as $invoice)
            <tr>
                <td>{{ $invoice->dateString() }}</td>
                <td>{{ $invoice->dollars() }}</td>
                <td><a href="/user/invoice/{{ $invoice->id }}">下载</a></td>
            </tr>
        @endforeach
    </table>

<a name="generating-invoice-pdfs"></a>
#### 生成发票的 PDF

在路由或是控制器中，使用 `downloadInvoice` 方法可以触发发票的 PDF 下载动作，此方法会自动生成正确的 HTTP 响应并发送下载动作至浏览器：

    Route::get('user/invoice/{invoice}', function ($invoiceId) {
        return Auth::user()->downloadInvoice($invoiceId, [
            'vendor'  => '你的公司',
            'product' => '你的产品',
        ]);
    });


