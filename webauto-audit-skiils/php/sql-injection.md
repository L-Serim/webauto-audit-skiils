# PHP SQL注入

## 确认条件

用户输入直接拼接到SQL字符串中，被数据库执行。

## 检测

```php
// 原生 mysql_ (PHP 5.5 废弃, 7.0 移除)
$sql = "SELECT * FROM users WHERE id = " . $_GET['id'];
$result = mysql_query($sql);

// mysqli 拼接
$sql = "SELECT * FROM users WHERE name = '" . $_GET['name'] . "'";
$result = $db->query($sql);

// PDO 拼接（不使用预处理）
$sql = "SELECT * FROM users WHERE id = " . $_GET['id'];
$stmt = $pdo->query($sql);

// ORDER BY 拼接（不支持参数绑定）
$sql = "SELECT * FROM users ORDER BY " . $_GET['sort'];
$result = $db->query($sql);

// LIKE 拼接
$sql = "SELECT * FROM products WHERE name LIKE '%" . $_GET['q'] . "%'";

// IN 子句拼接
$ids = implode(",", $_GET['ids']);
$sql = "SELECT * FROM users WHERE id IN ($ids)";

// 动态表名/字段名
$sql = "SELECT * FROM " . $_GET['table'];
$sql = "SELECT " . $_GET['field'] . " FROM users";

// 框架原始查询
User::whereRaw("status = " . $_GET['status']);
DB::raw("SELECT * FROM users WHERE id = " . $_GET['id']);
DB::select("SELECT * FROM users WHERE name = '" . $_GET['name'] . "'");
```

## 修复

```php
// PDO 预处理
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$_GET['id']]);

// 命名参数
$stmt = $pdo->prepare("SELECT * FROM users WHERE name = :name");
$stmt->execute([':name' => $_GET['name']]);

// Laravel Eloquent
User::where('id', $_GET['id'])->get();

// ORDER BY 白名单
$allowed = ['id', 'name', 'created_at'];
$sort = in_array($_GET['sort'], $allowed) ? $_GET['sort'] : 'id';
$sql = "SELECT * FROM users ORDER BY " . $sort;

// 框架安全用法
User::where('status', $_GET['status'])->get();
```

## 利用Payload

```sql
' OR '1'='1' --
' OR 1=1 #
' UNION SELECT 1,2,3,4 --
' UNION SELECT table_name,2,3 FROM information_schema.tables --
' AND SLEEP(5) --
' AND (SELECT 1 FROM (SELECT COUNT(*),CONCAT(VERSION(),FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)a) --
```

## 检测命令

```bash
# 搜索字符串拼接SQL
grep -rn "SELECT.*\\\$_" --include="*.php"
grep -rn "query\(.*\\\$_" --include="*.php"
grep -rn "whereRaw\|DB::raw\|DB::select.*\\\$_" --include="*.php"

# 搜索 ORDER BY/IN/LIMIT 动态拼接
grep -rn "ORDER BY.*\\\$_" --include="*.php"
grep -rn "implode.*\\\$_" --include="*.php"

# 搜索非预处理调用
grep -rn "->query(" --include="*.php"
grep -rn "mysql_query\|mysqli_query" --include="*.php"
```
