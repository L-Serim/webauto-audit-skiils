# ThinkPHP 安全审计

## 识别特征

```
文件: think, thinkphp/ 目录
版本: thinkphp/library/think/App.php
```

## 重点关注

### SQL注入

```php
// whereRaw/whereExp + 用户输入
Db::table('users')->whereRaw("status = " . input('status'));
Db::table('users')->whereExp('name', input('name'));

// order/field 参数动态
Db::table('users')->order(input('order'));
// 白名单
$allow = ['id', 'created_at'];
$order = in_array(input('order'), $allow) ? input('order') : 'id';
```

### 模板 (View)

```php
// 未转义
{$content|raw}       // XSS
{:html_entity_decode($content)}

// 默认转义(安全)
{$content}           // 自动 htmlspecialchars
```

### 反序列化

ThinkPHP 历史版本存在反序列化RCE链:
- TP 5.0.x / 5.1.x: `__destruct` → `Request::__call` → `call_user_func_array`
- TP 6.x: 需要检查自定义 `__destruct`/`__toString` 实现

```bash
grep -rn "unserialize(" thinkphp/ app/
```

### 路由

```
// 检查路由是否闭合
Route::get('index', 'Index@index');
// 检查是否有未受保护的强制路由 (/admin, /install)
```

### 文件上传

```php
// 检查验证规则
$file = request()->file('image');
$info = $file->validate(['ext'=>'jpg,png,gif'])->move('uploads/');
// ext 验证是否为白名单
// move() 是否生成了随机文件名
```

### 缓存/日志文件

```
runtime/log/ 目录 → 可能含SQL日志、敏感参数
runtime/cache/ 目录 → 模板编译缓存
```

## 检测清单

- [ ] 无 whereRaw/whereExp 拼接用户输入
- [ ] order/field 使用白名单
- [ ] 模板未使用 |raw 输出用户内容
- [ ] APP_DEBUG=false
- [ ] 文件上传有 ext 白名单
- [ ] runtime/ 目录不可web访问
