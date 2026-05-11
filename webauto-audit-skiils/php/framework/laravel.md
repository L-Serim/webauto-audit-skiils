# Laravel 安全审计

## 识别特征

```
文件: artisan, routes/web.php, app/Http/
目录: app/, config/, resources/, bootstrap/cache/
版本: composer.json → "laravel/framework": "^10.0"
```

## 重点关注

### Mass Assignment

```php
// $fillable 空或 $guarded 空
class User extends Model {
    protected $guarded = [];  // 允许全部字段批量赋值
}
// POST /users { "is_admin": true } → 权限提升

// 安全
protected $fillable = ['name', 'email'];
User::create($request->validated());
```

### Debug 模式

```
.env: APP_DEBUG=true → 错误页面暴露源码路径/配置
/.env 文件直接可访问 → 数据库密码泄露
```

### Eloquent SQL注入

```php
// 漏洞
User::whereRaw("status = " . $_GET['status']);
DB::raw("SELECT * FROM users WHERE id = " . $_GET['id']);
DB::select("SELECT * FROM users WHERE email = '" . $email . "'");

// 安全
User::where('status', $_GET['status']);
DB::select("SELECT * FROM users WHERE email = ?", [$email]);
```

### Blade 模板

```php
{!! $content !!}     // 未转义 → XSS
{{ $content }}       // 转义 → 安全
@json($data)         // 安全JS输出
```

### 反序列化

```php
// 高危入口
unserialize($request->cookie('laravel_session'));
// 框架自身反序列化Gadget链 (PHPGGC Laravel/RCE*)
```

### 路由安全

```bash
php artisan route:list  # 查看所有路由
# 检查: 敏感路由是否有 middleware 保护
# /admin/* 是否全部有 auth + role:admin
```

### 文件上传

```php
$request->file('file')->store('uploads');  // 使用原始扩展名
$request->file('file')->storeAs('uploads', $name);  // 自定义名称
```

## 检测清单

- [ ] APP_DEBUG=false (生产环境)
- [ ] $fillable 限制而非 $guarded
- [ ] 无 whereRaw/DB::raw 用户输入拼接
- [ ] Blade 未使用 {!! !!} 输出用户内容
- [ ] 敏感路由有 auth + role 中间件
- [ ] 文件上传有扩展名白名单
- [ ] .env 文件不可直接访问
- [ ] APP_KEY 已生成且非默认值
