# PHP 路径遍历

## 确认条件

文件路径包含用户输入且未限制在基础目录内。

## 检测

```php
$file = $_GET['file'];
$content = file_get_contents('files/' . $file);
// ?file=../../etc/passwd

include('pages/' . $_GET['page']);

unlink('uploads/' . $_GET['file']);           // 任意文件删除
readfile('templates/' . $_GET['tpl']);

// 压缩包解压
$zip = new ZipArchive();
$zip->extractTo('uploads/');                   // 无路径校验 → Zip Slip
```

## 修复

```php
$base = realpath(__DIR__ . '/files/');
$path = realpath($base . '/' . $_GET['file']);
if (!$path || strpos($path, $base) !== 0) {
    die('Invalid path');
}
$content = file_get_contents($path);

// Zip Slip 防护
while ($entry = $zip->getFromIndex($i)) {
    $target = realpath('uploads/') . '/' . $entry;
    if (strpos(realpath(dirname($target)), realpath('uploads/')) !== 0) {
        throw new Exception('Path traversal detected');
    }
}
```

## 绕过技巧

```
../../../etc/passwd
....//....//....//etc/passwd
..\/..\/..\/etc/passwd
..%2f..%2f..%2fetc/passwd
..%252f..%252f..%252fetc/passwd  (二次编码)
/var/www/files/../../../etc/passwd
```

## 检测命令

```bash
grep -rn "file_get_contents(\|readfile(\|unlink(\|fopen(" --include="*.php" | grep '\$'
grep -rn "include.*\\\$\|require.*\\\$\|include_once.*\\\$" --include="*.php"
grep -rn "ZipArchive\|extractTo" --include="*.php"
```
