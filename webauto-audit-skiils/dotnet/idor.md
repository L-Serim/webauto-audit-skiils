# .NET IDOR 越权

## 检测

```csharp
// MVC: 无归属校验
public IActionResult OrderDetail(int id) {
    var order = db.Orders.Find(id);   // 可查看任何人的订单
    return View(order);
}

// API: 修改他人资源
[HttpPut]
public IActionResult UpdateUser(int id, [FromBody] User user) {
    var entity = db.Users.Find(id);
    entity.Email = user.Email;        // 可修改任何人的信息
    db.SaveChanges();
}

// 批量操作
[HttpPost]
public IActionResult BatchDelete([FromBody] List<int> ids) {
    db.Orders.RemoveRange(db.Orders.Where(o => ids.Contains(o.Id)));
    db.SaveChanges();
}

// 文件下载
public IActionResult Download(string filename) {
    var path = Path.Combine(_webHostEnvironment.WebRootPath, "invoices", filename);
    return File(System.IO.File.ReadAllBytes(path), "application/pdf");
}
```

## 修复

```csharp
// 资源归属校验
public IActionResult OrderDetail(int id) {
    int userId = GetCurrentUserId();
    var order = db.Orders.FirstOrDefault(o => o.Id == id && o.UserId == userId);
    if (order == null) return Forbid();
    return View(order);
}

// 只能修改自己
[HttpPut]
public IActionResult UpdateProfile([FromBody] UpdateProfileDTO dto) {
    int userId = GetCurrentUserId();
    var user = db.Users.Find(userId);
    user.Email = dto.Email;
    db.SaveChanges();
}

// 批量归属校验
[HttpPost]
public IActionResult BatchDelete([FromBody] List<int> ids) {
    int userId = GetCurrentUserId();
    var orders = db.Orders.Where(o => ids.Contains(o.Id) && o.UserId == userId);
    db.Orders.RemoveRange(orders);
    db.SaveChanges();
}
```

## 检测命令

```bash
grep -rn "\.Find(\|\.FindById\|FirstOrDefault.*Id" --include="*.cs" | grep -v "UserId\|userId"
grep -rn "\[HttpGet\|\[HttpPut\|\[HttpDelete\]" --include="*.cs"
```
