# PHP XSS

## 确认条件

用户输入输出到HTML/JS上下文未经转义。

## 检测

```php
// 直接 echo
echo "Hello, " . $_GET['name'];
echo "<script>var x = '" . $_GET['x'] . "';</script>";
echo "<input value='" . $_GET['v'] . "'>";

// 存储型
$conn->query("INSERT INTO comments SET content='" . $_POST['msg'] . "'");
// 其他页面: echo $row['content'];

// 模板未转义
echo $twig->render('template', ['content' => $_GET['msg']]);  // Twig自动转义 → 安全
echo $twig->render('template.twig', ['content' => new \Twig\Markup($_GET['msg'], 'UTF-8')]); // 绕过!

// Laravel
{!! $content !!}       // 未转义输出
{{ $content }}         // 自动转义 → 安全

// ThinkPHP
{$content|raw}         // 未转义输出
{$content}             // 自动转义 → 安全
```

## 修复

```php
echo htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');

// 不同上下文
// HTML:    htmlspecialchars($input, ENT_QUOTES, 'UTF-8')
// JS:      json_encode($input)
// URL:     urlencode($input)
// CSS:     避免用户输入进入CSS
// 属性:     htmlspecialchars($input, ENT_QUOTES) + 引号包裹
```

## 利用Payload

```html
<script>alert(document.cookie)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
" onfocus=alert(1) autofocus="
';alert(1);'
javascript:alert(1)
```

## 检测命令

```bash
grep -rn "echo.*\\\$_GET\|echo.*\\\$_POST\|echo.*\\\$_REQUEST\|echo.*\\\$_SERVER" --include="*.php"
grep -rn "print.*\\\$_GET\|print.*\\\$_POST" --include="*.php"
grep -rn "|raw\|new.*Markup\|{!!" --include="*.php" --include="*.twig" --include="*.html"
```
