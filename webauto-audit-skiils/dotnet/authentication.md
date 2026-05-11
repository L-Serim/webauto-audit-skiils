# .NET 认证授权

## ASP.NET Identity

```csharp
// 无授权注解
public IActionResult AdminPanel() {
    return View();  // 任何人可访问
}

// 角色检查缺失
[Authorize]  // 仅检查登录，未检查角色
public IActionResult DeleteUser(int id) { ... }

// 安全
[Authorize(Roles = "Admin")]
public IActionResult DeleteUser(int id) { ... }

// 策略
[Authorize(Policy = "RequireAdminRole")]
public IActionResult AdminPanel() { ... }
```

## JWT

```csharp
// 弱密钥
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => {
        options.TokenValidationParameters = new TokenValidationParameters {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes("1234567890123456")  // 弱密钥
            )
        };
    });

// 未验证过期
ValidateLifetime = false

// 强密钥 + 完整验证
IssuerSigningKey = new SymmetricSecurityKey(
    Encoding.UTF8.GetBytes(Configuration["JwtSecret"])  // >= 256bit
),
ValidateIssuer = true,
ValidateAudience = true,
ValidateLifetime = true,
ClockSkew = TimeSpan.Zero  // 无时间偏差容忍
```

## Forms Authentication

```csharp
// 未更新 Session ID
FormsAuthentication.SetAuthCookie(username, false);
// 登录后 Session 未 Regenerate → Session Fixation

// 安全
Session.Abandon();
Session.Clear();
FormsAuthentication.SetAuthCookie(username, createPersistentCookie: false);
```

## 检测命令

```bash
grep -rn "\[Authorize\]\|\[AllowAnonymous\]" --include="*.cs"
grep -rn "IssuerSigningKey\|ValidateLifetime" --include="*.cs"
grep -rn "FormsAuthentication\|SetAuthCookie" --include="*.cs"
```
