# PHP SSTI 模板注入

## 确认条件

用户输入被传给模板引擎作为模板内容解析。

## Twig

```php
// 用户控制模板内容
$loader = new \Twig\Loader\ArrayLoader(['index' => $userInput]);
$twig = new \Twig\Environment($loader);
echo $twig->render('index');

// Sandbox 绕过 (1.x): {{_self.env.registerUndefinedFilterCallback("exec")}}
// Sandbox 绕过 (2.x): {{_self.env.setSandboxed(false)}}
// 通过 filter: {{["id"]|map("system")}}

// 用户输入作为变量而非模板
$twig->render('index.twig', ['user_input' => $userInput]);
```

## Smarty

```php
//
$smarty = new Smarty();
$smarty->display('string:' . $userInput);

// 利用 (Smarty <= 3): {php}phpinfo(){/php}
// 利用 (Smarty >= 4, security 关闭): {system('id')}

//
$smarty->assign('input', $userInput);
$smarty->display('template.tpl');
```

## Blade (Laravel)

```php
//
$compiled = Blade::compileString($userInput);
// {!! $var !!} — 未转义输出（非SSTI但可XSS）

//
return view('hello', ['name' => $userInput]);
```

## 检测命令

```bash
grep -rn "ArrayLoader\|display.*string\|compileString\|Blade::render" --include="*.php"
grep -rn "render(\|display(" --include="*.php" | grep '\$'
```
