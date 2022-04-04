## 模型关联 (32 提示)

  - [在 Eloquent 关系中使用 OrderBy](#在-eloquent-关系中使用-orderby)
  - [在 Eloquent 关系中添加条件](#在-eloquent-关系中添加条件)
  - [DB 原生查询 havingRaw ()](#db-原生查询-havingraw-)
  - [Eloquent 使用 has () 实现多层调用查询](#eloquent-使用-has--实现多层调用查询)
  - [一对多关系中获取符合指定数量的信息](#一对多关系中获取符合指定数量的信息)
  - [默认模型](#默认模型)
  - [一对多关系中一次创建多条关联数据](#一对多关系中一次创建多条关联数据)
  - [多层级预加载](#多层级预加载)
  - [预加载特定字段](#预加载特定字段)
  - [轻松更新父级 updated_at](#轻松更新父级-updated_at)
  - [一直检查关联是否存在](#一直检查关联是否存在)
  - [使用 withCount () 来统计关联记录数](#使用-withcount--来统计关联记录数)
  - [关联关系中过滤查询](#关联关系中过滤查询)
  - [动态预加载相关模型](#动态预加载相关模型)
  - [使用 hasMany 代替 belongsTo](#使用-hasmany-代替-belongsto)
  - [重命名 pivot 表名称](#重命名-pivot-表名称)
  - [仅用一行代码更新归属关系](#仅用一行代码更新归属关系)
  - [Laravel 7+ 的外键](#laravel-7-的外键)
  - [两种 「whereHas」 组合使用](#两种-wherehas-组合使用)
  - [检查关系方法是否已经存在](#检查关系方法是否已经存在)
  - [获取中间表中的关联关系数据](#获取中间表中的关联关系数据)
  - [便捷获取一对多关系中子集的数量](#便捷获取一对多关系中子集的数量)
  - [对关联模型数据进行随机排序](#对关联模型数据进行随机排序)
  - [过滤一对多关联](#过滤一对多关联)
  - [通过中间表字段过滤多对多关联](#通过中间表字段过滤多对多关联)
  - [whereHas 的一个更简短的方法](#wherehas-的一个更简短的方法)
  - [你可以为你的模型关联添加条件](#你可以为你的模型关联添加条件)
  - [新的 Eloquent 查询构建器方法 whereBelongsTo()](#新的-eloquent-查询构建器方法-wherebelongsto)
  - [使用is()方法比较一对一关系模型](#使用is方法比较一对一关系模型)
  - [whereHas多连接](#wherehas多连接)
  - [更新存在的中间记录](#更新存在的中间记录)
  - [获取最新或最老数据的关系](#获取最新或最老数据的关系)


### 在 Eloquent 关系中使用 OrderBy

您可以在关联关系中直接指定 `orderBy ()`。

```php
public function products()
{
    return $this->hasMany(Product::class);
}

public function productsByName()
{
    return $this->hasMany(Product::class)->orderBy('name');
}
```

### 在 Eloquent 关系中添加条件

假如你经常在模型关联关系中添加某些相同的 `where `条件，可以单独创建一个方法。

Model:

```php
public function comments()
{
    return $this->hasMany(Comment::class);
}

public function approved_comments()
{
    return $this->hasMany(Comment::class)->where('approved', 1);
}
```

### DB 原生查询 havingRaw ()

你可以在很多地方使用原始数据库查询，比如在 `groupBy()` 后面调用 `havingRaw()`

```php
Product::groupBy('category_id')->havingRaw('COUNT(*) > 1')->get();
```

### Eloquent 使用 has () 实现多层调用查询

你可以在关联关系查询中使用 `has()` 实现两层关联查询。

```php
// Author -> hasMany(Book::class);
// Book -> hasMany(Rating::class);
$authors = Author::has('books.ratings')->get();
```

### 一对多关系中获取符合指定数量的信息

在`hasMany()`中，你可以通过条件过滤，获取符合的数据。

```php
// Author -> hasMany(Book::class)
$authors = Author::has('books', '>', 5)->get();
```

### 默认模型

你可以在 `belongsTo `关系中设置返回一个默认的模型，从而避免类似于使用 `{{ $post->user->name }}` 当 $post->user 不存在的时候，引起的致命的错误

```php
public function user()
{
    return $this->belongsTo('App\User')->withDefault();
}
```

### 一对多关系中一次创建多条关联数据

在一对多关系中，你可以使用 `saveMany` 通过一次提交，保存多条关联数据。

```php
$post = Post::find(1);
$post->comments()->saveMany([
    new Comment(['message' => 'First comment']),
    new Comment(['message' => 'Second comment']),
]);
```

### 多层级预加载

在 `Laravel `中，你可以在一条语句中预加载多个层级，在这个例子中，我们不仅加载作者关系，而且还加载作者模型上的国家关系。

```php
$users = App\Book::with('author.country')->get();
```

### 预加载特定字段

你可以在 `Laravel `中预加载并指定关联中的特定字段。

```php
$users = App\Book::with('author:id,name')->get();
```

你同样可以在深层级中这样做，如第二层级关系：

```php
$users = App\Book::with('author.country:id,name')->get();
```

### 轻松更新父级 updated_at

如果你想更新一条数据同时更新它父级关联的 `updated_at` 字段 （例如：你添加一条帖子评论，想同时更新帖子的 posts.updated_at），只需要在子模型中使用 `$touches = ['post'];` 属性。

```php
class Comment extends Model
{
    protected $touches = ['post'];
}
```

### 一直检查关联是否存在

永远不要在不检查关联是否存在时使用 `$model->relationship->field`
它可能因为任何原因，如在你的代码之外，被别人的队列任务等等被删除。

用 `if-else`，或者在 Blade 模板中 `{{$model->relationship->field ? ? '' }}`，或者 `{{optional($model->relationship)->field }}` 。在 php8 中，你甚至可以使用 null 安全操作符 {{ $model->relationship?->field) }}

### 使用 withCount () 来统计关联记录数

如果你有 `hasMany()` 的关联，并且你想统计子关联记录的条数，不要写一个特殊的查询。例如，如果你的用户模型上有帖子和评论，使用 `withCount()`。

```php
public function index()
{
    $users = User::withCount(['posts', 'comments'])->get();
    return view('users', compact('users'));
}
```

同时，在 Blade 文件中，您可以通过使用 `{relationship}_count` 属性获得这些数量：

```blade
@foreach ($users as $user)
<tr>
    <td>{{ $user->name }}</td>
    <td class="text-center">{{ $user->posts_count }}</td>
    <td class="text-center">{{ $user->comments_count }}</td>
</tr>
@endforeach
```

也可以按照这些统计字段进行排序：

```php
User::withCount('comments')->orderBy('comments_count', 'desc')->get(); 
```

### 关联关系中过滤查询

假如您想加载关联关系的数据，同时需要指定一些限制或者排序的闭包函数。例如，您想获取人口最多的前 3 座城市信息，可以按照如下方式实现:

```php
$countries = Country::with(['cities' => function($query) {
    $query->orderBy('population', 'desc');
}])->get();
```

### 动态预加载相关模型

您不仅可以实现对关联模型的实时预加载，还可以根据情况动态设置某些关联关系，需要在模型初始化方法中处理：

```php
class ProductTag extends Model
{
    protected $with = ['product'];

    public function __construct() {
        parent::__construct();
        $this->with = ['product'];
        
        if (auth()->check()) {
            $this->with[] = 'user';
        }
    }
}
```

### 使用 hasMany 代替 belongsTo

在关联关系中，如果创建子关系的记录中需要用到父关系的 ID ，那么使用  `hasMany ` 比使用 `belongsTo `更简洁。

```php
// if Post -> belongsTo(User), and User -> hasMany(Post)...
// Then instead of passing user_id...
Post::create([
    'user_id' => auth()->id(),
    'title' => request()->input('title'),
    'post_text' => request()->input('post_text'),
]);

// Do this
auth()->user()->posts()->create([
    'title' => request()->input('title'),
    'post_text' => request()->input('post_text'),
]);
```

### 重命名 pivot 表名称

如果你想要重命名`pivot`并用其他的什么方式来调用关系，你可以在你的关系声明中使用 `->as('name')` 来为关系取名。
模型 ：

```php
public function podcasts() {
    return $this->belongsToMany('App\Podcast')
        ->as('subscription')
        ->withTimestamps();
}
```

控制器:

```php
$podcasts = $user->podcasts();
foreach ($podcasts as $podcast) {
    // instead of $podcast->pivot->created_at ...
    echo $podcast->subscription->created_at;
}
```

### 仅用一行代码更新归属关系

如果有一个 `belongsTo()` 关系，你可以在仅仅一条语句中更新这个 `Elquent` 关系：

```php
// if Project -> belongsTo(User::class)
$project->user->update(['email' => 'some@gmail.com']); 
```

### Laravel 7+ 的外键

从 Laravel 7 开始，你不需要在迁移（migration）中为一些关系字段写两行代码 —— 一行是字段，一行是外键。你可以使用 `foreignId()` 方法。

```php
// Before Laravel 7
Schema::table('posts', function (Blueprint $table)) {
    $table->unsignedBigInteger('user_id');
    $table->foreign('user_id')->references('id')->on('users');
}

// From Laravel 7
Schema::table('posts', function (Blueprint $table)) {
    $table->foreignId('user_id')->constrained();
}

// Or, if your field is different from the table reference
Schema::table('posts', function (Blueprint $table)) {
    $table->foreignId('created_by_id')->constrained('users', 'column');
}
```

### 两种 「whereHas」 组合使用

在 Eloquent 中，你可以在同一条语句中使用 `whereHas()` 和 `orDoesntHave()`。

```php
User::whereHas('roles', function($query) {
    $query->where('id', 1);
})
->orDoesntHave('roles')
->get();
```

### 检查关系方法是否已经存在

如果你的 `Eloquent` 关系名是动态的，那么你需要检查项目中是否存在相同名称的关系。你可以使用这个 PHP 方法  `method_exists($object, $methodName)`。

```php
$user = User::first();
if (method_exists($user, 'roles')) {
	// Do something with $user->roles()->...
}
```

### 获取中间表中的关联关系数据

在多对多关系中，您定义的中间表里面可能会包含扩展字段，甚至可能包含其它的关联关系。
下面生成一个中间表模型

```bash
php artisan make:model RoleUser --pivot
```

然后，给  `belongsToMany()` 指定 `->using()` 方法。下面就是见证奇迹的时刻：

```php
// in app/Models/User.php
public function roles()
{
	return $this->belongsToMany(Role::class)
	    ->using(RoleUser::class)
	    ->withPivot(['team_id']);
}

// app/Models/RoleUser.php: notice extends Pivot, not Model
use Illuminate\Database\Eloquent\Relations\Pivot;

class RoleUser extends Pivot
{
	public function team()
	{
	    return $this->belongsTo(Team::class);
	}
}

// Then, in Controller, you can do:
$firstTeam = auth()->user()->roles()->first()->pivot->team->name;
```

### 便捷获取一对多关系中子集的数量

除了可以使用 `Eloquent` 中的 `withCount()` 方法统计子集数量外，还可以直接用 `loadCount()` 更加便捷和快速获取：

```php
// if your Book hasMany Reviews...
$book = App\Book::first();

$book->loadCount('reviews');
// Then you get access to $book->reviews_count;

// Or even with extra condition
$book->loadCount(['reviews' => function ($query) {
    $query->where('rating', 5);
}]);
```

### 对关联模型数据进行随机排序

您可以使用 `inRandomOrder()` 对 Eloquent 的查询结果进行随机排序，同时也可以作用于关联关系中，实现关联数据的随机排序。

```php
// If you have a quiz and want to randomize questions...

// 1. If you want to get questions in random order:
$questions = Question::inRandomOrder()->get();

// 2. If you want to also get question options in random order:
$questions = Question::with(['answers' => function($q) {
    $q->inRandomOrder();
}])->inRandomOrder()->get();
```

### 过滤一对多关联

通过我项目中的一个代码例子，展示了过滤一对多关系的可能性。`TagType -> hasMany tags -> hasMany examples`

如果你想查询所有的标签类型，伴随他们的标签，但只包含有实例的标签，并按照实例数量倒序

```php
$tag_types = TagType::with(['tags' => function ($query) {
    $query->has('examples')
        ->withCount('examples')
        ->orderBy('examples_count', 'desc');
    }])->get();
```

### 通过中间表字段过滤多对多关联

如果你有一个多对多关联，你可以在中间表中添加一个额外字段，这样你可以在查询列表时用它排序。

```php
class Tournament extends Model
{
    public function countries()
    {
        return $this->belongsToMany(Country::class)->withPivot(['position']);
    }
}
```

```php
class TournamentsController extends Controller

public function whatever_method() {
    $tournaments = Tournament::with(['countries' => function($query) {
            $query->orderBy('position');
        }])->latest()->get();
}
```

### whereHas 的一个更简短的方法

在 `Laravel 8.57` 中发布：通过包含一个简单条件的简短方法来写 `whereHas()`。

```php
// Before
User::whereHas('posts', function ($query) {
    $query->where('published_at', '>', now());
})->get();

// After
User::whereRelation('posts', 'published_at', '>', now())->get();
```

### 你可以为你的模型关联添加条件

```php
class User
{
    public function posts()
    {
        return $this->hasMany(Post::class);
    }
    
    // with a getter
    public function getPublishedPostsAttribute()
    {
        return $this->posts->filter(fn ($post) => $post->published);
    }
    
    // with a relationship
    public function publishedPosts()
    {
        return $this->hasMany(Post::class)->where('published', true);
    }
}
```

由 [@anwar_nairi](https://twitter.com/anwar_nairi/status/1441718371335114756)提供

### 新的 Eloquent 查询构建器方法 whereBelongsTo()

Laravel `8.63.0` 版本带有一个新的 `Eloquent` 查询构建器方法 `whereBelongsTo()` 。这允许你从你的查询中删除 `BelongsTo` 外键名称，并使用关联方法替代（该方法会根据类名自动确定关联与外键，也可以添加第二个参数手动关联）

```php
// From:
$query->where('author_id', $author->id)

// To:
$query->whereBelongsTo($author)

// Easily add more advanced filtering:
Post::query()
    ->whereBelongsTo($author)
    ->whereBelongsTo($cateogry)
    ->whereBelongsTo($section)
    ->get();

// Specify a custom relationship:
$query->whereBelongsTo($author, 'author')
```

由 [@danjharrin](https://twitter.com/danjharrin/status/1445406334405459974)提供

### 使用is()方法比较一对一关系模型

我们可以在相关联的模型中做比较 而不需要其他数据库访问。

```php
// BEFORE: the foreign key is taken from the Post model
$post->author_id === $user->id;
// BEFORE: An additional request is made to get the User model from the Author relationship
$post->author->is($user);
// AFTER
$post->author()->is($user);
```

由 [@PascalBaljet](https://twitter.com/pascalbaljet)提供

### whereHas多连接

```php
// User Model
class User extends Model
{
    protected $connection = 'conn_1';
    public function posts()
    {
        return $this->hasMany(Post::class);
    }
}
// Post Model
class Post extends Model
{
    protected $connection = 'conn_2';
    public function user()
    {
        return $this->belongsTo(User::class, 'user_id');
    }
}
// wherehas()
$posts = Post::whereHas('user', function ($query) use ($request) {
      $query->from('db_name_conn_1.users')->where(...);
  })->get();
```

由 [@adityaricki](https://twitter.com/adityaricki2)提供

### 更新存在的中间记录

如果要更新表上存在中间记录，请使用`updateExistingPivot`而不是`syncWithPivotValues`。

```php
// Migrations
Schema::create('role_user', function ($table) {
    $table->unsignedId('user_id');
    $table->unsignedId('role_id');
    $table->timestamp('assigned_at');
})
// first param for the record id
// second param for the pivot records
$user->roles()->updateExistingPivot(
    $id, ['assigned_at' => now()],    
);
```

 [@sky_0xs](https://twitter.com/sky_0xs/status/1461414850341621760)提供

### 获取最新或最老数据的关系

在一个模型中，我们可以定义一个关系，该关系将获得另一个关系的最新（或最旧）项。

```php
public function historyItems(): HasMany
{
    return $this
        ->hasMany(ApplicationHealthCheckHistoryItem::class)
        ->orderByDesc('created_at');
}
public function latestHistoryItem(): HasOne
{
    return $this
        ->hasOne(ApplicationHealthCheckHistoryItem::class)
        ->latestOfMany();
}
```



