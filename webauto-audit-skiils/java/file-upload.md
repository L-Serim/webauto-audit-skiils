# Java 文件上传

## 检测

```java
// Servlet — 未验证扩展名
Part filePart = request.getPart("file");
String fileName = filePart.getSubmittedFileName();
filePart.write("/var/webapp/uploads/" + fileName);

// Spring MultipartFile
@PostMapping("/upload")
public String upload(@RequestParam("file") MultipartFile file) {
    String originalName = file.getOriginalFilename();
    file.transferTo(new File("/uploads/" + originalName));
}

// 仅验证 Content-Type
if (file.getContentType().equals("image/jpeg")) {
    file.transferTo(new File("/uploads/" + file.getOriginalFilename()));
}

// 路径遍历 — 文件名包含 ../
String fileName = file.getOriginalFilename();
file.transferTo(new File("/uploads/" + fileName));
// ../../../webapps/ROOT/shell.jsp
```

## 修复

```java
@PostMapping("/upload")
public String upload(@RequestParam("file") MultipartFile file) {
    String ext = StringUtils.getFilenameExtension(file.getOriginalFilename());
    if (!Arrays.asList("jpg", "png", "gif", "pdf").contains(ext.toLowerCase())) {
        throw new InvalidFileException();
    }
    String safeName = UUID.randomUUID().toString() + "." + ext.toLowerCase();
    File dest = new File("/var/www/uploads/" + safeName);
    // 确保规范化路径不超出上传目录
    if (!dest.getCanonicalPath().startsWith("/var/www/uploads/")) {
        throw new SecurityException();
    }
    file.transferTo(dest);
}
```

## 绕过技术

```
扩展名: .jsp → .jspx .jsw .jsv .jspf
Tomcat: .jsp;.jpg (IIS+Tomcat)
空字节: shell.jsp%00.jpg (旧版本)
大小写: shell.Jsp
```

## 检测命令

```bash
grep -rn "getOriginalFilename\|getSubmittedFileName" --include="*.java"
grep -rn "transferTo\|filePart\.write" --include="*.java"
grep -rn "MultipartFile" --include="*.java"
```
