# Express 安全审计

## 识别特征

```javascript
require('express')
const app = express();
app.get() / app.post() / app.use()
```

## 中间件顺序

```javascript
// 正确顺序
app.use(helmet());                       // 1. 安全Headers
app.use(express.json({ limit: '1mb' })); // 2. Body解析+限制
app.use(session({ ... }));               // 3. Session
app.use(passport.initialize());          // 4. 认证初始化
app.use(passport.session());             // 5. Session认证
app.use(flash());                        // 6. Flash消息
app.use('/', routes);                    // 7. 路由
app.use(express.static('public'));       // 8. 静态文件
app.use(errorHandler);                   // 9. 错误处理(最后!)
```

## Helmet (安全 Headers)

```javascript
// 必须启用
const helmet = require('helmet');
app.use(helmet());

// 关键子项
helmet.contentSecurityPolicy()    // XSS防护
helmet.hsts()                     // HSTS
helmet.frameguard()               // Clickjacking防护
```

## Session

```javascript
// Session配置不安全
app.use(session({
    secret: 'abc123',             // 弱密钥
    resave: true,
    saveUninitialized: true,      // 空Session也存储
    cookie: { secure: false }     // 非HTTPS
}));

// 安全
app.use(session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
        secure: true,             // 仅HTTPS
        httpOnly: true,           // 不可JS访问
        sameSite: 'strict',       // CSRF防护
        maxAge: 24 * 60 * 60 * 1000
    }
}));
```

## CORS

```javascript
// 全开
app.use(cors());
app.use(cors({ origin: '*' }));

// 反射式Origin
app.use(cors({ origin: true }));

// 白名单
app.use(cors({
    origin: ['https://app.com', 'https://admin.app.com'],
    credentials: true
}));
```

## 速率限制

```javascript
const rateLimit = require('express-rate-limit');

// 登录限流
app.use('/login', rateLimit({
    windowMs: 15 * 60 * 1000,  // 15分钟
    max: 5                      // 5次
}));

// 全局限流
app.use(rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100
}));
```

## 检测清单

- [ ] Helmet 已启用
- [ ] 中间件顺序: 认证→路由→静态文件→错误处理
- [ ] Session: secret强密钥, secure=true, httpOnly=true, sameSite=strict
- [ ] CORS: 白名单而非通配符
- [ ] 速率限制: 登录/API端点
- [ ] body-parser 限制 size
- [ ] 无 `x-powered-by` 暴露
- [ ] `app.disable('x-powered-by')`
