## TCPCopy_usage ##

### 背景

 * 线上机器: 172.20.19.187
 * 测试机器：172.20.19.197
 * 测试机器的nginx和lvs地址为:172.20.19.196
 * 辅助测试机器: 172.20.19.172

需要不存的一个网段, 此处使用 172.20.190.0/24

### 配置说明

### 测试服务器(Target-Server)配置

```
Set route commands appropriately to route response packets to the assistant server

  For example:

     Assume 61.135.233.161 is the IP address of the assistant server. We set the
     following route command to route all responses to the 62.135.200.x's clients
     to the assistant server.

       route add -net 62.135.200.0 netmask 255.255.255.0 gw 61.135.233.161
```
执行以下命令

```
 route add -net  172.20.190.0 netmask 255.255.255.0 gw 172.20.19.172
```

### 辅助测试服务器(Target-Server)配置

```
./intercept -F <filter> -i <device,>

   Note that the filter format is the same as the pcap filter.
   For example:

      ./intercept -i eth0 -F 'tcp and src port 8080' -d

      intercept will capture response packets of the TCP based application which listens
      on port 8080 from device eth0
```
拦截REST请求, 执行以下命令:

```
./intercept -i eth0 -F 'tcp and src port 80' -d
```

拦截长连接, 执行以下命令:

```
./intercept -i eth0 -F 'tcp and src port 5227' -d
```

```
 ./intercept -i eth0 -F 'tcp and src port 5223' -d
```

连接http-bind,执行以下命令

```
./intercept -i eth0 -F 'tcp and src port 7070' -d
```

### 线上服务器配置

```
./tcpcopy -x localServerPort-targetServerIP:targetServerPort -s <intercept server,>
 [-c <ip range,>]

 For example(assume 61.135.233.160 is the IP address of the target server):

   ./tcpcopy -x 80-61.135.233.160:8080 -s 61.135.233.161 -c 62.135.200.x

   tcpcopy would capture port '80' packets on current server, change client IP address
   to one of 62.135.200.x series, send these packets to the target port '8080' of the
   target server '61.135.233.160', and connect 61.135.233.161 for asking intercept to
   pass response packets to it.

   Although "-c" parameter is optional, it is set here in order to simplify route
   commands.
```

拦截REST请求, 执行以下命令:

```
 ./tcpcopy -x 9090-172.20.19.197:80 -s 172.20.19.172 -c 172.20.190.x
```

拦截长连接, 执行以下命令:

```
./tcpcopy -x 5227-172.20.19.197:5227 -s 172.20.19.172 -c 172.20.190.x
```

```
./tcpcopy -x 5223-172.20.19.197:5223 -s 172.20.19.172 -c 172.20.190.x
```

连接http-bind,执行以下命令

```
./tcpcopy -x 7070-172.20.19.197:7070 -s 172.20.19.172 -c 172.20.190.x
```

