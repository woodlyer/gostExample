# How to install gost as service

copy gost binary to /usr/local/bin/gost  
copy gost.service to systemd didrectory.  
start the gost service.  
```
cp  gost /usr/local/bin/gost
cp  gost.service  /usr/lib/systemd/system/gost.service
   
systemctl reload
systemctl start gost.service
```

more help to see    
https://github.com/BlueSkyXN/EasyGost


