Socklient
=========
A Socks5 Client written in C# that implements RFC 1928 &amp; 1929.

Features
========
Support TCP (Connect) & UDP Associate

NuGet
=====
https://www.nuget.org/packages/Socklient

Example
=======

TCP Relay:
```csharp
// Find a socks5 server by yourself, google search is a easy way.
var socks5ServerHostNameOrAddress = "xxx.xxx.xxx.xxx"; // or a hostname
var serverPort = 0; // assign service port you found of socks5 server

var targetHost = "httpbin.org"; // a target host you want to visit
var targetPort = 80; // target port 

// Tcp Example
try {
    // init with socks5 server information
    var socklient = new Socklient(socks5ServerHostNameOrAddress, serverPort);
    // you can pass a NetworkCredential contains username/password if socks server need a basic authencation
    //var socklient = new Socklient(socks5ServerHostNameOrAddress, serverPort, new System.Net.NetworkCredential("user", "pwd"));

    // tell socks server connect to target server
    socklient.Connect(targetHost, targetPort);

    Console.WriteLine($"TCP: Supported, {socklient.BoundType} {(socklient.BoundType == AddressType.Domain ? socklient.BoundDomain : socklient.BoundAddress.ToString())}:{socklient.BoundPort}");

    // write a http request or others you want
    socklient.Write("GET http://httpbin.org/ip HTTP/1.1\r\nHost: httpbin.org\r\n\r\n");

    // receive reply data
    var buffer = new byte[1024];
    socklient.Read(buffer, 0, buffer.Length);

    Console.WriteLine($"Receive: {Environment.NewLine}{Encoding.UTF8.GetString(buffer)}");

    // close connection
    socklient.Close();

} catch (Exception e) {
    Console.WriteLine($"TCP: {e.Message}");
}
```

UDP Relay:
```csharp
// Udp Example
try {
    var socklient = new Socklient(socks5ServerHostNameOrAddress, serverPort);
    // you can pass a NetworkCredential contains username/password if socks server need a basic authencation
    //var socklient = new Socklient(socks5ServerHostNameOrAddress, serverPort, new System.Net.NetworkCredential("user", "pwd"));

    // find some udp service by yourself, for example: UDP echo, DNS, etc...
    targetHost = "anyhost.provide.udpservice";
    // assign a service port
    targetPort = 0;

    // tell socks server do udp relay to targethost:targetport.
    // the last argument srcPort means socklient use localport 10000 to communicate with socks server, you can assign 0 if you dont care
    socklient.UdpAssociate(targetHost, targetPort, 10000);
    // set timeout for receive
    socklient.UDP.Client.ReceiveTimeout = 5000;

    Console.WriteLine($"UDP: Supported, {socklient.BoundType} {(socklient.BoundType == AddressType.Domain ? socklient.BoundDomain : socklient.BoundAddress.ToString())}:{socklient.BoundPort}");

    // send data via socks5 server
    socklient.Send(new byte[] { 1, 2, 3, 4 });

    // also, you can send to a different host:port 
    // for all kinds of Nat (Port-Restricted cone NAT, Address-Restricted cone NAT, Full cone NAT, etc...)
    // socklient.Send(new byte[] { 1, 2, 3, 4 }, "anotherhost.what.you.want", 5555);

    // receive data and remote host information that sent back data to socks5 server
    var received = BitConverter.ToString(socklient.Receive(out var remoteHost, out var remoteAddress, out var remotePort));
    Console.WriteLine($"Receive from {remoteHost}:{remotePort} {received}" );

    socklient.Close();

} catch (Exception e) {
    Console.WriteLine($"UDP: {e.Message}");
}
```
