# Next.js 安全审计

## 识别特征

```
pages/api/ → API Routes
pages/ → 页面路由
middleware.ts → 中间件
getServerSideProps / getStaticProps
```

## API Routes 认证

```typescript
// API Route 无认证
// pages/api/user/data.ts
export default async function handler(req, res) {
    const data = await db.users.findMany();
    res.json(data);  // 任何人可调用
}

// 认证检查
import { getToken } from 'next-auth/jwt';

export default async function handler(req, res) {
    const token = await getToken({ req });
    if (!token) return res.status(401).json({ error: 'Unauthorized' });
    const data = await db.users.findMany();
    res.json(data);
}

// 或使用 next-auth 的 unstable_getServerSession
import { unstable_getServerSession } from 'next-auth';
import { authOptions } from './auth/[...nextauth]';

export default async function handler(req, res) {
    const session = await unstable_getServerSession(req, res, authOptions);
    if (!session) return res.status(401).json({ error: 'Unauthorized' });
    // ...
}
```

## SSR 凭证泄露

```typescript
// getServerSideProps 返回敏感数据
export async function getServerSideProps(context) {
    const user = await db.user.findUnique({
        where: { id: context.params.id }
    });
    return {
        props: {
            user: JSON.parse(JSON.stringify(user))  // 返回所有字段(含password hash!)
        }
    };
}

// 选择性返回
export async function getServerSideProps(context) {
    const user = await db.user.findUnique({
        where: { id: context.params.id },
        select: { id: true, name: true, email: true }  // 仅公开字段
    });
    return { props: { user } };
}
```

## Middleware 绕过

```typescript
// middleware.ts
export function middleware(request) {
    // 仅检查 /admin 路径
    if (request.nextUrl.pathname.startsWith('/admin')) {
        // 认证逻辑
    }
}

// 绕过技巧:
// /ADMIN/  (大小写)
// /admin;/  (分号)
// /AdMiN/  (混合大小写)
// /admin%2f/ (URL编码)

// 规范化路径
const path = request.nextUrl.pathname.toLowerCase().replace(/\/+$/, '');
if (path.startsWith('/admin')) { ... }
```

## 环境变量泄露

```typescript
// NEXT_PUBLIC_ 前缀变量暴露到客户端
// .env: NEXT_PUBLIC_API_KEY=secret-123
// → 浏览器端可读取 process.env.NEXT_PUBLIC_API_KEY

// 只有客户端需要使用的变量才用 NEXT_PUBLIC_ 前缀
// 服务端密钥不应使用 NEXT_PUBLIC_ 前缀
```

## 检测清单

- [ ] 每个 API Route 有认证检查
- [ ] getServerSideProps 不返回敏感字段
- [ ] Middleware 路径检查做了规范化
- [ ] NEXT_PUBLIC_ 前缀仅用于客户端需要的变量
- [ ] CSP headers 已配置
- [ ] next.config.js 中的 security headers
- [ ] Image 组件使用 remotePatterns 限制外部图片源
