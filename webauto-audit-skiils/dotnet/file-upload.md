# .NET 文件上传

## 检测

```csharp
// 未验证扩展名
var file = Request.Form.Files[0];
var path = Path.Combine(env.WebRootPath, "uploads", file.FileName);
using (var stream = new FileStream(path, FileMode.Create)) {
    file.CopyTo(stream);
}

// 仅验证 Content-Type
if (file.ContentType == "image/jpeg") {
    var path = Path.Combine("uploads", file.FileName);
    using (var stream = new FileStream(path, FileMode.Create)) {
        file.CopyTo(stream);  // Content-Type 可伪造
    }
}

// 路径遍历 — FileName 包含 ../
var fileName = Path.GetFileName(file.FileName);  // 去除路径
var path = Path.Combine("uploads", fileName);    // 仍需验证扩展名
```

## 修复

```csharp
string[] allowed = { ".jpg", ".png", ".gif", ".pdf" };
var ext = Path.GetExtension(file.FileName).ToLower();
if (!allowed.Contains(ext)) throw new InvalidFileException();

var safeName = Guid.NewGuid().ToString() + ext;
var fullPath = Path.GetFullPath(Path.Combine(uploadDir, safeName));

// 确保不超出上传目录
if (!fullPath.StartsWith(Path.GetFullPath(uploadDir),
    StringComparison.OrdinalIgnoreCase)) {
    throw new SecurityException();
}

using (var stream = new FileStream(fullPath, FileMode.Create)) {
    file.CopyTo(stream);
}
```

## 绕过技术

```
扩展名: .aspx → .ashx .asmx .svc .asax .ascx .asp
IIS 6: .asp;.jpg / .asp/
IIS 7.5: .jpg/.php (PHP映射时)
NTFS: aspx. (尾随空格) / aspx::$DATA
```

## 检测命令

```bash
grep -rn "\.FileName\|\.ContentType" --include="*.cs"
grep -rn "FileStream\|CopyTo\|SaveAs" --include="*.cs"
```
