# nfs

## nfs

sudo apt\-get install nfs\-kernel\-server \# 安装 NFS服务器端

sudo apt\-get install nfs\-common \# 安装 NFS客户端

sudo /etc/init.d/nfs\-kernel\-server start

sudo /etc/init.d/nfs\-kernel\-server restart

exportfs \-arv

tcp:111 2049

udp:111 4046

showmount \-e 8.142.177.235

showmount.exe \-e 8.142.177.235

nfs.client.mount.options = vers=4

/data/nfs \*\(rw,sync,no\_root\_squash,no\_subtree\_check,insecure\)

mount \-t nfs \-o resvport 8.142.177.235:/data/nfs /Users/mayanyan/data/test\_nfs

win10

打开或关闭Windows功能选项\-\> NFS服务

## smaba

修改配置文件：

```
[global]
min protocol = NT1#小米电视只能支持1.0版本的samba协议

[share]
comment = share
path = media/mayanyan/软件/guoguo
public = yes
browseable = yes
writable = yes
read only = false
valid users = mayanyan              //用户名，ubuntu的user
directory mask = 0777
#force user = graysen 
#force group = graysen 
available = yes
guest ok = yes
```

#### 给硬盘修改 label

wxfat格式：（apt install exfatprogs）

> exfatlabel /dev/sdb2 dior

ntfs格式：

> ntfslabel /d

查看无线网站：

> wconfig

列出当前USB端口上的设备信息

> lsusb

nfs%0A\-\-\-%0A%0Asudo%20apt\-get%20install%20nfs\-kernel\-server%20%23%20%E5%AE%89%E8%A3%85%20NFS%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF%20%0Asudo%20apt\-get%20install%20nfs\-common%20%23%20%E5%AE%89%E8%A3%85%20NFS%E5%AE%A2%E6%88%B7%E7%AB%AF%0A%0Asudo%20%2Fetc%2Finit.d%2Fnfs\-kernel\-server%20start%20%0Asudo%20%2Fetc%2Finit.d%2Fnfs\-kernel\-server%20restart%0A%0A%0A%20exportfs%20\-arv%0A%20%0A%20%0A%20tcp%3A111%202049%0A%20udp%3A111%204046%0A%20%0A%20showmount%20\-e%208.142.177.235%0A%20showmount.exe%20\-e%208.142.177.235%0A%0Anfs.client.mount.options%20%3D%20vers%3D4%0A%0A%2Fdata%2Fnfs%20%20\*\(rw%2Csync%2Cno\_root\_squash%2Cno\_subtree\_check%2Cinsecure\)%0A%0A%0Amount%20\-t%20nfs%20\-o%20resvport%208.142.177.235%3A%2Fdata%2Fnfs%20%2FUsers%2Fmayanyan%2Fdata%2Ftest\_nfs%0A%0Awin10%0A%0A%0A%E6%89%93%E5%BC%80%E6%88%96%E5%85%B3%E9%97%ADWindows%E5%8A%9F%E8%83%BD%E9%80%89%E9%A1%B9\-%3E%20NFS%E6%9C%8D%E5%8A%A1%0A%0A%0Asmaba%0A\-\-\-%0A%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%EF%BC%9A%0A%60%60%60%0A%5Bglobal%5D%0Amin%20protocol%20%3D%20NT1%23%E5%B0%8F%E7%B1%B3%E7%94%B5%E8%A7%86%E5%8F%AA%E8%83%BD%E6%94%AF%E6%8C%811.0%E7%89%88%E6%9C%AC%E7%9A%84samba%E5%8D%8F%E8%AE%AE%0A%0A%0A%0A%5Bshare%5D%0Acomment%20%3D%20share%0Apath%20%3D%20media%2Fmayanyan%2F%E8%BD%AF%E4%BB%B6%2Fguoguo%0Apublic%20%3D%20yes%0Abrowseable%20%3D%20yes%0Awritable%20%3D%20yes%0Aread%20only%20%3D%20false%0Avalid%20users%20%3D%20mayanyan%20%20%20%20%20%20%20%20%20%20%20%20%20%20%2F%2F%E7%94%A8%E6%88%B7%E5%90%8D%EF%BC%8Cubuntu%E7%9A%84user%0Adirectory%20mask%20%3D%200777%0A%23force%20user%20%3D%20graysen%20%0A%23force%20group%20%3D%20graysen%20%0Aavailable%20%3D%20yes%0Aguest%20ok%20%3D%20yes%0A%60%60%60%0A%0A%20%23%23%23%23%20%E7%BB%99%E7%A1%AC%E7%9B%98%E4%BF%AE%E6%94%B9%20label%0A%20wxfat%E6%A0%BC%E5%BC%8F%EF%BC%9A%EF%BC%88apt%20install%20exfatprogs%EF%BC%89%0A%3Eexfatlabel%20%2Fdev%2Fsdb2%20dior%0A%0Antfs%E6%A0%BC%E5%BC%8F%EF%BC%9A%0A%3Entfslabel%20%2Fd%0A%0A%0A%E6%9F%A5%E7%9C%8B%E6%97%A0%E7%BA%BF%E7%BD%91%E7%AB%99%EF%BC%9A%0A%3Ewconfig%0A%0A%E5%88%97%E5%87%BA%E5%BD%93%E5%89%8DUSB%E7%AB%AF%E5%8F%A3%E4%B8%8A%E7%9A%84%E8%AE%BE%E5%A4%87%E4%BF%A1%E6%81%AF%0A%3Elsusb
