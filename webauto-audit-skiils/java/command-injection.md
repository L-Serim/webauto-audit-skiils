# Java 命令注入

## 确认条件

用户输入拼接到系统命令中执行。

## 检测

```java
// Runtime.exec 拼接
Runtime.getRuntime().exec("ping " + request.getParameter("ip"));
Runtime.getRuntime().exec(new String[]{"sh", "-c", "ping " + request.getParameter("ip")});

// ProcessBuilder 拼接
new ProcessBuilder("sh", "-c", "ping " + request.getParameter("ip")).start();

// ProcessBuilder 参数位置拼接
new ProcessBuilder("cmd", "/c", "ping", request.getParameter("ip")).start();
// 127.0.0.1 & calc.exe

// ScriptEngine
ScriptEngine engine = new ScriptEngineManager().getEngineByName("js");
engine.eval("var x = " + userInput);
// 1; new java.lang.ProcessBuilder("calc.exe").start()

// Groovy Shell
GroovyShell shell = new GroovyShell();
shell.evaluate(userInput);
// "calc".execute()

// BeanShell
Interpreter i = new Interpreter();
i.eval(userInput);
```

## 修复

```java
// 参数分离（ProcessBuilder 自动处理）
ProcessBuilder pb = new ProcessBuilder("ping", "-c", "4", request.getParameter("ip"));
pb.start();

// 白名单命令
Set<String> allowed = Set.of("ping", "tracert", "nslookup");
if (!allowed.contains(cmd)) throw new SecurityException();

// 不使用 ScriptEngine 处理用户输入
// 使用 SimpleEvaluationContext 代替 StandardEvaluationContext
```

## 检测命令

```bash
grep -rn "Runtime\.getRuntime\|exec(" --include="*.java"
grep -rn "ProcessBuilder\|new ProcessBuilder" --include="*.java"
grep -rn "ScriptEngine\|ScriptEngineManager\|eval(" --include="*.java" | grep -v "\.equals"
grep -rn "GroovyShell\|evaluate(" --include="*.java"
```
