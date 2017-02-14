# Laravel 的收费系统 Cashier

- [简介](#introduction)
- [配置](#configuration)
    - [Stripe](#stripe-configuration)
    - [Braintree](#braintree-configuration)
    - [货币配置](#currency-configuration)
- [订阅列表](#subscriptions)
    - [创建订阅](#creating-subscriptions)
    - [查询订阅状态](#checking-subscription-status)
    - [更改订阅计划](#changing-plans)
    - [按量计费订阅](#subscription-quantity)
    - [订阅税额设置](#subscription-taxes)
    - [取消订阅](#cancelling-subscriptions)
    - [恢复订阅](#resuming-subscriptions)
    - [更新信用卡信息](#updating-credit-cards)
- [试用订阅](#subscription-trials)
    - [必须提供信用卡信息](#with-credit-card-up-front)
    - [非必须提供信用卡](#without-credit-card-up-front)
- [处理 Stripe Webhooks](#handling-stripe-webhooks)
    - [定义 Webhook 事件处理器](#defining-webhook-event-handlers)
    - [订阅失败](#handling-failed-subscriptions)
- [处理 Braintree Webhooks](#handling-braintree-webhooks)
    - [定义 Webhook 事件处理器](#defining-braintree-webhook-event-handlers)
    - [订阅失败](#handling-braintree-failed-subscriptions)
- [一次性收费](#single-charges)
- [发票](#invoices)
    - [生成发票的 PDFs](#generating-invoice-pdfs)

<a name="introduction"></a>
## 简介

Laravel Cashier 提供了直观、流畅的接口来接入 [Stripe's](https://stripe.com) 和 [Braintree's](https://braintreepayments.com) 订阅付费服务。它可以处理几乎所有你写起来非常头疼付费订阅代码。除了提供基本的订阅管理之外，Cashier 还可以帮你处理优惠券，更改订阅计划, 按量计费订阅, 已取消订阅宽限期管理, 甚至生成发票的 PDF 文件。

> {note} 如果你只是需要提供一次性的收费服务，建议直接使用 Stripe 和 Braintree 的 SDK，而无需使用 Cashier。

<a name="configuration"></a>
## 配置

<a name="stripe-configuration"></a>
### Stripe

#### Composer

首先，将 Cashier Stripe 扩展包添加到 `composer.json` 文件，然后运行 `composer update` 命令:

    "laravel/cashier": "~7.0"

#### Service 服务提供者

接下来，需要注册 `Laravel\Cashier\CashierServiceProvider` [服务提供者](/docs/{{version}}/providers) 到你的 `config/app.php` 配置文件。

#### 数据库迁移

在使用 Cashier 之前，我们需要 [准备一下数据库](/docs/{{version}}/migrations)。需要在 `users` 表中新增几列，以及创建一个新的 `subscriptions` 表来存储客户的订阅信息：

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

创建好数据库迁移文件之后，需要运行一下 `migrate` Artisan 命令。

#### Billable 模型

现在，需要添加 `Billable` trait 到你的模型定义。`Billable` trait 提供了丰富的方法去允许你完成常见的支付任务，比如创建订阅，使用优惠券以及更新信用卡信息。

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### API Keys

最后，你需要在 `services.php` 配置文件中设置你的 Stripe key。你可以在 Stripe 的控制面板获取到相关的 API keys。

    'stripe' => [
        'model'  => App\User::class,
        'secret' => env('STRIPE_SECRET'),
    ],

<a name="braintree-configuration"></a>
### Braintree

#### Braintree 注意事项

对于大多数操作，Stripe 和 Braintree 在 Cashier 的实现方法是一致的，都提供对信用卡订阅计费的支持。但是 Braintree 是通过 Paypal 进行支付的，所以相对 Stripe 来说会缺少一些功能的支持。在选择该使用 Stripe 还是 Braintree 的时候需要留意这些区别。

<div class="content-list" markdown="1">
- Braintree 支持 PayPal，但是 Stripe 不支持。
- Braintree 并不支持对 subscriptions 使用 `increment` 和 `decrement` 方法。这是 Braintree 本身的限制，并不是 Cahier 的限制。
- Braintree 并不支持基于百分比的折扣。 这也是 Braintree 本身的限制，并不是 Cahier 的限制。
</div>

#### Composer

首先，将 Cashier Braintree 扩展包添加到 `composer.json` 文件，然后运行 `composer update` 命令:

    "laravel/cashier-braintree": "~2.0"

#### 服务提供者

接下来，需要注册 `Laravel\Cashier\CashierServiceProvider` [服务提供者](/docs/{{version}}/providers) 到你的 `config/app/php` 配置文件。

#### 信用卡优惠计划

在使用 Cashier 之前，你需要首先在 Braintree 控制面板定义一个 `plan-credit` 折扣。这个折扣会根据用户选择的支付选项匹配合适的折扣比例，比如选择年付还是月付。

在 Braintree 控制面板中配置的折扣总额可以随意填，Cashier 会在每次使用优惠券的时候根据我们自己的定制覆盖该默认值。我们需要这个优惠券计划的原因是 Braintree 并不原生支持按照订阅频率来匹配折扣比例。

#### 数据库迁移

在使用 Cashier 之前，我们需要 [准备一下数据库](/docs/{{version}}/migrations)。需要在 `users` 表中新增几列，以及创建一个新的 `subscriptions` 表来存储客户的订阅信息：

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

创建好数据库迁移文件之后，需要运行一下 `migrate` Artisan 命令。

#### Billable 模型

接下来，添加 `Billable` trait 到你的模型定义。

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### API Keys

现在，你需要参考下面的示例在 `services.php` 文件中配置相关选项：

    'braintree' => [
        'model'  => App\User::class,
        'environment' => env('BRAINTREE_ENV'),
        'merchant_id' => env('BRAINTREE_MERCHANT_ID'),
        'public_key' => env('BRAINTREE_PUBLIC_KEY'),
        'private_key' => env('BRAINTREE_PRIVATE_KEY'),
    ],

然后你需要将 Braintree SDK 添加到你的 `AppServiceProvider` 的 `boot` 方法中：

    \Braintree_Configuration::environment(config('services.braintree.environment'));
    \Braintree_Configuration::merchantId(config('services.braintree.merchant_id'));
    \Braintree_Configuration::publicKey(config('services.braintree.public_key'));
    \Braintree_Configuration::privateKey(config('services.braintree.private_key'));

<a name="currency-configuration"></a>
### 货币配置

Cashier 使用美元（USD）作为默认货币。你可以在某个服务提供者的 `boot` 方法中使用 `Cashier::useCurrency` 方法来修改默认的货币设置。`useCurrency` 方法接受两个字符串参数：货币名称和货币符号。

    use Laravel\Cashier\Cashier;

    Cashier::useCurrency('eur', '€');

<a name="subscriptions"></a>
## 订阅列表

<a name="creating-subscriptions"></a>
### 创建订阅

创建订阅，首先需要获取到一个 billabel 模型实例，一般情况下就是 `App\User` 实例。然后你可以使用 `newSubscription` 方法来创建该模型的订阅：

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')->create($stripeToken);

`newSubscription` 方法的第一个参数应该是订阅的名称。如果你的应用程序只是提供一个简单的订阅，你可以简单的设置为 `main` 或者 `primary`。第二个参数需要指定用户的 Stripe / Braintree 订阅计划。这里的值应该和在 Stripe 或 Braintree 的中的对应值保持一致。

`create` 方法会创建订阅并且更新客户 ID 和相关的支付信息到数据库。

#### Additional User Details

如果你需要添加额外的客户细节信息，你可以在 `create` 方法的第二个参数中进行传递：

    $user->newSubscription('main', 'monthly')->create($stripeToken, [
        'email' => $email,
    ]);

查阅一下 Stripe 的 [创建客户文档](https://stripe.com/docs/api#create_customer) 或对应的 [Braintree 文档](https://developers.braintreepayments.com/reference/request/customer/create/php) 了解更多支持的客户细节信息的内容。

#### 优惠券

如果你想在创建订阅的时候使用优惠券，你可以使用 `withCoupon` 方法：

    $user->newSubscription('main', 'monthly')
         ->withCoupon('code')
         ->create($stripeToken);

<a name="checking-subscription-status"></a>
### 查询订阅状态

一旦用户在你的应用程序中完成了订阅，你可以使用大量便利的方法来查询他们的订阅状态。首先, `subscribed` 方法会返回 `true` 如果用户已经激活了订阅，即使该订阅正在试用期内。

    if ($user->subscribed('main')) {
        //
    }

也可以在 [路由中间件](/docs/{{version}}/middleware) 中使用 `subscribed` 方法，来基于用户的订阅状态过滤用户请求：

    public function handle($request, Closure $next)
    {
        if ($request->user() && ! $request->user()->subscribed('main')) {
            // 用户并没有完成支付...
            return redirect('billing');
        }

        return $next($request);
    }

如果你想知道用户是否在一个常识周期内，你可以使用 `onTrial` 方法。该方法可以用来提示用户他们还在试用期之内：


    if ($user->subscription('main')->onTrial()) {
        //
    }

`subscribedToPlan` 方法可以基于给定的 Stripe / Braintree 计划 ID 来判断用户是否有订阅该计划。下面的示例，我们在判断用户的 `main` 订阅是否成功激活订阅了 `monthly` 计划。

    if ($user->subscribedToPlan('monthly', 'main')) {
        //
    }

#### 取消订阅的状态

可以使用 `cancelled` 方法来判断用户是否之前已经完成了订阅，但是又已经取消了他们的订阅：

    if ($user->subscription('main')->cancelled()) {
        //
    }

可以去判断用户是否已经取消了他们的订阅，但是还是在订阅的「宽限期」直到订阅完全到期。比方说，如果一个用户在 3 月 5 号取消了订阅，但是原本订阅定于 3 月 10 号到期，那么该用户就会处于「宽限期」直到 3 月 10 号。需要注意的是，`subscribed` 方法在宽限期还是会返回 `true` :

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

<a name="changing-plans"></a>
### 更改订阅计划

在用户完成订阅之后，有时会更改订阅计划。将用户切换到一个新的订阅计划，需要传递订阅计划的 ID 给 `swap` 方法：

    $user = App\User::find(1);

    $user->subscription('main')->swap('provider-plan-id');

如果用户在试用期，试用期的期限会被保留。如果订阅有限量的「份额」存在，该份额数目也会被保留。

如果你想在更改用户订阅计划的时候取消用户当前订阅的试用期，可以使用 `skipTrial` 方法：

    $user->subscription('main')
            ->skipTrial()
            ->swap('provider-plan-id');

<a name="subscription-quantity"></a>
### 订阅份额

> {note} 按量计费订阅方式只支持 Stripe，因为 Braintree 并没有按量计费订阅之类的功能。

有些时候订阅是会受「数量」影响的。举个例子，你的应用程序的付费方式可能是每个账户 $10 /人/月。你可以使用 `incrementQuantity` 和 `decrementQuantity` 方法轻松的增加或者减少你的订阅数量。

    $user = User::find(1);

    $user->subscription('main')->incrementQuantity();

    // 当前的订阅数量加 5 
    $user->subscription('main')->incrementQuantity(5);

    $user->subscription('main')->decrementQuantity();

    // 当前的订阅数量减 5 
    $user->subscription('main')->decrementQuantity(5);

另外，你可以使用 `updateQuantity` 方法指定订阅的数量：

    $user->subscription('main')->updateQuantity(10);

查阅 [Stripe 文档](https://stripe.com/docs/guides/subscriptions#setting-quantities) ，了解更多关于按量订阅的信息。

<a name="subscription-taxes"></a>
### 订阅税额设置

为了设置每笔订阅的税收比例，需要在 billable 模型内实现 `taxPercentage` 方法，该方法返回一个 0 到 100 的数字，精度不超过小数点后两位。

    public function taxPercentage() {
        return 20;
    }

`taxPercentage` 方法可以让你在模型基础上应用税率，这对于用户群跨越多个国家和税收的情况非常有帮助。

> {note} 但是需要注意的是 `taxPercentage` 方法仅适用于订阅付费模式。如果你使用 Cashier 完成一次性收费的业务，你需要手动的指定税率。

<a name="cancelling-subscriptions"></a>
### 取消订阅

取消订阅，可以直接在用户的订阅上调用 `cancel` 方法：

    $user->subscription('main')->cancel();

当订阅被取消之后，Cashier 自动会填充数据库中的 `ends_at` 列。这列用来指定 `subscribed` 方法什么时候应该返回 `false`。比如，如果一个客户在 3 月 1 号取消了订阅，但是订阅并不会结束直到 3 月 5 号才到期，所以 `subscribed` 方法会继续返回 `true` 直到 3 月 5 号。

你也可以使用 `onGracePeriod` 方法来判断用户是否已经取消了订阅，但是还在一个「宽限期」：

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

如果你想直接取消一个订阅，可以直接调用 `cancelNow` 方法：

    $user->subscription('main')->cancelNow();

<a name="resuming-subscriptions"></a>
### 恢复订阅

如果一个用户取消订阅之后，你可以使用 `resume` 方法来恢复他的订阅。该方法**只适用于**用户取消订阅之后的宽限期：

    $user->subscription('main')->resume();

如果用户取消订阅之后，然后恢复订阅之时订阅已经到期，他们并不会被立即支付费用。代替的是，订阅会被简单进入重新激活的状态，需要按照原本的支付流程再次进行支付。

<a name="updating-credit-cards"></a>
### 更新信用卡信息

`updateCard` 方法被用于更新客户的信用卡信息。该方法会接受一个 Stripe 的 token，并设置一个新的信用卡作为默认的结算来源：

    $user->updateCard($stripeToken);

<a name="subscription-trials"></a>
## 试用订阅

<a name="with-credit-card-up-front"></a>
### 必须提供信用卡信息

如果你想用户在选择试用订阅计划的时候必须提供信用卡信息，可以使用在创建订阅时使用 `trialDays` 方法：

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')
                ->trialDays(10)
                ->create($stripeToken);

该方法会设置一个试用期结束日期在数据库的订阅记录上，同时告知 Stripe / Braintree 在该日期之前并不开始结算。

> {note} 如果客户没有在试用期结束之前取消订阅，订阅会被自动结算，所以需要在试用期结束之前通知你的客户。

你可以使用用户实例的 `onTrial` 方法来判断用户是否在试用期内，或者使用订阅实例上的 `onTrial`方法也可以。下面是示例：

    if ($user->onTrial('main')) {
        //
    }

    if ($user->subscription('main')->onTrial()) {
        //
    }

<a name="without-credit-card-up-front"></a>
### 非必须提供信用卡

如果你打算提供一个试用期，并且不需要用户立马提交信用卡信息，你可以简单的在数据库的用户表记录中设置 `trial_ends_at` 列为你需要的试用期结束日期。这是用户注册过程中的典型手法：

    $user = User::create([
        // 填充其他用户属性...
        'trial_ends_at' => Carbon::now()->addDays(10),
    ]);

> {note}  请确保在你的模型中已经为 `trial_ends_at` 添加[日期转换器](/docs/{{version}}/eloquent-mutators#date-mutators)。

Cashier 把这种类型的试用引用为「generic trial」, 因为它并没有关联任何已存在的订阅。`User` 实例上的 `onTrial` 会返回 `true` 值，只要当前的日期没有超过 `trial_ends_at` 的值：

    if ($user->onTrial()) {
        // 用户在试用期内...
    }

你可以使用 `onGenericTrial` 方法来判断用户是否在一个「generic」 试用期间，其实并没有创建任何真实的订阅：

    if ($user->onGenericTrial()) {
        // 用户在 「generic」 试用期内...
    }

一旦你准备创建一个真实的订阅给用户，你通常可以使用 `newSubscription` 方法：

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')->create($stripeToken);

<a name="handling-stripe-webhooks"></a>
## 处理 Stripe Webhooks

Stripe 和 Braintree 都能通过 webhooks 通知大量的事件给你的应用程序。为了处理 Stripe webhooks，定义一个路由指向 Cashier's webhook 控制器。该控制器会处理所有传来的请求并分发到合适的控制器方法：

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

> {note} 注册好路由之后，记得在 Stripe 的控制面板中配置好 webhook 跳转的 URL。

Cashier 的控制器默认会在多次失败的支付尝试后自动取消该订阅（取决于你的 Stripe 设置）；但是，稍后你会发现，你可以根据自己的需要扩展该控制器。

#### Webhooks & CSRF 保护

因为 Stripe webhooks 需要绕过 Laravel 的 [CSRF 保护](/docs/{{version}}/csrf) ，所以需要确保将相关的 URI 作为例外添加到你的 `VerifyCsrfToken` 中间件或者在 `web` 中间件集合之外定义相关路由。

    protected $except = [
        'stripe/*',
    ];

<a name="defining-webhook-event-handlers"></a>
### 定义 Webhook 事件处理器

Cashier 对于失败的支付自动进行取消订阅的处理，但是如果你想对 Stripe webhook 事件进行额外的处理，可以简单的扩展一下 Webhook 控制器。你的方法名称应该与 Cashier 的预设惯例一直，特别的是，方法名应该以 `handle` 为前缀，然后以 「驼峰」命名的方式加上你要处理的 Stripe webhook 的名称。比如，如果你希望去处理 `invoice.payment_succeeded` webhook, 你应该添加一个 `handleInvoicePaymentSucceeded` 方法到控制器：

    <?php

    namespace App\Http\Controllers;

    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * 处理 Stripe webhook。
         *
         * @param  array  $payload
         * @return Response
         */
        public function handleInvoicePaymentSucceeded($payload)
        {
            // Handle The Event
        }
    }

<a name="handling-failed-subscriptions"></a>
### 订阅失败

如果客户的信用卡过期了怎么办？不用担心 - Cashier 的 webhook 控制器会轻易的取消客户的订阅。像上面提到的，你所需要做的知识简单的指定一个路由到该控制器：

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

就这么简单！失败的支付会被控制器捕捉、并处理。Cashier 控制器会自动取消客户的订阅当 Stripe 判断该订阅已经失败（正常会经过三次错误的支付尝试）。

<a name="handling-braintree-webhooks"></a>
## 处理 Braintree Webhooks

Stripe 和 Braintree 都能通过 webhooks 通知大量的事件给你的应用程序。为了处理 Braintree webhooks，定义一个路由指向 Cashier's webhook 控制器。该控制器会处理所有传来的请求并分发到合适的控制器方法：

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

> {note} 注册好路由之后，记得在 Braintree 的控制面板中配置好 webhook 跳转的 URL。

Cashier 的控制器默认会在多次失败的支付尝试后自动取消该订阅（取决于你的 Braintree 设置）；但是，稍后你会发现，你可以根据自己的需要扩展该控制器。

#### Braintree & CSRF 保护

因为 Stripe webhooks 需要绕过 Laravel 的 [CSRF 保护](/docs/{{version}}/csrf) ，所以需要确保将相关的 URI 作为例外添加到你的 `VerifyCsrfToken` 中间件或者在 `web` 中间件集合之外定义相关路由。


    protected $except = [
        'braintree/*',
    ];

<a name="defining-braintree-webhook-event-handlers"></a>
### 定义 Webhook 事件处理器

Cashier 对于失败的支付自动进行取消订阅的处理，但是如果你想对 Braintree webhook 事件进行额外的处理，可以简单的扩展一下 Webhook 控制器。你的方法名称应该与 Cashier 的预设惯例一直，特别的是，方法名应该以 `handle` 为前缀，然后以 「驼峰」命名的方式加上你要处理的 Braintree webhook 的名称。比如，如果你希望去处理 `dispute_opened` webhook, 你应该添加一个 `handleDisputeOpened` 方法到控制器：

    <?php

    namespace App\Http\Controllers;

    use Braintree\WebhookNotification;
    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * 处理 Braintree webhook。
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

如果客户的信用卡过期了怎么办？不用担心 - Cashier 的 webhook 控制器会轻易的取消客户的订阅。像上面提到的，你所需要做的知识简单的指定一个路由到该控制器：

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

就这么简单！失败的支付会被控制器捕捉、并处理。Cashier 控制器会自动取消客户的订阅当 Braintree 判断该订阅已经失败（正常会经过三次错误的支付尝试）。不要忘了：你需要在 Braintree 的控制面板正确配置 webhook URI。

<a name="single-charges"></a>
## 一次性收费

### 单次付费

> {note} 使用 Stripe 时, `charge` 方法是以你当前货币的**最小面值**作为单位的。但是，使用 Braintree 时，你应该传递完整的美元金额给 `charge` 方法:

如果你想对一个已订阅客户执行「单次」收费，你可以在 billable 模型实例上使用 `charge` 方法。

    // Stripe 以美分结算...
    $user->charge(100);

    // Braintree 以美元结算...
    $user->charge(1);

`charge` 方法可以接受一个数据作为第二个参数，允许你在 Stripe / Braintree 支付时传递任何可用的参数。查阅 Stripe 或 Braintree 的文档了解都有哪些可用的参数。

    $user->charge(100, [
        'custom_option' => $value,
    ]);

当支付结算失败是 `charge` 方法会抛出一个异常。如果结算成功，会返回完成的 Stripe / Braintree 响应。

    try {
        $response = $user->charge(100);
    } catch (Exception $e) {
        //
    }

### 结算时生成发票

有时候你只是需要完成一个一次性的支付，但是也需要为结算生成一个发票信息，这样你可以提供 PDF 的票据给你的客户。`invoiceFor` 方法就是做这个的。例如，让我们给客户开具一个 $5.00 的「单次收费」发票：

    // Stripe 以美分结算...
    $user->invoiceFor('One Time Fee', 500);

    // Braintree 以美元结算...
    $user->invoiceFor('One Time Fee', 5);

生成发票将立即对用户的信用卡收取费用。`invoiceFor` 方法可以接受一个数据作为第三个参数，允许你在 Stripe / Braintree 结算时传递任何可用的参数。

    $user->invoiceFor('One Time Fee', 500, [
        'custom-option' => $value,
    ]);

> {note} `invoiceFor` 方法在当多次尝试失败支付后也会产生一个 Stripe 发票。如果你不需要为失败重试的支付生成发票，你需要在第一次失败结算是就调用 Stripe 的 API 关闭它们。

<a name="invoices"></a>
## 发票

你可以使用 `invoices` 方法轻松获取 billable 模型的所有发票信息：

    $invoices = $user->invoices();

    // 在结果中包含待开发票...
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

在生成发票的 PDF 文件之前，你需要先安装 `dompdf` PHP 库：

    composer require dompdf/dompdf

然后，在路由或控制器内，使用 `downloadInvoice` 方法来生成一个发票的 PDF 的下载响应。浏览器会自动开始下载该文件：

    use Illuminate\Http\Request;

    Route::get('user/invoice/{invoice}', function (Request $request, $invoiceId) {
        return $request->user()->downloadInvoice($invoiceId, [
            'vendor'  => 'Your Company',
            'product' => 'Your Product',
        ]);
    });
