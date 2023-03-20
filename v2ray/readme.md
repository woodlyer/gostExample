
gost build a tunnel to transport vless-tcp.   
xray and v2fly are all compatiable with v2ray.  


```
[xray client] ------tcp--->  [gost client:1234]------ kcp/tls/quic ----------> [gost server:9000] ----tcp--------->[xray server:1234]
```

xray itself contains tls encryption.  So, this tunnel is like "Take off your pants and fart". 
https://github.com/XTLS/Xray-examples/tree/main/VLESS-TCP-TLS  






On server: run xray and gost.
```
# xray download 
wget https://github.com/XTLS/Xray-core/releases/download/v1.8.0/Xray-linux-64.zip

# unzip
unzip Xray-linux-64.zip

# run xray , listen on :1234
# you may want to modify the uuid in this file
./xray run -c server-vless.json


# run gost on server
./gost  -L relay+tls://:9000

# or use kcp,quic, ws...  as you need.
./gost  -L relay+kcp://:9000
```




On client: run gost to build the tunnel   
```
# gost client (windows)
.\gost.exe -L=tcp://127.0.0.1:1234/127.0.0.1:1234   -F relay+tls://server_ip:9000

```




vless test:
```
# vless para:
vless://3384d694-7af5-47f4-b9ee-a878ccd55047@localhost:1234?security=none&sni=localhost


# use xray to connect vless or other proxy tools. 
.\xray.exe run -c client-vless.json

```
