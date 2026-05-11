# Java SQL注入

## 确认条件

用户输入拼接到SQL中且非参数化执行。

## 检测

```java
// JDBC Statement 拼接
String sql = "SELECT * FROM users WHERE id = " + request.getParameter("id");
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(sql);

// PreparedStatement 拼接（错误用法）
String sql = "SELECT * FROM users WHERE name = '" + name + "'";
PreparedStatement ps = conn.prepareStatement(sql);  // 拼接后的SQL再次预编译无效
ps.executeQuery();

// LIKE 拼接
String sql = "SELECT * FROM users WHERE name LIKE '%" + search + "%'";
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(sql);

// ORDER BY 拼接
String sql = "SELECT * FROM users ORDER BY " + request.getParameter("sort");
stmt.executeQuery(sql);  // ORDER BY 不支持 PreparedStatement 参数化

// IN 子句拼接
String ids = request.getParameter("ids");
String sql = "SELECT * FROM users WHERE id IN (" + ids + ")";
stmt.executeQuery(sql);
```

## MyBatis 注入

```xml
<!-- 漏洞: ${} 直接拼接，不预编译 -->
<select id="findById" resultType="User">
    SELECT * FROM users WHERE id = ${id}
</select>

<select id="findAll">
    SELECT * FROM users ORDER BY ${sortBy}
</select>

<!-- 安全: #{} 预编译 -->
<select id="findById" resultType="User">
    SELECT * FROM users WHERE id = #{id}
</select>
```

## JPA/Hibernate

```java
// JPQL拼接
String jpql = "SELECT u FROM User u WHERE u.name = '" + name + "'";
Query query = entityManager.createQuery(jpql);

// Native Query 拼接
String sql = "SELECT * FROM users WHERE id = " + id;
Query query = entityManager.createNativeQuery(sql);

// 参数绑定
Query query = entityManager.createQuery(
    "SELECT u FROM User u WHERE u.name = :name"
);
query.setParameter("name", name);
```

## 检测命令

```bash
grep -rn "Statement\|createStatement" --include="*.java"
grep -rn "executeQuery\|executeUpdate" --include="*.java" | grep '+'
grep -rn '\${' --include="*.xml"  # MyBatis Mapper
grep -rn "createNativeQuery\|createQuery" --include="*.java" | grep '+'
```
