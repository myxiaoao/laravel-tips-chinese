## 数据库模型与 Eloquent

⬆️ [回到顶部](#Laravel-编码技巧) ➡️ [下一个 (模型关联)](#模型关联)

1. [复用或克隆query](#复用或克隆query)
2. [Eloquent where 日期方法](#Eloquent-where日期方法)
3. [增量和减量](#增量和减量)
4. [禁止 timestamp 列](#禁止-timestamp-列)
5. [软删除: 多行恢复](#软删除-多行恢复)
6. [Model all: columns](#Model-all-columns)
7. [To Fail or not to Fail](#To-Fail-or-not-to-Fail)
8. [列名修改](#列名修改)
9. [过滤结果集合](#过滤结果集合)
10. [修改默认的Timestamp 字段](#修改默认的-Timestamp-字段)
11. [按照created_at快速排序](#按照created_at快速排序)
12. [当创建记录时自动修改某些列的值](#当创建记录时自动修改某些列的值)
13. [数据库原始查询计算运行得更快](#数据库原始查询计算运行得更快)
14. [不止一个范围](#不止一个范围)
15. [无需转换 Carbon](#无需转换-Carbon)
16. [根据首字母分组](#根据首字母分组)
17. [永不更新某个字段](#永不更新某个字段)
18. [find () 查询多条数据](#find-()-查询多条数据)
19. [find多个模型并返回多列](#find多个模型并返回多列)
20. [按照键查找](#按照键查找)
21. [使用UUID替换auto-increment](#使用UUID替换auto-increment)
22. [Laravel 中的子查询](#Laravel-中的子查询)
23. [隐藏某些列](#隐藏某些列)
24. [确定DB报错](#确定DB报错)
25. [软删除与查询构造器](#软删除与查询构造器)
26. [SQL声明](#SQL声明)
27. [数据库事务](#数据库事务)
28. [更新或创建](#更新或创建)
29. [保存时移除缓存](#保存时移除缓存)
30. [修改Created_at和Updated_at的格式](#修改Created_at和Updated_at的格式)
31. [数组类型存储到 JSON 中](#数组类型存储到-JSON-中)
32. [复制一个模型](#复制一个模型)
33. [降低内存占用](#降低内存占用)
34. [忽略 $fillable/$guarded 并强制执行](#忽略-$fillable/$guarded-并强制执行)
35. [3层父子级结构](#3层父子级结构)
36. [使用 find() 来搜索更多的记录](#使用-find()-来搜索更多的记录)
37. [失败时执行任何操作](#失败时执行任何操作)
38. [检查记录是否存在否则显示 404](#检查记录是否存在否则显示-404)
39. [条件语句为否时中止](#条件语句为否时中止)
40. [在删除模型之前执行任何额外的操作](#在删除模型之前执行任何额外的操作)
41. [当你需要在保存数据到数据库时自动填充一个字段](#当你需要在保存数据到数据库时自动填充一个字段)
42. [获取查询语句的额外信息](#获取查询语句的额外信息)
43. [在 Laravel 中使用doesntExist()方法](#在-Laravel-中使用doesntExist()方法)
44. [在一些模型的 boot () 方法中自动调用一个特性](#在一些模型的-boot-()-方法中自动调用一个特性)
45. [在 Laravel 中有两种常见的方法来确定一个表是否为空表](#在-Laravel-中有两种常见的方法来确定一个表是否为空表)
46. [如何避免 property of non-object 错误](#如何避免-property-of-non-object-错误)
47. [Eloquent 数据改变后获取原始数据](#Eloquent-数据改变后获取原始数据)
48. [一种更简单创建数据库的方法](#一种更简单创建数据库的方法)
49. [Query构造器的crossJoinSub方法](#Query构造器的crossJoinSub 方法)
50. [belongsToMany的中间表命名](#belongsToMany的中间表命名)
51. [根据Pivot字段排序](#根据Pivot字段排序)
52. [从数据库中查询一条记录](#从数据库中查询一条记录)
53. [记录自动分块](#记录自动分块)
54. [更新模型而不触发事件](#更新模型而不触发事件)
55. [定时清理过期记录中的模型](#定时清理过期记录中的模型)
56. [不变的日期和对它们的强制转换](#不变的日期和对它们的强制转换)
57. [findorfail方法也接收ids数组](#findorfail方法也接收ids数组)

58. [从你的数据库中自动移除模型prunableTrait](#从你的数据库中自动移除模型prunableTrait)
59. [withAggregate方法](#withaggregate方法)
60. [日期转换](#日期转换)
61. [多模型更新/插入](#多模型更新插入)
62. [过滤结果集之后获取查询构造器](#过滤结果集之后获取查询构造器)
63. [选择聚合计算相关模型](#选择聚合计算相关模型)
64. [自定义强制转换](#自定义强制转换)
65. [保存中不要触发事件](#保存中不要触发事件)
66. [基于相关模型的平均值或总数排序](#基于相关模型的平均值或总数排序)
67. [返回事务结果](#返回事务结果)
68. [从query中移除多个公共scope](#从query中移除多个公共scope)

69. [JSON列属性排序](#JSON列属性排序)
70. [从第一个结果中获取单列的值](#从第一个结果中获取单列的值)
71. [检测模型属性是否被修改](#检测模型属性是否被修改)

### 复用或克隆query

通常，我们需要从过滤后的查询中进行更多次查询。因此，大多数时候我们使用 query() 方法，
让我们编写一个查询来获取今天创建的可用和不可用的产品

```php
$query = Product::query();


$today = request()->q_date ?? today();
if($today){
    $query->where('created_at', $today);
}

// 让我们获取可用和不可用的产品
$active_products = $query->where('status', 1)->get(); // 这一行 修改了$query 对象变量
$inactive_products = $query->where('status', 0)->get(); // 所以这里我们将获取不到任何不可用产品
```

但是，在获得 `$active products` 后，`$query` 会被修改。因此 `$inactive_products` 不会从 `$query` 中获取到不可用产品，并且每次都返回空集合。因为，它尝试从 `$active_products` 中查找不可用产品（`$query` 仅返回可用产品）。

为了解决这个问题,我们可以通过重用这个`$query`对象来查询多次。因此我们在做任何对`$query`修改操作的时候需要克隆这个`$query`。

```php
$active_products = (clone $query)->where('status', 1)->get(); // it will not modify the $query
$inactive_products = (clone $query)->where('status', 0)->get(); // so we will get inactive products from $query

```

### Eloquent where日期方法

在 Eloquent 中，使用 `whereDay()`、`whereMonth()`、`whereYear()`、`whereDate()` 和 `whereTime()` 函数检查日期。

```php
$products = Product::whereDate('created_at', '2018-01-31')->get();
$products = Product::whereMonth('created_at', '12')->get();
$products = Product::whereDay('created_at', '31')->get();
$products = Product::whereYear('created_at', date('Y'))->get();
$products = Product::whereTime('created_at', '=', '14:13:58')->get();
```

### 增量和减量

如果要增加数据库某个表中的某个列的值，只需要使用 `increment()` 函数。你不仅可以增加 1，还可以增加其他数字，如 50。

```php
Post::find($post_id)->increment('view_count');
User::find($user_id)->increment('points', 50);
```

### 禁止 timestamp 列

如果你的数据库表不包含 timestamp 字段 `created_at` 和 `updated_at`，你可以使用 `$timestamps = false` 属性指定 Eloquent 模型不使用它们。

```php
class Company extends Model
{
    public $timestamps = false;
}
```

### 软删除-多行恢复

使用软删除时，可以在一个句子中恢复多行。

```php
Post::onlyTrashed()->where('author_id', 1)->restore();
```

### Model all-columns

当调用Eloquent's `Model::all()`时你可以指定返回哪些列。

```php
$users = User::all(['id', 'name', 'email']);
```

### To Fail or not to Fail

除了 `findOrFail()` 之外，还有 Eloquent 方法 `firstOrFail()`，如果没有找到查询记录，它将返回 `404` 页面。

```php
$user = User::where('email', 'povilas@laraveldaily.com')->firstOrFail();
```

### 列名修改

在 `Eloquent Query Builder` 中，您可以像在普通 SQL 查询中一样指定`as`以返回任何列的别名。

```php
$users = DB::table('users')->select('name', 'email as user_email')->get();
```

### 过滤结果集合

在 `Eloquent` 查询到结果之后，您可以使用 Collections 中的 `map()` 函数来修改行数据。

```php
$users = User::where('role_id', 1)->get()->map(function (User $user) {
    $user->some_column = some_function($user);
    return $user;
});
```

### 修改默认的Timestamp 字段

如果您使用的是非 `Laravel` 数据库并且时间戳列的名称不同怎么办？也许，你有 `create_time` 和 `update_time`。 幸运的是，您也可以在模型中指定它们：

```php
class Role extends Model
{
    const CREATED_AT = 'create_time';
    const UPDATED_AT = 'update_time';
}
```

### 按照created_at快速排序

不用:

```php
User::orderBy('created_at', 'desc')->get();
```

你可以更快的使用排序:

```php
User::latest()->get();
```

默认情况下 `latest()` 将按照 `created_at`排序。

有一个相反的方法 `oldest()`，按 created_at 升序排序：

```php
User::oldest()->get();
```

您也可以指定另一列进行排序。 例如，如果你想使用 `updated_at`，你可以这样做：

```php
$lastUpdatedUser = User::latest('updated_at')->first();
```

### 当创建记录时自动修改某些列的值

如果您想在创建记录时生成一些 DB 列值，请将其添加到模型的 boot() 方法中。
例如，如果您有一个字段 「position」，并且想要赋值下一个可用位置（如 Country::max('position') + 1)，请执行以下操作：

```php
class Country extends Model {
    protected static function boot()
    {
        parent::boot();

        Country::creating(function($model) {
            $model->position = Country::max('position') + 1;
        });
    }
}
```

### 数据库原始查询计算运行得更快

使用类似 `whereRaw()` 方法的 SQL 原始查询，直接在查询中进行一些数据库特定计算，而不是在 Laravel 中，通常情况下结果会更快。 例如，如果您想获得注册后 30 天以上仍处于活跃状态的用户，代码如下：

```php
User::where('active', 1)
    ->whereRaw('TIMESTAMPDIFF(DAY, created_at, updated_at) > ?', 30)
    ->get();
```

### 不止一个范围

您可以在 Eloquent 中组合和链式调用查询范围，在一个`query`查询中使用多个范围。

Model文件内:

```php
public function scopeActive($query) {
    return $query->where('active', 1);
}

public function scopeRegisteredWithinDays($query, $days) {
    return $query->where('created_at', '>=', now()->subDays($days));
}
```

控制器内:

```php
$users = User::registeredWithinDays(30)->active()->get();
```

### 无需转换 Carbon

如果你正使用 `whereDate()` 查询今日的记录，可以直接使用 `Carbon` 的 `now()` 方法，它会自动转换为日期进行查询，而不需要指定 ->toDateString()。

```php
// 不用
$todayUsers = User::whereDate('created_at', now()->toDateString())->get();
// 不用做转换 只需要用 now()
$todayUsers = User::whereDate('created_at', now())->get();
```

### 根据首字母分组

你可以用任意自定义条件对 Eloquent 结果进行分组，下面的示例是由用户名的第一个单词进行分组:

```php
$users = User::all()->groupBy(function($item) {
    return $item->name[0];
});
```

### 永不更新某个字段

如果有一个数据库字段你想只设置一次并不想再次更新，您可以在Eloquent的模型上使用一个修改器设置该限制：

```php
class User extends Model
{
    public function setEmailAttribute($value)
    {
        if ($this->email) {
            return;
        }

        $this->attributes['email'] = $value;
    }
}
```

### find () 查询多条数据

`find()`方法可以接受多参数, 传入多个值时会返回所有找到记录的集合，而不是一个模型:

```php
// 返回 Eloquent Model
$user = User::find(1);
// 返回 Eloquent Collection
$users = User::find([1,2,3]);
```

技巧来自 [@tahiriqbalnajam](https://twitter.com/tahiriqbalnajam/status/1436120403655671817)

### find多个模型并返回多列

`find`方法可接受多参数 使得结果集返回指定列的模型集合，而不是模型的所有列:

```php
// Will return Eloquent Model with first_name and email only
$user = User::find(1, ['first_name', 'email']);
// Will return Eloquent Collection with first_name and email only
$users = User::find([1,2,3], ['first_name', 'email']);
```

技巧来自 [@tahiriqbalnajam](https://github.com/tahiriqbalnajam)

### 按照键查找

您还可以使用`whereKey()`方法根据您指定的主键查找多条记录。(默认`id`但是你可以在Eloquent 模型中覆盖掉)

```php
$users = User::whereKey([1,2,3])->get();
```

### 使用UUID替换auto-increment

您不想在模型中使用自动递增 ID？

迁移:

```php
Schema::create('users', function (Blueprint $table) {
    // $table->increments('id');
    $table->uuid('id')->unique();
});
```

模型:

```php
class User extends Model
{
    public $incrementing = false;
    protected $keyType = 'string';

    protected static function boot()
    {
        parent::boot();

        User::creating(function ($model) {
            $model->setId();
        });
    }

    public function setId()
    {
        $this->attributes['id'] = Str::uuid();
    }
}
```

### Laravel 中的子查询

从 Laravel 6 开始，您可以在 Eloquent 语句中使用 `addSelect()`方法，对列进行一些计算。

```php
return Destination::addSelect(['last_flight' => Flight::select('name')
    ->whereColumn('destination_id', 'destinations.id')
    ->orderBy('arrived_at', 'desc')
    ->limit(1)
])->get();
```

### 隐藏某些列

在进行 Eloquent 查询时，如果您想在返回中隐藏特定字段，最快捷的方法之一是在集合结果上添加 `->makeHidden()`。

```php
$users = User::all()->makeHidden(['email_verified_at', 'deleted_at']);
```

### 确定DB报错

如果您想捕获 Eloquent Query 异常，请使用特定的 `QueryException` 代替默认的 `Exception` 类，您将能够获得SQL确切的错误代码。

```php
try {
    // Some Eloquent/SQL statement
} catch (\Illuminate\Database\QueryException $e) {
    if ($e->getCode() === '23000') { // integrity constraint violation
        return back()->withError('Invalid data');
    }
}
```

### 软删除与查询构造器

注意 当你用到 `Eloquent`时 软删除将会起作用，但是查询构造器不行。

```php
// 将排除软删除的条目
$users = User::all();

// 将不会排除软删除的条目
$users = User::withTrashed()->get();

// 将不会排除软删除的条目
$users = DB::table('users')->get();
```

### SQL声明

如果你需要执行一个简单的 SQL 查询，但没有方案 —— 比如改变数据库模式中的某些东西，只需执行 DB::statement()。

```php
DB::statement('DROP TABLE users');
DB::statement('ALTER TABLE projects AUTO_INCREMENT=123');
```

### 数据库事务

如果您执行了两个数据库操作，第二个可能会出错，那么您应该回滚第一个，对吗？
为此，我建议使用 DB Transactions，它在 Laravel 中非常简单：

```php
DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
});
```

### 更新或创建

如果你需要检查记录是否存在，然后更新它，或者创建一个新记录，你可以用一句话来完成 - 使用 Eloquent updateOrCreate() 方法：

```php
// Instead of this
$flight = Flight::where('departure', 'Oakland')
    ->where('destination', 'San Diego')
    ->first();
if ($flight) {
    $flight->update(['price' => 99, 'discounted' => 1]);
} else {
	$flight = Flight::create([
	    'departure' => 'Oakland',
	    'destination' => 'San Diego',
	    'price' => 99,
	    'discounted' => 1
	]);
}
// Do it in ONE sentence
$flight = Flight::updateOrCreate(
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    ['price' => 99, 'discounted' => 1]
);
```

### 保存时移除缓存

由 [@pratiksh404](https://github.com/pratiksh404)提供

如果您缓存了一个键存储了 `posts` 这个集合，想在新增或更新时移除缓存键，可以在您的模型上调用静态的 saved 函数：

```php
class Post extends Model
{
    // Forget cache key on storing or updating
    public static function boot()
    {
        parent::boot();
        static::saved(function () {
           Cache::forget('posts');
        });
    }
}
```

### 修改Created_at和Updated_at的格式

由[@syofyanzuhad](https://github.com/syofyanzuhad)提供

想要改变 `created_at`的格式，您可以在模型中添加一个方法，如下所示:

```php
public function getCreatedAtFormattedAttribute()
{
   return $this->created_at->format('H:i d, M Y');
}
```

你可以在需要改变时间格式时使用 `$entry->created_at_formatted` ，它会返回 `created_at` 的属性如同 `04:19 23, Aug 2020`。

你也可以用同样的方法更改 `updated_at`：

```php
public function getUpdatedAtFormattedAttribute()
{
   return $this->updated_at->format('H:i d, M Y');
}
```

在有需要的时候使用 `$entry->updated_at_formatted`。它会返回 `updated_at` 的属性如同: `04:19 23, Aug 2020` 。

### 数组类型存储到 JSON 中

由[@pratiksh404](https://github.com/pratiksh404)提供

如果你的输入字段有一个数组需要存储为 JSON 格式，你可以在模型中使用 `$casts` 属性。 这里的 `images` 是 JSON 属性

```php
protected $casts = [
    'images' => 'array',
];
```

这样你可以以 JSON 格式存储它，但当你从 DB 中读取时，它会以数组方式使用。

### 复制一个模型

如果你有两个非常相似的模型（比如送货地址和账单地址），而且你想要复制其中一个作为另一个，你可以使用 replicate() 方法并更改一部分属性。

[官方文档的示例](https://laravel.com/docs/8.x/eloquent#replicating-models)

```php
$shipping = Address::create([
    'type' => 'shipping',
    'line_1' => '123 Example Street',
    'city' => 'Victorville',
    'state' => 'CA',
    'postcode' => '90001',
]);

$billing = $shipping->replicate()->fill([
    'type' => 'billing'
]);

$billing->save();
```

### 降低内存占用

有时我们需要将大量的数据加载到内存中，比如：

```php
$orders = Order::all();
```

但如果我们有非常庞大的数据库，这可能会很慢，因为 `Laravel ` 会准备好模型类的对象。在这种情况下，`Laravel `有一个很方便的函数 `toBase()`。

```php
$orders = Order::toBase()->get();
//$orders 将是一个由`StdClass`组成的 `Illuminate\Support\Collection`
```

通过调用这个方法，它将从数据库中获取数据，但它不会准备模型类。同时，向 `get()` 方法传递一个字段数组通常是个好主意，这样可以防止从数据库中获取所有字段。

### 忽略 $fillable/$guarded 并强制执行

如果你为其他开发者创建了一个 Laravel 模板, 然后你不能控制他们以后会在模型的 $fillable/$guarded 中填写什么，你可以使用 forceFill()

```php
$team->update(['name' => $request->name])
```

如果 name 不在`team`模型的 `$fillable` 中，怎么办？或者如果根本就没有 `$fillable/$guarded`， 怎么办？

```php
$team->forceFill(['name' => $request->name])
```

这将忽略该查询的 $fillable 并强制执行。

### 3层父子级结构

If you have a 3-level structure of parent-children, like categories in an e-shop, and you want to show the number of products on the third level, you can use `with('yyy.yyy')` and then add `withCount()` as a condition

```php
class HomeController extend Controller
{
    public function index()
    {
        $categories = Category::query()
            ->whereNull('category_id')
            ->with(['subcategories.subcategories' => function($query) {
                $query->withCount('products');
            }])->get();
    }
}
```

```php
class Category extends Model
{
    public function subcategories()
    {
        return $this->hasMany(Category::class);
    }
    
    public function products()
    {
    return $this->hasMany(Product::class);
    }
}
```

```php
<ul>
    @foreach($categories as $category)
        <li>
            {{ $category->name }}
            @if ($category->subcategories)
                <ul>
                @foreach($category->subcategories as $subcategory)
                    <li>
                        {{ $subcategory->name }}
                        @if ($subcategory->subcategories)
                            <ul>
                                @foreach ($subcategory->subcategories as $subcategory)
                                    <li>{{ $subcategory->name }} ({{ $subcategory->product_count }})</li>
                                @endforeach
                            </ul>
                        @endif
                    </li>
                @endforeach
                </ul>
            @endif
        </li>
    @endforeach           
</ul>
```

### 使用 find() 来搜索更多的记录

你不仅可以用 find() 来搜索单条记录，还可以用 IDs 的集合来搜索更多的记录，方法如下：

```php
return Product::whereIn('id', $this->productIDs)->get();
```

这么做:

```php
return Product::find($this->productIDs)
```

### 失败时执行任何操作

当查询一条记录时，如果没有找到，你可能想执行一些操作。除了用 ->firstOrFail() 会抛出 404 之外，你可以在失败时执行任何操作，只需要使用 

`->firstOr(function() { ... })`

```php
$model = Flight::where('legs', '>', 3)->firstOr(function () {
    // ...
})
```

### 检查记录是否存在否则显示 404

不要使用 find() ，然后再检查记录是否存在，使用 `findOrFail()`

```php
$product = Product::find($id);
if (!$product) {
    abort(404);
}
$product->update($productDataArray);
```

更简单的方法:

```php
$product = Product::findOrFail($id); // shows 404 if not found
$product->update($productDataArray);
```

### 条件语句为否时中止

可以使用 `abort_if()` 作为判断条件和抛出错误页面的快捷方式。

```php
$product = Product::findOrFail($id);
if($product->user_id != auth()->user()->id){
    abort(403);
}
```

更简单的方法:

```php
/* abort_if(CONDITION, ERROR_CODE) */
$product = Product::findOrFail($id);
abort_if ($product->user_id != auth()->user()->id, 403)
```

### 在删除模型之前执行任何额外的操作

由[@back2Lobby](https://github.com/back2Lobby)提供

我们可以使用 `Model::delete()` 执行额外的操作来覆盖原本的删除方法

```php
// App\Models\User.php

public function delete(){

	//extra steps here whatever you want
	
	//now perform the normal deletion
	Model::delete();
}
```

### 当你需要在保存数据到数据库时自动填充一个字段

当你需要在保存数据到数据库时自动填充一个字段 （例如: slug），使用模型观察者来代替重复编写代码

```php
use Illuminate\Support\Str;

class Article extends Model
{
    ...
    protected static function boot()
    {
        parent:boot();
        
        static::saving(function ($model) {
            $model->slug = Str::slug($model->title);
        });
    }
}
```

由[@sky_0xs](https://twitter.com/sky_0xs/status/1432390722280427521)提供

### 获取查询语句的额外信息

你可以使用 `explain()` 方法来获取查询语句的额外信息

```php
Book::where('name', 'Ruskin Bond')->explain()->dd();
```

```php
Illuminate\Support\Collection {#5344
    all: [
        {#15407
            +"id": 1,
            +"select_type": "SIMPLE",
            +"table": "books",
            +"partitions": null,
            +"type": "ALL",
            +"possible_keys": null,
            +"key": null,
            +"key_len": null,
            +"ref": null,
            +"rows": 9,
            +"filtered": 11.11111164093,
            +"Extra": "Using where",
        },
    ],
}
```

由 [@amit_merchant](https://twitter.com/amit_merchant/status/1432277631320223744)提供

### 在 Laravel 中使用doesntExist()方法

```php
// 一个例子
if ( 0 === $model->where('status', 'pending')->count() ) {
}

// 我不关心它有多少数据只要它是0
// Laravel 的 exists() 方法会很清晰
if ( ! $model->where('status', 'pending')->exists() ) {
}

// 但我发现上面这条语句中的！很容易被忽略。
// 那么 doesntExist() 方法会让这个例子更加清晰
if ( $model->where('status', 'pending')->doesntExist() ) {
}
```

由 [@ShawnHooper](https://twitter.com/ShawnHooper/status/1435686220542234626)提供

### 在一些模型的 boot () 方法中自动调用一个特性

如果你有一个特性，你想把它添加到几个模型中，自动调用它们的 `boot()` 方法，你可以把特性的方法作为boot （特性名称）来调用

```php
class Transaction extends  Model
{
    use MultiTenantModelTrait;
}
```

```php
class Task extends  Model
{
    use MultiTenantModelTrait;
}
```

```php
trait MultiTenantModelTrait
{
    // This method's name is boot[TraitName]
    // It will be auto-called as boot() of Transaction/Task
    public static function bootMultiTenantModelTrait()
    {
        static::creating(function ($model) {
            if (!$isAdmin) {
                $isAdmin->created_by_id = auth()->id();
            }
        })
    }
}
```

### Laravel 的 find () 方法，比只传一个 ID 更多的选择

```php
// 在 find($id) 方法中第二个参数可以是返回字段
Studdents::find(1, ['name', 'father_name']);
// 这样我们可以查询 ID 为 '1' 并返回 name , father_name 字段

// 我们可以用数组的方式传递更多的 ID
Studdents::find([1,2,3], ['name', 'father_name']);
// 输出: ID 为 1,2,3 并返回他们的 name , father_name 字段
```

### 在 Laravel 中有两种常见的方法来确定一个表是否为空表

在 Laravel 中，有两种常见的方法来确定一个表是否为空表。 直接在模型上使用 `exists()` 或者 `count()`
不等于一个返回严格的布尔值，另一个返回一个整数，你都可以在条件语句中使用。

```php
public function index()
{
    if (\App\Models\User::exists()) {
        // returns boolean true or false if the table has any saved rows
    }
    
    if (\App\Models\User::count()) {
        // returns the count of rows in the table
    }
}
```

由 [@aschmelyun](https://twitter.com/aschmelyun/status/1440641525998764041)提供

### 如何避免 property of non-object 错误

```php
// 设定默认模型
// 假设你有一篇 Post （帖子） 属于一个 Author （作者），代码如下:
$post->author->name;

// 当然你可以像这样阻止错误:
$post->author->name ?? ''
// 或者
@$post->author->name

// 但你可以在Eloquent关系层面上做到这一点。
// 如果没有作者关联帖子，这种关系将返回一个空的App/Author模型。
public function author() {
    return $this->belongsTo('App\Author')->withDefault();
}
// 或者
public function author() {
    return $this->belongsTo('App\Author')->withDefault([
        'name' => 'Guest Author'
    ]);
}
```

由 [@coderahuljat](https://twitter.com/coderahuljat/status/1440556610837876741)提供

### Eloquent 数据改变后获取原始数据

Eloquent 模型数据改变后，你可以使用 getOriginal () 方法来获取原始数据

```php
$user = App\User::first();
$user->name; // John
$user->name = "Peter"; // Peter
$user->getOriginal('name'); // John
$user->getOriginal(); // Original $user record
```

由 [@devThaer](https://twitter.com/devThaer/status/1442133797223403521)提供

### 一种更简单创建数据库的方法

Laravel 还可以使用 .sql 文件来更简单的创建数据库

```php
DB::unprepared(
    file_get_contents(__DIR__ . './dump.sql')
);
```

由 [@w3Nicolas](https://twitter.com/w3Nicolas/status/1447902369388249091)提供

### Query构造器的crossJoinSub方法

使用CROSS JOIN交叉连接

```php
use Illuminate\Support\Facades\DB;
$totalQuery = DB::table('orders')->selectRaw('SUM(price) as total');
DB::table('orders')
    ->select('*')
    ->crossJoinSub($totalQuery, 'overall')
    ->selectRaw('(price / overall.total) * 100 AS percent_of_total')
    ->get();
```

由 [@PascalBaljet](https://twitter.com/pascalbaljet)提供

### belongsToMany的中间表命名

为了决定 关系表的中间表, Eloquent将按字母顺序连接两个相关的型号名称。

这意味着可以这样添加“Post”和“Tag”之间的连接：

```php
class Post extends Model
{
    public $table = 'posts';
    public function tags()
    {
        return $this->belongsToMany(Tag::class);
    }
}
```

但是，您可以自由重写此约定，并且需要在第二个参数中指定联接表。

```php
class Post extends Model
{
    public $table = 'posts';
    public function tags()
    {
        return $this->belongsToMany(Tag::class, 'posts_tags');
    }
}
```

如果希望明确说明主键，还可以将其作为第三个和第四个参数提供。

```php
class Post extends Model
{
    public $table = 'posts';
    public function tags()
    {
        return $this->belongsToMany(Tag::class, 'post_tag', 'post_id', 'tag_id');
    }
}
```

由 [@iammikek](https://twitter.com/iammikek)提供

### 根据Pivot字段排序

`BelongsToMany::orderByPivot()` 允许你直接对`BelongsToMany `关系查询的结果集进行排序。

```php
class Tag extends Model
{
    public $table = 'tags';
}
class Post extends Model
{
    public $table = 'posts';
    public function tags()
    {
        return $this->belongsToMany(Tag::class, 'posts_tags', 'post_id', 'tag_id')
            ->using(PostTagPivot::class)
            ->withTimestamps()
            ->withPivot('flag');
    }
}
class PostTagPivot extends Pivot
{
    protected $table = 'posts_tags';
}
// Somewhere in the Controller
public function getPostTags($id)
{
    return Post::findOrFail($id)->tags()->orderByPivot('flag', 'desc')->get();
}
```

由 [@PascalBaljet](https://twitter.com/pascalbaljet)提供



### 从数据库中查询一条记录

`sole()`方法将会只返回一条匹配标准的记录。如果没找到，将会抛出`NoRecordsFoundException` 异常。如果发现了多条记录，抛出`MultipleRecordsFoundException` 异常

```php
DB::table('products')->where('ref', '#123')->sole();
```

由 [@PascalBaljet](https://twitter.com/pascalbaljet)提供

### 记录自动分块

与`each()`相同，但是更简单使用。`chunks`自动将记录分成多块。

```php
return User::orderBy('name')->chunkMap(fn ($user) => [
    'id' => $user->id,
    'name' => $user->name,
]), 25);
```

由[@PascalBaljet](https://twitter.com/pascalbaljet)提供

### 更新模型而不触发事件

有时候你需要更新一个模型但是不想发送任何事件 我们可以使用`updateQuietly`做到, 底层使用了`saveQuietly`方法。

```php
$flight->updateQuietly(['departed' => false]);
```

由 [@PascalBaljet](https://twitter.com/pascalbaljet)提供

### 定时清理过期记录中的模型

定期清理过时记录的模型。有了这个特性，Laravel将自动完成这项工作，只需调整内核类中`model:prune`命令的频率

```php
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Prunable;
class Flight extends Model
{
    use Prunable;
    /**
     * Get the prunable model query.
     *
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function prunable()
    {
        return static::where('created_at', '<=', now()->subMonth());
    }
}
```

此外，在修剪方法中，可以设置删除模型之前必须执行的操作：

```php
protected function pruning()
{
    // Removing additional resources,
    // associated with the model. For example, files.
    Storage::disk('s3')->delete($this->filename);
}
```

由 [@PascalBaljet](https://twitter.com/pascalbaljet)提供

### 不变的日期和对它们的强制转换

Laravel 8.53 介绍了`immutable_date` 和`immutable_datetime` 将日期转换为Immutable`.

转换成`CarbonImmutable `，而不是常规的`Carbon `实例。

```php
class User extends Model
{
    public $casts = [
        'date_field'     => 'immutable_date',
        'datetime_field' => 'immutable_datetime',
    ];
}
```

由 [@PascalBaljet](https://twitter.com/pascalbaljet)提供

### findorfail方法也接收ids数组

findorfail方法也接收ids数组。若无ids被找到 则失败。

若你想拿到一个模型的集合 并不想检测返回数量为你想得到的数量时很好用。



```php
User::create(['id' => 1]);
User::create(['id' => 2);
User::create(['id' => 3]);
// Retrives the user...
$user = User::findOrFail(1);
// Throws a 404 because the user doesn't exist...
User::findOrFail(99);
// Retrives all 3 users...
$users = User::findOrFail([1, 2, 3]);
// Throws because it is unable to find *all* of the users
User::findOrFail([1, 2, 3, 99]);
```

由 [@timacdonald87](https://twitter.com/timacdonald87/status/1457499557684604930) 提供

### 从你的数据库中自动移除模型prunableTrait

Laravel 8.50新特性:

你可以使用`prunable trait`从你的数据库中自动移除模型。举例:你可以在几天后永久移除软删除的模型。



```php
class File extends Model
{
    use SoftDeletes;
    
    // Add Prunable trait
    use Prunable;
    
    public function prunable()
    {
        // Files matching this query will be pruned
        return static::query()->where('deleted_at', '<=', now()->subDays(14));
    }
    
    protected function pruning()
    {
        // Remove the file from s3 before deleting the model
        Storage::disk('s3')->delete($this->filename);
    }
}
// Add PruneCommand to your shedule (app/Console/Kernel.php)
$schedule->command(PruneCommand::class)->daily();
```

由 [@Philo01](https://twitter.com/Philo01/status/1457626443782008834)提供

### withaggregate方法

`withAvg/withCount/withSum`和其他方法可以使用 "withAggregate"方法。可以使用此方法基于关系添加子查询。

```php
// Eloquent Model
class Post extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
// Instead of eager loading all users...
$posts = Post::with('user')->get();
// You can add a subselect to only retrieve the user's name...
$posts = Post::withAggregate('user', 'name')->get();
// This will add a 'user_name' attribute to the Post instance:
$posts->first()->user_name;
```

由 [@pascalbaljet](https://twitter.com/pascalbaljet/status/1457702666352594947)提供

### 日期转换

当标记改变时 原来用布尔值来控制模型的可见性，现在可以使用``something_at` `替换。比如 一个产品变成可见:

```php
// Migration
Schema::table('products', function (Blueprint $table) {
    $table->datetime('live_at')->nullable();
});
// In your model
public function live()
{
    return !is_null($this->live_at);
}
// Also in your model
protected $dates = [
    'live_at'
];
```

由 [@alexjgarrett](https://twitter.com/alexjgarrett/status/1459174062132019212)提供

### 多模型更新插入

`upsert()`方法将插入/更新多个记录。

- 第一个参数数组:要更新/插入的值
- 第二个:查询表达式中使用的唯一标识列
- 第三个:若记录存在 你想要更新的列

```php
Flight::upsert([
    ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
    ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150],
], ['departure', 'destination'], ['price']);
```

由 [@mmartin_joo](https://twitter.com/mmartin_joo/status/1461591319516647426)提供

### 过滤结果集之后获取查询构造器

你可以使用 `toQuery()` 在过滤结果集之后获取查询构造器。

该方法在内部使用集合的第一个模型 并使用集合模型上的“whereKey”比较器。(此处翻译拗口 存疑。但是使用方法很明确。)

```php
// Retrieve all logged_in users
$loggedInUsers = User::where('logged_in', true)->get();
// Filter them using a Collection method or php filtering
$nthUsers = $loggedInUsers->nth(3);
// You can't do this on the collection
$nthUsers->update(/* ... */);
// But you can retrieve the Builder using ->toQuery()
if ($nthUsers->isNotEmpty()) {
    $nthUsers->toQuery()->update(/* ... */);
}
```

由 [@RBilloir](https://twitter.com/RBilloir/status/1462529494917566465)提供

### 选择聚合计算相关模型

选择聚合计算相关模型。

需要指出的是在一组相关模型上使用`count`方法要慢一点。

```php
// In your controller
$user = User::withCount('articles');
// Or, to add a constraint to the aggregate
$user = User::withCount([
    'articles' => fn ($query) => $query->live();
]);
// In your view
$user->articles_count
// Instead of
$user->articles->count();
```

由 [@alexjgarrett](https://twitter.com/alexjgarrett/status/1462753602385108995)提供

### 自定义强制转换

你可以自定义强制转换来让`Laravel`自动格式化你的模型数据。

下面是一个在检索或更改用户名时将其大写的示例。

```php
class CapitalizeWordsCast implements CastsAttributes
{
    public function get($model, string $key, $value, array $attributes)
    {
        return ucwords($value);
    }
    
    public function set($model, string $key, $value, array $attributes)
    {
        return ucwords($value);
    }
}
class User extends Model
{
    protected $casts = [
        'name'  => CapitalizeWordsCast::class,
        'email' => 'string',
    ]; 
}
```

由 [@mattkingshott](https://twitter.com/mattkingshott/status/1462828232206659586)提供

### 保存中不要触发事件

若你不想触发模型事件 使用`saveQuietly()`方法

```php
public function quietly()
{
    $user = User::findOrFail(1);
    $user->name = 'Martin Joo';
    
    // Will not trigger any model event
    $user->saveQuietly();
}
```

由 [@mmartin_joo](https://twitter.com/mmartin_joo/status/1465289689154265091)提供

### 基于相关模型的平均值或总数排序

你是否曾需要基于关系模型的平均值或总数来排序？

这很简单

```php
public function bestBooks()
{
    Book::query()
        ->withAvg('ratings as average_rating', 'rating')
        ->orderByDesc('average_rating');
}
```

由 [@mmartin_joo](https://twitter.com/mmartin_joo/status/1466769691385335815)提供

### 返回事务结果

若你有一个`DB`事务 并且你想返回它的结果 至少有两种方法:

```php
// 1. You can pass the parameter by reference
$invoice = NULL;
DB::transaction(function () use (&$invoice) {
    $invoice = Invoice::create(...);
    $invoice->items()->attach(...);
})
// 2. Or shorter: just return trasaction result
$invoice = DB::transaction(function () {
    $invoice = Invoice::create(...);
    $invoice->items()->attach(...);
    
    return $invoice;
});
```

### 从query中移除多个公共scope

当使用`Global Scopes`时 你不仅可以使用多个 `scope` 而且可以在不需要的时候通过提供的``withoutGlobalScopes`方法移除他们


[Link to docs](https://laravel.com/docs/8.x/eloquent#global-scopes)

### JSON列属性排序

你可以使用JSON列属性排序

```php
// JSON column example:
// bikes.settings = {"is_retired": false}
$bikes = Bike::where('athlete_id', $this->athleteId)
        ->orderBy('name')
        ->orderByDesc('settings->is_retired')
        ->get();
```

由 [@brbcoding](https://twitter.com/brbcoding/status/1473353537983856643)提供

### 从第一个结果中获取单列的值

你可以使用`value`方法从第一个结果中获取单列的值。

```php
// Instead of
Integration::where('name', 'foo')->first()->active;
// You can use
Integration::where('name', 'foo')->value('active');
// or this to throw an exception if no records found
Integration::where('name', 'foo')->valueOrFail('active')';
```

由 [@justsanjit](https://twitter.com/justsanjit/status/1475572530215796744)提供

### 检测模型属性是否被修改

想知道您对模型所做的更改是否改变了键的值吗？没问题，只需`originalIsEquivalent`方法即可。

```php
$user = User::first(); // ['name' => "John']
$user->name = 'John';
$user->originalIsEquivalent('name'); // true
$user->name = 'David'; // Set directly
$user->fill(['name' => 'David']); // Or set via fill
$user->originalIsEquivalent('name'); // false
```

由 [@mattkingshott]提供