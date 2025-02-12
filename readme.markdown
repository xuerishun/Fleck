Fleck
===

分支功能:
---
增加自动连接脚本功能，实现采用http访问时可自动连接本服务器，无须自己构建客户端脚本

分支示例
---
服务端代码详见示例,编译并运行后，在浏览器中输入【http://localhost:8000/auto 】即可访问本服务器
后缀/auto不可少
```cs
private string GetAutoScript(string host, string path,string encode)
{
   const string LocalHost = "127.0.0.1:8000";//脚本中的服务器地址
   //加载自动连接脚本模板，根据路径选择不同模板
   var fileClient = (string.IsNullOrEmpty(path) 
      || !path.EndsWith("/debug",StringComparison.OrdinalIgnoreCase))
       ? Path.Combine(nvr.ModelsDir, "client.html")
       : Path.Combine(nvr.ModelsDir, "debug.html");
   if (!File.Exists(fileClient))
       return string.Empty;
   var script = File.ReadAllText(fileClient, Encoding.ASCII);     
   if(!string.IsNullOrEmpty(script))
       script= script.Replace(LocalHost, host);//用host替换脚本中的服务器地址
   return script;
}
        
var server = new WebSocketServer("ws://0.0.0.0:8000");
 server.GetAutoScript += GetAutoScript;
...
```

[![Build status](https://ci.appveyor.com/api/projects/status/k0s8hq5y4emak5j3/branch/master?svg=true)](https://ci.appveyor.com/project/statianzo/fleck/branch/master) [![NuGet](https://img.shields.io/nuget/v/Fleck.svg)](https://www.nuget.org/packages/Fleck/)

Fleck is a WebSocket server implementation in C#. Branched from the
[Nugget][nugget] project, Fleck requires no inheritance, container, or
additional references.

Fleck has no dependency on `HttpListener` or `HTTP.sys` meaning that it
will work on Windows 7 and Server 2008 hosts. [WebSocket Remarks - Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/api/system.net.websockets.websocket?redirectedfrom=MSDN&view=netframework-4.5#remarks)

Example
---

The following is an example that will echo to a client.

```c#

var server = new WebSocketServer("ws://0.0.0.0:8181");
server.Start(socket =>
{
  socket.OnOpen = () => Console.WriteLine("Open!");
  socket.OnClose = () => Console.WriteLine("Close!");
  socket.OnMessage = message => socket.Send(message);
});
        
```

Supported WebSocket Versions
---

Fleck supports several WebSocket versions of modern web browsers

- Hixie-Draft-76/Hybi-00 (Safari 5, Chrome < 14, Firefox 4 (when enabled))
- Hybi-07 (Firefox 6)
- Hybi-10 (Chrome 14-16, Firefox 7)
- Hybi-13 (Chrome 17+, Firefox 11+, Safari 6+, Edge 13+(?))

Secure WebSockets (wss://)
---

Enabling secure connections requires two things: using the scheme `wss` instead
of `ws`, and pointing Fleck to an x509 certificate containing a public and
private key

```cs
var server = new WebSocketServer("wss://0.0.0.0:8431");
server.Certificate = new X509Certificate2("MyCert.pfx");
server.Start(socket =>
{
  //...use as normal
});
```

Having issues making a certificate? See this
[guide to creating an x509](https://github.com/statianzo/Fleck/issues/214#issuecomment-364413879)
by [@AdrianBathurst](https://github.com/AdrianBathurst)

SubProtocol Negotiation
---

To enable negotiation of subprotocols, specify the supported protocols on
the `WebSocketServer.SupportedSubProtocols` property. The negotiated
subprotocol will be available on the socket's `ConnectionInfo.NegotiatedSubProtocol`.

If no supported subprotocols are found on the client request (the
Sec-WebSocket-Protocol header), the connection will be closed.

```cs
var server = new WebSocketServer("ws://0.0.0.0:8181");
server.SupportedSubProtocols = new []{ "superchat", "chat" };
server.Start(socket =>
{
  //socket.ConnectionInfo.NegotiatedSubProtocol is populated
});
```

Custom Logging
---

Fleck can log into Log4Net or any other third party logging system. Just override the `FleckLog.LogAction` property with the desired behavior.

```cs
ILog logger = LogManager.GetLogger(typeof(FleckLog));

FleckLog.LogAction = (level, message, ex) => {
  switch(level) {
    case LogLevel.Debug:
      logger.Debug(message, ex);
      break;
    case LogLevel.Error:
      logger.Error(message, ex);
      break;
    case LogLevel.Warn:
      logger.Warn(message, ex);
      break;
    default:
      logger.Info(message, ex);
      break;
  }
};

```

Disable Nagle's Algorithm
---

Set `NoDelay` to `true` on the `WebSocketConnection.ListenerSocket`

```cs
var server = new WebSocketServer("ws://0.0.0.0:8181");
server.ListenerSocket.NoDelay = true;
server.Start(socket =>
{
  //Child connections will not use Nagle's Algorithm
});
```

Auto Restart After Listen Error
---

Set `RestartAfterListenError` to `true` on the `WebSocketConnection`

```cs
var server = new WebSocketServer("ws://0.0.0.0:8181");
server.RestartAfterListenError = true;
server.Start(socket =>
{
  //...use as normal
});
```

[nugget]: http://nugget.codeplex.com/ 
