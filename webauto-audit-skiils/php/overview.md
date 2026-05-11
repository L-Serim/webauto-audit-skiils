# PHP 安全审计总览

## 指纹识别

```yaml
Headers: ["X-Powered-By: PHP", "Set-Cookie: PHPSESSID="]
Extensions: [.php, .phtml, .php5, .php7, .inc]
Key files: [composer.json, .env, artisan, index.php, phpinfo.php, .htaccess]
Error signatures: ["PHP Notice:", "Fatal error:", "Parse error:", "Warning:"]
```

## 框架速查

| 框架 | 识别特征 | 重点关注 |
|------|---------|---------|
| Laravel | `artisan` 文件, `routes/web.php`, `app/Http/` | Mass Assignment, Debug模式, Blade未转义, 反序列化 |
| ThinkPHP | `think` 文件, `application/`, `thinkphp/` 目录 | whereRaw注入, 模板未转义, 路由RCE历史 |
| Yii2 | `yii` 文件, `controllers/`, `models/` | ActiveRecord注入, Cache反序列化 |
| WordPress | `wp-config.php`, `wp-content/` | WPDB拼接, 插件漏洞, XMLRPC |
| Drupal | `sites/default/settings.php` | SQL注入, CVE-2018-7600 |
| 原生PHP | 无框架文件 | 审计所有用户输入到Sink的路径 |

## PHP 危险函数速查

```php
// 命令执行
system()  exec()  shell_exec()  passthru()  popen()  proc_open()  `` 反引号

// 代码执行
eval()  assert()  preg_replace('/e')  create_function()  call_user_func()

// 文件操作
include()  require()  include_once()  require_once()
file_get_contents()  file_put_contents()  fopen()  readfile()
move_uploaded_file()  unlink()  rename()  copy()

// 反序列化
unserialize()  simplexml_load_string( , LIBXML_NOENT)

// SSRF
file_get_contents($url)  curl_exec()  fopen($url)  readfile($url)

// 变量覆盖
extract()  parse_str()  $$var  import_request_variables()
```

## 审计优先级

```
P0 (必须审): 反序列化(unserialize/phar) / 文件包含 / 命令注入
P1 (高优先): SQL注入 / 类型混淆 / 变量覆盖 / 文件上传
P2 (标准):   SSRF / XXE / SSTI / 认证授权 / IDOR
P3 (补充):   XSS / CSRF / 路径遍历 / 信息泄露
```

## PHP 配置安全基线

```ini
allow_url_include = Off
allow_url_fopen  = Off
expose_php       = Off
display_errors   = Off
session.use_strict_mode = 1
session.use_only_cookies = 1
session.cookie_httponly  = 1
disable_functions = exec,system,passthru,shell_exec,popen,proc_open
```
