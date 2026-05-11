# .NET SSRF

## 危险API

```csharp
WebClient.DownloadString(url) / DownloadData(url)
HttpWebRequest.Create(url)
HttpClient.GetStringAsync(url) / GetAsync(url)
WebRequest.Create(url)
RestClient (Refit/RestSharp)
```

## 检测

```csharp
// WebClient
string url = Request["url"];
using (WebClient client = new WebClient()) {
    string result = client.DownloadString(url);
}

// HttpWebRequest
string url = Request["url"];
HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
using (HttpWebResponse response = (HttpWebResponse)request.GetResponse()) {
    using (StreamReader reader = new StreamReader(response.GetResponseStream())) {
        string result = reader.ReadToEnd();
    }
}

// HttpClient
string url = Request["url"];
using (HttpClient client = new HttpClient()) {
    string result = await client.GetStringAsync(url);
}

// RestSharp
var client = new RestClient();
var request = new RestRequest(Request["url"]);
var response = client.Execute(request);
```

## 修复

```csharp
// URL 白名单
Uri uri = new Uri(url);
string[] allowed = { "api.internal.com", "cdn.example.com" };
if (!allowed.Contains(uri.Host)) throw new SecurityException();

// 协议白名单
if (uri.Scheme != "http" && uri.Scheme != "https") throw new SecurityException();

// DNS 重绑定防护
IPAddress[] addresses = Dns.GetHostAddresses(uri.Host);
foreach (var addr in addresses) {
    if (IsPrivateAddress(addr)) throw new SecurityException();
}
```

## 利用目标

```
http://169.254.169.254/latest/meta-data/        # AWS
http://100.100.100.200/latest/meta-data/         # 阿里云
http://metadata.google.internal/                  # GCP
file:///C:/Windows/win.ini                       # 文件读取
gopher://127.0.0.1:6379/_INFO                     # Redis
```

## 检测命令

```bash
grep -rn "WebClient\|HttpWebRequest\|WebRequest\.Create\|HttpClient" --include="*.cs" | grep "Request\["
```
