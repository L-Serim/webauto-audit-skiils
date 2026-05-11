# PHP 变量覆盖

## 确认条件

用户可通过函数或语法将外部变量注入当前作用域，覆盖已有的安全变量。

## 检测

```php
// extract() — GET/POST 参数覆盖
extract($_GET);
// ?auth=1 → $auth 被覆盖为 1

// parse_str() — 解析查询字符串
parse_str($_SERVER['QUERY_STRING']);
// ?auth=true → $auth = true

// parse_str 无第二个参数
parse_str($_GET['data']);
// data=auth%3D1 → $auth = 1

// $$ 动态变量
foreach ($_GET as $k => $v) {
    $$k = $v;
}
// ?is_admin=1 → $is_admin = 1

// 动态变量赋值
${$_GET['var']} = $_GET['val'];
// ?var=is_admin&val=1 → $is_admin = 1

// 自定义注册 (register_globals: PHP 4-5.3)
// 通过 php.ini 设置 register_globals=On
// ?user=admin → $user = 'admin'
```

## 典型利用场景

```php
$auth = false;
extract($_GET);
// ?auth=1 → $auth = true → 认证绕过

$admin = false;
parse_str($_SERVER['QUERY_STRING']);
// ?admin=1 → $admin = true → 权限提升
```

## 修复

```php
// 不使用 extract/parse_str/$$ 处理用户输入

// 明确从输入源读取
$name = $_GET['name'] ?? '';
$page = $_GET['page'] ?? 'home';

// extract 使用标志位限制
extract($_GET, EXTR_SKIP);    // 不覆盖已有变量
extract($_GET, EXTR_PREFIX_ALL, 'input_');  // 加前缀
```

## 检测命令

```bash
grep -rn "extract(\|parse_str(" --include="*.php"
grep -rn "\\$\\\$" --include="*.php"
grep -rn "\\${\\\$_" --include="*.php"
```
