# PHP 类型混淆

## 确认条件

松比较（`==`）或类型转换不当导致认证绕过/逻辑缺陷。

## 检测

```php
// 松散比较 (Magic Hashes)
if ($_POST['password'] == $row['password']) // "0e12345" == "0e67890" → true
// 若密码哈希以 0e 开头，输入 0e 开头的任意md5值即可绕过

// strcmp 绕过 (PHP < 5.3)
$ret = strcmp($_POST['password'], $row['password']);
if ($ret == 0) { /* 登录成功 */ }
// password[]=xxx → strcmp(数组,字符串) → null == 0 → true

// in_array 非严格模式
$allowed = ['admin', 'user'];
if (in_array($_GET['role'], $allowed)) { ... }
// ?role=0 → in_array(0, ['admin', 'user']) → true (0 == 'admin')

// switch 松散比较
switch ($_GET['action']) {
    case 'admin':
        break;
    default:
        break;
}
// ?action=0 → 0 == 'admin' → 进入 admin 分支

// json_decode 类型混淆
$data = json_decode($input, true);
if ($data['admin'] == true) { ... }
// {"admin": true} → 通过
// {"admin": "any"} → "any" == true → 通过！
if ($data['admin'] === true) { ... }  // 安全：严格比较

// md5 弱比较
if (md5($a) == md5($b)) { ... }
// a=QNKCDZO&b=240610708 → 两个md5都是以0e开头的科学计数法值
```

## 修复

```php
// 密码比较
hash_equals($expected, $input);    // 时间安全 + 严格按字节比较

// 严格比较
if ($input === $expected) { ... }

// in_array 严格模式
in_array($value, $allowed, true);  // 第三个参数启用严格模式

// 类型转换
$val = (int)$_GET['id'];          // 明确类型
if (!is_numeric($val)) die();     // 类型检查
```

## 检测命令

```bash
grep -rn "\=\= " --include="*.php" | grep -v "===\|!=="
grep -rn "strcmp(" --include="*.php"
grep -rn "in_array(" --include="*.php" | grep -v "true"
grep -rn "switch.*\\\$_" --include="*.php"
```
