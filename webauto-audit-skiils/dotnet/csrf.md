# .NET CSRF

## 检测

```csharp
// 无 AntiForgeryToken
[HttpPost]
public IActionResult Delete(int id) {
    // 无 ValidateAntiForgeryToken
    db.Users.Remove(db.Users.Find(id));
    db.SaveChanges();
}

// GET 执行写操作
[HttpGet]
public IActionResult DeleteUser(int id) { ... }

// ASP.NET Core 未开启 AntiForgery
// Startup.cs 中未调用 services.AddAntiforgery()

// Cookie 认证 API 无 CSRF 防护
[HttpPost]
[Route("api/transfer")]
public IActionResult Transfer([FromBody] TransferRequest req) {
    // Cookie 认证 → 浏览器自动携带 → CSRF 风险
}
```

## 修复

```csharp
// MVC: ValidateAntiForgeryToken
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Delete(int id) { ... }

// 前端表单
<form method="post">
    @Html.AntiForgeryToken()
    <input name="id" value="1" />
</form>

// API: JWT Bearer Token（天然免疫）
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(...);

// AJAX 自动附带 Token
$.ajax({
    headers: { 'RequestVerificationToken': $('[name="__RequestVerificationToken"]').val() }
});
```

## 检测命令

```bash
grep -rn "ValidateAntiForgeryToken\|AutoValidateAntiforgeryToken" --include="*.cs"
grep -rn "\[HttpPost\]\|\[HttpPut\]\|\[HttpDelete\]" --include="*.cs" | grep -v "AntiForgery\|Validate"
```
