## 授权 

### 一次检查多个权限

除了 `@can Blade` 指令外，你知道可以用 `@canany` 指令一次检查多个权限吗？

```blade
@canany(['update', 'view', 'delete'], $post)
    // The current user can update, view, or delete the post
@elsecanany(['create'], \App\Post::class)
    // The current user can create a post
@endcanany
```

### 更多关于用户注册的事件

希望在新用户注册后执行一些操作？ 转到 `app/Providers/EventServiceProvider.php` 和 添加更多的监听类，然后在 `$event->user` 对象中实现 `handle()` 方法。

```php
class EventServiceProvider extends ServiceProvider
{
    protected $listen = [
        Registered::class => [
            SendEmailVerificationNotification::class,

            // You can add any Listener class here
            // With handle() method inside of that class
        ],
    ];
```

### 你知道Authonce吗

你可以用用户登录一个请求，使用方法 `Auth::once()`。
不会使用任何会话或 cookie，这意味着该方法在构建无状态 API 时可能很有帮助。

```php
if (Auth::once($credentials)) {
    //
}
```

### 更改用户密码更新的API令牌

当用户的密码更改时，可以方便地更改用户的 API 令牌。
模型：

```php
public function setPasswordAttribute($value)
{
    $this->attributes['password'] = $value;
    $this->attributes['api_token'] = Str::random(100);
}
```

### 覆盖超级管理员的权限

如果你已经定义了网关（Gates）但是又想要覆盖超级管理员的所有权限。 给超级管理员所有权限，你可以在  `AuthServiceProvider.php` 文件中用 `Gate::before()` 语句拦截网关（Gates）。

```php
// Intercept any Gate and check if it's super admin
Gate::before(function($user, $ability) {
    if ($user->is_super_admin == 1) {
        return true;
    }
});

// Or if you use some permissions package...
Gate::before(function($user, $ability) {
    if ($user->hasPermission('root')) {
        return true;
    }
});
```

