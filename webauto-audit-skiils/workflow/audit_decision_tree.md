# 审计决策树 - 核心编排逻辑

## 概述

当用户提交审计任务时，按以下决策树逐级展开，确保不遗漏关键攻击面。

## 层级0：审计触发判定

```
用户描述包含以下关键字之一 → 进入审计模式:
  "审计" "audit" "安全" "security" "漏洞" "vuln" "代码审查"
  "白盒" "黑盒" "渗透" "pentest"

进入审计模式后 → 读取本文件，按决策树执行
```

## 层级1：技术栈识别

```
源数据:
  ├── 项目文件: pom.xml/build.gradle(Java) / composer.json(PHP) / package.json(Node) / *.csproj(.NET)
  ├── 目录结构: src/main/java/ / app/Http/ / pages/api/ / Controllers/
  ├── 配置文件: application.yml / .env / web.config / appsettings.json
  └── 用户描述: "这是一个Spring项目" / "Laravel项目"等

判定逻辑:
  if exists("pom.xml") or exists("build.gradle") → Java
  if exists("composer.json") → PHP
  if exists("package.json") and exists("node_modules") → Node.js
  if exists("*.csproj") or exists("Web.config") → .NET
  
  if Java + exists("*Application.java") → Spring Boot
  if Java + exists("struts.xml") → Struts2
  if Java + exists("pom.xml") and grep("fastjson") → 注意Fastjson反序列化
  if PHP + exists("artisan") → Laravel
  if PHP + exists("think") → ThinkPHP
```

## 层级2：漏洞优先级矩阵

按平台×漏洞类型，决定审计顺序（数字越小优先级越高）:

```
┌───────────────────────┬─────┬──────┬──────┬────────┐
│ 漏洞类型               │ PHP │ Java │ .NET │ Node   │
├───────────────────────┼─────┼──────┼──────┼────────┤
│ 反序列化               │  4  │  1☠  │  2☠  │  --    │
│ JNDI注入               │ --  │  1☠  │  --  │  --    │
│ SpEL/OGNL注入          │ --  │  1☠  │  --  │  --    │
│ Fastjson反序列化        │ --  │  1☠  │  --  │  --    │
│ Shiro rememberMe       │ --  │  1☠  │  --  │  --    │
│ SQL注入                │  2   │  3   │  3   │   4    │
│ SSTI                   │  3   │  2☠  │  2☠  │  2☠   │
│ 命令注入                │  2   │  3   │  3   │   3    │
│ 文件上传               │  2   │  4   │  4   │   5    │
│ 原型链污染              │ --  │  --  │  --  │  1☠   │
│ SSRF                  │  4   │  3   │  3   │   4    │
│ XXE                   │  5   │  3   │  3   │  --    │
│ 认证绕过               │  2   │  2   │  2   │   2    │
│ IDOR                  │  3   │  2   │  2   │   2    │
│ 路径遍历               │  3   │  5   │  5   │   5    │
│ XSS                   │  4   │  5   │  5   │   5    │
│ CSRF                  │  5   │  5   │  5   │   5    │
│ API安全/JWT            │  3   │  2   │  2   │   2    │
└───────────────────────┴─────┴──────┴──────┴────────┘
☠ = 平台"招牌漏洞"，极高优先级，必须审查
```

### 优先级规则

```
P0 (必须审): 平台招牌漏洞（☠标记）+ 反序列化
P1 (高优先): SQL注入 / 认证绕过 / IDOR / SSTI（若框架使用模板引擎）
P2 (标准): 命令注入 / 文件上传 / SSRF / XXE / 路径遍历
P3 (补充): XSS / CSRF / 信息泄露 / 配置缺陷
```

## 层级3：静态分析触发（按P0→P3执行）

### P0 阶段：平台招牌漏洞（必须审查）

```
Java项目:
  ├── 读取 pom.xml, 搜索: fastjson/jackson/commons-collections/hessian/dubbo/shiro/struts2
  │   若命中 → 打开对应 deep-dive 文件
  ├── 搜索 SpEL: rg "parseExpression|@Value\(#\{|SpelExpressionParser"
  ├── 搜索 JNDI: rg "InitialContext|lookup\(|JndiManager|log4j"
  ├── 搜索 Shiro: rg "rememberMe|CookieRememberMeManager|AesCipherService"
  ├── 搜索 Fastjson: rg "JSON\.parse|parseObject\(.*Object\.class|@type"
  └── 搜索 反序列化: rg "ObjectInputStream|readObject|readUnshared"

Node.js项目:
  ├── 搜索 prototype pollution: rg "Object\.assign|\.extend\(|merge\(|cloneDeep\(|lodash"
  ├── 搜索 eval/Function: rg "eval\(|new Function\(|vm\.run"
  ├── 检查 中间件顺序: 读取主路由文件，检查 app.use 顺序
  └── 搜索 ReDoS: rg "new RegExp\(.*\+|new RegExp\(.*req\."

PHP项目:
  ├── 搜索 反序列化: rg "unserialize\(|phar://|__destruct|__wakeup"
  ├── 搜索 类型混淆: rg "\=\=.*\$_|!\=\=.*hash|strcmp"
  ├── 搜索 变量覆盖: rg "extract\(|parse_str|\\$\$"
  └── 搜索 文件包含: rg "include\(.*\\\$|require\(.*\\\$"

.NET项目:
  ├── 搜索 ViewState: rg "EnableViewStateMac|ViewStateUserKey|LosFormatter"
  ├── 搜索 反序列化: rg "BinaryFormatter|SoapFormatter|TypeNameHandling"
  ├── 搜索 MachineKey: rg "machineKey|validationKey|decryptionKey"
  └── 搜索 Request Validation: rg "ValidateRequest|AllowHtml|ValidateInput"
```

### P1 阶段：高危通用漏洞

```
所有平台:
  ├── SQL注入: 搜索字符串拼接 + execute/query/executeQuery
  │   rg "SELECT.*\+|SELECT.*\\.\\$|query\(.*\\\+|execute\(.*\\\+"
  ├── 认证绕过: 检查权限中间件注册顺序、硬编码凭证
  │   rg "password\s*=\s*\"|secret\s*=\s*\"|isAdmin.*return true"
  ├── IDOR: 搜索资源操作中无所有权检查
  │   rg "findById|getById|params\.id|req\.params" + 上下文判断
  └── SSTI (如果框架使用模板引擎):
       rg "Template\(.*input|render\(.*req|\.display\(|compileString"
```

### P2-P3 阶段：标准漏洞 + 补充

```
按 common/ 文件逐项审查，优先级如矩阵所示
```

## 层级4：动态验证触发条件

```
静态分析发现可疑点后 → 进入动态调试:

条件1: 代码路径可达（HTTP请求→Controller→Service→Sink）
  → 使用 dynamic/debug_audit.md 的拦截点设置
  → Java: 使用 Arthas watch 拦截关键方法
  → PHP: 使用 Xdebug trace 跟踪函数调用
  → Node: 使用 --inspect 断点

条件2: 代码路径不清晰（动态代理/反射/AOP/中间件隐式调用）
  → 使用 stack 命令查看完整调用链
  → 追踪用户输入从入口到 Sink 的完整路径

条件3: 需要确认WAF/过滤器是否生效
  → 发送 payload 观察是否被拦截
  → 拦截过滤方法确认参数是否经过处理

条件4: 反序列化链需要验证（gadget class是否在classpath中）
  → 使用 Arthas sc 命令搜索类
  → 确认 JDK 版本和 Serialization Filter 配置
```

## 层级5：漏洞确认标准

```
漏洞确认（五要素制）:
  ┌────────────────────────────────────────────────────┐
  │ 1. Source (源): 用户可控的输入点 (明确)               │
  │ 2. Sink (汇): 危险函数/操作 (明确)                   │
  │ 3. Path (路径): Source→Sink 的完整调用链 (无安全处理)  │
  │ 4. Impact (影响): 可达成的安全影响 (具体)              │
  │ 5. Evidence (证据): 静态代码证据 + 动态行为证据        │
  └────────────────────────────────────────────────────┘

五要素不全时:
  ├── Source不明确 → 追溯调用链找到最初的HTTP入口
  ├── Sink不明确 → 不确认为漏洞，标记为"可疑点"
  ├── Path有安全处理 → 检查是否可绕过（二次编码/字符集/协议差异）
  ├── Impact不明确 → 评估最坏情况影响
  └── Evidence不足 → 进入动态调试补充证据
```

## 层级6：利用验证决策

```
确认漏洞后 → 是否生成POC:

安全环境 (白盒审计/CTF/授权渗透):
  → 可以生成完整利用代码
  → 调用 exploit/script_generation.md 生成交互式脚本
  → 调用 exploit/exploit_guide.md 选择利用链

生产环境:
  → 仅生成检测脚本（不执行破坏性操作）
  → Payload限制: 时间延迟/错误触发/DNS外带（不写文件/不执行命令）
  → 向用户明确说明风险后确认执行

不确定:
  → 生成"只检测不利用"的payload
  → 如: SLEEP(2) 时间检测 → 确认SQL注入，但不UNION SELECT
  → 如: 访问 /actuator/health → 确认信息泄露，但不读取配置
```

## 层级7：漏洞影响链评估

```
确认多个漏洞后 → 按 vuln_chaining.md 评估组合影响:

单漏洞评估:
  P0反序列化/JNDI/SpEL → 直接RCE → 严重 (9.0+)
  SQL注入(可UNION) → 数据泄露+RCE → 严重 (8.0+)
  认证绕过 → 未授权访问 → 高危 (7.0+)
  IDOR → 越权访问 → 高危 (6.0+)
  SSRF(可访问内网) → 内网渗透 → 高危 (7.0+)
  文件上传(可执行) → RCE → 严重 (8.0+)

组合评估 (两个中危 → 一个严重):
  XSS + CSRF → 蠕虫式攻击 → 升级为高危
  SSRF(受限) + 内网未授权服务 → 升级为严重
  IDOR + 敏感信息泄露 → 批量数据窃取 → 升级为严重
  LFI + 文件上传(无执行) → log文件包含 → 升级为RCE
  路径遍历 + 配置文件读取 → 凭证泄露 → 升级为严重
```

## 层级8：报告生成

```
按优先级输出:
  1. 严重漏洞详情 (每个包含: Source/Sink/Path/Impact/Evidence/修复建议)
  2. 高危漏洞详情
  3. 中危漏洞详情
  4. 漏洞组合链分析
  5. 修复优先级建议

可选自动化:
  → 调用 exploit/script_generation.md 生成检测脚本
  → 调用 remediation/remediation_guide.md 生成修复代码
```

## 快速审计模式（时间受限时）

```
15分钟模式:
  1. 指纹识别 → 2. 仅审P0（平台招牌漏洞）→ 3. 仅审SQL注入
  → 输出: 只报告严重漏洞

30分钟模式:
  1. 指纹识别 → 2. 审P0+P1 → 3. 关键Sink点快速grep
  → 输出: 严重+高危漏洞

60分钟模式:
  完整五阶段审计: 指纹 → P0→P3 → 动态验证关键漏洞 → 利用确认
  → 输出: 完整审计报告

无限制模式:
  完整七阶段审计 + 漏洞链分析 + 交互式脚本生成
```
