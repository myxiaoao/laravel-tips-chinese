## 视图 

### foreach 语句中的 $loop 变量

在 `foreach` 循环中，使用 `$loop` 变量来查看当前是否是第一次 / 最后一次循环。

```blade
@foreach ($users as $user)
     @if ($loop->first)
        This is the first iteration.
     @endif

     @if ($loop->last)
        This is the last iteration.
     @endif

     <p>This is user {{ $user->id }}</p>
@endforeach
```

同样也有诸如 `$loop->iteration` 或 `$loop->count` 等属性。可以在 [官方文档](https://laravel.com/docs/master/blade#the-loop-variable)中查看更多相关内容。

### 视图是否存在

你可以在视图实际加载之前确认该视图文件是否存在。

```php
if (view()->exists('custom.page')) {
 // Load the view
}
```

你甚至可以使用一个数组来加载视图，这样只有第一个视图文件确实存在的视图会被加载。

```php
return view()->first(['custom.dashboard', 'dashboard'], $data);
```

### 错误代码视图页面

如果你想为一些特殊的 HTTP 返回码建立特定的错误页面，比如 `500` —— 只需要使用该码值创建视图文件，比如  `resources/views/errors/500.blade.php` ，或者是 `403.blade.php` 等等，这些视图会在对应的错误码出现时自动被加载。

### 脱离控制器的视图

如果你想让一个路由仅仅显示某个视图，不需要创建控制器，只需要使用 Route::view() 方法即可。

```php
// Instead of this
Route::get('about', 'TextsController@about');
// And this
class TextsController extends Controller
{
    public function about()
    {
        return view('texts.about');
    }
}
// Do this
Route::view('about', 'texts.about');
```

### Blade @auth 指令

不需要使用 if 来检查用户是否登录，使用 @auth 指令即可。

比较典型的方式是：

```blade
@if(auth()->user())
    // The user is authenticated.
@endif
```

更短的用法：

```blade
@auth
    // The user is authenticated.
@endauth
```

与 @auth 相对的是 @guest 指令：

```blade
@guest
    // The user is not authenticated.
@endguest
```

### Blade 视图中的二级 $loop 变量

你甚至可以在 `Blade` 视图的二级 `foreach` 循环中使用 `$loop` 变量来表示外层的循环变量。

```blade
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

### 创建你自己的 Blade 指令

你只需要在 `app/Providers/AppServiceProvider.php` 中添加你自己的方法。举个例子，如果你需要将 `<br>` 标签替换为换行：

```blade
<textarea>@br2nl($post->post_text)</textarea>
```

然后将这个指令添加到 `AppServiceProvider` 的 `boot()` 方法中：

```php
public function boot()
{
    Blade::directive('br2nl', function ($string) {
        return "<?php echo preg_replace('/\<br(\s*)?\/?\>/i', \"\n\", $string); ?>";
    });
}
```

### 视图指令IncludeIf IncludeWhen IncludeFirst

如果你不确定 Blade 文件是否存在，你可以使用这些条件指令。
仅当 Blade 文件存在时载入 header：

```blade
@includeIf('partials.header')
```

仅当用户的 role_id == 1 的时候载入 header：

```blade
@includeWhen(auth()->user()->role_id == 1, 'partials.header')
```

尝试加载 adminlte.header ，如果不存在，则加载 default.header：

```blade
@includeFirst('adminlte.header', 'default.header')
```

### 使用Laravel Blade-X 变量绑定节省更多空间

```php
// Using include, the old way
@include("components.post", ["title" => $post->title])

// Using Blade-X
<x-post link="{{ $post->title }}" />

// Using Blade-X variable binding
<x-post :link="$post->title" />
```

由 [@anwar_nairi](https://twitter.com/anwar_nairi/status/1442441888787795970)提供

### Blade 组件属性

```php
// button.blade.php
@props(['rounded' => false])

<button {{ $attributes->class([
    'bg-red-100 text-red-800',
    'rounded' => $rounded
    ]) }}>
    {{ $slot }}
</button>

// view.blade.php
// Non-rounded:
<x-button>Submit</x-button>

// Rounded:
<x-button rounded>Submit</x-button>
```

由 [@godismyjudge95](https://twitter.com/godismyjudge95/status/1448825909167931396)提供

### Blade类型提示

```php
@php
    /* @var App\Models\User $user */
@endphp
<div>
    // your ide will typehint the property for you 
    {{$user->email}}
</div>
```

由 [@freekmurze](https://twitter.com/freekmurze/status/1455466663927746560)提供

### 组件语法提示

在组件参数之前传入 `:` 你可以直接传入变量而不需要使用 `{{}}`表达式

```php
<x-navbar title="{{ $title }}"/>
// you can do instead
<x-navbar :title="$title"/>
```

由 [@sky_0xs](https://twitter.com/sky_0xs/status/1457056634363072520)提供

### 自动高亮导航链接

当精确的URL匹配或传递路径或路由名称模式时，自动突出显示导航链接

带有请求和CSS类助手的 `blade` 组件使显示活动/非活动状态变得非常简单。

```php
class NavLink extends Component
{
    public function __construct($href, $active = null)
    {
        $this->href = $href;
        $this->active = $active ?? $href;        
    }
    
    public function render(): View
    {
        $classes = ['font-medium', 'py-2', 'text-primary' => $this->isActive()];
        
        return view('components.nav-link', [
            'class' => Arr::toCssClasses($classes);
        ]);
    }
    
    protected function isActive(): bool
    {
        if (is_bool($this->active)) {
            return $this->active;
        }
        
        if (request()->is($this->active)) {
            return true;
        }
        
        if (request()->fullUrlIs($this->active)) {
            return true;
        }
        
        return request()->routeIs($this->active);
    }
}
```

```blade
<a href="{{ $href }}" {{ $attributes->class($class) }}>
    {{ $slot }}
</a>
```

```blade
<x-nav-link :href="route('projects.index')">Projects</x-nav-link>
<x-nav-link :href="route('projects.index')" active="projects.*">Projects</x-nav-link>
<x-nav-link :href="route('projects.index')" active="projects/*">Projects</x-nav-link>
<x-nav-link :href="route('projects.index')" :active="$tab = 'projects'">Projects</x-nav-link>
```

[@mpskovvang](https://twitter.com/mpskovvang/status/1459646197635944455)提供

### 简化循环

你知道Blade`@each`指令可以帮助清理模板中的循环吗？

```blade
// good
@foreach($item in $items)
    <div>
        <p>Name: {{ $item->name }}
        <p>Price: {{ $item->price }}
    </div>
@endforeach
// better (HTML extracted into partial)
@each('partials.item', $items, 'item')
```

[@kirschbaum_dev](https://twitter.com/kirschbaum_dev/status/1463205294914297861)提供

### 整理blade视图的简单方法

整理刀片视图的简单方法<br>

使用`forelse·，而不是嵌套在`if`语句中的`foreach`

```blade
<!-- if/loop combination -->
@if ($orders->count())
    @foreach($orders as $order)
        <div>
            {{ $order->id }}
        </div>
    @endforeach
@else
    <p>You haven't placed any orders yet.</p>
@endif
<!-- Forelse alternative -->
@forelse($orders as $order)
    <div>
        {{ $order->id }}
    </div>
@empty
    <p>You haven't placed any orders yet.</p>
@endforelse
```

由 [@alexjgarrett]提供
