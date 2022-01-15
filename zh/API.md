## API

⬆️ [回到顶部](../REAME-zh.md) ⬅️ [上一个 (日志与调试)](./Log_and_Debug.md) ➡️ [下一个 (其他)](./Other.md)

1. [API 返回一切正常](#API-返回一切正常)
2. [去掉额外的内部数据包装](#去掉额外的内部数据包装)
3. [API resource中避免N+1查询](#避免N+1查询)


由 [@phillipmwaniki](https://twitter.com/phillipmwaniki/status/1445230637544321029)提供

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



### 去掉额外的内部数据包装

当创建一个 `Laravel Resource` 集合 你可以去除数据外层包装, 通过在 `AppServiceProvider`中添加

`JsonResource::withoutWrapping()`

```php
public function boot()
{
    JsonResource::withoutWrapping();
}
```

### 避免N+1查询

在`API resource`资源中你可以使用`whenLoaded`方法避免`N+1`查询。

如果`Employee `模型准备好了加载的时候 才会被加载。
如果没有`whenLoaded` `department`每次都会执行查询。
Without `whenLoaded()` there is always a query for the department

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

