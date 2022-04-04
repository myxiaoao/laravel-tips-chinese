## 邮件 

### 测试邮件存入laravel.log

如果你想在你的应用中测试邮件内容但是无法或不愿意设置类似 `Mailgun` 的东西，使用 `.env` 参数 `MAIL_DRIVER=log` 然后所有的邮件都会被保存到 `storage/logs/laravel.log` 文件，而不是真实的发送。

### 预览邮件

如果你使用 `Mailables` 发送邮件，您可以在浏览器中预览结果，而无需发送。返回一个 Mailables 作为路由结果:

```php
Route::get('/mailable', function () {
    $invoice = App\Invoice::find(1);
    return new App\Mail\InvoicePaid($invoice);
});
```

### 不使用Mailables预览邮件

不使用Mailables你也可以预览你的邮件。举个例子，当你创建通知的时候你可以指定你的邮件通知中可能用到的markdown文件。

```php
use Illuminate\Notifications\Messages\MailMessage;
Route::get('/mailable', function () {
    $invoice = App\Invoice::find(1);
    return (new MailMessage)->markdown('emails.invoice-paid', compact('invoice'));
});
```

你也可以使用``MailMessage` `对象提供的`view`方法和其他方法。

由 [@raditzfarhan](https://github.com/raditzfarhan)提供

### Laravel 通知中的默认邮件主题

如果您发送 `Laravel` 通知，并且没有在 `toMail()` 中指定主题，默认主题是您的通知类名，驼峰命名进入控制器。
那么，你可以：

```php
class UserRegistrationEmail extends Notification {
    //
}
```

然后您将收到一封主题为` 用户注册的电子邮件` 的电子邮件。

### 向任何人发送通知

你不仅可以发送 `Laravel` 通知 给特定的用户 `$user->notify()`，而且可以发送给你想发给的任何人，通过 `Notification::route()` ，所谓的 “按需” 通知：

```php
Notification::route('mail', 'taylor@example.com')
        ->route('nexmo', '5555555555')
        ->route('slack', 'https://hooks.slack.com/services/...')
        ->notify(new InvoicePaid($invoice));
```

