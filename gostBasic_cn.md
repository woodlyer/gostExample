# gost基本知识
----------------------------------------------


## gost 默认是一个socks5代理服务
```
gost -L :8080
gost -L admin:123456@:1080
gost -L :8080?auth=YWRtaW46MTIzNDU2
```

auth的值为 user:pass的base64编码值， 生成方法：
```
echo -n 'user:pass' | base64
```
也可以通过secrets参数来设定多组认证信息：
```
gost -L=:8080?secrets=secrets.txt
```
secrets.txt文件格式为按行分割的认证信息，每一行认证信息为用空格分割的user-pass对，以 # 开始的行为注释行。

```
# period for live reloading
reload      10s

# username password

admin           #123456
test\user001    123456
test.user@002   12345678
```
reload - 此配置文件支持热更新。此选项用来指定文件检查周期，默认不开启热更新。  
注意： 当secrets参数用于shadowsocks协议时，仅会使用第一项作为认证信息。  
所有的认证信息都是用于协议层(Protocol)。  


## -F转发

-F 用来将socks5请求转发出去。
```
gost -L :8080 -F 192.168.1.1:8081
```
当GOST去连接一个代理节点时，会先按照传输层设置的传输类型进行交互，当传输层建立以后，再按照协议层设置的协议类型进行交互。  

- 协议类型(Protocols)
支持的协议类型有：
```
http - HTTP
http2 - HTTP2
socks4 - SOCKS4 (2.4+)
socks4a - SOCKS4A (2.4+)
socks5 - SOCKS5
ss - Shadowsocks
ss2 - Shadowsocks with AEAD support (2.8+)
sni - SNI (2.5+)
forward - Forward
relay - TCP/UDP relay (2.11+)
```
- 传输类型(Transports)
支持的传输类型有：
```
tcp - 原始TCP
tls - TLS
mtls - Multiplex TLS，在TLS上增加多路复用功能 (2.5+)
ws - Websocket
mws - Multiplex Websocket，在Websocket上增加多路复用功能 (2.5+)
wss - Websocket Secure，基于TLS加密的Websocket
mwss - Multiplex Websocket Secure，在基于TLS加密的Websocket上增加多路复用功能 (2.5+)
kcp - KCP (2.3+)
quic - QUIC (2.4+)
ssh - SSH (2.4+)
h2 - HTTP2 (2.4+)
h2c - HTTP2 Cleartext (2.4+)
obfs4 - OBFS4 (2.4+)
ohttp - HTTP Obfuscation (2.7+)
otls - TLS Obfuscation (2.11+)
```

传输层默认为是原始TCP类型。  
对于-L参数，协议层未指定时，默认为是HTTP & SOCKS5，对于-F参数，协议层未指定时，默认为是HTTP类型。  

### 仅指定协议类型
当仅指定协议类型时，传输层默认为原始TCP类型。
```
gost -L http://:8080 -F socks5://:1080
```
### 仅指定传输类型
当仅指定传输类型时，对于-L参数，协议类型默认为HTTP+SOCKS5。对于-F参数，协议层默认为是HTTP类型。
```
gost -L tls://:443 -F ws://:1443
```



## 端口转发

- TCP本地端口转发
tcp://:port_A/ip_B:port_B
将本地的TCP端口A映射到指定的目标TCP端口B，连接至端口A，实际上是连接到远处的端口B。
```
./gost -L :1080
./gost -L=tcp://:2222/192.168.1.1:22  [-F=...] 
```
将本机TCP端口2222上的数据(通过代理链)转发到（服务端看到的）192.168.1.1:22上。  
当你连接时 127.0.0.1:2222时，其实是连接到服务端那边的 192.168.1.1:22。

- TCP远程端口转发
rtcp://:port_A/ip_B:port_B
将目标TCP端口B映射到远程TCP端口A，连接到到端口A，即连接到端口B。 
```
./gost -L :1080       # server on 172.24.10.1
# client
./gost -L=rtcp://:2222/192.168.1.1:22  [-F=...]  -F=socks5://172.24.10.1:1080
```
当你连接172.24.10.1:2222时，最终是连接到本地这边的看到的的192.168.1.1:22上，有点像反向代理  
[ssh root@172.24.10.1 -p 2222]    ->   [client 端的 192.168.1.1:22]




## SOCKS5多路复用模式
在2.5版本中，SOCKS5的BIND方法增加了对多路复用的支持，远程端口转发可以利用这个特性提高传输效率。
```
gost -L rtcp://:8080/192.168.1.1:80 -F socks5://:1080?mbind=true
```
客户端通过mbind=true参数开启SOCKS5的BIND多路复用模式。




## Relay协议
Relay是GOST(2.11+)支持的一种协议类型(Protocol)。  
Relay协议本身不具备加密功能，如果需要对数据进行加密传输，可以配合加密隧道使用。  

Relay协议同时具有代理和转发功能，可同时处理TCP和UDP的数据，并支持用户认证。  
参数说明
nodelay - true/false (默认值为false)。默认情况下relay协议会等待客户端的请求数据，当收到请求数据后会把协议头部信息与请求数据一起发给服务端。
当此参数设为true后，协议头部信息会立即发给服务端，不再等待客户端的请求。

- 代理功能  
Relay协议可以像HTTP/SOCKS5一样用作代理协议。
```
# 服务端
gost -L relay+tls://username:password@:12345
#客户端
gost -L :8080 -F relay+tls://username:password@:12345?nodelay=false
```

- 转发功能  
Relay转发有两种模式，一种是配合端口转发使用，另一种是配合转发隧道使用。两种模式均可以同时转发TCP和UDP数据。  
端口转发:  
```
# 服务端
gost -L relay://:12345
# 客户端
gost -L udp://:1053/:53    -L tcp://:1053/:53  -F relay://:12345
```

转发隧道:  
```
# 服务端
gost -L relay://:12345/:53
# 客户端
gost -L tcp://:1053 -F relay://:12345
```




















