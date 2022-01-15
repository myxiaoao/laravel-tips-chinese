## 验证

⬆️ [回到顶部](../REAME-zh.md) ⬅️ [上一个 (路由)](./Routing.md) ➡️ [下一个 (集合)](./Collections.md)

1. [图片验证](#图片验证)
2. [自定义验证错误的信息](#自定义验证错误的信息)
3. [用now或yesterday来验证日期](#用now或yesterday来验证日期)
4. [具有某些条件的验证规则](#具有某些条件的验证规则)
5. [更改默认验证消息](#更改默认验证消息)
6. [预验证](#预验证)
7. [第一次验证错误时停止](#第一次验证错误时停止)
8. [不使用validate或者FormRequest就抛出422](#不使用validate或者FormRequest就抛出422)
9. [规则取决于其他条件](#规则取决于其他条件)
10. [使用属性设置首次验证失败时停止](#使用属性设置首次验证失败时停止)
11. [unique规则在软删除全局作用域中无效](#unique规则在软删除全局作用域中无效)
12. [sometimes方法允许你定义验证器在什么时候被应用](#sometimes方法允许你定义验证器在什么时候被应用)
13. [提交自定义验证规则](#提交自定义验证规则)
14. [数组元素验证](#数组元素验证)
15. [Password的defaults方法](#Password的defaults方法)
16. [表单验证重定向请求](#表单验证重定向请求)

17. [Mac验证规则](#Mac验证规则)

### 图片验证

在验证上传的图片时，可以指定所需的尺寸

```php
['photo' => 'dimensions:max_width=4096,max_height=4096']
```

### 自定义验证错误的信息

只需在 `resources/lang/xx/validation.php` 文件创建适当的数组结构，就可以定义定每个 字段、规则和语言的验证错误消息。

```php
'custom' => [
     'email' => [
        'required' => 'We need to know your e-mail address!',
     ],
],
```

### 用now或yesterday来验证日期

您可以使用 `before/after` 的规则验证日期，并将各种字符串作为参数传递，比如: `tomorrow`, `now`, `yesterday`。例如: `'start_date' => 'after:now'`。它在底层下使用 `strtotime ()`。

```php
$rules = [
    'start_date' => 'after:tomorrow',
    'end_date' => 'after:start_date'
];
```

### 具有某些条件的验证规则

如果验证规则依赖于某些条件，则可以通过将 `withValidator()` 添加到 `FormRequest` 类中来修改规则，并在那里指定自定义逻辑。例如，如果您只想为某些用户角色添加验证规则。

```php
use Illuminate\Validation\Validator;
class StoreBlogCategoryRequest extends FormRequest {
    public function withValidator(Validator $validator) {
        if (auth()->user()->is_admin) {
            $validator->addRules(['some_secret_password' => 'required']);
        }
    }
}
```

### 更改默认验证消息

如果要更改特定字段和特定验证规则的默认验证错误消息，只需将 `messages()` 方法添加到` FormRequest `类中。

```php
class StoreUserRequest extends FormRequest
{
    public function rules()
    {
        return ['name' => 'required'];
    }
    
    public function messages()
    {
        return ['name.required' => 'User name should be real name'];
    }
}
```

### 预验证

如果你想在默认的` Laravel `验证之前修改某个字段，或者，换句话说，“准备” 那个字段， `FormRequest `类中有一个方法 `prepareForValidation () `

```php
protected function prepareForValidation()
{
    $this->merge([
        'slug' => Illuminate\Support\Str::slug($this->slug),
    ]);
}
```

### 第一次验证错误时停止

默认情况下，将在列表中返回 Laravel 验证错误，检查所有验证规则。但是如果你想要在第一个错误之后停止这个过程，使用验证规则叫做 bail:

```php
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

如果你需要停止首次错误验证，可以设置`FormRequest` 类中`$stopOnFirstFailure`为`true`:

```php
protected $stopOnFirstFailure = true;
```

### 不使用validate或者FormRequest就抛出422

如果您不使用 `validate()` 或 `Form Request`，但仍然需要使用相同的 `422` 状态码和错误结构抛出错误，那么可以手动抛出 `throw ValidationException::withMessages()`

```php
if (! $user || ! Hash::check($request->password, $user->password)) {
    throw ValidationException::withMessages([
        'email' => ['The provided credentials are incorrect.'],
    ]);
}
```

### 规则取决于其他条件

如果您的规则是动态的并且依赖于其他条件，那么您可以动态地创建该规则数组

```php
    public function store(Request $request)
    {
        $validationArray = [
            'title' => 'required',
            'company' => 'required',
            'logo' => 'file|max:2048',
            'location' => 'required',
            'apply_link' => 'required|url',
            'content' => 'required',
            'payment_method_id' => 'required'
        ];

        if (!Auth::check()) {
            $validationArray = array_merge($validationArray, [
                'email' => 'required|email|unique:users',
                'password' => 'required|confirmed|min:5',
                'name' => 'required'
            ]);
        }
        //
    }
```

### 使用属性设置首次验证失败时停止

在`request`类中使用这个属性设置首次验证失败时停止。

注意 这个跟 `Bail`规则不一样 只在单个规则失败时就停止

```php
/**
* Indicated if the validator should stop
 * the entire validation once a single
 * rule failure has occurred.
 */
protected $stopOnFirstFailure = true;
```

由 [@Sala7JR](https://twitter.com/Sala7JR/status/1436172331198603270)提供

### unique规则在软删除全局作用域中无效

`Rule::unique` 默认不在软删除的全局范围内。但是`使用`withoutTrashed` 时可用。

```php
Rule::unique('users', 'email')->withoutTrashed();
```

由 [@Zubairmohsin33](https://twitter.com/Zubairmohsin33/status/1438490197956702209)提供

### sometimes方法允许你定义验证器在什么时候被应用

`Validator::sometimes` 方法允许你定义验证器在什么时候被应用，基于提供的输入。
这个片段展示了如果购买的物品数量不够，如何禁止使用优惠券。

```php
$data = [
    'coupon' => 'PIZZA_PARTY',
    'items' => [
        [
            'id' => 1,
            'quantity' => 2
        ],
        [
            'id' => 2,
            'quantity' => 2,
        ],
    ],
];

$validator = Validator::make($data, [
    'coupon' => 'exists:coupons,name',
    'items' => 'required|array',
    'items.*.id' => 'required|int',
    'items.*.quantity' => 'required|int',
]);

$validator->sometimes('coupon', 'prohibited', function (Fluent $data) {
    return collect($data->items)->sum('quantity') < 5;
});

// throws a ValidationException as the quantity provided is not enough
$validator->validate();
```

Tip given by [@cerbero90](https://twitter.com/cerbero90/status/1440226037972013056)

### 数组元素验证

如果你想要验证提交的数组元素，使用带`*`号的点符号。

```php
// say you have this array
// array in request 'user_info'
$request->validated()->user_info = [
    [
        'name' => 'Qasim',
        'age' => 26,
    ],
    [
        'name' => 'Ahmed',
        'age' => 23,
    ],
];
// Rule
$rules = [
    'user_info.*.name' => ['required', 'alpha'],
    'user_info.*.age' => ['required', 'numeric'],
];
```

由[HydroMoon](https://github.com/HydroMoon)提供

### 提交自定义验证规则

感谢`Rule::when` 我们可以指定提交验证规则。

下面例子我们可以验证用户是否真的可以对文章点赞。

```php
use Illuminate\Validation\Rule;

public function rules()
{
    return [
        'vote' => Rule::when($user->can('vote', $post), 'required|int|between:1,5'),
    ]
}
```

由 [@cerbero90](https://twitter.com/cerbero90/status/1434426076198014976)提供

### Password的defaults方法

使用`Password::defaults`方法验证用户提供的密码时，可以强制执行特定规则。它包括要求字母、数字、符号等的选项。

```php
class AppServiceProvider
{
    public function boot(): void
    {
        Password::defaults(function () {
            return Password::min(12)
                ->letters()
                ->numbers()
                ->symbols()
                ->mixedCase()
                ->uncompromised();
        })
    }
}
request()->validate([
    ['password' => ['required', Password::defaults()]]
])
```

 [@mattkingshott](https://twitter.com/mattkingshott/status/1463190613260603395)提供

### 表单验证重定向请求

使用表单请求进行验证时，默认情况下，验证错误将重定向回上一页，但您可以覆盖它<br>

只需定义`$redirect`或`redirectRoute`的属性即可


[Link to docs](https://laravel.com/docs/8.x/validation#customizing-the-redirect-location)

### Mac验证规则

`Laravel 8.77`添加了新的`mac_address`规则

```php
$trans = $this->getIlluminateArrayTranslator();
$validator = new Validator($trans, ['mac' => '01-23-45-67-89-ab'], ['mac' => 'mac_address']);
$this->assertTrue($validator->passes());
```

 [@Teacoders] 提供