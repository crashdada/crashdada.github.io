# sometips

esxi备份

```shell
cd C:\Program Files (x86)\VMware\VMware Workstation\OVFTool

.\ovftool.exe vi://root@192.168.31.254/win10 D:\esxi-back

.\ovftool.exe -ds=datastore1 -dm=thin -n=ubuntu-s1 "D:\esxi-back\win10\win10.ovf" vi://root@192.168.31.254

```



```
PermitRootLogin yes
PasswordAuthentication yes
ClientAliveInterval 30
MaxSessions 10
```

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

```
export Namesilo_Key="b980f7ec95b1981bb6c32b"

export Namesilo_Key="bd13c169162f37a55788251"
```

vs-home
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
d19d0796fb799c05ccce690aae1d4020

docs
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



```
/usr/local/acme.sh/acme.sh --installcert -d xxx.jaymz.top \
--key-file /usr/local/nginx/conf/ssl/xxx.jaymz.top/xxx.jaymz.top.key \
--fullchain-file /usr/local/nginx/conf/ssl/xxx.jaymz.top/fullchain.cer \
--reloadcmd     "nginx -s reload"
```

```
/usr/local/acme.sh/acme.sh --force --issue --server letsencrypt --dns dns_namesilo --dnssleep 20 -d jaymz.top  -d *.jaymz.top
```

```
/usr/local/acme.sh/acme.sh --force --issue --server letsencrypt --dns dns_namesilo --dnssleep 200 -d zzz.jaymz.top
```

