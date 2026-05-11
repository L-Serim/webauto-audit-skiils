# .NET SQL注入

## 检测

```csharp
// ADO.NET 拼接
string sql = "SELECT * FROM users WHERE id = " + Request["id"];
SqlCommand cmd = new SqlCommand(sql, conn);
SqlDataReader reader = cmd.ExecuteReader();

// LIKE 拼接
string sql = "SELECT * FROM users WHERE name LIKE '%" + search + "%'";
SqlCommand cmd = new SqlCommand(sql, conn);

// ORDER BY 拼接
string sql = "SELECT * FROM users ORDER BY " + Request["sort"];
SqlCommand cmd = new SqlCommand(sql, conn);

// Entity Framework 原始SQL
var users = db.Database.SqlQuery<User>(
    "SELECT * FROM users WHERE id = " + id
).ToList();

// EF FromSqlRaw 拼接
var users = db.Users.FromSqlRaw(
    "SELECT * FROM users WHERE name = '" + name + "'"
).ToList();

// EF 动态表名
db.Database.ExecuteSqlRaw(
    "DELETE FROM " + tableName
);
```

## 修复

```csharp
// 参数化查询
SqlCommand cmd = new SqlCommand(
    "SELECT * FROM users WHERE id = @id", conn
);
cmd.Parameters.AddWithValue("@id", id);

// Entity Framework LINQ
db.Users.Where(u => u.Id == id).ToList();

// EF FromSqlRaw 参数化
db.Users.FromSqlRaw("SELECT * FROM users WHERE id = {0}", id);

// ORDER BY 白名单
string[] allowed = { "id", "name", "created_at" };
string sort = allowed.Contains(Request["sort"]) ? Request["sort"] : "id";
```

## 检测命令

```bash
grep -rn "SqlCommand\|ExecuteReader\|ExecuteNonQuery" --include="*.cs" | grep '+'
grep -rn "SqlQuery\|FromSqlRaw\|ExecuteSqlRaw" --include="*.cs" | grep '+'
grep -rn "AddWithValue" --include="*.cs"  # 应存在（安全标记）
```
