## Artisan 

### Artisan 命令参数

创建 Artisan 命令时，您可以各种方式询问输入：`$this->confirm()` （确认），`$this->perialipate()` (预期输入)，`$this->choice()` (选择)。

```php
// Yes or no?
if ($this->confirm('Do you wish to continue?')) {
    //
}

// Open question with auto-complete options
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

// One of the listed options with default index
$name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);
```

### 维护模式

如果你想要在页面上启用维护模式，执行下面的 Artisan 命令:

```bash
php artisan down
```

然后人们会看到默认的 503 页面。
在 Laravel 8 里，你还可以提供的标识：

- 用户将会重定向的路径地址
- 预渲染的维护模式视图页面
- 绕过维护模式的秘钥
- 维护模式返回的状态吗
- 每 X 秒重新加载页面

```bash
php artisan down --redirect="/" --render="errors::503" --secret="1630542a-246b-4b66-afa1-dd72a4c43515" --status=200 --retry=60
```

在 Laravel 8 之前有：

- 维护模式显示的消息
- 每 X 秒重新加载页面
- 允许访问的 IP 地址

```bash
php artisan down --message="Upgrading Database" --retry=60 --allow=127.0.0.1
```

当你完成了维护工作，只需要运行：

```bash
php artisan up
```

### Artisan 命令行帮助

要查看 `Artisan` 命令的相关选项，可以运行 `Artisan` 命令带上 `--help` 标识参数，比如 `php artisan make:model --help` 然后就可以看到你可以用到的诸多选项

```
Options:
  -a, --all             Generate a migration, seeder, factory, and resource controller for the model
  -c, --controller      Create a new controller for the model
  -f, --factory         Create a new factory for the model
      --force           Create the class even if the model already exists
  -m, --migration       Create a new migration file for the model
  -s, --seed            Create a new seeder file for the model
  -p, --pivot           Indicates if the generated model should be a custom intermediate table model
  -r, --resource        Indicates if the generated controller should be a resource controller
      --api             Indicates if the generated controller should be an API controller
  -h, --help            Display this help message
  -q, --quiet           Do not output any message
  -V, --version         Display this application version
      --ansi            Force ANSI output
      --no-ansi         Disable ANSI output
  -n, --no-interaction  Do not ask any interactive question
      --env[=ENV]       The environment the command should run under
  -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
```

### 确认 Laravel 的版本

通过以下命令行，可以查看并确认你的应用所使用 Lavavel 版本
`php artisan --version`

### 从任意处使用 Artisan 命令

你不仅可以在命令行中启动 `Artisan` 命令，还可以携带参数地在代码中启动它，使用 `Artisan::call()· 方法即可：

```php
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
});
```

### 隐藏你的自定义命令
如果你不想在 artisan 命令列表中显示一个特定的命令，请将 `hidden` 属性设置为 `true` 。
```php
class SendMail extends Command
{
    protected $signature = 'send:mail';
    protected $hidden = true;
}
```

如果你输入了 `php artisan`，你就不会在可用命令中看到 `send:mail`。

由 [@sky_0xs](https://twitter.com/sky_0xs/status/1487921500023832579) 提供

### 跳过方法
Laravel的调度器中的跳过方法

你可以在你的命令中使用 `skip` 来跳过一个执行过程
```php
$schedule->command('emails:send')->daily()->skip(function () {
    return Calendar::isHoliday();
});
```

由 [@cosmeescobedo](https://twitter.com/cosmeescobedo/status/1494503181438492675) 提供
