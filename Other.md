## 其他 (50 提示)


### localhost配置

不要忘记将 `.env` 文件中的 `app_url` 从 `http://localhost` 中改为真实的 `URL`，因为它将是你的电子邮件通知和任何其他链接的基础。

```
APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:9PHz3TL5C4YrdV6Gg/Xkkmx9btaE93j7rQTUZWm2MqU=
APP_DEBUG=true
APP_URL=http://localhost
```

### 何时运行或不运行composer-update

与`Laravel`不是很相关，但是… 永远不要在生产服务器上运行 `composer update` ，它很慢，会 “破坏” 存储库。始终在你电脑上本地运行 `composer update` ，将新的 `composer.lock` 提交到存储库，然后再在生产服务器运行 `composer install`。

### Composer检查新版本

如果你想找出 `composer.json` 包中已经发布的较新版本，直接运行 `composer outdated`。你会得到一个包含所有信息的完整列表，如下所示。

```
phpdocumentor/type-resolver 0.4.0 0.7.1
phpunit/php-code-coverage   6.1.4 7.0.3 Library that provides collection, processing, and rende...
phpunit/phpunit             7.5.9 8.1.3 The PHP Unit Testing framework.
ralouphie/getallheaders     2.0.5 3.0.3 A polyfill for getallheaders.
sebastian/global-state      2.0.0 3.0.0 Snapshotting of global state
```

### 自动大写翻译

在翻译文件中`（resources/lang）`，你不仅可以指定变量为`:variable` ，也可以指定大写为`:VARIABLE` 或 `:Variable`，然后你传递的值也会自动大写。

```php
// resources/lang/en/messages.php
'welcome' => 'Welcome, :Name'

// Result: "Welcome, Taylor"
echo __('messages.welcome', ['name' => 'taylor']);
```

### 仅含小时的Carbon

如果你想有当前日期不包含秒或者分钟，用`Carbon`的方法比如：`setSeconds(0)` 或者  `setMinutes(0)`。

```php
// 2020-04-20 08:12:34
echo now();

// 2020-04-20 08:12:00
echo now()->setSeconds(0);

// 2020-04-20 08:00:00
echo now()->setSeconds(0)->setMinutes(0);

// Another way - even shorter
echo now()->startOfHour();
```

### 单动作控制器

如果你想创建一个只有一个动作的控制器，你可以使用 __invoke() 方法创建「可调用（invokable）」控制器。
路由：

```php
Route::get('user/{id}', 'ShowProfile');
```

Artisan 命令:

```bash
php artisan make:controller ShowProfile --invokable
```

控制器 

```php
class ShowProfile extends Controller
{
    public function __invoke($id)
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

### 重定向到特定的控制器方法

你不仅可以 `redirect()` 到 `URL` 或特定的路由，而且可以跳转到一个特定的控制器里的特定方法，甚至向其传递参数。像这样：

```php
return redirect()->action('SomeController@method', ['param' => $value]);
```

### 使用旧版本的Laravel

如果你想用旧版本而非新版本的 `Laravel`，使用这个命令：

```bash
composer create-project --prefer-dist laravel/laravel project "7.*"
```

将 `7.*` 更改为任何你想要的版本。

### 为分页链接添加参数

在默认的分页链接中，你可以传递其他参数，保留原始的查询字符串，甚至指向一个特定的 `#xxxxx` 锚点。

```blade
{{ $users->appends(['sort' => 'votes'])->links() }}

{{ $users->withQueryString()->links() }}

{{ $users->fragment('foo')->links() }}
```

### 可重复回调函数

如果你又一个需要多次重复调用的回调函数，你可以将其声明在一个变量中，然后反复使用它。

```php
$userCondition = function ($query) {
    $query->where('user_id', auth()->id());
};

// Get articles that have comments from this user
// And return only those comments from this user
$articles = Article::with(['comments' => $userCondition])
    ->whereHas('comments', $userCondition)
    ->get();
```


### $request->hasAny

你不仅可以使用 `$request->has()` 方法来查看一个参数，而且可以使用 `$request->hasAny()` 来查看传入的多个参数。

```php
public function store(Request $request) 
{
    if ($request->hasAny(['api_key', 'token'])) {
        echo 'We have API key passed';
    } else {
        echo 'No authorization parameter';
    }
}
```

### 简单分页组件

在分页组件中，如果你只需要「上一页 / 下一页」的链接，而不是需要所有页码，也因此可以使用更少的数据库查询，你只需要将 `simplePaginate() `更改为` paginate()`

```php
// Instead of 
$users = User::paginate(10);

// You can do this
$users = User::simplePaginate(10);
```

### 获取数据的方法

如果你有一个具有复杂数据结构的数组，例如带对象嵌套的数组，你可以使用 `data_get() `助手函数配合通配符和「点」符号，来从嵌套数组或对象中检索值。

```php
// We have an array
[ 
  0 => 
	['user_id' =>'some user id', 'created_at' => 'some timestamp', 'product' => {object Product}, etc], 
  1 =>  
  	['user_id' =>'some user id', 'created_at' => 'some timestamp', 'product' => {object Product}, etc],  
  2 =>  etc
]

// Now we want to get all products ids. We can do like this:

data_get($yourArray,  '*.product.id');

// Now we have all products ids [1, 2, 3, 4, 5, etc...]
```

### Blade指令增加真假条件

`Laravel 8.51` 新增 `@class` 指令，用于添加控制 CSS 类的真 / 假条件。可以在 [文档](https://laravel.com/docs/8.x/blade#conditional-classes) <br>中了解更多
之前: 

```php
<div class="@if ($active) underline @endif">`
```

现在:

```php
<div @class(['underline' => $active])>
```

```php
@php
    $isActive = false;
    $hasError = true;
@endphp

<span @class([
    'p-4',
    'font-bold' => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>

<span class="p-4 text-gray-500 bg-red"></span>
```

由 [@Teacoders](https://twitter.com/Teacoders/status/1445417511546023938)提供

### 任务允许脱离队列

在文档中，任务是在`队列`章节进行讨论的，但是你可以脱离队列来使用`job`，就像传统的委托任务的类一样。只需在控制器中调用 `$this->dispatchNow()` 即可。

```php
public function approve(Article $article)
{
    //
    $this->dispatchNow(new ApproveArticle($article));
    //
}
```

### 在工厂类或seeders外部使用Faker

如果你想要生成一些假数据，你可以在模型工厂或 Seeds 中，甚至任何类的外部使用 Faker。

注意：要在生产模式 `production` 中使用它的话，你需要在 `composer.json` 中，将 faker 从 "require-dev" 移动到 "require" 中。

```php
use Faker;

class WhateverController extends Controller
{
    public function whatever_method()
    {
        $faker = Faker\Factory::create();
        $address = $faker->streetAddress;
    }
}
```

### 可以定时执行的事情

你可以让一些事情以每小时、每天，或是其他时间模式执行。

你可以安排 `artisan` 命令、作业类、可调用类、回调函数、甚至是 `shell` 脚本去定时执行。

```php
use App\Jobs\Heartbeat;

$schedule->job(new Heartbeat)->everyFiveMinutes();
```

```php
$schedule->exec('node /home/forge/script.js')->daily();
```

```php
use App\Console\Commands\SendEmailsCommand;

$schedule->command('emails:send Taylor --force')->daily();

$schedule->command(SendEmailsCommand::class, ['Taylor', '--force'])->daily();
```

```php
protected function schedule(Schedule $schedule)
{
    $schedule->call(function () {
        DB::table('recent_users')->delete();
    })->daily();
}
```

### 检索Laravel文档

如果你想使用一些关键词来检索 `Laravel` 文档，默认情况下只会给出 5 个结果。或许还能给出更多结果。

如果你想要看全部的结果，你可以前往 `Laravel` 文档 的` Github` [仓库](https://github.com/laravel/docs) 直接搜索。



### 过滤route-list

Laravel 8.34 新增： `php artisan route:list` 获得了新的参数 `--except-path`，你可以把一些你不想看见的路由过滤掉。

 [原始PR](https://github.com/laravel/framework/pull/36619)

### 自定义Blade指令

如果你在不同的 `Blade` 文件中格式化数据，可以尝试创建自己的 `Blade` 指令。
下面这一段是来自 `Laravel Cashier` 包的例子：

```php
"require": {
        "laravel/cashier": "^12.9",
}
```

```php
public function boot()
{
    Blade::directive('money', function ($expression) {
        return "<?php echo Laravel\Cashier\Cashier::formatAmount($expression, config('cashier.currency')); ?>";
    });
}
```

```php
<div>Price: @money($book->price)</div>
@if($book->discount_price)
    <div>Discounted price: @money($book->dicount_price)</div>
@endif
```

### Artisan命令帮助

如果您不确定某些 Artisan 命令的参数，或者您想知道可用的参数，只需键入 `php artisan help [command]`。




### 当运行测试时禁用懒加载

当运行测试时如果你想排除掉懒加载 你可以禁用掉懒加载

```php
Model::preventLazyLoading(!$this->app->isProduction() && !$this->app->runningUnitTests());
```

由 [@djgeisi](https://twitter.com/djgeisi/status/1435538167290073090)提供

### 使用两个很好用的辅助函数会带来魔法效果

在这个例子中 服务将会被调用并重试。如果仍然失败 将会被报告。但是请求不会失败。(rescue)

```php
rescue(function () {
    retry(5, function () {
        $this->service->callSomething();
    }, 200);
});
```

由 [@JuanDMeGon](https://twitter.com/JuanDMeGon/status/1435466660467683328)提供

### 请求参数的默认值

以下是我们检测我们使用的`per_page`值是否存在 否则我们用默认值

```php
// Isteand of this
$perPage = request()->per_page ? request()->per_page : 20;

// You can do this
$perPage = request('per_page', 20);
```

由 [@devThaer](https://twitter.com/devThaer/status/1437521022631165957)提供

### 在路由中直接传入中间件而不是注册它

```php
Route::get('posts', PostController::class)
    ->middleware(['auth', CustomMiddleware::class])
```

由 [@sky_0xs](https://twitter.com/sky_0xs/status/1438258486815690766)提供

### 将数组转化成css类

```php
use Illuminate\Support\Arr;

$array = ['p-4', 'font-bold' => $isActive, 'bg-red' => $hasError];

$isActive = false;
$hasError = true;

$classes = Arr::toCssClasses($array);

/*
 * 'p-4 bg-red'
 */
```

由 [@dietsedev](https://twitter.com/dietsedev/status/1438550428833271808)提供

### Laravel-Cashier中的upcomingInvoice方法

您可以显示客户将在下一个计费周期支付的金额。<br>

在`Laravel Cashier（Stripe）`中有一个“upcomingInvoice”方法来获取即将到来的发票详细信息。

```php
Route::get('/profile/invoices', function (Request $request) {
    return view('/profile/invoices', [
        'upcomingInvoice' => $request->user()->upcomingInvoice(),
        'invoices' => $request-user()->invoices(),
    ]);
});
```

由 [@oliverds_](https://twitter.com/oliverds_/status/1439997820228890626)提供

### $request->exists与has

```php
// https://example.com?popular
$request->exists('popular') // true
$request->has('popular') // false 

// https://example.com?popular=foo
$request->exists('popular') // true
$request->has('popular') // true
```

由 [@coderahuljat](https://twitter.com/coderahuljat/status/1442191143244951552)提供

### 返回带变量视图的多种方法

```php
// First way ->with()
return view('index')
    ->with('projects', $projects)
    ->with('tasks', $tasks)

// Second way - as an array
return view('index', [
        'projects' => $projects,
        'tasks' => $tasks
    ]);

// Third way - the same as second, but with variable
$data = [
    'projects' => $projects,
    'tasks' => $tasks
];
return view('index', $data);

// Fourth way - the shortest - compact()
return view('index', compact('projects', 'tasks'));
```

### 调度标准shell命令

我们可以使用`scheduled  command`调度标准shell命令

```php
// app/Console/Kernel.php

class Kernel extends ConsoleKernel
{
    protected function shedule(Schedule $shedule)
    {
        $shedule->exec('node /home/forge/script.js')->daily();
    }
}
```

由y [@anwar_nairi](https://twitter.com/anwar_nairi/status/1448985254794915845)提供



### 无需SSL验证的HTTP请求

有时候你可能会在本地发送一个无需`SSL`验证的`HTTP`请求 可以如下这么干:

```php
return Http::withoutVerifying()->post('https://example.com');
```

如果想设置一些选项 可以使用 ``withOptions``

```php
return Http::withOptions([
    'verify' => false,
    'allow_redirects' => true
])->post('https://example.com');
```

由 [@raditzfarhan](https://github.com/raditzfarhan)提供

### 未经验证的HTTP客户端请求

有时，您可能希望在本地环境中发送HTTP请求而不验证SSL，您可以这样做：

```php
return Http::withoutVerifying()->post('https://example.com');
```

如果要设置多个选项，可以使用`withOptions`。

```php
return Http::withOptions([
    'verify' => false,
    'allow_redirects' => true
])->post('https://example.com');
```

Tip given by [@raditzfarhan](https://github.com/raditzfarhan)

### 不断言任何内容的测试

不断言任何内容的测试，只需启动一些可能引发或不引发异常的内容

```php
class MigrationsTest extends TestCase
{
    public function test_successful_foreign_key_in_migrations()
    {
        // We just test if the migrations succeeds or throws an exception
        $this->expectNotToPerformAssertions();
    }
}
```

### Str的mask方法

Laravel 8.69发布了“Str:：mask（）”方法，该方法使用重复字符屏蔽字符串的一部分.

```php
class PasswordResetLinkController extends Controller
{
    public function sendResetLinkResponse(Request $request)
    {
        $userEmail = User::where('email', $request->email)->value('email'); // username@domain.com
        
        $maskedEmail = Str::mask($userEmail, '*', 4); // user***************
        
        // If needed, you provide a negative number as the third argument to the mask method,
        // which will instruct the method to begin masking at the given distance from the end of the string
        
        $maskedEmail = Str::mask($userEmail, '*', -16, 6); // use******domain.com
    }
}
```

[@Teacoders](https://twitter.com/Teacoders/status/1457029765634744322)

### 扩展Laravel类

在许多内置的`Laravel`类上有一个名为`macro`的方法。例如，集合、Str、Arr、请求、缓存、文件等<br>

您可以在这些类上定义自己的方法，如下所示：

```php
Str::macro('lowerSnake', function (string $str) {
    return Str::lower(Str::snake($str));
});
// Will return: "my-string"
Str::lowerSnake('MyString');
```

Tip given by [@mmartin_joo](https://twitter.com/mmartin_joo/status/1457635252466298885)

### Can特性

Laravel `v8.70`中, 你可以链式调用`can()` 方法替代 `middleware('can:..')`

```php
// instead of
Route::get('users/{user}/edit', function (User $user) {
    ...
})->middleware('can:edit,user');
// you can do this
Route::get('users/{user}/edit', function (User $user) {
    ...
})->can('edit' 'user');
// PS: you must write UserPolicy to be able to do this in both cases
```

[@sky_0xs](https://twitter.com/sky_0xs/status/1458179766192853001)

### 临时下载url

您可以使用云存储资源的临时下载URL来防止不必要的访问。例如，当一个用户想要下载一个文件时，我们重定向到一个s3资源，但是URL在5秒内过期。

```php
public function download(File $file)
{
    // Initiate file download by redirecting to a temporary s3 URL that expires in 5 seconds
    return redirect()->to(
        Storage::disk('s3')->temporaryUrl($file->name, now()->addSeconds(5))
    );
}
```

[@Philo01](https://twitter.com/Philo01/status/1458791323889197064)

### 处理深度嵌套数组

处理深度嵌套数组可能会导致缺少键/值异常。幸运的是，Laravel的data_get（）助手使这一点很容易避免。它还支持深度嵌套的对象<br>

深度嵌套的数组可能缺少所需的属性，这是一场噩梦<br>

在下面的示例中，如果缺少'request'，'user'或'name'，则会出现错误。

```php
$value = $payload['request']['user']['name']
```

Instead, use the `data_get()` helper to access a deeply-nested array item using dot notation.

```php
$value = data_get($payload, 'request.user.name');
```

We can also avoid any errors caused by missing properties by supplying a default value.

```php
$value = data_get($payload, 'request.user.name', 'John');
```

[@mattkingshott](https://twitter.com/mattkingshott/status/1460970984568094722)

### 自定义异常的呈现方式

```php
abstract class BaseException extends Exception
{
    public function render(Request $request)
    {
        if ($request->expectsJson()) {
            return response()->json([
                'meta' => [
                    'valid'   => false,
                    'status'  => static::ID,
                    'message' => $this->getMessage(),
                ],
            ], $this->getCode());
        }
        
        return response()->view('errors.' . $this->getCode(), ['exception' => $this], $this->getCode());
    }
}
```

```php
class LicenseExpiredException extends BaseException
{
    public const ID = 'EXPIRED';
    protected $code = 401;
    protected $message = 'Given license has expired.'
}
```

[@Philo01](https://twitter.com/Philo01/status/1461331239240192003/)

### tap助手函数

`tap`助手是在对对象调用方法后删除单独的`return`语句的好方法。使事物变得美好和干净.

```php
// without tap
$user->update(['name' => 'John Doe']);
return $user;
// with tap()
return tap($user)->update(['name' => 'John Doe']);
```

Tip given by [@mattkingshott](https://twitter.com/mattkingshott/status/1462058149314183171)

### 重置所有剩余的时间单位

您可以在`DateTime::createFromFormat`方法中插入感叹号以重置所有剩余的时间单位

```php
// 2021-10-12 21:48:07.0
DateTime::createFromFormat('Y-m-d', '2021-10-12');
// 2021-10-12 00:00:00.0
DateTime::createFromFormat('!Y-m-d', '2021-10-12');
2021-10-12 21:00:00.0
DateTime::createFromFormat('!Y-m-d H', '2021-10-12');
```

Tip given by [@SteveTheBauman](https://twitter.com/SteveTheBauman/status/1448045021006082054)

### 控制台内核中的计划命令问题可以自动通过电子邮件发送其输出

您知道吗，如果出现问题，您在控制台内核中调度的任何命令都可以自动通过电子邮件发送其输出

```php
$schedule
    ->command(PruneOrganizationsCOmmand::class)
    ->hourly()
    ->emailOutputOnFailure(config('mail.support'));
```

 [@mattkingshott](https://twitter.com/mattkingshott/status/1463160409905455104)

### 使用GET参数构造自定义筛选查询时要小心

```php
if (request()->has('since')) {
    // example.org/?since=
    // fails with illegal operator and value combination
    $query->whereDate('created_at', '<=', request('since'));
}
if (request()->input('name')) {
    // example.org/?name=0
    // fails to apply query filter because evaluates to false
    $query->where('name', request('name'));
}
if (request()->filled('key')) {
    // correct way to check if get parameter has value
}
```

[@mc0de](https://twitter.com/mc0de/status/1465209203472146434)

### 清理你臃肿的路由文件

清掉臃肿的路由文件，并将其拆分以保持组织有序

```php
class RouteServiceProvider extends ServiceProvider
{
    public function boot()
    {
        $this->routes(function () {
            Route::prefix('api/v1')
                ->middleware('api')
                ->namespace($this->namespace)
                ->group(base_path('routes/api.php'));
                
            Route::prefix('webhooks')
                ->namespace($this->namespace)
                ->group(base_path('routes/webhooks.php'));
    
            Route::middleware('web')
                ->namespace($this->namespace)
                ->group(base_path('routes/web.php'));
                
            if ($this->app->environment('local')) {
                Route::middleware('web')
                    ->namespace($this->namespace)
                    ->group(base_path('routes/local.php'));
            }
        });
    }
}
```

[@Philo01](https://twitter.com/Philo01/status/1466068376330153984)

### 自定义邮件日志存储位置

你可以自定义日志存储位置.

通过设置环境变量如下

```
MAIL_MAILER=log
MAIL_LOG_CHANNEL=mail
```

也可以设置日志渠道

```php
'mail' => [
    'driver' => 'single',
    'path' => storage_path('logs/mails.log'),
    'level' => env('LOG_LEVEL', 'debug'),
],
```

[@mmartin_joo](https://twitter.com/mmartin_joo/status/1466362508571131904)

### markdown简单创建

Laravel提供了一个接口来转换HTML中的即时标记，而无需安装新的composer软件包。

```php
$html = Str::markdown('# Changelogfy')
```

Output:

```
<h1>Changelogfy</h1>
```

[@paulocastellano](https://twitter.com/paulocastellano/status/1467478502400315394)

### 给中间件传参数

您可以通过在值后面附加“:”向中间件传递特定路由的参数。例如，我使用一个中间件基于路由实施不同的身份验证方法。

```php
Route::get('...')->middleware('auth.license');
Route::get('...')->middleware('auth.license:bearer');
Route::get('...')->middleware('auth.license:basic');
```

```php
class VerifyLicense
{
    public function  handle(Request $request, Closure $next, $type = null)
    {
        $licenseKey  = match ($type) {
            'basic'  => $request->getPassword(),
            'bearer' => $request->bearerToken(),
            default  => $request->get('key')
        };
        
        // Verify license and return response based on the authentication type
    }
}
```

 [@Philo01](https://twitter.com/Philo01/status/1471864630486179840)

### 获取并删除session

如果您需要从Laravel会话中获取一些东西，并立即忘记它，考虑使用`Session()->pull(value)`。它为您完成了这两个步骤。

```php
// Before
$path = session()->get('before-github-redirect', '/components');
session()->forget('before-github-redirect');
return redirect($path);
// After
return redirect(session()->pull('before-github-redirect', '/components'))
```

 [@jasonlbeggs](https://twitter.com/jasonlbeggs/status/1471905631619715077)提供

### request的date方法

Laravel v8.77: `$request->date()` 方法

现在您不需要手动调用Carbon，您可以执行以下操作：`$post->publish_at=$request->date（'publish_at'）->>addHour()->startOfHour()`
[Link to full pr](https://github.com/laravel/framework/pull/39945) by [@DarkGhostHunter](https://twitter.com/DarkGhostHunter)

### 使用分页时请使用through而不是map

当您想要映射分页数据并只返回字段的子集时，请使用`through`而不是`map`。`map`会打断分页对象并更改其标识。而，`through`作用于分页数据本身

```php
// Don't: Mapping paginated data
$employees = Employee::paginate(10)->map(fn ($employee) => [
    'id' => $employee->id,
    'name' => $employee->name
])
// Do: Mapping paginated data
$employees = Employee::paginate(10)->through(fn ($employee) => [
    'id' => $employee->id,
    'name' => $employee->name
])
```

 [@bhaidar](https://twitter.com/bhaidar/status/1475073910383120399)

