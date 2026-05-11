# PHP 认证授权

## 认证绕过

```php
// 弱密码哈希
$pass = md5($_POST['password']);  // md5 弱加密
if ($pass === $row['password'])   // md5("240610708") === md5("QNKCDZO") → 绕过

// JWT 算法混淆
$header = json_decode(base64_decode($parts[0]));
if ($header->alg === 'none') { /* 无签名验证 */ }

// JWT 弱密钥
$secret = '123456';  // 可爆破
JWT::decode($token, $secret, ['HS256']);

// Cookie 伪造
$user = unserialize(base64_decode($_COOKIE['user']));
if ($user->isAdmin()) { ... }  // 可伪造序列化数据
```

## 水平越权

```php
// 未验证资源归属
public function view($id) {
    $order = Order::find($id);       // 可查看任何人的订单
    return view('order', ['order' => $order]);
}
// /order/123 → /order/124

// 批量操作
public function batchDelete() {
    Order::destroy($_POST['ids']);   // 未校验每个订单归属
}
```

## 修复

```php
// 密码
$hash = password_hash($password, PASSWORD_BCRYPT);
password_verify($password, $hash);

// 资源归属校验
public function view($id) {
    $order = Order::where('id', $id)
        ->where('user_id', auth()->id())
        ->firstOrFail();
    return view('order', ['order' => $order]);
}

// JWT
JWT::decode($token, $secret, ['HS256']);      // 固定算法
JWT::requireIssuer('my-app');                  // 验证签发者
JWT::verifyExpiration();                       // 验证过期

// 中间件
Route::get('/admin', [AdminController::class, 'index'])
    ->middleware(['auth', 'role:admin']);
```

## 常见未保护路径

```
/admin  /admin.php  /phpinfo.php  /phpmyadmin
/.env  /.git/config  /composer.json
/backup.zip  /dump.sql  /wp-config.php
```

## 检测命令

```bash
grep -rn "md5(\|sha1(\|md5(" --include="*.php"
grep -rn "->middleware\|middleware(" --include="*.php"
grep -rn "auth()->id\|auth()->user" --include="*.php"
```
