# Spring Boot 安全审计

## 识别特征

```
文件: Application.java, application.yml/properties
依赖: pom.xml → spring-boot-starter-*
端点: /actuator
```

## Actuator 暴露

```yaml
# 漏洞: 全暴露无保护
management:
  endpoints:
    web:
      exposure:
        include: "*"

# 安全: 仅暴露必要端点 + 认证
management:
  endpoints:
    web:
      exposure:
        include: health,info
  endpoint:
    health:
      show-details: when-authorized
```

敏感端点: `/actuator/env` `/actuator/configprops` `/actuator/heapdump` `/actuator/loggers`

## DevTools

```yaml
# 生产环境必须禁用
spring.devtools.remote.secret=  # 不配置则无认证
spring.devtools.restart.enabled=true  # 远程重启风险
```

## H2 Console

```java
// 生产环境开启
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
// 无认证 → 数据库完全暴露

// 生产环境必须禁用
spring.h2.console.enabled=false
```

## SpEL 注入

```java
// @Value 引用用户可控属性
@Value("#{${user.expression}}")
private Object injected;

// @Cacheable key 表达式
@Cacheable(key = "#" + request.getParameter("key"))
public Object get(String param) { ... }
```

## RequestMapping

```java
// 检查所有 @RequestMapping / @GetMapping 是否有权限注解
@GetMapping("/api/admin/users")   // 无 @PreAuthorize → 漏洞
```

## 检测命令

```bash
grep -rn "management.endpoints\|actuator" application.yml application.properties
grep -rn "@PreAuthorize\|@Secured" --include="*.java"
grep -rn "h2.console\|devtools" application.yml application.properties
grep -rn "@Value.*#{" --include="*.java"
```
