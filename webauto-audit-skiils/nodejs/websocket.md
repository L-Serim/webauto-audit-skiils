# Node.js WebSocket 安全

## ws 库

```javascript
// 无认证
const wss = new WebSocket.Server({ port: 8080 });
wss.on('connection', (ws, req) => {
    ws.on('message', (data) => {
        handleMessage(ws, data);  // 任何人都可连接
    });
});

// Token验证
wss.on('connection', (ws, req) => {
    const token = new URL(req.url, 'http://localhost').searchParams.get('token');
    if (!verifyToken(token)) {
        ws.close(4001, 'Unauthorized');
        return;
    }
    ws.userId = extractUserId(token);
});
```

## Socket.io

```javascript
// 无认证 + 全开CORS
const io = require('socket.io')(server, {
    cors: { origin: "*" }
});
io.on('connection', (socket) => { ... });

// 认证中间件
io.use((socket, next) => {
    const token = socket.handshake.auth.token;
    if (!verifyToken(token)) return next(new Error('Unauthorized'));
    socket.userId = extractUserId(token);
    next();
});
```

## 消息级授权

```javascript
// 未校验操作权限
socket.on('join_room', (roomId) => {
    socket.join(roomId);           // 可加入任意房间
});
socket.on('admin_command', (cmd) => {
    executeAdminCommand(cmd);      // 未校验管理员
});

// 逐消息校验
socket.on('join_room', (roomId) => {
    if (!userCanJoinRoom(socket.userId, roomId)) {
        return socket.emit('error', 'Forbidden');
    }
    socket.join(roomId);
});
```

## 速率限制

```javascript
// 无消息频率限制 → DoS
// 消息频率控制
const rateLimit = new Map();
wss.on('connection', (ws, req) => {
    ws.ip = req.socket.remoteAddress;
    ws.on('message', (data) => {
        const limiter = rateLimit.get(ws.ip) || { count: 0, time: Date.now() };
        if (Date.now() - limiter.time > 1000) {
            limiter.count = 0;
            limiter.time = Date.now();
        }
        limiter.count++;
        if (limiter.count > 60) {
            ws.close(4002, 'Rate limited');
            return;
        }
        rateLimit.set(ws.ip, limiter);
    });
});
```

## 检测命令

```bash
grep -rn "new WebSocket\.Server\|socket\.io\|Socket\.Server" --include="*.js"
grep -rn "\.on('connection\|\|\.use(" --include="*.js"
```
