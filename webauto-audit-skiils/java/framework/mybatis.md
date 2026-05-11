# MyBatis 安全审计

## 识别特征

```
文件: *Mapper.xml, *Mapper.java
语法: #{} 和 ${}
```

## SQL注入: ${} vs #{}

```xml
<!-- 漏洞: ${} 直接字符串拼接，不预编译 -->
<select id="findById" resultType="User">
    SELECT * FROM users WHERE id = ${id}
</select>
<!-- 传入: id=1 OR 1=1 → WHERE id = 1 OR 1=1 -->

<!-- 漏洞: ORDER BY ${} -->
<select id="findAll">
    SELECT * FROM users ORDER BY ${sortBy}
</select>
<!-- 安全替代: 后端白名单校验后使用 ${} -->

<!-- 漏洞: LIKE ${} -->
<select id="search">
    SELECT * FROM users WHERE name LIKE '%${keyword}%'
</select>

<!-- 安全: #{} 预编译 -->
<select id="findById" resultType="User">
    SELECT * FROM users WHERE id = #{id}
</select>
<!-- 生成: SELECT * FROM users WHERE id = ? -->
```

## 动态 SQL 注入

```xml
<!-- 漏洞: 动态表名/字段名 -->
<select id="findAll">
    SELECT * FROM ${tableName}
</select>

<!-- 漏洞: IN 子句 — 错误用法 -->
<select id="findByIds">
    SELECT * FROM users WHERE id IN (${ids})
</select>

<!-- 安全: foreach -->
<select id="findByIds">
    SELECT * FROM users WHERE id IN
    <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

## 注解方式

```java
// 漏洞
@Select("SELECT * FROM users WHERE id = ${id}")
User findById(@Param("id") String id);

// 安全
@Select("SELECT * FROM users WHERE id = #{id}")
User findById(@Param("id") String id);
```

## 检测命令

```bash
grep -rn '\${' --include="*.xml"  # 所有 ${} 使用
grep -rn '\${' --include="*.xml" | grep -v "jdbc\|driver\|url\|password\|log"
# 排除配置类 ${}，关注 SQL 中的 ${}
```
