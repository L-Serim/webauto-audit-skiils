# Java 认证授权

## Spring Security

```java
// 路径未保护
http.authorizeRequests()
    .antMatchers("/admin/**").permitAll()       // 管理员路径公开
    .anyRequest().authenticated();

// 方法无注解
@GetMapping("/api/admin/users")
public List<User> listUsers() {                 // 无 @PreAuthorize
    return userRepository.findAll();
}

// CSRF 关闭
http.csrf().disable();

// 安全
@GetMapping("/api/admin/users")
@PreAuthorize("hasRole('ADMIN')")
public List<User> listUsers() { ... }

// Mass Assignment 防护
@PostMapping("/user")
public User create(@Valid @RequestBody CreateUserDTO dto) {
    User user = new User();
    user.setName(dto.getName());                // 只设置允许的字段
    user.setEmail(dto.getEmail());
    return userRepository.save(user);
}
```

## Shiro

```java
// 路径绕过
filterRule.put("/admin/**", "authc");
filterRule.put("/admin/backup/**", "anon");     // 细粒度规则覆盖粗粒度

// rememberMe 弱密钥
// 默认 AES key: kPH+bIxk5D2deZiIxcaaaA==
// 用户未修改 → ysoserial 可构造恶意 rememberMe Cookie

// 强制使用自定义强密钥
cookieRememberMeManager.setCipherKey(
    Base64.decode("your-strong-random-key-here")
);
```

## JWT

```java
// 算法混淆
Jwts.parser().setSigningKey(secret).parse(token);  // 未限制算法

// 弱密钥
String secret = "123456";

// 未验证过期
Claims claims = Jwts.parser().setSigningKey(key).parseClaimsJws(token).getBody();
// 未检查 exp

// 安全
Jwts.parserBuilder()
    .setSigningKey(secretKey)
    .requireIssuer("my-app")
    .build()
    .parseClaimsJws(token);  // 自动验证 exp/iat/nbf
```

## 检测命令

```bash
grep -rn "@PreAuthorize\|@Secured\|@RolesAllowed" --include="*.java"
grep -rn "antMatchers.*permitAll\|\.anonymous()" --include="*.java"
grep -rn "rememberMe\|setCipherKey\|getCipherKey" --include="*.java"
grep -rn "setSigningKey\|parseClaimsJws" --include="*.java"
```
