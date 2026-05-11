# webauto-audit-skiils — Web 代码审计技能skills

<p align="center">
  <b>按语言/框架组织 · PHP / Java / .NET / Node.js · 50+ 审计文件 · 动态调试 · 漏洞链挖掘</b>
</p>

---

## 项目简介

webauto-audit-skiils是一套适用于 Claude Code 的 Web 代码审计技能skills。

**核心设计理念：按语言和框架组织。** 每种语言拥有完整、独立的审计文件集合——从总览（危险函数/优先级）到每个漏洞的详细模式（漏洞代码 ↔ 安全代码对比），再到框架专项审计。审计者无需在多文件间跳转，一种语言一个目录即可完成完整审计。

## 特性

- **语言优先**：每个语言自成一体的审计目录，包含漏洞模式 + grep命令 + Payload
- **12 个框架覆盖**：Laravel / ThinkPHP / Spring Boot / Struts2 / Shiro / MyBatis / ASP.NET MVC / WebForms / Express / NestJS / Next.js
- **漏洞↔安全代码对比**：每个漏洞文件包含漏洞模式和修复代码的并列对比
- **grep 检测命令**：每个文件附有可执行的 grep 搜索命令，可直接用于源码扫描
- **静态+动态**：代码模式匹配 → Arthas/Xdebug/--inspect 运行时验证
- **漏洞链挖掘**：9 类常见漏洞组合链（中危+中危→严重）
- **交互式脚本生成**：确认漏洞后自动生成 Python 检测/利用脚本

## 目录结构

```
web-audit-skills/
├── SKILL.md                              # 技能入口
├── INDEX.md                              # 完整索引（含难度标注）
├── README.md                             # 本文件
│
├── php/                                  # PHP 完整审计
│   ├── overview.md                       #   总览：危险函数/框架识别/优先级
│   ├── sql-injection.md                  #   SQL注入（原生/PDO/Laravel/TP）
│   ├── command-injection.md              #   命令注入（system/exec/shell_exec）
│   ├── file-upload.md                    #   文件上传（扩展名/Content-Type绕过）
│   ├── file-include.md                   #   文件包含（LFI/RFI/Wrapper/日志污染）
│   ├── deserialization.md                #   反序列化（unserialize/Phar/Session）
│   ├── ssrf.md                           #   SSRF（file_get_contents/curl）
│   ├── xxe.md                            #   XXE（SimpleXML/DOMDocument）
│   ├── ssti.md                           #   SSTI（Twig/Smarty/Blade）
│   ├── xss.md                            #   XSS（echo/模板未转义）
│   ├── type-juggling.md                  #   类型混淆（== vs ===/strcmp/md5）
│   ├── variable-override.md              #   变量覆盖（extract/parse_str/$$）
│   ├── authentication.md                 #   认证授权（密码Hash/JWT/Cookie）
│   ├── idor.md                           #   IDOR越权
│   ├── csrf.md                           #   CSRF
│   ├── path-traversal.md                 #   路径遍历
│   ├── business-logic.md                 #   业务逻辑（竞争条件/价格操纵）
│   ├── api-security.md                   #   API安全（Mass Assignment/CORS）
│   └── framework/
│       ├── laravel.md                    #   Laravel专项
│       └── thinkphp.md                   #   ThinkPHP专项
│
├── java/                                 # Java 完整审计
│   ├── overview.md                       #   总览：危险API/框架识别/优先级
│   ├── sql-injection.md                  #   SQL注入（JDBC/MyBatis/JPA）
│   ├── deserialization.md                #   反序列化（readObject/Fastjson/Jackson/Hessian）
│   ├── jndi-injection.md                 #   JNDI注入（Log4j/LDAP/RMI）
│   ├── spel-injection.md                 #   SpEL/OGNL注入
│   ├── ssti.md                           #   SSTI（Freemarker/Velocity/Thymeleaf）
│   ├── ssrf.md                           #   SSRF（URL/RestTemplate/HttpClient）
│   ├── xxe.md                            #   XXE（DocumentBuilder/SAXParser）
│   ├── command-injection.md              #   命令注入（Runtime.exec/ProcessBuilder）
│   ├── file-upload.md                    #   文件上传（Servlet/MultipartFile）
│   ├── authentication.md                 #   认证授权（Spring Security/Shiro/JWT）
│   ├── idor.md                           #   IDOR越权
│   ├── api-security.md                   #   API安全（Actuator/Mass Assignment）
│   ├── business-logic.md                 #   业务逻辑（竞争条件/状态机）
│   └── framework/
│       ├── spring-boot.md                #   Spring Boot专项
│       ├── struts2.md                    #   Struts2专项
│       ├── shiro.md                      #   Shiro专项
│       └── mybatis.md                    #   MyBatis专项
│
├── dotnet/                               # .NET 完整审计
│   ├── overview.md                       #   总览
│   ├── sql-injection.md                  #   SQL注入（ADO.NET/EF）
│   ├── deserialization.md                #   反序列化（BinaryFormatter/Json.NET）
│   ├── viewstate.md                      #   ViewState/MachineKey
│   ├── ssrf.md                           #   SSRF（WebClient/HttpClient）
│   ├── xxe.md                            #   XXE（XmlDocument）
│   ├── ssti.md                           #   SSTI（Razor）
│   ├── command-injection.md              #   命令注入（Process.Start）
│   ├── file-upload.md                    #   文件上传
│   ├── authentication.md                 #   认证授权（Identity/JWT）
│   ├── idor.md                           #   IDOR越权
│   ├── csrf.md                           #   CSRF
│   └── framework/
│       ├── aspnet-mvc.md                 #   ASP.NET MVC/Core专项
│       └── aspnet-webforms.md            #   WebForms专项
│
├── nodejs/                               # Node.js 完整审计
│   ├── overview.md                       #   总览
│   ├── sql-injection.md                  #   SQL注入（Sequelize/Knex）
│   ├── nosql-injection.md                #   NoSQL注入（MongoDB/Redis/ES）
│   ├── prototype-pollution.md            #   原型链污染
│   ├── ssti.md                           #   SSTI（Pug/EJS/Handlebars/Nunjucks）
│   ├── ssrf.md                           #   SSRF（http/axios/fetch）
│   ├── command-injection.md              #   命令注入（exec/spawn/eval）
│   ├── deserialization.md                #   反序列化（js-yaml/node-serialize）
│   ├── authentication.md                 #   认证授权（Passport/JWT/IDOR）
│   ├── path-traversal.md                 #   路径遍历（sendFile/二次解码）
│   ├── redos.md                          #   ReDoS（灾难性回溯）
│   ├── graphql.md                        #   GraphQL安全
│   ├── websocket.md                      #   WebSocket安全
│   └── framework/
│       ├── express.md                    #   Express专项
│       ├── nestjs.md                     #   NestJS专项
│       └── nextjs.md                     #   Next.js专项
│
├── dynamic/                              # 动态调试
│   ├── debug_audit.md                    #   动态调试方法论（4种模式）
│   └── java_arthas_cmds.md              #   Arthas安全审计命令速查
│
├── workflow/                             # 审计编排
│   ├── audit_decision_tree.md            #   8层审计决策树
│   └── vuln_chaining.md                  #   漏洞链组合逻辑（9类链）
│
├── exploit/                              # 利用与脚本
│   ├── script_generation.md              #   交互式脚本生成
│   ├── template_audit_script.py          #   Python脚本模板
│   └── exploit_guide.md                  #   Payload速查
│
└── remediation/                          # 修复方案
    └── remediation_guide.md              #   各平台修复代码+安全配置
```

## 快速开始

### 安装到 Claude Code

```bash
cp -r web-audit-skills ~/.claude/skills/
```

### 使用

在 Claude Code 对话中提出审计请求：

```
审计这个 Spring Boot 项目的安全性
检查这个 Laravel 项目是否存在 SQL 注入
```

AI 将自动：识别技术栈 → 加载对应语言目录 → 按优先级审计 → 动态验证 → 生成报告。

### 独立阅读

所有文件为纯 Markdown，可直接用编辑器阅读：

```bash
# 先读语言总览
cat php/overview.md
cat java/overview.md

# 按漏洞查阅
cat java/deserialization.md
cat nodejs/prototype-pollution.md
```

## 审计流程

```
指纹识别 → 语言总览 → 静态grep → 动态验证 → 脚本/报告
   │           │          │          │           │
   ▼           ▼          ▼          ▼           ▼
响应头/URL    对应语言    grep命令    Arthas/    交互式
项目文件      overview   模式匹配    Xdebug/    检测脚本
确定技术栈    危险API    逐项审查    --inspect   审计报告
             审计优先级             运行时拦截
```

## 漏洞确认五要素

1. **Source** — 用户可控输入点
2. **Sink** — 危险函数/操作
3. **Path** — Source→Sink 完整调用链（无安全处理）
4. **Impact** — 可达成的具体安全影响
5. **Evidence** — 静态代码证据 + 动态行为证据

## 审计时间预算

| 模式 | 时间 | 覆盖范围 |
|------|:----:|---------|
| 快速扫描 | 15min | 语言 overview → P0 招牌漏洞 |
| 标准审计 | 30min | P0 + P1（严重+高危） |
| 完整审计 | 60min | 全部漏洞 + 关键动态验证 |
| 深度审计 | 不限 | 全漏洞 + 全部动态 + 漏洞链 + 交互脚本 |

## 平台指纹速查

```yaml
PHP:
  Headers: ["X-Powered-By: PHP", "Set-Cookie: PHPSESSID="]
  Extensions: [.php, .phtml, .php5]
  Files: [composer.json, .env, artisan]
  Frameworks: Laravel(artisan) ThinkPHP(think/) Yii2(yii)

Java:
  Headers: ["X-Powered-By: Servlet", "JSESSIONID=", "rememberMe=deleteMe"]
  Extensions: [.jsp, .do, .action]
  Files: [pom.xml, build.gradle, WEB-INF/web.xml, application.yml]
  Frameworks: SpringBoot(Application.java) Struts2(struts.xml) Shiro(rememberMe)

.NET:
  Headers: ["X-AspNet-Version:", "ASPSESSIONID="]
  Extensions: [.aspx, .asmx, .ashx]
  Files: [Web.config, *.csproj, Global.asax]
  Frameworks: ASP.NET MVC(Controllers/) WebForms(.aspx)

Node.js:
  Headers: ["X-Powered-By: Express", "x-powered-by"]
  Extensions: [.njs, .ejs]
  Files: [package.json, node_modules/, server.js]
  Frameworks: Express(app.js) NestJS(module.ts) Next.js(pages/api/)
```

## 贡献

欢迎提交 Issue 和 PR：
- 补充新的漏洞模式或绕过技巧
- 增加平台/框架专项覆盖
- 改进 Payload 和利用链

## 许可

MIT License

---

<p align="center">
  <sub>谛听——听辨万物真伪，守护代码安全</sub>
</p>
