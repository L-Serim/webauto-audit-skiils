# Node.js SSRF

## 危险API

```javascript
http.get(url) / https.get(url)
http.request(url)
axios.get(url)
fetch(url) / node-fetch
request(url)              // 已废弃的 request 库
got(url)
superagent.get(url)
```

## 检测

```javascript
// http.get
const http = require('http');
http.get(req.query.url, (res) => {
    let data = '';
    res.on('data', chunk => data += chunk);
    res.on('end', () => res.send(data));
});

// axios
const axios = require('axios');
const response = await axios.get(req.body.url);
res.send(response.data);

// fetch
const response = await fetch(req.query.url);
const data = await response.text();

// request (废弃)
request(req.query.url, (error, response, body) => {
    res.send(body);
});

// 文件读取 (file://)
const content = fs.readFileSync(req.query.path);
```

## 修复

```javascript
// URL 白名单
const url = new URL(req.query.url);
const allowed = ['api.github.com', 'api.internal.com'];
if (!allowed.includes(url.hostname)) {
    return res.status(403).send('Blocked');
}

// 协议白名单
if (!['http:', 'https:'].includes(url.protocol)) {
    return res.status(403).send('Only HTTP/HTTPS');
}

// DNS 重绑定防护
const { resolve } = require('dns');
const addresses = await resolve(url.hostname);
const privateRanges = ['10.', '172.16.', '192.168.', '127.', '0.'];
if (addresses.some(ip => privateRanges.some(range => ip.startsWith(range)))) {
    return res.status(403).send('Private IP blocked');
}
```

## 利用目标

```
http://169.254.169.254/latest/meta-data/        # AWS
http://metadata.google.internal/                  # GCP
http://127.0.0.1:3000/admin                      # 内网API
file:///etc/passwd                                # 文件读取
http://127.0.0.1:6379/                            # Redis
```

## 检测命令

```bash
grep -rn "http\.get\|http\.request\|https\.get\|https\.request\|axios\|fetch(" --include="*.js" | grep "req\."
```
