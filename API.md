## API 

### 去掉额外的内部数据包装

当创建一个 `Laravel Resource` 集合 你可以去除数据外层包装, 通过在 `app/Providers/AppServiceProvider.php` 中添加

`JsonResource::withoutWrapping()`

```php
public function boot()
{
    JsonResource::withoutWrapping();
}
```

由 [@phillipmwaniki](https://twitter.com/phillipmwaniki/status/1445230637544321029) 提供

### API 返回一切正常

如果你有 API 端口执行某些操作但是没有响应，那么您只想返回 “一切正常”, 您可以返回 204 状态代码 “No content”。在 Laravel 中，很简单: `return response()->noContent();`

```php
public function reorder(Request $request)
{
    foreach ($request->input('rows', []) as $row) {
        Country::find($row['id'])->update(['position' => $row['position']]);
    }

    return response()->noContent();
}
```

### 避免 N+1 查询

在 `API resource` 资源中你可以使用 `whenLoaded` 方法避免 `N+1` 查询。

如果 `Employee ` 模型准备好了加载的时候 才会被加载。

如果没有 `whenLoaded` `department` 每次都会执行查询。

```php
class EmplyeeResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->uuid,
            'fullName' => $this->full_name,
            'email' => $this->email,
            'jobTitle' => $this->job_title,
            'department' => DepartmentResource::make($this->whenLoaded('department')),
        ];
    }
}
```

Tip given by [@mmartin_joo](https://twitter.com/mmartin_joo/status/1473987501501071362)

### 从 Authorizationheader 中获取 BearerToken

当你使用 api 并想访问 bearerToken 时 `bearerToken` 方法很方便.

```php
// Don't parse API headers manually like this:
$tokenWithBearer = $request->header('Authorization');
$token = substr($tokenWithBearer, 7);
//Do this instead:
$token = $request->bearerToken();
```

由 [@iamharis010](https://twitter.com/iamharis010/status/1488413755826327553) 提供

### 对你的 API 结果进行排序

单一字段 api 升降序排列

```php
// Handles /dogs?sort=name and /dogs?sort=-name
Route::get('dogs', function (Request $request) {
    // Get the sort query parameter (or fall back to default sort "name")
    $sortColumn = $request->input('sort', 'name');

    // Set the sort direction based on whether the key starts with -
    // using Laravel's Str::startsWith() helper function
    $sortDirection = Str::startsWith($sortColumn, '-') ? 'desc' : 'asc';
    $sortColumn = ltrim($sortColumn, '-');

    return Dog::orderBy($sortColumn, $sortDirection)
        ->paginate(20);
});
```

多字段 api 升降序排列 (例如. ?sort=name,-weight)

```php
// Handles ?sort=name,-weight
Route::get('dogs', function (Request $request) {
    // Grab the query parameter and turn it into an array exploded by ,
    $sorts = explode(',', $request->input('sort', ''));

    // Create a query
    $query = Dog::query();

    // Add the sorts one by one
    foreach ($sorts as $sortColumn) {
        $sortDirection = Str::startsWith($sortColumn, '-') ? 'desc' : 'asc';
        $sortColumn = ltrim($sortColumn, '-');

        $query->orderBy($sortColumn, $sortDirection);
    }

    // Return
    return $query->paginate(20);
});
```
