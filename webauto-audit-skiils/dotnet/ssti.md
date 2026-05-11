# .NET SSTI 模板注入

## 确认条件

用户输入进入模板引擎作为模板内容执行。

## Razor

```csharp
// 用户控制模板内容
string template = "Hello @Model.Name! " + userInput;
string result = RazorEngine.Parse(template, model);

// RazorEngine 直接渲染用户内容
Engine.Razor.RunCompile(userInput, "templateKey", null, model);

// 用户输入作为 Model 属性
string template = "Hello @Model.Name! @Model.Message";
RazorEngine.Parse(template, new { Name = "Guest", Message = userInput });
```

## ASPX/WebForms

```asp
<!-- 漏洞: 内联表达式未转义 -->
<%= Request["name"] %>
<%= Request["data"] %>

<!-- 漏洞: Response.Write 直接输出 -->
<% Response.Write(Request["name"]); %>

<!-- 安全: 编码输出 -->
<%: Request["name"] %>
<% Server.HtmlEncode(Request["name"]); %>
```

## 检测命令

```bash
grep -rn "RazorEngine\|RunCompile\|Run" --include="*.cs"
grep -rn "<%= " --include="*.aspx" --include="*.ascx" | grep -v "<%=:"
```
