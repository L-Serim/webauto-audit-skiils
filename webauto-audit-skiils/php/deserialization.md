# PHP 反序列化

## 确认条件

`unserialize()` 处理用户可控的序列化数据。

## 检测

```php
// 直接反序列化用户输入
$data = unserialize($_GET['data']);
$obj = unserialize(base64_decode($_COOKIE['user']));
$config = unserialize(file_get_contents($_GET['file']));

// Phar 反序列化（无需 unserialize）
file_exists('phar://' . $_GET['path']);
include('phar://' . $_GET['path']);
file_get_contents('phar://' . $_GET['path']);

// Session 反序列化
session_start();
$_SESSION['user'] = $_POST['user'];
// 条件: session.serialize_handler 配置不一致
// PHP使用php_serialize, 攻击者以php格式写入 → 反序列化攻击者的数据

// __destruct / __wakeup 利用
class Logger {
    public $logFile;
    function __destruct() {
        file_put_contents($this->logFile, 'Log: ' . date('Y-m-d'), FILE_APPEND);
    }
}
// 构造: O:6:"Logger":1:{s:7:"logFile";s:15:"/var/www/shell.php";}
```

## 修复

```php
// PHP 7.3+ 限制允许的类
$data = unserialize($input, ['allowed_classes' => ['User', 'Order']]);

// 使用 JSON 替代
$data = json_decode($input, true);
```

## Magic Methods 利用链

```
__destruct()    → 对象销毁时触发 → GC 自动触发
__wakeup()      → unserialize() 时触发
__toString()    → 对象被转为字符串时触发
__call()        → 调用不存在的方法时触发
__get()         → 访问不存在的属性时触发
__set()         → 设置不存在的属性时触发
```

## 利用工具

```bash
phpggc -l                              # 列出所有 Gadget Chain
phpggc Laravel/RCE1 system id          # 生成 Laravel RCE Payload
phpggc Guzzle/FW1 system id -b         # Base64 编码输出
```

## 检测命令

```bash
grep -rn "unserialize(" --include="*.php"
grep -rn "__destruct\|__wakeup\|__toString\|__call\|__get\|__set" --include="*.php"
grep -rn "function __destruct\|function __wakeup" --include="*.php"  # 自定义Gadget
grep -rn "phar://\|Phar::" --include="*.php"                         # Phar攻击面
grep -rn "session.serialize_handler" --include="*.ini" --include="*.php"
```
