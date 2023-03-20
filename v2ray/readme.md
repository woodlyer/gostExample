

On server: run xray and gost.
```
# xray download 
wget https://github.com/XTLS/Xray-core/releases/download/v1.8.0/Xray-linux-64.zip

# unzip
unzip Xray-linux-64.zip

# run xray , listen on :1234
./xray run -c server-vless.json


# run gost on server
./gost  -L relay+tls://:9000

# or use kcp,quic, ws...  as you need.
./gost  -L relay+kcp://:9000
```




On client: run gost to do the tunnel   
```
# gost client (windows)
.\gost.exe -L=tcp://127.0.0.1:1234/127.0.0.1:1234   -F relay+tls://server_ip:9000

```




vless para:
```
# vless para:
vless://3384d694-7af5-47f4-b9ee-a878ccd55047@localhost:1234?security=none&sni=localhost


# use xray to connect vless or other proxy tools. 
.\xray.exe run -c client-vless.json

```
