# PHP CSRF

## 确认条件

状态变更操作无CSRF Token验证且使用Cookie认证。

## 检测

```php
// 无 Token 验证
Route::post('/user/delete', function () {
    User::destroy(request('id'));
});

// 表单无 _token
<form method="POST" action="/admin/delete">
    <input name="id" value="1">
</form>

// GET 执行写操作
Route::get('/user/delete/{id}', function ($id) {
    User::destroy($id);
});
// <img src="https://target.com/user/delete/1">

// API 使用 Cookie 认证但无 CSRF 防护
Route::post('/api/transfer', function () {
    // 浏览器自动携带 Cookie → 跨站可伪造请求
});
```

## 修复

```php
// Laravel 自动 CSRF 保护
<form method="POST" action="/user/delete">
    @csrf
    <input name="id" value="1">
</form>

// API 使用 Bearer Token（浏览器不自动携带，天然免疫）
Route::post('/api/transfer', function () {
    // 前端: Authorization: Bearer <token>
})->middleware('auth:api');

// 关键操作二次验证
Route::post('/user/delete', function () {
    if (!password_verify(request('password'), auth()->user()->password)) {
        return error('需要密码确认');
    }
});
```

## 检测命令

```bash
grep -rn "Route::post\|Route::put\|Route::delete" --include="*.php" | grep -v "middleware\|@csrf"
grep -rn "Route::get.*delete\|Route::get.*create\|Route::get.*update" --include="*.php"
```
