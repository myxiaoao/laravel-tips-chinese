## 工厂 (6 提示)

  - [工厂回调](#工厂回调)
  - [生成带图像的数据工厂或填充](#生成带图像的数据工厂或填充)
  - [使用自定义逻辑覆盖值](#使用自定义逻辑覆盖值)
  - [使用带关联关系的工厂](#使用带关联关系的工厂)
  - [创建模型而不触发任意事件](#创建模型而不触发任意事件)
  - [有用的for方法](#有用的for方法)

### 工厂回调

使用工厂类进行填充数据时，您可以在插入记录后提供回调函数来执行某种操作。

```php
$factory->afterCreating(App\User::class, function ($user, $faker) {
    $user->accounts()->save(factory(App\Account::class)->make());
});
```

### 生成带图像的数据工厂或填充

你是否知道伪造类 (Faker) 不仅可以生成文本值，还可以生成图像？看此处的 avatar 字段，它将生成一个 50x50 的图像:

```php
$factory->define(User::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'email_verified_at' => now(),
        'password' => bcrypt('password'),
        'remember_token' => Str::random(10),
        'avatar' => $faker->image(storage_path('images'), 50, 50)
    ];
});
```

### 使用自定义逻辑覆盖值

当使用工厂类创建记录时，可以使用序列类 (Sequence) 来输入自定义逻辑并将值覆盖

```php
$users = User::factory()
                ->count(10)
                ->state(new Sequence(
                    ['admin' => 'Y'],
                    ['admin' => 'N'],
                ))
                ->create();
```

### 使用带关联关系的工厂

当使用带关联关系的工厂时，`laravel`也提供了魔术方法:

```php
// magic factory relationship methods
User::factory()->hasPosts(3)->create();

// instead of
User::factory()->has(Post::factory()->count(3))->create();
```

由 [@oliverds_](https://twitter.com/oliverds_/status/1441447356323430402)提供

### 创建模型而不触发任意事件

有时，您可能希望`update`给定的模型，而不发送任何事件。您可以使用`updateQuietly`方法来完成此操作

```php
Post::factory()->createOneQuietly();
Post::factory()->count(3)->createQuietly();
Post::factory()->createManyQuietly([
    ['message' => 'A new comment'],
    ['message' => 'Another new comment'],
]);
```

### 有用的for方法

Laravel工厂有一个非常有用的`for`方法。您可以使用它来创建`belongsTo`关系。

```php
public function run()
{
    Product::factory()
        ->count(3);
        ->for(Category::factory()->create())
        ->create();    
}
```

由 [@mmartin_joo](https://twitter.com/mmartin_joo/status/1461002439629361158)提供
