## 日志与调试 (5 提示)


### 日志记录参数

你可以使用 `Log::info()`，或使用更短的 `info()` 额外参数信息，来了解更多发生的事情

```php
Log::info('User failed to login.', ['id' => $user->id]);
```

### 更方便的 DD

你可以在你的 `Eloquent` 句子或者任何集合结尾添加 `->dd()`，而不是使用 `dd($result)`

```php
// Instead of
$users = User::where('name', 'Taylor')->get();
dd($users);
// Do this
$users = User::where('name', 'Taylor')->get()->dd();
```

### 使用 context 日志

在最新的 `Laravel 8.49` 中：`Log::withContext()` 将帮助您区分不同请求之间的日志消息。
如果你创建了中间件并且设置了 `context`，所有的长消息将包含在 `context` 中，你将会搜索更容易。

```php
public function handle(Request $request, Closure $next)
{
    $requestId = (string) Str::uuid();

    Log::withContext(['request-id' => $requestId]);

    $response = $next($request);

    $response->header('request-id', $requestId);

    return $response;
}
```

由 [@LaraibiM](https://twitter.com/LaraibiM/status/1437857603263078421)提供

### 快速输出Query的sql

如果你想快速输出一个 `Eloquent query`的sql 你可以调用 `toSql()`方法如下:

```php
$invoices = Invoice::where('client', 'James pay')->toSql();

dd($invoices)
// select * from `invoices` where `client` = ? 
```

 [@devThaer](https://twitter.com/devThaer/status/1438816135881822210)提供

### 开发模式打印所有数据库查询

如果要在开发期间记录所有数据库查询，请将此代码段添加到`AppServiceProvider`

```php
public function boot()
{
    if (App::environment('local')) {
        DB::listen(function ($query) {
            logger(Str::replaceArray('?', $query->bindings, $query->sql));
        });
    }
}
```

 [@mmartin_joo](https://twitter.com/mmartin_joo/status/1473262634405449730)提供

