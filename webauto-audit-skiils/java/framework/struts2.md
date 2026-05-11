# Struts2 安全审计

## 识别特征

```
文件: struts.xml, struts.properties
类: extends ActionSupport, BaseAction
依赖: struts2-core, xwork-core
URL: .action, .do
```

## OGNL 注入

S2-xxx 系列漏洞核心通常是 OGNL 表达式注入。

```xml
<!-- struts.xml 中的动态方法调用 -->
<constant name="struts.enable.DynamicMethodInvocation" value="true" />

<!-- 排除类配置缺失/不足 -->
<constant name="struts.excludedClasses"
    value="java.lang.Object,java.lang.Runtime,java.lang.ProcessBuilder" />
```

```java
// JSP 标签 OGNL
<s:property value="%{userInput}" />
<s:if test="%{userInput}">
<s:url action="%{userInput}">

// URL 参数注入
// /action?name=${@java.lang.Runtime@getRuntime().exec('id')}
// /action?redirect:${#context['com.opensymphony.xwork2.dispatcher.HttpServletResponse'].addHeader('X',...)} 
```

## 已知高危版本

| 版本系列 | 代表 CVE |
|---------|---------|
| Struts 2.0.x - 2.3.x | S2-001~S2-057 |
| Struts 2.5.x | S2-045~S2-062 |

## 安全配置

```xml
<!-- 排除高危类 -->
<constant name="struts.excludedClasses"
    value="
        java.lang.Runtime,
        java.lang.ProcessBuilder,
        java.lang.System,
        java.lang.reflect.AccessibleObject,
        ognl.MemberAccess,
        ognl.DefaultMemberAccess,
        com.opensymphony.xwork2.ognl.SecurityMemberAccess" />

<!-- 禁止动态方法调用 -->
<constant name="struts.enable.DynamicMethodInvocation" value="false" />

<!-- OGNL 表达式长度限制 -->
<constant name="struts.ognl.expressionMaxLength" value="256" />
```

## 检测命令

```bash
grep -rn "struts2-core\|xwork-core" pom.xml
grep -rn "struts.enable.DynamicMethodInvocation\|struts.excludedClasses" --include="*.xml"
grep -rn "ActionSupport\|BaseAction" --include="*.java"
```
