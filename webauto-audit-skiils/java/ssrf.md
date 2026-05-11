# Java SSRF

## 危险API

```java
URL.openConnection() / URL.openStream()
HttpURLConnection
RestTemplate.getForObject() / exchange()
WebClient (Spring 5+)
HttpClient.execute() (Apache)
OkHttpClient.newCall()
```

## 检测

```java
// URL.openStream
String url = request.getParameter("url");
URL u = new URL(url);
InputStream in = u.openStream();

// HttpURLConnection
URL u = new URL(request.getParameter("url"));
HttpURLConnection conn = (HttpURLConnection) u.openConnection();
conn.setRequestMethod("GET");
BufferedReader reader = new BufferedReader(new InputStreamReader(conn.getInputStream()));

// RestTemplate
String url = request.getParameter("url");
RestTemplate rt = new RestTemplate();
String result = rt.getForObject(url, String.class);

// Spring WebClient
WebClient client = WebClient.create();
String result = client.get()
    .uri(request.getParameter("url"))
    .retrieve()
    .bodyToMono(String.class)
    .block();

// Apache HttpClient
CloseableHttpClient client = HttpClients.createDefault();
HttpGet httpGet = new HttpGet(request.getParameter("url"));
CloseableHttpResponse response = client.execute(httpGet);

// XXE + SSRF (XML 解析器)
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
Document doc = builder.parse(new URL(request.getParameter("url")).openStream());
```

## 修复

```java
// URL 白名单
URI uri = new URI(url);
String host = uri.getHost();
if (!host.endsWith(".trusted.com") && !host.equals("api.internal.com")) {
    throw new SecurityException("Blocked");
}

// 协议白名单
if (!uri.getScheme().equals("http") && !uri.getScheme().equals("https")) {
    throw new SecurityException("Only HTTP/HTTPS allowed");
}

// RestTemplate 限制
RestTemplate rt = new RestTemplate();
// 自定义 ClientHttpRequestFactory 添加白名单校验

// DNS 重绑定防护
URL u = new URL(url);
InetAddress address = InetAddress.getByName(u.getHost());
if (!isAddressAllowed(address.getHostAddress())) {
    throw new SecurityException("Address not allowed");
}
```

## 利用目标

```
http://169.254.169.254/latest/meta-data/          # AWS
http://100.100.100.200/latest/meta-data/           # 阿里云
http://metadata.google.internal/                    # GCP
http://127.0.0.1:9200/                             # ES
http://127.0.0.1:8080/actuator/env                 # Spring Actuator
http://127.0.0.1:6379/                              # Redis
```

## 检测命令

```bash
grep -rn "openConnection\|openStream" --include="*.java"
grep -rn "RestTemplate\|WebClient\|HttpClient" --include="*.java" | grep "getFor\|exchange\|execute"
grep -rn "new URL(" --include="*.java" | grep -v "new URL\[\]"
```
