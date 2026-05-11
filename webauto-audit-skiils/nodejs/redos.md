# Node.js ReDoS

## 确认条件

用户输入作为正则表达式匹配的输入，正则存在灾难性回溯。

## 检测

```javascript
// 用户输入作为正则模式
const pattern = new RegExp(req.query.pattern);
const result = pattern.test(input);
// pattern=^(a+)+$ & input=aaaaaaaaaaaaaaaac → CPU 100%

// 邮箱正则 (±quantifier 嵌套)
const emailRegex = /^([a-zA-Z0-9_\.\-])+@(([a-zA-Z0-9\-])+\.)+([a-zA-Z0-9]{2,4})+$/;
emailRegex.test(req.body.email);

// 路径正则
const pathRegex = /^(\/[a-zA-Z0-9_-]+)+\/?$/;
pathRegex.test(req.path);

// 字符串替换递归
const trimRegex = /^\s+|\s+$/g;
// 对超长字符串可能导致性能问题

// Markdown 解析器正则
// 多个 (.*) 嵌套在复杂正则中
```

## 识别灾难性回溯

```
(a+)+       → aaaaaaaaaaaaaaac
(a|aa)+     → aaaaaaaaaaaaaaab
([a-z]+)+   → 大量重复字符
(.*a){n}    → n 次匹配 + a 不匹配
```

## 修复

```javascript
// 不使用用户输入作为正则模式
// 使用 re2 库（线性时间复杂度）
const RE2 = require('re2');
const safeRegex = new RE2(pattern);

// 正则超时
const result = await Promise.race([
    regex.test(input),
    new Promise((_, reject) =>
        setTimeout(() => reject(new Error('Timeout')), 1000))
]);

// 使用 validator.js 而非自定义正则
const validator = require('validator');
validator.isEmail(req.body.email);  // 安全的预建正则

// 限制输入长度
if (input.length > 1000) throw new Error('Input too long');
```

## 检测命令

```bash
grep -rn "new RegExp\|\.test(\|\.match(" --include="*.js" | grep "req\."
grep -rn "\+\*\|\.\+\..*\+\|\.\*.*\.\*" --include="*.js"  # 潜在回溯模式
```
