# 通用修复指南

## 编码规范

### SQL注入
```
PHP:   预处理语句/ORM →  $stmt->execute([':id'=>$id])
Java:  PreparedStatement / #{param} → ps.setInt(1, id)
.NET:  SqlParameter / LINQ → cmd.Parameters.AddWithValue("@id", id)
```

### XSS
```
输出编码（按上下文）:
HTML体:   &lt;&gt;&amp;&quot;&#x27; → htmlspecialchars()
HTML属性: 同上，加引号包裹
JavaScript: \x3c\x3e → json_encode() 或 \uXXXX
URL:       %3C%3E → urlencode()
CSS:      \3c\3e → 避免用户输入进入CSS
```

### 文件上传
```
白名单扩展名 + 随机重命名 + 目录禁止执行:
PHP:  in_array($ext, ['jpg','png']) + md5(uniqid()) + .htaccess deny
Java: ext白名单 + UUID + 上传目录外配置
.NET: ext白名单 + Guid + web.config <location> deny
```

## 各平台安全配置

### PHP (php.ini)
```ini
allow_url_include = Off
allow_url_fopen  = Off
expose_php       = Off
display_errors   = Off
session.use_strict_mode = 1
session.use_only_cookies = 1
session.use_trans_sid = 0
session.cookie_httponly = 1
session.cookie_samesite = "Lax"
disable_functions = exec,system,passthru,shell_exec,popen,proc_open,phpinfo
```

### Java (Tomcat/Spring)
```xml
<!-- web.xml: 自定义错误页避免信息泄露 -->
<error-page>
  <error-code>500</error-code>
  <location>/error.html</location>
</error-page>
```
```properties
# Spring: actuator保护
management.endpoints.web.exposure.exclude=*
# 或
management.endpoints.web.exposure.include=health,info
```
```java
// Spring Security
http.csrf().and().authorizeRequests()
    .antMatchers("/admin/**").hasRole("ADMIN")
    .anyRequest().authenticated();
```

### .NET (Web.config)
```xml
<configuration>
  <system.web>
    <httpRuntime enableVersionHeader="false" />
    <customErrors mode="RemoteOnly" defaultRedirect="error.html" />
    <authentication mode="Forms">
      <forms requireSSL="true" />
    </authentication>
    <httpCookies httpOnlyCookies="true" sameSite="Strict" />
  </system.web>
</configuration>
```
