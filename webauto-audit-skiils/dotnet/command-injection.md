# .NET 命令注入

## 检测

```csharp
// Process.Start 拼接
Process.Start("cmd.exe", "/c ping " + Request["ip"]);

Process p = new Process();
p.StartInfo.FileName = "cmd.exe";
p.StartInfo.Arguments = "/c " + Request["ip"];
p.Start();

// ProcessStartInfo
ProcessStartInfo psi = new ProcessStartInfo();
psi.FileName = "cmd.exe";
psi.Arguments = "/c " + Request["ip"];
psi.UseShellExecute = true;  // shell 解析 → 更危险
Process.Start(psi);

// RedirectStandardOutput (输出重定向)
Process proc = new Process();
proc.StartInfo.FileName = "cmd.exe";
proc.StartInfo.Arguments = "/c " + Request["ip"];
proc.StartInfo.RedirectStandardOutput = true;
proc.Start();
string output = proc.StandardOutput.ReadToEnd();
```

## 修复

```csharp
// 参数分离
Process.Start("ping", "-n 4 " + ip);

// 白名单
string[] allowed = { "ping", "tracert", "nslookup" };
if (!allowed.Contains(cmd)) throw new Exception();

// UseShellExecute = false
psi.UseShellExecute = false;  // 限制 shell 特性
```

## 检测命令

```bash
grep -rn "Process\.Start\|ProcessStartInfo" --include="*.cs" | grep '+'
grep -rn "UseShellExecute.*true\|ShellExecute" --include="*.cs"
```
