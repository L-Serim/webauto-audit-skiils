# Node.js 原型链污染

## 确认条件

对象合并/拷贝操作允许通过 `__proto__` 或 `constructor.prototype` 污染 Object 原型。

## 检测

```javascript
// 不安全的 merge
function merge(target, source) {
    for (let key in source) {
        if (typeof source[key] === 'object') {
            if (!target[key]) target[key] = {};
            merge(target[key], source[key]);
        } else {
            target[key] = source[key];
        }
    }
}
// merge({}, JSON.parse('{"__proto__":{"isAdmin":true}}'))
// 后果: 所有对象({}).isAdmin === true

// lodash.merge (CVE-2018-3721, CVE-2020-8203)
_.merge(target, req.body);

// jQuery.extend (CVE-2019-11358)
$.extend(true, {}, req.body);

// Object.assign (不递归, 仅第一层污染)
Object.assign(target, req.body);
// {"constructor":{"prototype":{"isAdmin":true}}}

// set-value (CVE-2021-23440)
set(target, '__proto__.isAdmin', true);

// 路径设置
app.put('/config', (req, res) => {
    const config = {};
    req.body.keys.forEach(key => {
        lodash.set(config, key, req.body.value);
    });
    // keys=["__proto__.isAdmin"], value=true
});
```

## 利用链

```
Object原型污染 → 影响服务端代码 → RCE/权限绕过

1. 污染 isAdmin → 认证绕过
2. 污染 shell/exec options → RCE (child_process)
3. 污染 template options → SSTI → RCE (Pug/EJS)
4. 污染 env → 覆盖 NODE_OPTIONS → RCE
```

## 修复

```javascript
// 禁止 __proto__ 和 constructor
function safeMerge(target, source) {
    for (let key in source) {
        if (key === '__proto__' || key === 'constructor') continue;
        if (typeof source[key] === 'object' && source[key] !== null) {
            if (!target[key]) target[key] = {};
            safeMerge(target[key], source[key]);
        } else {
            target[key] = source[key];
        }
    }
}

// 使用 Object.create(null)
const config = Object.create(null);

// 使用 Map
const config = new Map();

// 冻结 Object.prototype
Object.freeze(Object.prototype);
```

## 检测命令

```bash
grep -rn "\.merge\|\.extend\|Object\.assign\|\.defaultsDeep" --include="*.js"
grep -rn "__proto__\|constructor\.prototype" --include="*.js"
grep -rn "lodash\.set\|set-value\|dot-prop" --include="*.js"
grep -rn "JSON\.parse.*req\." --include="*.js" | grep -v "try"
```
