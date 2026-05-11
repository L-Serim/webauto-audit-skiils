# .NET ViewState 安全

## 确认条件

ViewState MAC 验证关闭或 MachineKey 泄露。

## ViewState 基础

```
ViewState: 页面状态持久化机制，Base64编码存储
MAC: Message Authentication Code，防篡改
__VIEWSTATE: 隐藏字段，存储序列化后的页面状态
```

## 检测

```xml
<!-- Web.config: 关闭 MAC 验证 -->
<pages enableViewStateMac="false" />

<!-- MachineKey 硬编码 -->
<machineKey 
    validationKey="AAAA...BBBB"
    decryptionKey="CCCC...DDDD"
    validation="SHA1"
    decryption="AES" />
```

```csharp
// 代码中关闭 MAC
Page.EnableViewStateMac = false;

// 关闭请求验证
<% @Page ValidateRequest="false" %>
[ValidateInput(false)]

// 允许 HTML
[AllowHtml]
public string Content { get; set; }
```

## MachineKey 泄露利用

```
已知 MachineKey → 伪造 ViewState:
1. 使用 ysoserial.net 生成恶意序列化 Payload
2. 用已知 validationKey/decryptionKey 加密签名
3. 将加密后的 ViewState 作为 __VIEWSTATE 提交
4. 服务端反序列化 → 执行代码
```

## 修复

```xml
<!-- 不显式配置 MachineKey（使用自动生成的） -->
<!-- .NET 4.5.2+: enableViewStateMac 默认 true 且不可关闭 -->

<!-- 请求验证保持开启 -->
<pages validateRequest="true" />
```

## 检测命令

```bash
grep -rn "enableViewStateMac\|ViewStateMac" --include="*.config" --include="*.aspx"
grep -rn "machineKey\|validationKey\|decryptionKey" --include="*.config"
grep -rn "ValidateRequest\|ValidateInput\|AllowHtml" --include="*.cs" --include="*.aspx"
```
