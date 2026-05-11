# Node.js 安全审计总览

## 指纹识别

```yaml
Headers: ["X-Powered-By: Express", "x-powered-by"]
Extensions: [.njs, .ejs, .pug, .hbs]
Key files: [package.json, node_modules/, server.js, app.js, .env]
Error signatures: ["Error: ", "at ", "node_modules/", "TypeError:"]
```

## 框架速查

| 框架 | 识别特征 | 招牌漏洞 |
|------|---------|---------|
| Express | `require('express')`, `app.use()`, `app.get()` | 中间件顺序, 无Helmet, CORS全开 |
| NestJS | `@Module()`, `@Controller()`, `@Injectable()` | Guard遗漏, Validation缺失 |
| Next.js | `pages/api/`, `getServerSideProps` | API Route认证, SSR凭证泄露 |
| Koa | `require('koa')`, `app.use(async ctx` | 中间件顺序, 错误处理 |

## 危险API

```javascript
// 命令执行
exec() / execSync()           // child_process
spawn() / spawnSync()
eval() / new Function()
vm.runInNewContext()

// 代码/SSTI注入
eval()
new Function()
template.render(string, options)   // 直接渲染用户字符串
setTimeout(string) / setInterval(string)

// SSRF
http.get(url) / https.get(url)
axios.get(url)
fetch(url)
node-fetch(url)

// NoSQL注入
MongoDB: findOne({password: {$ne: ""}})
Redis: CRLF注入

// 原型链污染
_.merge(target, source)       // lodash
Object.assign(target, source)
$.extend(target, source)      // jQuery
```

## 审计优先级

```
P0 (必须审): 原型链污染 / eval/Function注入 / SQL/NoSQL注入
P1 (高优先): SSTI / 认证绕过 / SSRF / 中间件顺序
P2 (标准):   命令注入 / IDOR / 路径遍历 / ReDoS
P3 (补充):   XSS / deserialization / 信息泄露
```

## 安全中间件基线

```javascript
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const cors = require('cors');

app.use(helmet());
app.use(cors({ origin: 'https://trusted.com' }));
app.use(rateLimit({ windowMs: 15*60*1000, max: 100 }));
app.use(express.json({ limit: '1mb' }));
```
