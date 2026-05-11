# Node.js 命令注入

## 检测

```javascript
// exec — 拼接（最危险，shell解析）
const { exec } = require('child_process');
exec(`ping -c 4 ${req.query.ip}`, (err, stdout) => {
    res.send(stdout);
});
// 127.0.0.1; id

// execSync
const result = execSync(`ping -c 4 ${req.query.ip}`);

// spawn — 参数拼接
const { spawn } = require('child_process');
spawn('sh', ['-c', `ping -c 4 ${req.query.ip}`]);
// 127.0.0.1; id

// eval
eval(`const result = ${req.body.expression};`);
// require('child_process').execSync('id')

// new Function
const fn = new Function(`return ${req.body.expression}`);
// global.process.mainModule.require('child_process').execSync('id')

// setTimeout / setInterval 字符串形式
setTimeout(`doSomething(${req.body.data})`, 1000);

// vm (沙箱逃逸风险)
const vm = require('vm');
vm.runInNewContext(req.body.code);
// 沙箱逃逸: this.constructor.constructor('return this.process')().mainModule.require('child_process').execSync('id')
```

## 修复

```javascript
// spawn 参数分离
const { spawn } = require('child_process');
const proc = spawn('ping', ['-c', '4', req.query.ip]);
// spawn 在未使用 shell=true 时不会调用 /bin/sh

// execFile 参数分离
const { execFile } = require('child_process');
execFile('ping', ['-c', '4', req.query.ip], callback);

// 避免 eval/Function/string setTimeout
// 使用 JSON.parse 或表达式解析库替代 eval
```

## 检测命令

```bash
grep -rn "exec(\|execSync\|spawn.*sh\|spawn.*-c" --include="*.js"
grep -rn "eval(\|new Function(" --include="*.js"
grep -rn "vm\.\|runInNewContext\|runInThisContext" --include="*.js"
```
