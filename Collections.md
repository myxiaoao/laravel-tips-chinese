## 集合

⬆️ [回到顶部](../README-zh.md) ⬅️ [上一个 (验证)](./Validation.md) ➡️ [下一个 (授权)](./Auth.md)

1. [不要使用NULL过滤集合](#不要使用NULL过滤集合)
2. [使用自定义的回调函数对集合分组](#使用自定义的回调函数对集合分组)
3. [针对行的集合方法](#针对行的集合方法)
4. [对分页集合求和](#对分页集合求和)
5. [分页组件中的唯一标识](#分页组件中的唯一标识)
6. [高阶集合方法](#高阶集合方法)
7. [Higher order collection message](#higher-order-collection-message)

### 不要使用NULL过滤集合

你可以在 Eloquent 中使用 `NULL` 过滤，但是你不能用 `NULL` 过滤 集合 - 你应该换成空字符串过滤，字段中已经没有 "null"。(意思是全字符串的形式的过滤不能使用 NULL，因为会被格式化为 ["field is null", "=", true])

```php
// This works
$messages = Message::where('read_at is null')->get();

// Won’t work - will return 0 messages
$messages = Message::all();
$unread_messages = $messages->where('read_at is null')->count();

// Will work
$unread_messages = $messages->where('read_at', '')->count();
```

### 使用自定义的回调函数对集合分组

如果你想对结果分组，且分组字段不对应数据库中的字段，你可以提供一个回调函数来返回自定义的分组字段。
例如，通过用户的注册日分组，代码如下：

```php
$users = User::all()->groupBy(function($item) {
    return $item->created_at->format('Y-m-d');
});
```

注意：这个方法是在 `Collection `类上的，所以将会在数据库的返回结果上执行。(意思不会在数据库 sql 层面分组)

### 针对行的集合方法

你可以用 `->all(`) ,` ->get()` 方法查询数据，然后在这个返回的集合上执行各种集合方法，执行集合操作不会每次都查询数据库。

```php
$users = User::all();
echo 'Max ID: ' . $users->max('id');
echo 'Average age: ' . $users->avg('age');
echo 'Total budget: ' . $users->sum('budget');
```

### 对分页集合求和

如何对分页返回的结果集求和？使用相同的查询构建器，在分页查询之前执行求和操作


```php
// How to get sum of post_views with pagination?
$posts = Post::paginate(10);
// This will be only for page 1, not ALL posts
$sum = $posts->sum('post_views');

// Do this with Query Builder
$query = Post::query();
// Calculate sum
$sum = $query->sum('post_views');
// And then do the pagination from the same query
$posts = $query->paginate(10);
```

### 分页组件中的唯一标识

我们可以在分页组件中像序列号那样使用每趟循环中的索引 `index `，作为分页组件的唯一标识。

```php
   ...
   <th>Serial</th>
    ...
    @foreach ($products as $product)
    <tr>
        <td>{{ $loop->index + $product->firstItem() }}</td>
        ...
    @endforeach
```

这可以解决下一页（?page=2&...）索引的计数问题。

### 高阶集合方法

集合具有更高阶的可以链式调用的方法，例如 `groupBy()` `map()` 等，给你流畅的语法体验。下面的例子计算了一个需求单中每组产品的价格。

```php
$offer = [
        'name'  => 'offer1',
        'lines' => [
            ['group' => 1, 'price' => 10],
            ['group' => 1, 'price' => 20],
            ['group' => 2, 'price' => 30],
            ['group' => 2, 'price' => 40],
            ['group' => 3, 'price' => 50],
            ['group' => 3, 'price' => 60]
        ]
];
                
$totalPerGroup = collect($offer->lines)->groupBy('group')->map(fn($group) => $group->sum('price')); 
```

### 高阶集合排序

集合还支持“高阶排序”，这是对集合执行常见操作的捷径。

此示例计算报价中每组产品的价格。

```php
$offer = [
        'name'  => 'offer1',
        'lines' => [
            ['group' => 1, 'price' => 10],
            ['group' => 1, 'price' => 20],
            ['group' => 2, 'price' => 30],
            ['group' => 2, 'price' => 40],
            ['group' => 3, 'price' => 50],
            ['group' => 3, 'price' => 60]
        ]
];
                
$totalPerGroup = collect($offer['lines'])->groupBy->group->map->sum('price');
```