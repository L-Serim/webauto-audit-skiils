---
name: di-ting-audit
description: 谛听——按语言/框架组织的跨平台Web安全审计系统，覆盖PHP/Java/.NET/Node.js四大平台
type: skill
---

# 谛听 (DiTing) — 跨平台Web安全审计

覆盖 **PHP / Java / .NET / Node.js** 四大平台，按语言和框架组织。

## 审计流程

| # | 阶段 | 操作 | 参考 |
|---|------|------|------|
| 1 | **指纹** | 响应头/URL/项目文件确定技术栈 | 下方指纹速查 |
| 2 | **总览** | 打开对应语言的 `overview.md` 了解危险函数和优先级 | `{lang}/overview.md` |
| 3 | **静态** | 按优先级 grep 可疑模式，对照语言目录下的漏洞文件 | `{lang}/*.md` |
| 4 | **动态** | 对关键 Sink 运行时拦截确认 | `dynamic/` |
| 5 | **脚本** | 生成交互式检测/利用脚本 | `exploit/` |

## 目录结构

```
SKILL.md                    ← 入口（本文件）
INDEX.md                    ← 完整索引
README.md                   ← GitHub 说明
│
php/                        ← PHP 完整审计（15个漏洞 + 2框架）
java/                       ← Java 完整审计（14个漏洞 + 5框架）
dotnet/                     ← .NET 完整审计（11个漏洞 + 2框架）
nodejs/                     ← Node.js 完整审计（14个漏洞 + 3框架）
│
dynamic/                    ← 动态调试（4平台）
workflow/                   ← 审计决策树 + 漏洞链组合
exploit/                    ← 利用脚本生成
remediation/                ← 修复方案
```

## 各语言漏洞覆盖

| PHP | Java | .NET | Node.js |
|-----|------|------|---------|
| sql-injection | sql-injection | sql-injection | sql-injection |
| command-injection | command-injection | command-injection | command-injection |
| file-upload | file-upload | file-upload | nosql-injection |
| file-include | deserialization | deserialization | prototype-pollution |
| deserialization | jndi-injection | viewstate | ssti |
| ssrf | spel-injection | ssrf | ssrf |
| xxe | ssti | xxe | deserialization |
| ssti | ssrf | ssti | authentication |
| xss | xxe | authentication | path-traversal |
| type-juggling | authentication | idor | redos |
| variable-override | idor | csrf | graphql |
| authentication | api-security | | websocket |
| idor | business-logic | | |
| csrf | | | |
| path-traversal | | | |
| business-logic | | | |
| api-security | | | |

## 各语言框架覆盖

| PHP | Java | .NET | Node.js |
|-----|------|------|---------|
| Laravel | Spring Boot | ASP.NET MVC | Express |
| ThinkPHP | Struts2 | ASP.NET WebForms | NestJS |
| | Shiro | | Next.js |
| | MyBatis | | |

## 平台指纹速查

```yaml
PHP:
  Headers: ["X-Powered-By: PHP", "Set-Cookie: PHPSESSID="]
  Extensions: [.php, .phtml, .php5]
  Files: [composer.json, .env, artisan, index.php]
  Error: ["PHP Notice:", "Fatal error:", "Parse error:"]
  Frameworks: Laravel(artisan) ThinkPHP(think/) Yii2(yii)

Java:
  Headers: ["X-Powered-By: Servlet", "JSESSIONID=", "rememberMe=deleteMe"]
  Extensions: [.jsp, .do, .action, .jspx]
  Files: [pom.xml, build.gradle, WEB-INF/web.xml, application.yml]
  Error: ["java.lang.", "ServletException", "at org.springframework"]
  Frameworks: SpringBoot(Application.java) Struts2(struts.xml) Shiro(rememberMe)

.NET:
  Headers: ["X-AspNet-Version:", "ASPSESSIONID="]
  Extensions: [.aspx, .asmx, .ashx, .asp, .svc]
  Files: [Web.config, *.csproj, Global.asax, appsettings.json]
  Error: ["Server Error in '/' Application", "Runtime Error"]

Node.js:
  Headers: ["X-Powered-By: Express", "x-powered-by"]
  Extensions: [.njs, .ejs, .pug]
  Files: [package.json, node_modules/, server.js, app.js]
  Error: ["Error: ", "at ", "node_modules/"]
```
