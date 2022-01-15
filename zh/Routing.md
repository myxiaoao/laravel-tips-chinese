## 路由

⬆️ [回到顶部](../README-zh.md) ⬅️ [上一个 (视图)](./Views.md) ➡️ [下一个 (验证)](./Validation.md)

1. [分组中的分组](#分组中的分组)
2. [通配符子域名](#通配符子域名)
3. [Auth::routes调用之后是什么](#routes调用之后是什么)
4. [路由模型绑定-你可以定义一个Key](#路由模型绑定-你可以定义一个Key)
5. [快速从路由导航到控制器](#快速从路由导航到控制器)
6. [备选路由-当没有匹配到任何路由时](#备选路由-当没有匹配到任何路由时)
7. [使用正则进行路由参数验证](#使用正则进行路由参数验证)
8. [限流-全局配置与按用户配置](#限流-全局配置与按用户配置)
9. [路由中的URL参数](#路由中的URL参数)
10. [按文件为路由分类](#按文件为路由分类)
11. [翻译资源中的动词](#翻译资源中的动词)
12. [自定义资源路由名称](#自定义资源路由名称)
13. [可读性更强的路由列表](#可读性更强的路由列表)
14. [预加载](#预加载)
15. [本地化资源URI](#本地化资源URI)
16. [资源控制器命名](#资源控制器命名)
17. [更简单地高亮你的导航栏](#更简单地高亮你的导航栏)
18. [使用route()辅助函数生成绝对路径](#使用route()辅助函数生成绝对路径)
19. [为你的每个模型重写路由绑定解析器](#为你的每个模型重写路由绑定解析器)
20. [如果你需要一个公共URL但是你想让他们更安全](#如果你需要一个公共URL但是你想让他们更安全)
21. [在中间件中使用Gate](#在中间件中使用Gate)
22. [简单路由-使用箭头函数](#简单路由-使用箭头函数)

### 分组中的分组

在路由中，你可以在分组中创建分组，来实现仅仅为父分组中的某些路由分配中间件。

```php
Route::group(['prefix' => 'account', 'as' => 'account.'], function() {
    Route::get('login', 'AccountController@login');
    Route::get('register', 'AccountController@register');
    
    Route::group(['middleware' => 'auth'], function() {
        Route::get('edit', 'AccountController@edit');
    });
});
```

### 通配符子域名

你可以在分组中定义变量，来创建动态子域名分组，然后将这个变量传递给每一个子路由。

```php
Route::domain('{username}.workspace.com')->group(function () {
    Route::get('user/{id}', function ($username, $id) {
        //
    });
});
```

### routes调用之后是什么

从 Laravel 7 之后，它被分在一个单独的包中，你可以查阅文件 `/vendor/laravel/ui/src/AuthRouteMethods.php`。

```php
public function auth()
{
    return function ($options = []) {
        // Authentication Routes...
        $this->get('login', 'Auth\LoginController@showLoginForm')->name('login');
        $this->post('login', 'Auth\LoginController@login');
        $this->post('logout', 'Auth\LoginController@logout')->name('logout');
        // Registration Routes...
        if ($options['register'] ?? true) {
            $this->get('register', 'Auth\RegisterController@showRegistrationForm')->name('register');
            $this->post('register', 'Auth\RegisterController@register');
        }
        // Password Reset Routes...
        if ($options['reset'] ?? true) {
            $this->resetPassword();
        }
        // Password Confirmation Routes...
        if ($options['confirm'] ?? class_exists($this->prependGroupNamespace('Auth\ConfirmPasswordController'))) {
            $this->confirmPassword();
        }
        // Email Verification Routes...
        if ($options['verify'] ?? false) {
            $this->emailVerification();
        }
    };
}
```

Laravel 7 之前，请查阅  `/vendor/laravel/framework/src/illuminate/Routing/Router.php`。

### 路由模型绑定-你可以定义一个Key

你可以像 `Route::get('api/users/{user}', function (App\User $user) { … }` 这样来进行路由模型绑定，但不仅仅是 ID 字段，如果你想让 {user} 是 username，你可以把它放在模型中：

```php
public function getRouteKeyName() {
    return 'username';
}
```

### 快速从路由导航到控制器

在 `Laravel 8` 之前，这件事情是可选的。在 `Laravel 8` 中这将成为路由的标准语法

不用这么写:

```php
Route::get('page', 'PageController@action');
```

你可以将控制器标识为 `class` 类名:

```php
Route::get('page', [\App\Http\Controllers\PageController::class, 'action']);
```

这样，你就可以在 `PhpStorm`中点击 `PageController` 来跳转到控制器定义，而不是手动去搜索它

或者你想要让路由的定义更简洁，你可以在路由文件的开始提前引入控制器的类。

```php
use App\Http\Controllers\PageController;

// Then:
Route::get('page', [PageController::class, 'action']);
```

### 备选路由-当没有匹配到任何路由时

如果你想为未找到的路由指定其它逻辑，而不是直接显示 404 页面，你可以在路由文件的最后为其创建一个特殊的路由。

```php
Route::group(['middleware' => ['auth'], 'prefix' => 'admin', 'as' => 'admin.'], function () {
    Route::get('/home', 'HomeController@index');
    Route::resource('tasks', 'Admin\TasksController');
});

// Some more routes....
Route::fallback(function() {
    return 'Hm, why did you land here somehow?';
});
```

### 使用正则进行路由参数验证

我们可以在路由中使用 `where`参数 来直接验证参数。一个典型的例子是，当使用语言区域的参数来作为路由前缀时，像是 `fr/blog` 和 `en/article/333` 等，这时我们如何来确保这两个首字母没有被用在其他语言呢？

`routes/web.php`:

```php
Route::group([
    'prefix' => '{locale}',
    'where' => ['locale' => '[a-zA-Z]{2}']
], function () {
    Route::get('/', 'HomeController@index');
    Route::get('article/{id}', 'ArticleController@show');
});
```

### 限流-全局配置与按用户配置

你可以使用 `throttle:60,1` 来限制一些 URL 在每分钟内最多被访问 60 次。

```php
Route::middleware('auth:api', 'throttle:60,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```

另外，你也可以为公开请求和登录用户分别配置：

```php
// maximum of 10 requests for guests, 60 for authenticated users
Route::middleware('throttle:10|60,1')->group(function () {
    //
});
```

此外，你也可以使用数据库字段 `users.rate_limit` 为一些特殊用户设定此值。

```php
Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```

### 路由中的URL参数

如果你在路由中使用数组传入了其它参数，这些键 / 值将会自动配对并且带入 `URL` 查询参数中。

```php
Route::get('user/{id}/profile', function ($id) {
    //
})->name('profile');

$url = route('profile', ['id' => 1, 'photos' => 'yes']); // Result: /user/1/profile?photos=yes
```

### 按文件为路由分类

如果你有一组与某些功能相关的路由，你可以将它们放在一个特殊的文件 `routes/XXXXX.php` 中，然后在 `routes/web.php` 中使用 include 引入它。`

Taylor Otwell 在  [Laravel Breeze](https://github.com/laravel/breeze/blob/1.x/stubs/routes/web.php) 中的例子：
`routes/auth.php`


```php
Route::get('/', function () {
    return view('welcome');
});

Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth'])->name('dashboard');

require __DIR__.'/auth.php';
```

然后，在 `routes/auth.php` 中:

```php
use App\Http\Controllers\Auth\AuthenticatedSessionController;
use App\Http\Controllers\Auth\RegisteredUserController;
// ... more controllers

use Illuminate\Support\Facades\Route;

Route::get('/register', [RegisteredUserController::class, 'create'])
                ->middleware('guest')
                ->name('register');

Route::post('/register', [RegisteredUserController::class, 'store'])
                ->middleware('guest');

// ... A dozen more routes
```

但是，你应该只在路由都各自具有相同的前缀 / 中间件配置时使用 `include()` 来引入路由，否则，更好的选择是将他们分类在 `app/Providers/RouteServiceProvider` 中。

```php
public function boot()
{
    $this->configureRateLimiting();

    $this->routes(function () {
        Route::prefix('api')
            ->middleware('api')
            ->namespace($this->namespace)
            ->group(base_path('routes/api.php'));

        Route::middleware('web')
            ->namespace($this->namespace)
            ->group(base_path('routes/web.php'));

        // ... Your routes file listed next here
    });
}
```

### 翻译资源中的动词

当你使用了资源控制器，但希望变更 `URL` 谓词以适应非英语语言环境下的 `SEO` ，以在路由中用 `/crear` 替换 `/create`，你可以使用 `App\Providers\RouteServiceProvider 中的 Route::resourceVerbs()` 配置。

```php
public function boot()
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);

    // ...
}
```

### 自定义资源路由名称

当使用资源路由时，你可以在 `routes/web.php` 中指定 `->names() 参数`，这样一来，在整个 Laravel 项目中，浏览器中的 URL 前缀和路由名称前缀可能会不同。

```php
Route::resource('p', ProductController::class)->names('products');
```

这行代码将会生成像 `/p`, `/p/{id}`, `/p/{id}/edit` 这样的路由，但是你可以在代码中使用` route('products.index')`, `route('products.create')` 等方式来调用它们。

### 可读性更强的路由列表

你有没有运行过 `php artisan route:list` ，然后发现这个列表又长，可读性又很差。
另一个方法是：
`php artisan route:list --compact`
这样只会输出 3 列，而非 6 列：只展示方法名、 URI 和方法

```
+----------+---------------------------------+-------------------------------------------------------------------------+
| Method   | URI                             | Action                                                                  |
+----------+---------------------------------+-------------------------------------------------------------------------+
| GET|HEAD | /                               | Closure                                                                 |
| GET|HEAD | api/user                        | Closure                                                                 |
| POST     | confirm-password                | App\Http\Controllers\Auth\ConfirmablePasswordController@store           |
| GET|HEAD | confirm-password                | App\Http\Controllers\Auth\ConfirmablePasswordController@show            |
| GET|HEAD | dashboard                       | Closure                                                                 |
| POST     | email/verification-notification | App\Http\Controllers\Auth\EmailVerificationNotificationController@store |
| POST     | forgot-password                 | App\Http\Controllers\Auth\PasswordResetLinkController@store             |
| GET|HEAD | forgot-password                 | App\Http\Controllers\Auth\PasswordResetLinkController@create            |
| POST     | login                           | App\Http\Controllers\Auth\AuthenticatedSessionController@store          |
| GET|HEAD | login                           | App\Http\Controllers\Auth\AuthenticatedSessionController@create         |
| POST     | logout                          | App\Http\Controllers\Auth\AuthenticatedSessionController@destroy        |
| POST     | register                        | App\Http\Controllers\Auth\RegisteredUserController@store                |
| GET|HEAD | register                        | App\Http\Controllers\Auth\RegisteredUserController@create               |
| POST     | reset-password                  | App\Http\Controllers\Auth\NewPasswordController@store                   |
| GET|HEAD | reset-password/{token}          | App\Http\Controllers\Auth\NewPasswordController@create                  |
| GET|HEAD | verify-email                    | App\Http\Controllers\Auth\EmailVerificationPromptController@__invoke    |
| GET|HEAD | verify-email/{id}/{hash}        | App\Http\Controllers\Auth\VerifyEmailController@__invoke                |
+----------+---------------------------------+-------------------------------------------------------------------------+
```

你还可以特别地指定所需要的列：

`php artisan route:list --columns=Method,URI,Name`

```
+----------+---------------------------------+---------------------+
| Method   | URI                             | Name                |
+----------+---------------------------------+---------------------+
| GET|HEAD | /                               |                     |
| GET|HEAD | api/user                        |                     |
| POST     | confirm-password                |                     |
| GET|HEAD | confirm-password                | password.confirm    |
| GET|HEAD | dashboard                       | dashboard           |
| POST     | email/verification-notification | verification.send   |
| POST     | forgot-password                 | password.email      |
| GET|HEAD | forgot-password                 | password.request    |
| POST     | login                           |                     |
| GET|HEAD | login                           | login               |
| POST     | logout                          | logout              |
| POST     | register                        |                     |
| GET|HEAD | register                        | register            |
| POST     | reset-password                  | password.update     |
| GET|HEAD | reset-password/{token}          | password.reset      |
| GET|HEAD | verify-email                    | verification.notice |
| GET|HEAD | verify-email/{id}/{hash}        | verification.verify |
+----------+---------------------------------+---------------------+
```

### 预加载

如果你使用了路由模型绑定，并且你认为不会在绑定关系中使用预加载，请你再想一想。
所以当你用了这样的路由模型绑定:

```php
public function show(Product $product) {
    //
}
```

但是你有一个从属关系，这时候就不能使用 `$product->with('category')` 预加载了吗？ 
你当然可以，使用 `->load()` 来加载关系

But you have a belongsTo relationship, and cannot use $product->with('category') eager loading?<br>
You actually can! Load the relationship with `->load()`

```php
public function show(Product $product) {
    $product->load('category');
    //
}
```

### 本地化资源URI

如果你使用了资源控制器，但是想要将 URL 谓词变为非英语形式的，比如你想要西班牙语的 `/crear` 而不是 `/create` ，你可以使用 `Route::resourceVerbs()` 方法来配置。

```php
public function boot()
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
    //
}
```

### 资源控制器命名

在资源控制器中，你可以在 `routes/web.php` 中指定 `->names()` 参数，这样 `URL` 前缀与路由前缀可能会不同
.
这样会生成诸如  `/p`, `/p/{id}`, `/p/{id}/edit` 等等，但是你可以这样调用它们：

`route('products.index)`
`route('products.create)`
等等

```php
Route::resource('p', \App\Http\Controllers\ProductController::class)->names('products');
```

### 更简单地高亮你的导航栏

使用`Route::is('route-name')`来更简单的高亮你的导航栏

```html
<ul>
    <li @if(Route::is('home')) class="active" @endif>
        <a href="/">Home</a>
    </li>
    <li @if(Route::is('contact-us')) class="active" @endif>
        <a href="/contact-us">Contact us</a>
    </li>
</ul>
```

由 [@anwar_nairi](https://twitter.com/anwar_nairi/status/1443893957507747849)提供

### 使用route辅助函数生成绝对路径

```php
route('page.show', $page->id);
// http://laravel.test/pages/1

route('page.show', $page->id, false);
// /pages/1
```

由 [@oliverds_](https://twitter.com/oliverds_/status/1445796035742240770)提供

### 为你的每个模型重写路由绑定解析器

你可以为你的所有模型重写路由绑定解析器。在这个例子里，我们没有对`URL` 中的 `@` 符号做任何处理，所以 用``resolveRouteBinding` `我可以移除`@`符号 然后解析模型

```php
// Route
Route::get('{product:slug}', Controller::class);

// Request
https://nodejs.pub/@unlock/hello-world

// Product Model
public function resolveRouteBinding($value, $field = null)
{
    $value = str_replace('@', '', $value);
    
    return parent::resolveRouteBinding($value, $field);
}
```

由 [@Philo01](https://twitter.com/Philo01/status/1447539300397195269)提供

### 如果你需要一个公共URL但是你想让他们更安全

如果你需要一个公共URL但是你想让他们更安全，使用 ` Laravel signed URL`

```php
class AccountController extends Controller
{
    public function destroy(Request $request)
    {
        $confirmDeleteUrl = URL::signedRoute('confirm-destroy', [
            $user => $request->user()
        ]);
        // Send link by email...
    }
    
    public function confirmDestroy(Request $request, User $user)
    {
        if (! $request->hasValidSignature()) {
            abort(403);
        }
        
        // User confirmed by clikcing on the email
        $user->delete();
        
        return redirect()->route('home');
    }
}
```

由 [@anwar_nairi](https://twitter.com/anwar_nairi/status/1448239591467589633)提供

### 在中间件中使用Gate

你可以在中间件中使用在 `App\Providers\AuthServiceProvider`设置的`Gate` 

怎么做呢?你可以在路由中添加`can:`和必要`gate`的名字

```php
Route::put('/post/{post}', function (Post $post) {
    // The current user may update the post...
})->middleware('can:update,post');
```

### 简单路由-使用箭头函数

在路由中你可以使用PHP的箭头函数 而不需要用匿名函数。

要做到这一点 你可以使用 `fn() =>` 这样看起来更简单。

```php
// Instead of
Route::get('/example', function () {
    return User::all();
});
// You can
Route::get('/example', fn () => User::all());
```

