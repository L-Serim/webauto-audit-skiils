# Node.js NoSQL注入

## MongoDB (Mongoose)

```javascript
// 直接传递请求体（操作符注入）
const user = await User.findOne({
    username: req.body.username,
    password: req.body.password
});
// {"username":"admin","password":{"$ne":""}} → 密码绕过

// $where 拼接
const query = { $where: `this.username == '${req.body.username}'` };
await User.find(query);
// username: "' || true || '"

// $regex 注入
const users = await User.find({
    email: { $regex: req.body.email }
});
// email: "^a.*" → 逐字符爆破

// 类型强制
const user = await User.findOne({
    username: String(req.body.username),
    password: String(req.body.password)
});
```

## Redis

```javascript
// Key名拼接 — CRLF注入
client.set('user:' + userId, JSON.stringify(data));
// userId = "foo\r\nSET admin:flag 1\r\n"

// 参数化
client.set(`user:${userId}`, JSON.stringify(data));
// 或白名单key名
if (!/^[a-zA-Z0-9_-]+$/.test(userId)) throw new Error();
```

## Elasticsearch

```javascript
// query_string 注入
const result = await client.search({
    query: { query_string: { query: req.query.q } }
});
// q=password:* OR _exists_:ssn

// match 查询限定字段
const result = await client.search({
    query: { match: { title: req.query.q } }
});
```

## 检测命令

```bash
grep -rn "findOne(\|find({" --include="*.js" | grep "req\."
grep -rn "\$where\|\\$regex" --include="*.js"
grep -rn "client\.set\|client\.get" --include="*.js"
```
