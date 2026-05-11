# PHP API 安全

## Mass Assignment

```php
// Laravel 漏洞: $fillable 不完整
class User extends Model {
    protected $fillable = ['name', 'email'];   // is_admin 不在列表中
}
User::create($request->all());  // 安全：仅允许 fillable 字段

// $guarded 为空或使用 forceCreate
class User extends Model {
    protected $guarded = [];  // 全部字段允许批量赋值
}
User::create($request->all());  // 可设置 is_admin=true

// update 不校验
$user->update($request->all());  // 未限制字段

// 安全
$request->only(['name', 'email']);
$request->validated();
```

## JWT

```php
// 漏洞
JWT::decode($token, 'weak-secret', ['HS256', 'none']);  // 允许 none
JWT::decode($token, '123456');                            // 弱密钥

// 安全
JWT::decode($token, env('JWT_SECRET'), ['HS256']);       // 固定算法 + 强密钥
JWT::checkExpiration();                                   // 验证过期
```

## CORS

```php
// 漏洞
header('Access-Control-Allow-Origin: *');
header('Access-Control-Allow-Credentials: true');  // 通配符+凭据=不安全

// 安全
$allowed = ['https://example.com'];
$origin = $_SERVER['HTTP_ORIGIN'] ?? '';
header('Access-Control-Allow-Origin: ' . (in_array($origin, $allowed) ? $origin : ''));
```

## 速率限制

```php
// 登录无限制
Route::post('/login', [AuthController::class, 'login']);

// Laravel 内置限流
Route::post('/login', [AuthController::class, 'login'])
    ->middleware('throttle:5,1');  // 1分钟最多5次
```

## 检测命令

```bash
grep -rn "->all()\|request()->all" --include="*.php"
grep -rn "\$fillable\|\$guarded" --include="*.php"
grep -rn "Access-Control-Allow-Origin" --include="*.php"
grep -rn "JWT::decode\|JWTAuth" --include="*.php"
```
