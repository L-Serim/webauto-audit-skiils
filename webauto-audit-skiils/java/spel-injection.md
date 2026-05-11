# Java SpEL 表达式注入

## 确认条件

SpEL 表达式从用户输入拼接构建并求值。

## 检测

```java
// Spring ExpressionParser
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression(request.getParameter("expr"));
Object result = exp.getValue();
// T(java.lang.Runtime).getRuntime().exec("id")

// Spring Template 模式
StandardEvaluationContext ctx = new StandardEvaluationContext();
String expr = "#{" + request.getParameter("expr") + "}";
parser.parseExpression(expr).getValue(ctx);

// @Value 注解
@Value("#{${userExpr}}")           // application.properties 中 ${userExpr} 可控
private Object injected;

// Spring XML Bean
<property name="expression" value="#{T(java.lang.Runtime).getRuntime().exec('id')}" />

// Cache 注解 SpEL
@Cacheable(key = "#" + request.getParameter("key"))
public Object get(String param) { ... }
```

## 修复

```java
// SimpleEvaluationContext 限制反射/类加载
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext ctx = SimpleEvaluationContext.forReadOnlyDataBinding().build();
parser.parseExpression(expr).getValue(ctx);

// 变量方式而非拼接
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("#name");
StandardEvaluationContext ctx = new StandardEvaluationContext();
ctx.setVariable("name", userInput);
exp.getValue(ctx);
```

## OGNL 注入 (Struts2)

```java
// Struts2 标签
<s:if test="%{userInput}">

// Action 属性注入
// ?name=${@java.lang.Runtime@getRuntime().exec('id')}
// ?(#_memberAccess['allowStaticMethodAccess']=true)(@java.lang.Runtime@getRuntime().exec('id'))

// Strict Method Invocation
<constant name="struts.excludedClasses"
    value="java.lang.Runtime,java.lang.ProcessBuilder,java.lang.System" />
```

## 检测命令

```bash
grep -rn "SpelExpressionParser\|parseExpression" --include="*.java"
grep -rn "@Value.*#{" --include="*.java"
grep -rn "StandardEvaluationContext" --include="*.java"
grep -rn "struts.xml\|ognl\|\%{" --include="*.xml" --include="*.java"
```
