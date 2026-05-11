# 谛听 (DiTing) — Web安全审计技能索引

按语言/框架组织的跨平台审计技能系统，覆盖 PHP / Java / .NET / Node.js。

## 新用户入口

**第一次审计？** 先读 [SKILL.md](SKILL.md) → 确定技术栈 → 打开对应语言的 `overview.md` → 按优先级逐项审查。

## PHP 审计

| 文件 | 内容 | 难度 |
|------|------|:----:|
| [overview](php/overview.md) | PHP审计总览/危险函数/优先级 | ⭐ |
| [sql-injection](php/sql-injection.md) | mysql_/mysqli_/PDO/Laravel/ThinkPHP | ⭐⭐ |
| [command-injection](php/command-injection.md) | system/exec/shell_exec/popen 等 | ⭐⭐ |
| [file-upload](php/file-upload.md) | move_uploaded_file 验证绕过 | ⭐⭐ |
| [file-include](php/file-include.md) | include/require LFI/RFI/Wrapper | ⭐⭐⭐ |
| [deserialization](php/deserialization.md) | unserialize/Phar/Session/Magic Method | ⭐⭐⭐⭐ |
| [ssrf](php/ssrf.md) | file_get_contents/curl/fopen | ⭐⭐ |
| [xxe](php/xxe.md) | SimpleXML/DOMDocument | ⭐⭐ |
| [ssti](php/ssti.md) | Twig/Smarty/Blade | ⭐⭐⭐ |
| [xss](php/xss.md) | echo/模板未转义 | ⭐ |
| [type-juggling](php/type-juggling.md) | == vs === / strcmp / in_array | ⭐⭐⭐ |
| [variable-override](php/variable-override.md) | extract/parse_str/$$ | ⭐⭐ |
| [authentication](php/authentication.md) | 认证/JWT/Session/IDOR | ⭐⭐ |
| [idor](php/idor.md) | 水平越权/批量操作 | ⭐⭐ |
| [csrf](php/csrf.md) | Token缺失/Cookie认证API | ⭐ |
| [path-traversal](php/path-traversal.md) | file_get/readfile/Zip Slip | ⭐⭐ |
| [business-logic](php/business-logic.md) | 竞争条件/价格操纵/OTP | ⭐⭐⭐ |
| [api-security](php/api-security.md) | Mass Assignment/JWT/CORS/Rate | ⭐⭐ |

### PHP 框架

| [Laravel](php/framework/laravel.md) | Eloquent/Blade/Debug/Mass Assignment | ⭐⭐ |
| [ThinkPHP](php/framework/thinkphp.md) | whereRaw/模板/反序列化/缓存 | ⭐⭐ |

## Java 审计

| 文件 | 内容 | 难度 |
|------|------|:----:|
| [overview](java/overview.md) | Java审计总览/危险API/优先级 | ⭐ |
| [sql-injection](java/sql-injection.md) | JDBC/MyBatis/JPA/Hibernate | ⭐⭐ |
| [deserialization](java/deserialization.md) | ObjectInputStream/Fastjson/Jackson/XStream | ⭐⭐⭐⭐⭐ |
| [jndi-injection](java/jndi-injection.md) | InitialContext/Log4j/LDAP/RMI | ⭐⭐⭐⭐ |
| [spel-injection](java/spel-injection.md) | SpEL/OGNL 表达式注入 | ⭐⭐⭐⭐ |
| [ssti](java/ssti.md) | Freemarker/Velocity/Thymeleaf | ⭐⭐⭐ |
| [ssrf](java/ssrf.md) | URL/RestTemplate/HttpClient | ⭐⭐ |
| [xxe](java/xxe.md) | DocumentBuilder/SAXParser/JAXB | ⭐⭐ |
| [command-injection](java/command-injection.md) | Runtime.exec/ProcessBuilder/ScriptEngine | ⭐⭐ |
| [file-upload](java/file-upload.md) | Servlet/MultipartFile | ⭐⭐ |
| [authentication](java/authentication.md) | Spring Security/Shiro/JWT | ⭐⭐⭐ |
| [idor](java/idor.md) | @PreAuthorize/归属校验 | ⭐⭐ |
| [api-security](java/api-security.md) | Actuator/Mass Assignment/CORS | ⭐⭐ |
| [business-logic](java/business-logic.md) | 竞争条件/状态机/价格操纵 | ⭐⭐⭐ |

### Java 框架

| [Spring Boot](java/framework/spring-boot.md) | Actuator/DevTools/H2/SpEL | ⭐⭐ |
| [Struts2](java/framework/struts2.md) | OGNL注入/S2-xxx系列 | ⭐⭐⭐⭐ |
| [Shiro](java/framework/shiro.md) | rememberMe反序列化/路径绕过 | ⭐⭐⭐⭐ |
| [MyBatis](java/framework/mybatis.md) | ${} vs #{} / 动态SQL | ⭐⭐ |

## .NET 审计

| 文件 | 内容 | 难度 |
|------|------|:----:|
| [overview](dotnet/overview.md) | .NET审计总览/危险API/优先级 | ⭐ |
| [sql-injection](dotnet/sql-injection.md) | ADO.NET/EF/SqlQuery | ⭐⭐ |
| [deserialization](dotnet/deserialization.md) | BinaryFormatter/Json.NET/LosFormatter | ⭐⭐⭐⭐ |
| [viewstate](dotnet/viewstate.md) | ViewState MAC/MachineKey | ⭐⭐⭐ |
| [ssrf](dotnet/ssrf.md) | WebClient/HttpClient/WebRequest | ⭐⭐ |
| [xxe](dotnet/xxe.md) | XmlDocument/XmlResolver | ⭐⭐ |
| [ssti](dotnet/ssti.md) | Razor/ASPX内联表达式 | ⭐⭐⭐ |
| [command-injection](dotnet/command-injection.md) | Process.Start/ShellExecute | ⭐⭐ |
| [file-upload](dotnet/file-upload.md) | FileName/ContentType验证 | ⭐⭐ |
| [authentication](dotnet/authentication.md) | Identity/JWT/Forms Auth | ⭐⭐⭐ |
| [idor](dotnet/idor.md) | 资源归属/MVC | ⭐⭐ |
| [csrf](dotnet/csrf.md) | AntiForgeryToken/Cookie认证 | ⭐ |

### .NET 框架

| [ASP.NET MVC](dotnet/framework/aspnet-mvc.md) | 中间件顺序/Model Binding/Authorize | ⭐⭐ |
| [ASP.NET WebForms](dotnet/framework/aspnet-webforms.md) | ViewState/WebMethod/ValidateRequest | ⭐⭐ |

## Node.js 审计

| 文件 | 内容 | 难度 |
|------|------|:----:|
| [overview](nodejs/overview.md) | Node审计总览/危险API/优先级 | ⭐ |
| [sql-injection](nodejs/sql-injection.md) | 原生/Sequelize/Knex | ⭐⭐ |
| [nosql-injection](nodejs/nosql-injection.md) | MongoDB/Redis/Elasticsearch | ⭐⭐⭐ |
| [prototype-pollution](nodejs/prototype-pollution.md) | merge/extend/lodash/jQuery | ⭐⭐⭐⭐ |
| [ssti](nodejs/ssti.md) | Pug/EJS/Handlebars/Nunjucks | ⭐⭐⭐ |
| [ssrf](nodejs/ssrf.md) | http/https/axios/fetch | ⭐⭐ |
| [command-injection](nodejs/command-injection.md) | exec/spawn/eval/Function | ⭐⭐ |
| [deserialization](nodejs/deserialization.md) | js-yaml/node-serialize | ⭐⭐⭐ |
| [authentication](nodejs/authentication.md) | Passport/JWT/中间件/IDOR | ⭐⭐⭐ |
| [path-traversal](nodejs/path-traversal.md) | sendFile/二次解码/Zip Slip | ⭐⭐ |
| [redos](nodejs/redos.md) | 灾难性回溯/re2/超时 | ⭐⭐⭐ |
| [graphql](nodejs/graphql.md) | Introspection/深度限制/字段授权 | ⭐⭐⭐ |
| [websocket](nodejs/websocket.md) | ws/Socket.io 认证/消息校验 | ⭐⭐⭐ |

### Node.js 框架

| [Express](nodejs/framework/express.md) | 中间件顺序/Helmet/CORS/Session | ⭐⭐ |
| [NestJS](nodejs/framework/nestjs.md) | Guards/Validation/DTO/Swagger | ⭐⭐ |
| [Next.js](nodejs/framework/nextjs.md) | API Routes/SSR凭证/Middleware | ⭐⭐ |

## 跨语言模块

| 文件 | 内容 |
|------|------|
| [dynamic/debug_audit.md](dynamic/debug_audit.md) | 动态调试方法论（4种核心模式） |
| [dynamic/java_arthas_cmds.md](dynamic/java_arthas_cmds.md) | Arthas 安全审计命令速查 |
| [workflow/audit_decision_tree.md](workflow/audit_decision_tree.md) | 8层审计决策树 |
| [workflow/vuln_chaining.md](workflow/vuln_chaining.md) | 漏洞链组合逻辑（9类链） |
| [exploit/script_generation.md](exploit/script_generation.md) | 交互式脚本生成指南 |
| [exploit/template_audit_script.py](exploit/template_audit_script.py) | Python 脚本模板 |
| [remediation/remediation_guide.md](remediation/remediation_guide.md) | 各平台修复方案 |

## 审计时间预算

| 模式 | 时间 | 覆盖 |
|------|:----:|------|
| 快速 | 15min | 语言 overview → P0 招牌漏洞 |
| 标准 | 30min | P0 + P1（严重+高危） |
| 完整 | 60min | 全部漏洞 + 动态验证关键漏洞 |
| 深度 | 不限 | 全漏洞 + 全部动态 + 漏洞链 + 交互脚本 |
