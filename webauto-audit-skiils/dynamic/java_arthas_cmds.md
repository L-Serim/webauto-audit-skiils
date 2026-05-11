# Arthas 安全审计命令速查

## 快速安装

```bash
# 在线安装
curl -O https://arthas.aliyun.com/arthas-boot.jar
java -jar arthas-boot.jar

# 选择目标Java进程后进入交互模式
# 也支持一键式:
java -jar arthas-boot.jar <PID> -c "watch java.lang.Runtime exec {params} -x 3 -n 1"
```

---

## 核心命令

### watch — 方法调用观察（最常用）

```bash
# 基本语法
watch <类名> <方法名> "<表达式>" -x <展开深度> [-n <次数>] [-f]

# 观察参数
watch java.lang.Runtime exec "{params}" -x 3

# 观察参数 + 返回值 + 异常
watch java.lang.Runtime exec "{params,returnObj,throwExp}" -x 3

# 只观察异常
watch java.lang.Runtime exec "{throwExp}" -x 3

# 条件观察：仅当参数包含特定字符串
watch java.sql.Statement executeQuery "{params}" -x 3 "params[0].contains('union')"

# 观察调用者
watch java.sql.Statement executeQuery "{params,throwExp}" -x 3 -n 5

# 观察返回值+方法耗时
watch java.sql.Statement executeQuery "{params,returnObj,throwExp}" -x 3 "#cost>100"

# 过滤高频调用（只打印前5次，防止刷屏）
watch java.lang.String toString "{params}" -x 2 -n 5
```

### trace — 方法内部调用链（定位漏洞触发路径）

```bash
# 追踪方法内部所有子调用及其耗时
trace org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping getHandler

# 追踪自定义Controller的完整执行路径
trace com.example.UserController getUser

# 追踪MyBatis SQL执行路径
trace org.apache.ibatis.executor.SimpleExecutor doQuery -n 3

# 追踪反序列化完整路径
trace java.io.ObjectInputStream readObject -n 3

# 追踪耗时 > 100ms 的路径
trace com.example.UserController getUser "#cost>100"

# 追踪 JDBC 连接路径
trace java.sql.DriverManager getConnection -n 5

# 将结果保存到文件
trace com.example.UserController getUser > /tmp/trace_result.log
```

### stack — 方法调用栈（确认谁调用了危险方法）

```bash
# 谁调用了 Runtime.exec
stack java.lang.Runtime exec -n 5

# 谁调用了 SQL 执行
stack java.sql.Statement executeQuery -n 5

# 谁调用了 JNDI lookup
stack javax.naming.InitialContext lookup -n 5

# 谁调用了 readObject
stack java.io.ObjectInputStream readObject -n 5

# 带条件
stack java.lang.Runtime exec -n 5 '#cost>50'
```

### tt — TimeTunnel 时空隧道（录制+回放请求）

```bash
# 录制所有对指定方法的调用
tt -t java.sql.Statement executeQuery

# 录制
tt -t org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter invokeHandlerMethod

# 查看录制列表
tt -l

# 查看某次调用的详情（包括参数、返回值、异常、耗时）
tt -i <index> -p

# 重新执行某次调用（回放漏洞请求）
tt -i <index> -p --replay-times 1

# 删除所有录制
tt -d --all

# 场景: 录制一次SQL注入请求，反复回放验证修复
# 1. 启动录制: tt -t java.sql.Statement executeQuery
# 2. 发送请求: curl "http://target/api/user?id=1' OR '1'='1"
# 3. 查看录制: tt -l
# 4. 查看详情: tt -i 1000 -p
# 5. 修复后回放: tt -i 1000 --replay-times 1
```

### monitor — 方法调用统计（发现异常调用模式）

```bash
# 统计反序列化调用频率（正常情况下应为0）
monitor java.io.ObjectInputStream readObject -c 5

# 统计SQL执行情况
monitor java.sql.Statement executeQuery -c 5

# 统计所有Controller的调用统计
monitor org.springframework.web.bind.annotation.RequestMapping getForPattern -c 10
```

### jad — 反编译运行时类（确认实际加载的代码）

```bash
# 反编译Controller
jad com.example.UserController

# 反编译具体方法
jad com.example.UserController getUser

# 反编译并显示行号
jad --source-only com.example.UserController

# 反编译 Security/SQL 相关类（确认是否有AOP注入）
jad org.apache.shiro.web.filter.AccessControlFilter
jad org.apache.ibatis.scripting.defaults.DefaultParameterHandler

# 比较源码和运行时类是否一致（防止热加载绕过）
# 如果反编译结果与Git仓库中的源码不一致，说明：
# 1. 有热补丁
# 2. AOP动态代理修改了行为
# 3. 部署的版本与源码不符
```

### sc — 类搜索（按特征定位类）

```bash
# 搜索所有实现 Job 接口的类（Quartz RCE 审计）
sc -d *Job

# 搜索所有 ObjectInputStream 子类
sc -d *ObjectInputStream

# 搜索包含特定方法的类
sc -d -m executeQuery

# 搜索实现了 Serializable 的接口
sc -d *Serializable

# 列出所有 Filter（确认认证过滤器是否生效）
sc -d *Filter

# 搜索所有 Controller
sc -d *Controller
```

### ognl — 表达式执行（运行时调用任意方法）

```bash
# 获取 Spring Context 中的 Bean
ognl '@org.springframework.web.context.ContextLoader@getCurrentWebApplicationContext().getBean("userServiceImpl")'

# 查看系统属性
ognl '@java.lang.System@getProperty("user.dir")'

# 查看数据库配置
ognl '@org.springframework.web.context.ContextLoader@getCurrentWebApplicationContext().getBean("dataSource").getConnection().getMetaData().getURL()'

# 检查 Shiro 配置
ognl '@org.apache.shiro.SecurityUtils@getSubject().getSession().getAttributeKeys()'

# 查看 Redis 中的缓存
ognl '@org.springframework.web.context.ContextLoader@getCurrentWebApplicationContext().getBean("redisTemplate").opsForValue().get("user::admin")'

# 运行时启用 Druid Wall（不想重启）
ognl '@com.alibaba.druid.wall.WallConfig@setSelecetWhereAlwayTrueCheck(true)'

# 检查 SecurityManager（反序列化防护）
ognl '@java.lang.System@getSecurityManager()'
```

### vmtool — JVM 级操作（强制GC、类加载器）

```bash
# 强制GC
vmtool --action forceGc

# 查看类加载器统计
vmtool --action getClassLoader

# 查看JVM信息
vmtool --action getInstances --class java.lang.String --limit 1
```

---

## 场景化组合命令

### 场景1: 全量反序列化审计

```bash
# 1. 搜索所有可能反序列化入口
sc -d *readObject
sc -d *ObjectInputStream

# 2. 设置自动追踪
watch java.io.ObjectInputStream readObject "{params,returnObj}" -x 3 -n 0

# 3. 追踪类加载链
watch java.io.ObjectInputStream resolveClass "{params,returnObj}" -x 3

# 4. 监控反序列化频率
monitor java.io.ObjectInputStream readObject -c 10

# 5. 检查是否有Serialization Filter
ognl '@java.io.ObjectInputFilter$Config@getSerialFilter()'
```

### 场景2: 全面SQL注入审计

```bash
# 1. 拦截所有JDBC执行
watch java.sql.Statement executeQuery "{params[0]}" -x 3 -n 0
watch java.sql.PreparedStatement executeQuery "{params[0]}" -x 3 -n 0

# 2. 跟踪耗时超过50ms的查询（可能是注入成功的大量数据返回）
watch java.sql.Statement executeQuery "{params,returnObj}" -x 3 "#cost>50"

# 3. 查看MyBatis实际生成的SQL
watch org.apache.ibatis.mapping.BoundSql getSql "{returnObj}" -x 3

# 4. 监控所有数据库调用
monitor java.sql.Statement executeQuery -c 10

# 5. 保存可疑SQL
watch java.sql.Statement executeQuery "{params[0]}" -x 3 > /tmp/sql_trace.log
```

### 场景3: 命令执行漏洞定位

```bash
# 1. 全量拦截
watch java.lang.Runtime exec "{params,throwExp}" -x 3 -n 0
watch java.lang.ProcessBuilder start "{params,throwExp}" -x 3 -n 0

# 2. 追踪调用链
stack java.lang.Runtime exec -n 5

# 3. 搜索所有可能执行命令的类
sc -d *Command*
sc -d *Process*

# 4. ProcessImpl (Java 9+)
watch java.lang.ProcessImpl start "{params}" -x 3
```

### 场景4: JWT 安全运行时审计

```bash
# 1. 查看实际token解析过程
watch io.jsonwebtoken.impl.DefaultJwtParser parse "{params,returnObj}" -x 3

# 2. 确认签名算法
watch io.jsonwebtoken.impl.DefaultJwtParser parse -x 3

# 3. 查看验签密钥
ognl '@io.jsonwebtoken.impl.crypto.HmacProvider@getSigningKey()'

# 4. 检查是否信任none算法
watch io.jsonwebtoken.JwtParser isSigned "{returnObj}"
```

### 场景5: 文件上传漏洞动态确认

```bash
# 1. 观察文件上传路径
watch org.springframework.web.multipart.commons.CommonsMultipartFile transferTo "{params}" -x 3

# 2. 观察文件保存路径（确认是否可预测）
watch java.io.FileOutputStream "<init>" "{params}" -x 3

# 3. 确认文件是否经过重命名
stack java.io.FileOutputStream "<init>" -n 5
```

### 场景6: 认证绕过动态分析

```bash
# 1. 查看请求拦截器链
watch org.apache.shiro.web.filter.mgt.PathMatchingFilterChainResolver getChain "{params,returnObj}" -x 3

# 2. 查看JWT Filter执行
watch org.jeecg.config.shiro.filters.JwtFilter isAccessAllowed "{params,returnObj}" -x 3

# 3. 确认权限检查过程
watch org.apache.shiro.authz.ModularRealmAuthorizer isPermitted "{params,returnObj}" -x 3

# 4. 检查Session中保存的用户信息
ognl '@org.apache.shiro.SecurityUtils@getSubject().getPrincipal()'
```

### 场景7: 竞态条件验证

```bash
# 终端1 - 观察事务
watch org.springframework.transaction.interceptor.TransactionAspectSupport invokeWithinTransaction "{params}" -x 3

# 终端2 - 观察SQL更新
watch java.sql.Statement executeUpdate "{params[0]}" -x 3

# 同时发送两个并发请求:
# curl "http://target/api/order/create?productId=1&qty=1" &
# curl "http://target/api/order/create?productId=1&qty=1" &

# 观察是否两个请求都通过了 SELECT stock 检查
```

---

## 审计中的实用技巧

### 防止刷屏
```bash
# 高频方法（如 toString）添加过滤条件
watch java.lang.String toString "{params}" -x 2 -n 5

# 使用条件表达式
watch java.sql.Statement executeQuery "{params}" -x 3 "params[0].contains('select')"
```

### 结果输出到文件
```bash
watch java.lang.Runtime exec "{params}" -x 3 > /tmp/security_audit.log
trace org.apache.shiro.web.filter.mgt.PathMatchingFilterChainResolver getChain > /tmp/shiro_chain.log
```

### 批量停止所有观察
```bash
# 停止所有 watch
watch --delete-all
# 或
watch -d

# 停止所有 trace
trace -d

# 停止所有 tt 录制
tt -d --all
```

### 按条件排除噪声
```bash
# 排除 Quartz 定时任务自身的SQL
watch java.sql.Statement executeQuery "{params}" -x 3 '#cost>20'

# 只看特定包路径下的调用
stack java.lang.Runtime exec -n 5 | grep "com.example"
```
