# Java 反序列化

## 确认条件

用户输入的序列化数据被反序列化API处理。

## 危险API

```java
ObjectInputStream.readObject()                // 原生反序列化
XStream.fromXML()                             // XStream
JSON.parseObject(json, Object.class)          // Fastjson (autoType开启)
ObjectMapper.enableDefaultTyping()            // Jackson
Yaml.load(input)                              // SnakeYAML
HessianInput.readObject()                     // Hessian
Kryo.readClassAndObject(input)               // Kryo
```

## 检测

```java
// 原生反序列化
byte[] data = Base64.getDecoder().decode(request.getParameter("data"));
ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(data));
Object obj = ois.readObject();

// Cookie 反序列化
byte[] cookie = Base64.getDecoder().decode(request.getCookies());
ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(cookie));
Object obj = ois.readObject();

// Fastjson
JSON.parse(jsonString);                         // autoType关闭时相对安全
JSON.parseObject(jsonString, Feature.SupportAutoType);  // 开启后危险
JSON.parseObject(jsonString, Object.class);     // 危险

// Jackson
ObjectMapper mapper = new ObjectMapper();
mapper.enableDefaultTyping();                   // 开启 DefaultTyping
mapper.readValue(json, Object.class);

// XStream
XStream xstream = new XStream();
xstream.fromXML(xmlInput);

// SnakeYAML
Yaml yaml = new Yaml();
yaml.load(yamlInput);                           // 危险
yaml.loadAs(yamlInput, Object.class);

// Hessian (Dubbo)
HessianInput input = new HessianInput(is);
input.readObject();                             // 绕过 ObjectInputFilter
```

## Gadget 库速查

| 库 | Gadget Chain | 影响版本 |
|----|-------------|---------|
| Commons Collections 3 | CC1-CC7 | 3.0-3.2.1 |
| Commons Collections 4 | CC2, CC4 | 4.0 |
| Commons BeanUtils | CB1, CB2 | 1.8.3+ |
| JDK 内置 | Jdk7u21, Jdk8u20, URLDNS | JDK 7/8 |
| C3P0 | JNDI注入 | 0.9.5+ |
| Fastjson | TemplatesImpl, JNDI | 1.2.x |
| XStream | CVE-2021-39139~39154 | 1.4.x |
| Rome | ROME链 | 1.x |
| Hibernate | Hibernate 5/6 链 | 5.x/6.x |

## 修复

```java
// Jackson 白名单
ObjectMapper mapper = new ObjectMapper();
mapper.activateDefaultTyping(
    BasicPolymorphicTypeValidator.builder()
        .allowIfSubType("com.example.")
        .build()
);

// JDK 9+ Serialization Filter
ObjectInputStream ois = new ObjectInputStream(fis);
ois.setObjectInputFilter(filter);
// 全局: -Djdk.serialFilter=com.example.*;!*

// Kryo 白名单
Kryo kryo = new Kryo();
kryo.setRegistrationRequired(true);
kryo.register(MyClass.class);
```

## 检测命令

```bash
grep -rn "readObject\|readUnshared" --include="*.java"
grep -rn "ObjectInputStream" --include="*.java"
grep -rn "JSON\.parse\|parseObject" --include="*.java"
grep -rn "enableDefaultTyping\|DefaultTyping" --include="*.java"
grep -rn "fromXML\|XStream" --include="*.java"
grep -rn "Yaml\.load" --include="*.java"
grep -rn "HessianInput\|Hessian2Input" --include="*.java"
grep -rn "Kryo\|readClassAndObject" --include="*.java"

# 扫描 classpath 中的高风险库
find WEB-INF/lib -name "commons-collections*.jar" -o -name "fastjson*.jar" -o -name "xstream*.jar"
```
