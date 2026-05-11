# Node.js 反序列化

## 确认条件

用户输入的序列化数据被反序列化函数处理。

## 漏洞API

```javascript
// js-yaml: load() 可执行 JS 函数
const yaml = require('js-yaml');
const obj = yaml.load(req.body.yaml);  // 危险: load() 支持 !!js/function
// 攻击:
// !!js/function >
//   require('child_process').execSync('id')

// safeLoad 已废弃, load 默认安全 (v4+)
const obj = yaml.load(req.body.yaml, { schema: yaml.DEFAULT_SCHEMA }); // 安全

// node-serialize
const serialize = require('node-serialize');
const obj = serialize.unserialize(req.body.data);
// {"rce":"_$$ND_FUNC$$_function(){require('child_process').execSync('id')}"}

// serialize-javascript (eval)
const deserialize = eval(`(${req.body.data})`);

// forEach 自定义反序列化
function deserialize(obj) {
    for (let key in obj) {
        if (typeof obj[key] === 'function') {  // 危险
            obj[key]();
        }
    }
}
```

## 修复

```javascript
// js-yaml: 使用 safeLoad 或限定 schema
const obj = yaml.load(data, { schema: yaml.FAILSAFE_SCHEMA });

// 避免 node-serialize / serialize-javascript
// 使用 JSON.parse 替代
const obj = JSON.parse(data);

// 使用 structuredClone 或 JSON 深拷贝
const copy = JSON.parse(JSON.stringify(obj));
```

## 检测命令

```bash
grep -rn "\.load(\|\.unserialize\|node-serialize\|serialize-javascript\|js-yaml" --include="*.js"
grep -rn "eval.*JSON\|eval.*req\." --include="*.js"
```
