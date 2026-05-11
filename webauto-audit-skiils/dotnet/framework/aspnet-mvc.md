# ASP.NET MVC / Core 安全审计

## 识别特征

```
MVC: Controllers/, Views/, Global.asax, RouteConfig.cs
Core: Startup.cs, appsettings.json, Program.cs
```

## 认证中间件顺序 (Core)

```csharp
// 中间件顺序错误
app.UseStaticFiles();                           // 先静态文件
app.UseAuthentication();                        // 后认证 → 静态文件无保护

// 错误处理中间件过早
app.UseExceptionHandler("/error");              // 吞掉认证异常
app.UseAuthentication();

// 安全
app.UseAuthentication();                        // 1. 认证
app.UseAuthorization();                         // 2. 授权
app.UseStaticFiles();                           // 3. 静态文件
app.UseExceptionHandler("/error");              // 4. 最后错误处理
```

## Authorize 属性

```csharp
// Controller 有 [Authorize] 但个别 Action 有 [AllowAnonymous]
[Authorize]
public class AdminController : Controller {
    [AllowAnonymous]
    public IActionResult SecretData() { ... }   // 公开访问
}

// 全局过滤器（推荐）
services.AddMvc(options => {
    options.Filters.Add(new AuthorizeFilter()); // 全局要求认证
});
```

## Model Binding (Mass Assignment)

```csharp
// 直接绑定实体
[HttpPost]
public IActionResult Update(User user) {
    db.Users.Update(user);  // 可设置 IsAdmin=true
    db.SaveChanges();
}

// 使用 DTO + 白名单
public class UpdateUserDTO {
    [Required] public string Name { get; set; }
    [Required] public string Email { get; set; }
    // 不包含 IsAdmin
}
[HttpPost]
public IActionResult Update(UpdateUserDTO dto) { ... }
```

## 错误/调试信息

```xml
<!-- Web.config: 详细错误-->
<customErrors mode="Off" />

<!-- 安全 -->
<customErrors mode="RemoteOnly" />
```

```csharp
// Core: 详细错误泄露
app.UseDeveloperExceptionPage();  // 生产禁用

// 安全
if (env.IsDevelopment()) {
    app.UseDeveloperExceptionPage();
} else {
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}
```

## 检测清单

- [ ] 中间件顺序: 认证→授权→路由→静态文件→错误处理
- [ ] [Authorize] vs [AllowAnonymous] 范围
- [ ] Model Binding 使用 DTO 而非实体
- [ ] customErrors mode != Off
- [ ] UseDeveloperExceptionPage 仅开发环境
- [ ] CORS 白名单而非通配符
- [ ] 无 Debug=true 残留
