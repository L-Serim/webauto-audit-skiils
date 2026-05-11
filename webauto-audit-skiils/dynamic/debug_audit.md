# 动态调试审计指南

## 概述

静态代码审计只能看到"代码应该怎么跑"，动态调试能确认"代码实际怎么跑"。通过运行时拦截关键方法调用，观察参数传递和数据流，可以**精准确认漏洞是否存在、触发条件是什么、影响范围有多大**。

## 审计流程中的动态调试

```
阶段一: 静态分析 → 找到可疑调用点（如 readObject、拼装SQL、不安全的类型转换）
     ↓
阶段二: 设置拦截点 → 在可疑方法入口/出口设置断点或Trace
     ↓
阶段三: 触发执行 → 发送HTTP请求、操作界面、调用API
     ↓
阶段四: 观察行为 → 记录参数值、返回值、调用栈、数据流
     ↓
阶段五: 验证漏洞 → 追踪用户输入是否到达Sink点、是否被转义
```

## 核心调试模式

### 模式1: 输入追踪法（Taint Tracking）

追踪用户输入从入口到Sink的完整路径，确认用户输入是否被"污染"（未经安全处理）到达敏感操作。

```
HTTP请求参数 → Controller → Service → Mapper/DAO → 数据库（SQL注入Sink）
             ↘                  ↙
          观察点1: 参数是否被转义/参数化?
          观察点2: 是否经过了白名单校验?
          观察点3: 最终拼接到SQL的原始值?
```

### 模式2: 方法拦截法（Method Interception）

对关键的安全方法设置拦截，观察每次调用时的参数和返回值。

```
目标方法:
  java.sql.Statement.executeQuery(String sql)
  java.lang.Runtime.exec(String cmd)
  javax.naming.InitialContext.lookup(String name)
  java.io.ObjectInputStream.readObject()
  org.springframework.web.bind.annotation.RequestMapping
  
拦截内容:
  ├── 调用栈（谁调用了这个方法）
  ├── 参数值（SQL语句/命令/JNDI地址）
  ├── 返回值（查询结果/执行结果）
  └── 上下文（当前用户/请求URL/Session）
```

### 模式3: 异常触发法（Exception Probing）

通过触发异常观察应用的错误处理行为，确认信息泄露或异常处理缺陷。

```
输入非法参数 → 观察错误响应:
  ├── 是否包含堆栈信息? → 调试模式未关闭
  ├── 是否暴露SQL语句? → SQL注入线索
  ├── 是否包含文件路径? → 路径遍历线索
  └── 是否包含认证信息? → Token/密码泄露
```

### 模式4: 热点对比法（Hotspot Comparison）

在同样的请求下，对比安全修复前后的行为差异。

```
修复前: 发送 payload ' OR 1=1-- → 返回全部用户数据
修复后: 发送相同 payload → 返回 "参数非法"

对比SQL日志:
  修复前: SELECT * FROM users WHERE id = '' OR 1=1--'
  修复后: SELECT * FROM users WHERE id = ?
```

---

## 平台调试工具速查

### Java

| 工具 | 用途 | 启动方式 |
|------|------|----------|
| **Arthas** | 运行时方法Trace/Watch/Stack，不重启 | `java -jar arthas-boot.jar <PID>` |
| **JDB** | JDK自带调试器，断点/单步 | `jdb -attach <PORT>` 或 `jdb -connect ...` |
| **BTrace** | 动态字节码Trace（Java 8+） | `btrace <PID> traceScript.java` |
| **JMX** | MBean监控，堆/线程/类加载 | `jconsole <PID>` |
| **JDWP** | 远程调试协议 | `-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005` |
| **Spring Actuator** | 运行时HTTP端点 | 内置 `/actuator/*` |
| **JMC (Java Mission Control)** | JDK飞行记录器，方法分析 | `jmc` |

### PHP

| 工具 | 用途 |
|------|------|
| **Xdebug** | 断点调试 + 函数Trace + 性能分析 |
| **phptrace** | 运行时Trace，不重启PHP |
| **strace** | 系统调用级跟踪 |

### Node.js

| 工具 | 用途 |
|------|------|
| **--inspect** | Chrome DevTools Protocol调试 |
| **async_hooks** | 异步资源生命周期跟踪 |
| **require-in-the-middle** | 模块加载拦截 |
| **clinic.js** | 运行时性能与行为分析 |

### .NET

| 工具 | 用途 |
|------|------|
| **dotnet-trace** | 运行时事件追踪 |
| **dotnet-dump** | 运行时堆分析 |
| **dnSpy** | .NET调试器 + 反编译 |
| **Visual Studio Remote Debugger** | 远程断点调试 |

---

## 审计关键拦截点

### SQL注入

```bash
# Java: 拦截所有SQL执行
Arthas: watch java.sql.Statement executeQuery "{params,throwExp}" -x 3
Arthas: watch java.sql.PreparedStatement executeQuery "{params,throwExp}" -x 3
Arthas: watch org.apache.ibatis.mapping.BoundSql getSql "{params,returnObj}" -x 3
Arthas: stack java.sql.Statement executeQuery -n 5

# MyBatis 参数追踪
Arthas: watch org.apache.ibatis.executor.SimpleExecutor doQuery "{params}" -x 3
Arthas: tt -t org.apache.ibatis.executor.SimpleExecutor doQuery

# Spring JDBC
Arthas: watch org.springframework.jdbc.core.JdbcTemplate query "{params}" -x 3
```

### 反序列化

```bash
# Java: 拦截反序列化入口
Arthas: watch java.io.ObjectInputStream readObject "{params,returnObj}" -x 3
Arthas: stack java.io.ObjectInputStream readObject -n 5

# Fastjson
Arthas: watch com.alibaba.fastjson.JSON parseObject "{params}" -x 3
Arthas: watch com.alibaba.fastjson.parser.DefaultJSONParser parseObject "{params}" -x 3

# Jackson
Arthas: watch com.fasterxml.jackson.databind.ObjectMapper readValue "{params}" -x 3

# Hessian (Dubbo)
Arthas: watch com.caucho.hessian.io.HessianInput readObject "{params}" -x 3

# 自动检测反序列化入口
watch java.io.ObjectInputStream resolveClass "{params,returnObj}" -x 3
watch java.io.ObjectInputStream readClassDescriptor "{params}" -x 3
```

### JNDI注入

```bash
# Java: 拦截所有JNDI lookup
Arthas: watch javax.naming.InitialContext lookup "{params,returnObj}" -x 3
Arthas: watch javax.naming.spi.NamingManager getObjectInstance "{params}" -x 3
Arthas: stack javax.naming.InitialContext lookup -n 5

# Log4j JNDI
Arthas: watch org.apache.logging.log4j.core.net.JndiManager lookup "{params}" -x 3
```

### 命令执行

```bash
# Java: 拦截命令执行
Arthas: watch java.lang.Runtime exec "{params,returnObj}" -x 3
Arthas: watch java.lang.ProcessBuilder start "{params,returnObj}" -x 3
Arthas: stack java.lang.Runtime exec -n 5

# ProcessImpl (Java 9+)
Arthas: watch java.lang.ProcessImpl start "{params}" -x 3
```

### SSRF

```bash
# Java: 拦截所有HTTP请求
Arthas: watch java.net.URL openConnection "{params,returnObj}" -x 3
Arthas: watch java.net.HttpURLConnection connect "{params,returnObj}" -x 3
Arthas: watch org.springframework.web.client.RestTemplate getForObject "{params}" -x 3
Arthas: watch java.net.URL openStream "{params}" -x 3
```

### 文件操作（路径遍历）

```bash
# Java: 拦截文件读写
Arthas: watch java.io.FileInputStream "<init>" "{params}" -x 3
Arthas: watch java.io.FileOutputStream "<init>" "{params}" -x 3
Arthas: watch java.io.File "<init>" "{params}" -x 3
Arthas: watch java.nio.file.Files newInputStream "{params}" -x 3
```

### 认证授权

```bash
# Java: 拦截认证检查
Arthas: watch org.apache.shiro.web.filter.AccessControlFilter onPreHandle "{params}" -x 3
Arthas: watch org.apache.shiro.subject.Subject checkPermission "{params}" -x 3
Arthas: watch org.springframework.security.access.intercept.AbstractSecurityInterceptor beforeInvocation "{params}" -x 3

# JWT
Arthas: watch io.jsonwebtoken.JwtParser parse "{params}" -x 3
Arthas: watch io.jsonwebtoken.impl.DefaultJwtParser parse "{params,returnObj}" -x 3
```

### XSS输出

```bash
# Java: 拦截输出
Arthas: watch javax.servlet.http.HttpServletResponse getWriter "{returnObj}" -x 2
# 通过查看返回内容确认是否转义
```

### XXE

```bash
# Java: XML解析器
Arthas: watch javax.xml.parsers.DocumentBuilder parse "{params}" -x 3
Arthas: watch org.xml.sax.helpers.DefaultHandler startElement "{params}" -x 3
```

### SpEL注入

```bash
# Java: SpEL表达式解析
Arthas: watch org.springframework.expression.spel.standard.SpelExpressionParser parseExpression "{params}" -x 3
Arthas: watch org.springframework.expression.Expression getValue "{params,returnObj}" -x 3
```

---

## 数据流追踪实战

### 场景: SQL注入动态确认

```bash
# 步骤1: 找到可疑参数入口点
Arthas: watch org.springframework.web.method.support.InvocableHandlerMethod invokeForRequest "{params[0]}" -x 2

# 步骤2: 设置SQL拦截点
Arthas: watch java.sql.Statement executeQuery "{params,throwExp}" -x 3

# 步骤3: 发送触发请求
# curl "http://target/api/user?id=1' OR '1'='1"

# 步骤4: 观察执行结果
# ┌── Statement.executeQuery() 被调用
# ├── 参数: SELECT * FROM user WHERE id = '1' OR '1'='1'
# ├── 调用栈: UserController.getUser() → UserServiceImpl.find() → JdbcTemplate.query() → Statement.executeQuery()
# └── 结论: ✗ 用户输入1' OR '1'='1直接拼入SQL，SQL注入确认

# 步骤5: 确认后查看返回值
Arthas: watch java.sql.Statement executeQuery "{returnObj}" -x 3
```

### 场景: 反序列化链追踪

```bash
# 步骤1: 拦截反序列化入口
Arthas: watch java.io.ObjectInputStream readObject "{params[0],returnObj}" -x 3

# 步骤2: 追踪类加载链
Arthas: watch java.io.ObjectInputStream resolveClass "{params,returnObj}" -x 3

# 步骤3: 触发反序列化请求
# curl -X POST "http://target/rpc" -d '...序列化数据...'

# 步骤4: 观察类加载序列
# ┌── readObject() 开始
# ├── resolveClass() → java.util.HashMap
# ├── resolveClass() → org.apache.commons.collections.map.LazyMap
# ├── resolveClass() → org.apache.commons.collections.functors.ChainedTransformer
# ├── resolveClass() → org.apache.commons.collections.functors.InvokerTransformer
# ├── resolveClass() → java.lang.Runtime
# └── ✗ Commons Collections Gadget Chain 确认!
```

### 场景: SSRF + JNDI 联动追踪

```bash
# 步骤1: 拦截URL连接
Arthas: watch java.net.URL openConnection "{params,returnObj}" -x 3

# 步骤2: 拦截JNDI
Arthas: watch javax.naming.InitialContext lookup "{params,returnObj}" -x 3

# 步骤3: 触发请求
# curl "http://target/fetch?url=http://127.0.0.1:8080/admin"

# 步骤4: 观察
# ┌── URL.openConnection("http://127.0.0.1:8080/admin")
# └── ✗ 服务端请求内网地址，SSRF确认
```

---

## 远程调试配置

### Java JDWP 远程调试

```bash
# 在 JVM 启动参数中添加:
JAVA_OPTS="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005"

# 然后使用 IDE 或 jdb 连接:
jdb -attach <target_ip>:5005

# 常用 JDB 命令:
#   stop in com.example.UserController.getUser   # 设置方法断点
#   stop at com.example.UserController:42         # 设置行断点
#   clear com.example.UserController.getUser      # 清除断点
#   locals                                        # 查看局部变量
#   dump this                                     # 查看当前对象
#   print param                                   # 打印参数值
#   step / stepi / next                           # 单步执行
#   where                                         # 查看调用栈
#   eval com.example.Util.check(param)            # 执行表达式
#   threads                                       # 列出线程
#   thread <id>                                   # 切换线程
```

### PHP Xdebug 远程调试

```ini
; php.ini 配置:
xdebug.mode=debug
xdebug.start_with_request=yes
xdebug.client_host=<审计机IP>
xdebug.client_port=9003
xdebug.idekey=PHPSTORM
```

### Node.js --inspect 调试

```bash
# 启动时添加:
node --inspect=0.0.0.0:9229 app.js
# 或启动后触发:
kill -USR1 <PID>  # 动态开启 inspect

# 用 Chrome DevTools 连接: chrome://inspect
# 或命令行调试:
node inspect <host>:<port>
```

---

## Docker/K8s 环境动态调试

```bash
# Docker: 动态附加到容器内 Java 进程
docker exec -it <container> bash
# 容器内安装 Arthas
curl -O https://arthas.aliyun.com/arthas-boot.jar
java -jar arthas-boot.jar

# Docker: 使用 jattach 注入
docker cp jattach <container>:/tmp/
docker exec <container> /tmp/jattach <PID> load /path/to/agent.jar

# K8s: 临时启动调试容器（sidecar）
kubectl debug -it <pod> --image=arthas:latest --target=<container>
# 或在容器内:
kubectl exec -it <pod> -c <container> -- java -jar arthas-boot.jar

# K8s: 端口转发远程调试
kubectl port-forward pod/<pod> 5005:5005
# 然后 jdb -attach localhost:5005
```

---

## 敏感信息运行时泄露检测

```bash
# 确认 Session/Token 是否能被其他线程访问
Arthas: watch javax.servlet.http.HttpSession getAttribute "{params,returnObj}" -x 3
Arthas: watch org.apache.shiro.session.Session getAttribute "{params,returnObj}" -x 3

# 确认密码是否出现在内存转储中
Arthas: watch java.lang.String "<init>" "{params}" -x 2 '1==1' -n 0 | grep -i password

# 确认异常信息中是否包含敏感数据
Arthas: watch java.lang.Exception "<init>" "{params}" -x 2
Arthas: watch java.lang.Throwable printStackTrace "{params}" -x 2
```

---

## 性能与竞态条件调试

```bash
# 并发请求下观察竞态
# 启动两个终端，同时发送请求

# 终端1-观察库存扣减
Arthas: watch com.example.OrderService createOrder "{params,returnObj}" -x 3

# 终端2-观察数据库操作
Arthas: watch java.sql.Statement executeUpdate "{params}" -x 3

# 观察结果: 两个请求同时通过库存检查
# ┌── 请求1: SELECT stock=1 → stock>0 → UPDATE stock=0
# ┌── 请求2: SELECT stock=1 → stock>0 → UPDATE stock=0
# └── ✗ 竞态条件确认: 同一商品被扣了2次，但初始库存为1
```

---

## 调试结束时清理

```bash
# Arthas: 停止所有 watch
Arthas: watch -d

# Arthas: 停止所有 trace
Arthas: trace -d

# Arthas: 停止所有 monitor
Arthas: monitor -d

# Arthas: 退出
Arthas: stop
```
