# .NET 反序列化

## 危险API

```csharp
BinaryFormatter.Deserialize()
JsonConvert.DeserializeObject(json, settings)  // TypeNameHandling
SoapFormatter.Deserialize()
LosFormatter.Deserialize()
JavaScriptSerializer.Deserialize()             // SimpleTypeResolver
ObjectStateFormatter.Deserialize()              // ViewState底层
NetDataContractSerializer.Deserialize()
FastJson.Deserialize()
```

## 检测

```csharp
// BinaryFormatter
byte[] data = Convert.FromBase64String(userInput);
BinaryFormatter formatter = new BinaryFormatter();
MemoryStream ms = new MemoryStream(data);
formatter.Deserialize(ms);

// SoapFormatter
SoapFormatter formatter = new SoapFormatter();
formatter.Deserialize(new MemoryStream(data));

// Json.NET TypeNameHandling
var settings = new JsonSerializerSettings {
    TypeNameHandling = TypeNameHandling.All       // 危险
};
JsonConvert.DeserializeObject(json, settings);

// TypeNameHandling.Auto 也不安全
TypeNameHandling = TypeNameHandling.Auto          // 仍然危险

// JavaScriptSerializer + SimpleTypeResolver
var serializer = new JavaScriptSerializer(new SimpleTypeResolver());
serializer.Deserialize<User>(json);               // 危险

// LosFormatter (ViewState)
LosFormatter formatter = new LosFormatter();
formatter.Deserialize(input);                     // 旧版ViewState
```

## 修复

```csharp
// 禁止 TypeNameHandling
TypeNameHandling = TypeNameHandling.None

// SerializationBinder 白名单
var settings = new JsonSerializerSettings {
    TypeNameHandling = TypeNameHandling.Auto,
    SerializationBinder = new CustomSafeBinder()
};

// .NET 5+: BinaryFormatter 默认不可用
// 使用 System.Text.Json 替代 Json.NET
System.Text.Json.JsonSerializer.Deserialize<User>(json);
```

## 利用工具

```bash
ysoserial.net -g ObjectDataProvider -c "cmd /c calc.exe" -f BinaryFormatter
```

## 检测命令

```bash
grep -rn "BinaryFormatter\|SoapFormatter\|LosFormatter" --include="*.cs"
grep -rn "TypeNameHandling" --include="*.cs" | grep -v "None"
grep -rn "SimpleTypeResolver\|JavaScriptSerializer" --include="*.cs"
```
