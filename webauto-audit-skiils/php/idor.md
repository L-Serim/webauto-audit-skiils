# PHP IDOR 越权

## 确认条件

通过修改资源ID可访问/操作非授权资源。

## 检测

```php
// API 无归属校验
Route::get('/api/order/{id}', function ($id) {
    return Order::find($id);  // 可查看任何人的订单
});

Route::post('/api/user/{id}/password', function ($id) {
    User::find($id)->update(['password' => request('new_password')]);
    // 可修改其他用户密码
});

// 文件下载
Route::get('/download/{filename}', function ($filename) {
    return response()->download(storage_path('invoices/' . $filename));
    // 可下载任何人的发票
});

// UUID 不等于安全
Route::get('/api/invoice/{uuid}', function ($uuid) {
    return Invoice::where('uuid', $uuid)->first();  // 无所有权检查
});

// 批量操作
Route::post('/api/orders/delete', function () {
    Order::destroy(request('ids'));  // 无归属校验
});
```

## 修复

```php
// 资源归属校验
Route::get('/api/order/{id}', function ($id) {
    return Order::where('id', $id)
        ->where('user_id', auth()->id())
        ->firstOrFail();
});

// 文件下载归属校验
Route::get('/download/{filename}', function ($filename) {
    $invoice = Invoice::where('filename', $filename)
        ->where('user_id', auth()->id())
        ->firstOrFail();
    return response()->download(storage_path('invoices/' . $filename));
});

// 批量操作
Route::post('/api/orders/delete', function () {
    $ids = request('ids');
    Order::whereIn('id', $ids)
        ->where('user_id', auth()->id())
        ->delete();
});
```

## 检测命令

```bash
grep -rn "::find(\|findOrFail(" --include="*.php" | grep -v "auth"
grep -rn "->where('id'\|->where('uuid'" --include="*.php" | grep -v "user_id"
```
