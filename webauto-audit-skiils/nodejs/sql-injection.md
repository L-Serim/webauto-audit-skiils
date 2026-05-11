# Node.js SQL注入

## 确认条件

用户输入拼接到SQL语句中执行。

## 检测

```javascript
// 字符串拼接查询
const query = "SELECT * FROM users WHERE id = " + req.query.id;
db.query(query);

// 模板字面量拼接
const query = `SELECT * FROM users WHERE name = '${req.body.name}'`;
db.execute(query);

// ORDER BY 拼接
const query = `SELECT * FROM users ORDER BY ${req.query.sort}`;
db.query(query);

// LIKE 拼接
const query = `SELECT * FROM posts WHERE title LIKE '%${req.query.q}%'`;
db.query(query);

// Sequelize 原始查询
const users = await sequelize.query(
    `SELECT * FROM users WHERE email = '${req.body.email}'`
);

// Knex 原始查询
const users = await knex.raw(
    `SELECT * FROM users WHERE id = ${req.params.id}`
);
```

## 修复

```javascript
// 参数化查询 (mysql2)
const [rows] = await db.execute(
    'SELECT * FROM users WHERE id = ?',
    [req.query.id]
);

// 命名参数
const [rows] = await db.execute(
    'SELECT * FROM users WHERE name = :name',
    { name: req.body.name }
);

// Sequelize 安全用法
const users = await User.findAll({
    where: { email: req.body.email }
});

// Knex 参数绑定
const users = await knex('users')
    .where('id', req.params.id);

// ORDER BY 白名单
const allowed = ['id', 'name', 'created_at'];
const sort = allowed.includes(req.query.sort) ? req.query.sort : 'id';
const query = `SELECT * FROM users ORDER BY ${sort}`;
```

## 检测命令

```bash
grep -rn "SELECT.*\$\{\|SELECT.*'+|\\.query(.*\\$" --include="*.js"
grep -rn "sequelize\.query\|knex\.raw\|db\.query" --include="*.js"
```
