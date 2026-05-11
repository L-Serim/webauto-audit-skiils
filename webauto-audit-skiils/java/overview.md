# Java 安全审计总览

## 指纹识别

```yaml
Headers: ["X-Powered-By: Servlet", "JSESSIONID=", "rememberMe=deleteMe"]
Extensions: [.jsp, .do, .action, .jspx]
Key files: [pom.xml, build.gradle, application.yml, WEB-INF/web.xml, application.properties]
Error signatures: ["java.lang.", "ServletException", "at org.springframework", "JBWEB"]
```

## 框架速查

| 框架 | 识别特征 | 招牌漏洞 |
|------|---------|---------|
| Spring Boot | `Application.java`, `application.yml`, `/actuator` | Actuator未授权, SpEL注入, DevTools |
| Spring MVC | `DispatcherServlet`, `@Controller` | Data Binding, Mass Assignment |
| Struts2 | `struts.xml`, `FilterDispatcher` | OGNL注入 (S2-xxx系列) |
| Shiro | `shiro.ini`, `rememberMe=deleteMe` Cookie | rememberMe反序列化, AES弱密钥 |
| MyBatis | Mapper XML, `${}` 语法 | SQL注入 (`${}` 不预编译) |
| Hibernate | `hibernate.cfg.xml`, `@Entity` | HQL注入, 缓存反序列化 |
| Dubbo | `dubbo.xml`, 泛化调用 | Hessian反序列化 |
| Fastjson | `JSON.parse()`, `parseObject()` | 反序列化 (autoType绕过) |

## 危险API速查

```java
// 命令执行
Runtime.getRuntime().exec()
ProcessBuilder.start()

// 反序列化
ObjectInputStream.readObject()
XStream.fromXML()
JSON.parse() / JSON.parseObject()   // Fastjson
ObjectMapper.enableDefaultTyping()  // Jackson
Yaml.load()                          // SnakeYAML

// JNDI注入
InitialContext.lookup()
Context.lookup()

// SpEL注入
SpelExpressionParser.parseExpression()
@Value("#{...}")

// SQL注入
Statement.executeQuery()            // 非PrepareStatement
MyBatis: ${param}                   // 不预编译
JPA: createNativeQuery()            // 拼接SQL

// SSRF
URL.openConnection() / URL.openStream()
RestTemplate.getForObject()
HttpClient.execute()
```

## 审计优先级

```
P0 (必须审): 反序列化 / JNDI注入 / SpEL注入 / Shiro rememberMe
P1 (高优先): SQL注入 / SSTI / 认证绕过 / IDOR
P2 (标准):   SSRF / XXE / 命令注入 / 文件上传
P3 (补充):   XSS / CSRF / 路径遍历 / 信息泄露
```

## 关键配置

```xml
<!-- JDK 9+: 全局反序列化过滤器 -->
-Djdk.serialFilter=!*

<!-- JNDI 保护 -->
-Dcom.sun.jndi.rmi.object.trustURLCodebase=false
-Dcom.sun.jndi.ldap.object.trustURLCodebase=false

<!-- SecurityManager (已废弃但旧系统可能使用) -->
-Djava.security.manager
```
