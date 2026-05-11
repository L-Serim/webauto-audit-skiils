# Apache Shiro 安全审计

## 识别特征

```
Cookie: rememberMe=deleteMe
配置: shiro.ini, application.yml → shiro:
依赖: shiro-core, shiro-spring, shiro-web
```

## rememberMe 反序列化

```java
// 硬编码/弱 AES 密钥
// Shiro < 1.2.4: 默认密钥 kPH+bIxk5D2deZiIxcaaaA==
// 密钥泄露 → ysoserial 构造 rememberMe Cookie → RCE

// Cookie 格式:
// rememberMe=base64(AES_CBC_encrypt(serialized_principal))

// 检测: Cookie 中出现 rememberMe=deleteMe → Shiro 存在

// 生成强随机密钥
// 使用 shiro-tools HashedCredentialsMatcher 生成
cookieRememberMeManager.setCipherKey(
    Base64.decode("your-256bit-random-key-here")
);
```

## 权限绕过 (Ant 路径匹配)

```java
// 路径匹配缺陷
filterChainDefinitionMap.put("/admin/**", "authc");
filterChainDefinitionMap.put("/admin/page/**", "anon");  // 细粒度覆盖

// 绕过技巧:
// /admin/../public/page  → 路径规范化差异
// /admin;/page           → 分号截断
// /Admin/page            → 大小写绕过

// 统一路径规范化
filterChainDefinitionMap.put("/**", "authc");
filterChainDefinitionMap.put("/login", "anon");
```

## 常见漏洞版本

```
Shiro 1.2.4: rememberMe 硬编码 AES key
Shiro 1.4.1-1.7.0: 路径绕过 CVE-2020-13933
Shiro 1.7.0-1.9.1: 路径绕过 CVE-2022-40664
```

## 检测命令

```bash
grep -rn "setCipherKey\|getCipherKey\|rememberMe\|cookieRememberMe" --include="*.java"
grep -rn "filterChainDefinition\|filterChainDefinitionMap" --include="*.java" --include="*.ini"
grep -rn "shiro-core\|shiro-spring" pom.xml
```
