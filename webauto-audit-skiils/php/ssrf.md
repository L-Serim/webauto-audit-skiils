# PHP SSRF

## 危险函数

```php
file_get_contents($url)
curl_exec($ch)         // curl_setopt($ch, CURLOPT_URL, $user_url)
readfile($url)
fopen($url, 'r')
fread(fopen($url, 'r'))
copy($url, $local)
include($url)          // allow_url_include=On 时
```

## 检测

```php
$url = $_GET['url'];
$content = file_get_contents($url);
echo $content;

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $_GET['url']);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$result = curl_exec($ch);

$fh = fopen($_GET['url'], 'r');
while ($line = fread($fh, 8192)) { echo $line; }
```

## 修复

```php
$allowed_hosts = ['api.weixin.qq.com', 'api.github.com'];
$host = parse_url($_GET['url'], PHP_URL_HOST);
if (!in_array($host, $allowed_hosts)) die('Invalid host');

// CURL 协议限制
curl_setopt($ch, CURLOPT_PROTOCOLS, CURLPROTO_HTTP | CURLPROTO_HTTPS);
curl_setopt($ch, CURLOPT_FOLLOWLOCATION, false);

// CURL DNS 重绑定防护
curl_setopt($ch, CURLOPT_DNS_CACHE_TIMEOUT, 0);
```

## 利用目标

```
http://169.254.169.254/latest/meta-data/        # AWS 元数据
http://100.100.100.200/latest/meta-data/         # 阿里云元数据
http://metadata.google.internal/                  # GCP 元数据
http://127.0.0.1:9200/_cat/indices               # ES
http://127.0.0.1:6379/                            # Redis
file:///etc/passwd                                # 文件读取
gopher://127.0.0.1:6379/_*3%0d%0a...             # Redis 协议
dict://127.0.0.1:6379/info                        # 端口探测
```

## 检测命令

```bash
grep -rn "file_get_contents\|curl_exec\|readfile\|fopen\|copy(" --include="*.php" | grep '\$'
grep -rn "CURLOPT_URL\|CURLOPT_PROTOCOLS" --include="*.php"
```
