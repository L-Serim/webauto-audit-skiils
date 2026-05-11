# Java API 安全

## Spring Boot Actuator

```
未保护端点:
/actuator/env          → 环境变量（含密钥）
/actuator/configprops  → 配置属性
/actuator/heapdump     → 堆转储（含内存中的凭证）
/actuator/loggers      → 日志级别（可隐藏攻击痕迹）
/actuator/mappings     → 所有路由映射
/actuator/threaddump   → 线程堆栈
```

```java
// Actuator 端点保护
management.endpoints.web.exposure.include=health,info
management.endpoint.health.show-details=when-authorized
```

## Mass Assignment

```java
// 直接绑定实体
@PostMapping("/user")
public User createUser(@RequestBody User user) {
    return userRepository.save(user);  // 可设置 isAdmin=true
}

// DTO 对象
@PostMapping("/user")
public User createUser(@Valid @RequestBody CreateUserDTO dto) {
    User user = new User();
    user.setName(dto.getName());
    user.setEmail(dto.getEmail());
    return userRepository.save(user);
}
```

## CORS

```java
// 全开
@CrossOrigin(origins = "*")
@CrossOrigin(origins = "*", allowCredentials = "true")  // 极不安全

// 白名单
@CrossOrigin(origins = "https://trusted-frontend.com")
```

## HTTP 方法

```java
// GET 执行写操作
@GetMapping("/api/order/delete")
public Result deleteOrder(@RequestParam Long id) { ... }

// 未限制方法
@RequestMapping("/api/order/{id}")

// 安全
@DeleteMapping("/api/order/{id}")
@PreAuthorize("hasPermission(#id, 'ORDER', 'DELETE')")
```

## Swagger/API文档

```bash
/swagger-ui.html
/v2/api-docs
/v3/api-docs
/swagger-resources
```

生产环境应禁用: `springdoc.api-docs.enabled=false`

## 检测命令

```bash
grep -rn "CrossOrigin" --include="*.java"
grep -rn "@RequestMapping\|@GetMapping.*delete\|@GetMapping.*create" --include="*.java"
grep -rn "management.endpoints" --include="*.properties" --include="*.yml"
```
