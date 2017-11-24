# Laravel 的收费系统 Cashier

- [简介](#introduction)
- [配置](#configuration)
    - [Stripe](#stripe-configuration)
    - [Braintree](#braintree-configuration)
    - [货币配置](#currency-configuration)
- [订阅](#subscriptions)
    - [创建订阅](#creating-subscriptions)
    - [检查订阅状态](#checking-subscription-status)
    - [修改订阅计划](#changing-plans)
    - [订阅量](#subscription-quantity)
    - [订阅税额](#subscription-taxes)
    - [取消订阅](#cancelling-subscriptions)
    - [恢复订阅](#resuming-subscriptions)
    - [更新信用卡](#updating-credit-cards)
- [试用订阅](#subscription-trials)
    - [有信用卡的情况下](#with-credit-card-up-front)
    - [在没有信用卡的情况下](#without-credit-card-up-front)
- [处理 Stripe Webhooks](#handling-stripe-webhooks)
    - [定义 Webhook 事件处理程序](#defining-webhook-event-handlers)
    - [订阅失败](#handling-failed-subscriptions)
- [处理 Braintree Webhooks    ](#handling-braintree-webhooks)
    - [定义 Webhook 事件处理程序](#defining-braintree-webhook-event-handlers)
    - [订阅失败](#handling-braintree-failed-subscriptions)
- [一次性收费](#single-charges)
- [发票](#invoices)
    - [生成发票的 PDFs](#generating-invoice-pdfs)

<a name="introduction"></a>

## 简介

 Laravel Cashier 提供了直观、流畅的接口来接入 [Stripe's](https://stripe.com) and [Braintree's](https://www.braintreepayments.com) 订阅付费服务。它可以处理几乎所有你写起来非常头疼付费订阅代码。除了提供基本的订阅管理之外，Cashier 还可以帮你处理优惠券，交换订阅、订阅“数量”、取消宽限期，甚至还可以生成pdf文档。

> {note} 如果你只是“一次性”的收费并且不提供订阅。你就不应该使用 Cashier 。可以直接使用 Stripe 和 Braintree 的 sdk。

<a name="configuration"></a>

## 配置

<a name="stripe-configuration"></a>

### Stripe

#### Composer

首先, 将 Stripe 的 Cashier 包添加到您的依赖项中：

    composer require "laravel/cashier":"~7.0"

#### 服务提供者
下一步, 在 `config/app.php` 配置文件中，注册 `Laravel\Cashier\CashierServiceProvider` [服务提供者](/docs/{{version}}/providers).

#### 数据库迁移

在使用 Cashier 之前，我们需要 [准备数据库](/docs/{{version}}/migrations). 我们需要向您的 `users` 表中添加几个列，并创建一个新的 `subscriptions` 表来保存所有客户的订阅：

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

一旦迁移文件建立好后，运行 Artisan 的 `migrate` 命令。

#### Billable 模型

下一步, 需要添加 `Billable` trait 到你的模型定义。trait 提供各种方法让您执行常见的帐单任务，例如：创建订阅、应用优惠券和更新信用卡信息：

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### API Keys

最后, 你需要在 `services.php` 配置文件中设置你的 Stripe key 。 你可以在 Stripe 的控制面板获取到相关的 API keys。

    'stripe' => [
        'model'  => App\User::class,
        'key' => env('STRIPE_KEY'),
        'secret' => env('STRIPE_SECRET'),
    ],

<a name="braintree-configuration"></a>

### Braintree

#### Braintree 注意事项

对于许多操作，Cashier 的 Stripe 和 Braintree 实现都是一样的。这两项服务都提供信用卡支付服务，但 Braintree 支持通过 PayPal 支付。然而，Braintree 也缺少一些由 Stripe 支持的功能。当你决定使用 Stripe 或 Braintree 时，你应该记住以下几点：

<div class="content-list" markdown="1">
- Braintree 支持 PayPal ，而 Stripe 不支持.
- Braintree 不支持订阅的 `increment` 和 `decrement` 方法。这是 Braintree 的限制，而不是 Cashier 限制。
- Braintree 不支持基于百分比的折扣。这是 Braintree 的限制，而不是 Cashier 限制。
</div>

#### Composer

首先，将 Braintree 的 Cashier 包添加到您的依赖项：

    composer require "laravel/cashier-braintree":"~2.0"

#### 服务提供者

下一步, 在 `config/app.php` 配置文件中，注册  `Laravel\Cashier\CashierServiceProvider` [服务提供者](/docs/{{version}}/providers)：

    Laravel\Cashier\CashierServiceProvider::class

#### 信用卡优惠计划

在使用 Cashier 之前，你需要首先在 Braintree 控制面板定义一个 `plan-credit` 折扣。这个折扣会根据用户选择的支付选项匹配合适的折扣比例，比如选择年付还是月付。

在 Braintree 控制面板中配置的折扣总额可以随意填，Cashier 会在每次使用优惠券的时候根据我们自己的定制覆盖该默认值。由于 Braintree 不支持在订阅频率上来匹配折扣比例，所以这一优惠券是必需的。

#### 数据库迁移

开始使用 Cashier 之前, 我们需要 [准备一下数据库](/docs/{{version}}/migrations).我们需要在数据库的 `users` 表中新增几列，以及创建一个新的 `subscriptions` 表来存储客户的订阅信息：

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

一旦迁移文件建立好后，运行 Artisan 的 `migrate` 命令。

#### Billable 模型

下一步, 添加 `Billable` trait 到你的模型定义 ：

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### API Keys

下一步, 你应该在 `services.php` 文件中，参考下面的配置选项:

    'braintree' => [
        'model'  => App\User::class,
        'environment' => env('BRAINTREE_ENV'),
        'merchant_id' => env('BRAINTREE_MERCHANT_ID'),
        'public_key' => env('BRAINTREE_PUBLIC_KEY'),
        'private_key' => env('BRAINTREE_PRIVATE_KEY'),
    ],

然后，你应该向 `AppServiceProvider` 服务提供商的 `boot` 方法中，添加以下的 Braintree SDK 调用：

    \Braintree_Configuration::environment(config('services.braintree.environment'));
    \Braintree_Configuration::merchantId(config('services.braintree.merchant_id'));
    \Braintree_Configuration::publicKey(config('services.braintree.public_key'));
    \Braintree_Configuration::privateKey(config('services.braintree.private_key'));

<a name="currency-configuration"></a>

### 货币配置

Cashier 使用美元（USD）作为默认货币。你可以通过在服务提供者的 `boot` 方法中调用 `Cashier::useCurrency` 方法来更改默认的货币。这个 `useCurrency` 方法接受两个字符串参数：货币和货币符号：

    use Laravel\Cashier\Cashier;

    Cashier::useCurrency('eur', '€');

<a name="subscriptions"></a>

## 订阅

<a name="creating-subscriptions"></a>

### 创建订阅

创建订阅，首先需要获取到一个 billable 模型实例，这通常是 `App\User` 的一个实例。一旦您获取了模型实例，您可以使用 `newSubscription` 方法创建模型的订阅:

    $user = User::find(1);

    $user->newSubscription('main', 'premium')->create($stripeToken);

 `newSubscription` 方法的第一个参数应该是订阅的名称。如果您的应用程序只提供一个订阅，那么您可以将其设置为 `main` or `primary`。
 第二个参数是用户订阅的 Stripe / Braintree 计划。这个值应该与 Stripe 或 Braintree 中的标识符对应。

 `create` 方法接受一个 Stripe 信用卡 / 源令牌，它将开始订阅，并使用客户 ID 和其他相关的账单信息更新数据库。

#### 用户其他的详细信息

如果您想要指定用户其他的详细信息，您可以通过将它们作为第二个参数传递给 `create` 方法：

    $user->newSubscription('main', 'monthly')->create($stripeToken, [
        'email' => $email,
    ]);

要了解更多关于 Stripe 或 Braintree 支持的额外字段，请查看 Stripe 的内容 [创建客户文档](https://stripe.com/docs/api#create_customer) 或对应的 [Braintree 文档](https://developers.braintreepayments.com/reference/request/customer/create/php)。

#### 优惠券

如果您想在创建订阅时使用优惠券，您可以使用 `withCoupon` 方法：

    $user->newSubscription('main', 'monthly')
         ->withCoupon('code')
         ->create($stripeToken);

<a name="checking-subscription-status"></a>

### 检查订阅状态

一旦用户在您的应用程序订阅了，您可以使用各种方便的方法轻松地检查他们的订阅状态。首先，如果用户有一个激活的订阅，那么 `subscribed` 的方法返回 `true` ，即使订阅当前处于试用阶段：

    if ($user->subscribed('main')) {
        //
    }

这个 `subscribed` 方法还可以在 [路由中间件](/docs/{{version}}/middleware) 使用，允许您根据用户的订阅状态对路由和控制器进行访问。

    public function handle($request, Closure $next)
    {
        if ($request->user() && ! $request->user()->subscribed('main')) {
            // 用户并没有完成支付...
            return redirect('billing');
        }

        return $next($request);
    }

如果您想要确定用户是否仍然处于测试阶段，您可以使用 `onTrial` 方法。这个方法对于向用户显示他们仍然处于试用期的警告是很有用的。

    if ($user->subscription('main')->onTrial()) {
        //
    }

可以使用 `subscribedToPlan` 方法可以基于给定的 Stripe / Braintree 计划 ID 来确定用户是否订阅了的该计划，在本例中，我们将确定用户的 `main` 订阅是否激活了 `monthly` 计划：

    if ($user->subscribedToPlan('monthly', 'main')) {
        //
    }

#### 取消订阅状态

为了确定用户是否曾经订阅，但是已经取消了他们的订阅，您可以使用 `cancelled` 方法:

    if ($user->subscription('main')->cancelled()) {
        //
    }

您还可以确定用户是否已经取消了订阅，但是仍然处于订阅的「宽限期」，直到订阅完全过期为止。例如，如果用户在3月5日取消了原定于3月10日到期的订阅，那么用户将在3月10日之前进行「宽限期」。请注意，在此期间，`subscribed` 方法仍然返回 `true` ：

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

<a name="changing-plans"></a>

### 修改订阅计划

用户在您的应用程序中订阅了之后，他们可能会偶尔想要更改一个新的订阅计划。要将一个用户切换到一个新的订阅，需将订阅计划的标识符传递给 `swap` 方法：
    $user = App\User::find(1);

    $user->subscription('main')->swap('provider-plan-id');
如果用户在试用期，试用期的期限会被保留。另外，如果订阅的数量存在「份额」，那么该份额也将保持。
如果你想在更改用户订阅计划的时候取消用户当前订阅的试用期，可以使用 `skipTrial` 方法：

    $user->subscription('main')
            ->skipTrial()
            ->swap('provider-plan-id');

<a name="subscription-quantity"></a>

### 订阅量

> {note} 
订阅数量仅由 Cashier 的 Stripe 支持。Braintree 没有一个对应于 Stripe 的「数量」的特性。

有些时候订阅是会受「数量」影响的。举个例子，你的应用程序的付费方式可能是每个账户 $10 /人/月。你可以使用 `incrementQuantity` 和 `decrementQuantity` 方法轻松的增加或者减少你的订阅数量：

    $user = User::find(1);

    $user->subscription('main')->incrementQuantity();

    // 对当前的订阅数量加 5 ...
    $user->subscription('main')->incrementQuantity(5);

    $user->subscription('main')->decrementQuantity();

    // 对订阅的当前数量减去 5 ...
    $user->subscription('main')->decrementQuantity(5);

或者，您可以使用 `updateQuantity` 方法设置一个特定的数量：

    $user->subscription('main')->updateQuantity(10);

要获得更多关于订阅量的信息，请参考 [Stripe 文档](https://stripe.com/docs/subscriptions/quantities).

<a name="subscription-taxes"></a>

### 订阅税额

要指定用户在订阅中支付的税率百分比，在计费模式上实现 `taxPercentage` 方法，并在0到100之间返回一个数值，不超过2个小数点。

    public function taxPercentage() {
        return 20;
    }

`taxPercentage` 方法使你能够在模型的基础上应用税率，这对于一个跨越多个国家和税率的用户群来说可能是有帮助的。

> {note} `taxPercentage` 方法只适用于订阅付费模式。如果你用 charges 来做 完成一次性收费的，你需要手工指定税率。

<a name="cancelling-subscriptions"></a>

### 取消订阅

要取消订阅，只需在用户的订阅上调用 `cancel` 方法：

    $user->subscription('main')->cancel();

当一个订阅被取消时，Cashier 会自动在你的数据库中设置 `ends_at` 列。这个列用于知道 `subscribed` 的方法何时应该开始返回 `false` 。例如，如果客户在3月1日取消订阅，但是订阅计划直到3月5日才会结束，`subscribed` 方法将会返回 `true` 一直到3月5日。

您可以使用 `onGracePeriod` 方法确定用户是否已经取消了订阅，但是仍然在一个「宽限期」：

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }
如果你想马上取消订阅，请在用户的订阅中调用 `cancelNow` 方法：

    $user->subscription('main')->cancelNow();

<a name="resuming-subscriptions"></a>

### 恢复订阅

如果用户取消了他们的订阅，可以使用 `resume` 的方法来恢复他们的订阅。用户还 **必须** 在他们的宽限期内才能恢复订阅：

    $user->subscription('main')->resume();

如果用户取消订阅，然后在订阅完全过期之前恢复订阅，他们将不会立即被计费。相反，他们的订阅将会被重新激活，需要按照原本的支付流程再次进行支付。

<a name="updating-credit-cards"></a>

### 更新信用卡信息

 `updateCard` 方法可以用来更新客户的信用卡信息。该方法接受一个 Stripe 的 token ，并将新信用卡分配为默认的支付源：

    $user->updateCard($stripeToken);

<a name="subscription-trials"></a>
## 试用订阅

<a name="with-credit-card-up-front"></a>

### 有信用卡的情况下

如果你想要给你的客户提供试用期，同时还要收集支付方法信息，那么你应该在创建订阅时使用 `trialDays` 方法：

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')
                ->trialDays(10)
                ->create($stripeToken);

此方法将在数据库内的订阅记录中设置试用期结束日期，并指示 Stripe / Braintree 在此日期之前不开始向客户收费。

> {note} 如果客户没有在试用期结束之前取消订阅，订阅会被自动结算，所以需要在试用期结束之前通知你的客户。

您可以使用用户实例的 `onTrial` 方法或订阅实例的 `onTrial` 方法来确定用户是否处于试用期。下面两个例子是相同的：
    if ($user->onTrial('main')) {
        //
    }

    if ($user->subscription('main')->onTrial()) {
        //
    }

<a name="without-credit-card-up-front"></a>

### 没有信用卡的情况下

如果您希望在不收集用户的支付方法信息的情况下提供试用期，您可以简单地将用户记录的 `trial_ends_at` 列设置为您想要的试用结束日期。这通常是在用户注册过程中完成的：

    $user = User::create([
        // 填充其他用户属性...
        'trial_ends_at' => Carbon::now()->addDays(10),
    ]);

> {note}请确保在你的模型中已经为 `trial_ends_at` [日期转换器](/docs/{{version}}/eloquent-mutators#date-mutators)。

Cashier 把这种类型的试用引用为「generic trial」, 因为它并没有关联任何已存在的订阅。如果当前的日期没有超过 `trial_ends_at` 的值，那么 `User` 实例上的  `onTrial` 方法将返回 `true` ：

    if ($user->onTrial()) {
        // 用户在试用期内...
    }

如果您希望明确地知道用户处于「generic」的试用期，并且还没有创建实际的订阅，那么您可以使用 `onGenericTrial` 方法：

    if ($user->onGenericTrial()) {
        // 用户在 「generic」 试用期内...
    }

一旦您准备好为用户创建实际的订阅，您可以像往常一样使用 `newSubscription` 方法：

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')->create($stripeToken);

<a name="handling-stripe-webhooks"></a>

## 处理 Stripe Webhooks

 Stripe 和 Braintree 都可以通过 webhooks 通知你的应用程序。要处理 Stripe Webhooks ，定义一个路由指向 Cashier 的 webhook 控制器的。该控制器将处理所有传入的 webhook 请求，并将它们发送到适当的控制器方法：

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

> {note} 一旦你注册了你的路由之后，一定要在你的 Stripe 控制面板设置中配置webhook URL。

默认情况下，该控制器将自动处理在多次失败的支付尝试后自动取消该订阅(根据您的 Stripe 设置所定义);但是，我们很快就会发现，您可以扩展这个控制器来处理任何您喜欢的 webhook 事件。

#### Webhooks & CSRF 保护

因为 Stripe webhooks 需要绕过 Laravel 的 [CSRF 保护](/docs/{{version}}/csrf)，一定要在 `VerifyCsrfToken` 中间件中列出 URI ，或者列出 `web` 中间件组之外的路由：

    protected $except = [
        'stripe/*',
    ];

<a name="defining-webhook-event-handlers"></a>

### 定义 Webhook 事件处理程序

 Cashier 对于失败的支付自动进行取消订阅的处理，但是如果你有其他的 Stripe webhook 事件，你想要处理，可以简单地扩展 Webhook 控制器。您的方法名应该与 Cashier 期望的约定相符，具体来说，方法应该是用 `handle` 前缀，和您希望处理的 Stripe webhook 的「驼峰」名称。例如，如果您希望处理 `invoice.payment_succeeded` webhook ，你应该添加一个 `handleInvoicePaymentSucceeded` 方法到控制器：

    <?php

    namespace App\Http\Controllers;

    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * Handle a Stripe webhook.
         *
         * @param  array  $payload
         * @return Response
         */
        public function handleInvoicePaymentSucceeded($payload)
        {
            // 处理事件
        }
    }

<a name="handling-failed-subscriptions"></a>

### 订阅失败


如果客户的信用卡过期了怎么办?不用担心，Cashier 会有一个 Webhook 控制器，它可以很容易地取消客户的订阅。如上所述，您所需要做的就是指定一个路由到该控制器：

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

就是这样！失败的支付将被控制器捕获和处理。当 Stripe 判断订阅失败时，控制器将取消客户的订阅(通常是在三次失败的付款尝试之后)。

<a name="handling-braintree-webhooks"></a>

## 处理 Braintree Webhooks

Stripe 和 Braintree 都可以通过 webhooks 通知你的应用程序。要处理 Braintree webhook ，可以定义一个路由指向 Cashier 的 webhook 控制器。该控制器将处理所有传入的 webhook 请求，并将它们发送到适当的控制器方法：

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

> {note} 一旦你注册了你的路由之后，一定要在你的 Braintree 控制面板设置中配置webhook URL。


默认情况下，该控制器将自动处理在多次失败的支付尝试后自动取消该订阅(根据您的 Braintree 设置所定义);但是，我们很快就会发现，您可以扩展这个控制器来处理任何您喜欢的 webhook 事件。

#### Webhooks & CSRF 保护

因为 Braintree webhooks 需要绕过 Laravel 的 [CSRF 保护](/docs/{{version}}/csrf)，所以一定要在 `VerifyCsrfToken` 中间件中列出这个 URI ，或者列出 `web` 中间件组之外的路由:

    protected $except = [
        'braintree/*',
    ];

<a name="defining-braintree-webhook-event-handlers"></a>

### 定义 Webhook 事件处理程序

Cashier 对于失败的支付自动进行取消订阅的处理，但是如果你想对 Braintree webhook 事件进行额外的处理，可以简单的扩展一下 Webhook 控制器。你的方法名称应该与 Cashier 的预设惯例一直，特别的是，方法名应该以 `handle` 为前缀，然后以 「驼峰」命名的方式加上你要处理的 Braintree webhook 的名称。比如，如果你希望去处理 `dispute_opened` webhook, 你应该添加一个 `handleDisputeOpened` 方法到控制器：

    <?php

    namespace App\Http\Controllers;

    use Braintree\WebhookNotification;
    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * Handle a Braintree webhook.
         *
         * @param  WebhookNotification  $webhook
         * @return Response
         */
        public function handleDisputeOpened(WebhookNotification $notification)
        {
            // Handle The Event
        }
    }

<a name="handling-braintree-failed-subscriptions"></a>

### 订阅失败

如果客户的信用卡过期了怎么办?不用担心，Cashier 会有一个 Webhook 控制器，它可以很容易地取消客户的订阅。如上所述，您所需要做的就是指定一个路由到该控制器：

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

就是这样！失败的支付将被控制器捕获和处理。当 Braintree 判断订阅失败时，控制器将取消客户的订阅(通常是在三次失败的付款尝试之后)。

<a name="single-charges"></a>

## 一次性收费

### 单次付费

> {note} 当使用 Stripe 时，`charge` 方法会接受你想要在 **应用程序使用的货币的最小面值** 中收取的金额。然而，当使用 Braintree 时，您应该将全部的美元金额传递给 `charge` 方法：

如果您想要对订阅的客户的信用卡进行「一次」的收费，那么您可以在一个 billable 模型实例上使用 `charge` 方法。

    // Stripe Accepts Charges In Cents...
    $user->charge(100);

    // Braintree Accepts Charges In Dollars...
    $user->charge(1);

`charge` 方法接受一个数组作为它的第二个参数，允许您将任何您希望的选项传递给底层的 Stripe / Braintree 费用创建。在创建收费时，请参考 Stripe 或 Braintree 文档，了解您可以选择的选项：

    $user->charge(100, [
        'custom_option' => $value,
    ]);
当支付结算失败是 `charge` 方法会抛出一个异常。如果结算成功，会返回完成的 Stripe / Braintree 响应：

    try {
        $response = $user->charge(100);
    } catch (Exception $e) {
        //
    }

### 费用与发票

有时候你只是需要完成一个一次性的支付，但也要为收费生成一张发票信息，这样你可以提供 PDF 的票据给你的客户。`invoiceFor` 方法就是做这个的。例如，让我们给客户开具一个 $5.00 的「一次性费用」发票：

    // Stripe 以美分结算...
    $user->invoiceFor('One Time Fee', 500);

    // Braintree 以美元结算...
    $user->invoiceFor('One Time Fee', 5);

生成发票将立即对用户的信用卡收取费用。invoiceFor 方法可以接受一个数据作为第三个参数，允许你在 Stripe / Braintree 结算时传递任何可用的参数：

    $user->invoiceFor('One Time Fee', 500, [
        'custom-option' => $value,
    ]);

> {note} `invoiceFor`  方法在当多次尝试失败支付后也会产生一个 Stripe 发票。如果你不需要为失败重试的支付生成发票，你需要在第一次失败结算是就调用 Stripe 的 API 关闭它们。

<a name="invoices"></a>

## 发票

你可以使用 `invoices` 方法轻松获取 billable 模型的所有发票信息：

    $invoices = $user->invoices();

    // Include pending invoices in the results...
    $invoices = $user->invoicesIncludingPending();

当将发票信息列出来给用户时，你可能需要使用发票的辅助函数来显示相关的发票信息。例如，你可能希望将发票列表以表格的形式显示，允许用户轻松的下载它们：

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

### 生成发票的 PDFs

从路由或控制器中，使用 `downloadInvoice` 方法生成发票的 PDF 下载。该方法将自动生成适当的 HTTP 响应，将下载发送到浏览器:

    use Illuminate\Http\Request;

    Route::get('user/invoice/{invoice}', function (Request $request, $invoiceId) {
        return $request->user()->downloadInvoice($invoiceId, [
            'vendor'  => 'Your Company',
            'product' => 'Your Product',
        ]);
    });

## 译者署名
| 用户名 | 头像 | 职能 | 签名 |
|---|---|---|---|
| [@孤雪飘寒](https://laravel-china.org/users/15752)  | <img class="avatar-66 rm-style" src="https://dn-phphub.qbox.me/uploads/avatars/15752_1493141445.jpeg">  |  翻译  | 全桟工程师，[Github](https://github.com/piaohan)，[CSDN](http://blog.csdn.net/msmile_my)|

--- 

> {note} 欢迎任何形式的转载，但请务必注明出处，尊重他人劳动共创开源社区。
> 
> 转载请注明：本文档由 Laravel China 社区 [laravel-china.org](https://laravel-china.org) 组织翻译，详见 [翻译召集帖](https://laravel-china.org/topics/5756/laravel-55-document-translation-call-come-and-join-the-translation)。
> 
> 文档永久地址： https://d.laravel-china.org