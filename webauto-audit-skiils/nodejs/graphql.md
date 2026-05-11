# Node.js GraphQL 安全

## Introspection

```javascript
// 生产环境 Introspection 开启
const server = new ApolloServer({
    schema,
    introspection: true       // 泄露全部 Schema
});

// 生产禁用
const server = new ApolloServer({
    schema,
    introspection: process.env.NODE_ENV !== 'production'
});
```

## 深度限制

```javascript
// 无深度限制 → DoS
// graphql-depth-limit
const depthLimit = require('graphql-depth-limit');
const server = new ApolloServer({
    schema,
    validationRules: [depthLimit(10)]  // 最大10层
});
```

## 成本分析

```javascript
// 批量查询无限制
// query { a1:user(id:1){name} a2:user(id:2){name} ... a1000:user(id:1000){name} }

// 成本限制
const { createComplexityLimitRule } = require('graphql-validation-complexity');
const server = new ApolloServer({
    schema,
    validationRules: [
        createComplexityLimitRule(1000, {
            onCost: (cost) => console.log('query cost:', cost)
        })
    ]
});
```

## 字段级授权

```javascript
// 敏感字段无保护
const typeDefs = gql`
    type User {
        id: ID!
        username: String!
        ssn: String!          # 应不可见
        internalNotes: String  # 应不可见
    }
`;

// Resolver 级授权
const resolvers = {
    User: {
        ssn: (parent, args, context) => {
            if (!context.user.isAdmin) throw new ForbiddenError();
            return parent.ssn;
        },
        internalNotes: () => {
            throw new ForbiddenError();  // 从未暴露
        }
    }
};
```

## 检测命令

```bash
grep -rn "introspection\|playground" --include="*.js" --include="*.ts"
grep -rn "depthLimit\|complexityLimit\|costAnalysis" --include="*.js"
```
