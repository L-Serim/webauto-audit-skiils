# .NET 安全审计总览

## 指纹识别

```yaml
Headers: ["X-AspNet-Version:", "X-Powered-By: ASP.NET", "ASPSESSIONID="]
Extensions: [.aspx, .asmx, .ashx, .asp, .svc, .ascx, .asax]
Key files: [Web.config, *.csproj, Global.asax, appsettings.json, packages.config]
Error signatures: ["Server Error in '/' Application", "Runtime Error", "ASP.NET"]
```

## 框架速查

| 框架 | 识别特征 | 招牌漏洞 |
|------|---------|---------|
| ASP.NET MVC | `Controllers/`, `Views/`, `@Html` | CSRF缺失, 反序列化 |
| ASP.NET WebForms | `.aspx`, `ViewState`, `PostBack` | ViewState篡改, 反序列化 |
| ASP.NET Core | `Startup.cs`, `appsettings.json` | Middleware顺序, JWT弱密钥 |
| Blazor | `_Imports.razor`, `App.razor` | SignalR认证 |

## 危险API

```csharp
// 命令执行
Process.Start("cmd.exe", "/c " + input)

// 反序列化
BinaryFormatter.Deserialize()
JsonConvert.DeserializeObject(settings)  // TypeNameHandling!=None
SoapFormatter.Deserialize()
LosFormatter.Deserialize()

// SSRF
WebClient.DownloadString()
HttpWebRequest.Create()
HttpClient.GetStringAsync()

// SQL注入
SqlCommand(query_string, conn)          // 非参数化
db.Database.SqlQuery<User>(rawSql)

// XXE
XmlDocument.LoadXml(input)              // XmlResolver != null
```

## 审计优先级

```
P0 (必须审): 反序列化 / ViewState篡改 / MachineKey泄露
P1 (高优先): SQL注入 / SSTI / 认证绕过
P2 (标准):   SSRF / XXE / 命令注入
P3 (补充):   XSS / CSRF / 路径遍历 / 信息泄露
```

## 安全配置基线

```xml
<!-- Web.config -->
<customErrors mode="RemoteOnly" />
<httpCookies httpOnlyCookies="true" requireSSL="true" />
<machineKey validation="HMACSHA256" decryption="AES" />

<!-- 请求验证（默认开启） -->
<pages validateRequest="true" />
```
