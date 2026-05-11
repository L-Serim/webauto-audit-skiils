# Node.js 路径遍历

## 确认条件

文件路径包含用户输入且未校验路径范围。

## 检测

```javascript
// 直接拼接路径
const path = require('path');
app.get('/files/:file', (req, res) => {
    res.sendFile(path.join(__dirname, 'files', req.params.file));
    // /files/../../etc/passwd
});

// fs.readFile 直接拼接
const data = fs.readFileSync(`./uploads/${req.query.file}`, 'utf8');

// 二次解码绕过
app.get('/files/:file', (req, res) => {
    let fileName = req.params.file;
    // Express 自动 decodeURIComponent 一次
    if (fileName.includes('..')) return res.status(403);
    // %252e%252e%252f → 第一次解码: %2e%2e%2f → 不包含".."
    // Express再解码 → "../" → 绕过
    res.sendFile(path.join(__dirname, 'files', fileName));
});

// Zip Slip
const AdmZip = require('adm-zip');
const zip = new AdmZip(req.file.buffer);
zip.extractAllTo('./uploads/', true);
```

## 修复

```javascript
const path = require('path');

app.get('/files/:file', (req, res) => {
    const baseDir = path.resolve(__dirname, 'files');
    const filePath = path.resolve(baseDir, req.params.file);

    if (!filePath.startsWith(baseDir)) {
        return res.status(403).send('Forbidden');
    }
    res.sendFile(filePath);
});
```

## 检测命令

```bash
grep -rn "sendFile\|readFile\|createReadStream" --include="*.js" | grep "req\."
grep -rn "path\.join\|path\.resolve" --include="*.js" | grep "req\."
grep -rn "AdmZip\|extractTo\|Unzipper" --include="*.js"
```
