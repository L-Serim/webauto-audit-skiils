# ASP.NET WebForms 安全审计

## 识别特征

```
文件: .aspx, .ascx, .master, .asmx
目录: App_Code/, App_Data/, App_Theme/
```

## ViewState 安全

```xml
<!-- 漏洞: 关闭 MAC 验证 -->
<pages enableViewStateMac="false" />

<!-- 漏洞: MachineKey 硬编码 -->
<machineKey validationKey="..." decryptionKey="..." validation="SHA1" />
```

## 输入验证

```asp
<!-- 漏洞: 关闭请求验证 -->
<% @Page ValidateRequest="false" %>

<!-- 漏洞: 控件关闭验证 -->
<asp:TextBox ValidateRequestMode="Disabled" />

<!-- 安全: 保持默认开启 -->
```

## 文件暴露

```
常见敏感文件:
/Web.config → 数据库连接字符串/密钥
/App_Data/ → 数据库文件
/Trace.axd → 请求跟踪信息
/elmah.axd → 错误日志
```

## WebMethod / ASMX

```csharp
// 未认证的 WebMethod
[WebMethod]
[ScriptMethod(ResponseFormat = ResponseFormat.Json)]
public string GetUserData(int userId) {
    // 无认证检查
    return JsonConvert.SerializeObject(db.Users.Find(userId));
}

// 安全
[WebMethod]
public string GetUserData(int userId) {
    if (!HttpContext.Current.User.Identity.IsAuthenticated)
        throw new UnauthorizedAccessException();
    // 校验归属
}
```

## 检测命令

```bash
grep -rn "enableViewStateMac\|ViewStateMac" --include="*.config" --include="*.aspx"
grep -rn "ValidateRequest.*false\|ValidateRequestMode.*Disabled" --include="*.aspx"
grep -rn "\[WebMethod\]" --include="*.asmx" --include="*.cs"
```
