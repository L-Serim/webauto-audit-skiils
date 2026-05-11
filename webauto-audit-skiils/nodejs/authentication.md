# Node.js 认证授权

## Express 中间件顺序

```javascript
// 静态文件先于认证
app.use(express.static('public'));
app.use(authMiddleware);
// → public/ 下所有文件无需认证

// 路由先于认证
app.use('/api/admin', adminRoutes);  // 无认证
app.use(authMiddleware);             // 太晚了

// 错误处理吞掉认证异常
app.use(errorHandler);
app.use(authMiddleware);
// → auth抛出的401被errorHandler捕获

// 安全顺序
app.use(authMiddleware);             // 1. 认证
app.use('/api', apiRoutes);          // 2. 路由
app.use(express.static('public'));   // 3. 静态文件
app.use(errorHandler);               // 4. 最后: 错误处理
```

## JWT

```javascript
// 允许 none 算法
jwt.verify(token, secret, { algorithms: ['HS256', 'none'] });

// 弱密钥
const secret = '123456';

// 未验证过期
jwt.verify(token, secret, { ignoreExpiration: true });

// 安全
jwt.verify(token, process.env.JWT_SECRET, {
    algorithms: ['HS256'],
    issuer: 'my-app',
    maxAge: '1h'
});
```

## Passport

```javascript
// 路由未挂载 Passport 中间件
app.get('/api/profile', (req, res) => {
    res.json(req.user);  // req.user 可能不存在
});

// 安全
app.get('/api/profile', passport.authenticate('jwt', { session: false }), (req, res) => {
    res.json(req.user);
});
```

## IDOR

```javascript
// 无归属校验
app.get('/api/order/:id', async (req, res) => {
    const order = await Order.findById(req.params.id);
    res.json(order);
});

// 验归属
app.get('/api/order/:id', auth, async (req, res) => {
    const order = await Order.findOne({
        _id: req.params.id,
        userId: req.user.id
    });
    if (!order) return res.status(403).send('Forbidden');
    res.json(order);
});
```

## 检测命令

```bash
grep -rn "jwt\.verify\|jwt\.sign" --include="*.js"
grep -rn "passport\.authenticate\|authMiddleware" --include="*.js"
grep -rn "app\.use.*auth\|app\.get.*admin" --include="*.js"
```
