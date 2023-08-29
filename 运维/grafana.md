# grafana

## Grafana

https://grafana.com/grafana/download

方式1

> wget https://dl.grafana.com/oss/release/grafana\-7.1.3\-1.x86\_64.rpm
> 
> 
> sudo yum install grafana\-7.1.3\-1.x86\_64.rpm

方式2

> https://dl.grafana.com/oss/release/grafana\-7.3.1.linux\-amd64.tar.gz
> 
> 
> 启动
> 
> 
> ./grafana\-server

39.107.127.244:3000

默认密码

admin

admin

强制修改密码

admin

mqzhifu

## grafana

先添加数据源，这里用promethue

data source

http://127.0.0.1:9000

save & test

没问题后，开始添加DashBoard

还是 data\-course 窗口，右侧有直接 有个tab ，选择 promethue stats 2.0 ,import

回到dashboard 页，就会看到了

服务器主要监控的关注点

cpu process num : cpu总数

mem cache

mem buffer

mem total

内存使用率：

100 \- \(\(node\_memory\_MemFree{instance="xxx"}\+node\_memory\_Cached{instance="xxx"}\+node\_memory\_Buffers{instance="xxx"}\)/node\_memory\_MemTotal\) \* 100

CPU使用率

100 \- \(avg by \(instance\) \(irate\(node\_cpu{instance="xxx", mode="idle"}\[5m\]\)\) \* 100\)

硬盘使用率

100 \- node\_filesystem\_free{instance="xxx",fstype\!~"rootfs|selinuxfs|autofs|rpc\_pipefs|tmpfs|udev|none|devpts|sysfs|debugfs|fuse._"} / node\_filesystem\_size{instance="xxx",fstype\!~"rootfs|selinuxfs|autofs|rpc\_pipefs|tmpfs|udev|none|devpts|sysfs|debugfs|fuse._"} \* 100

// 上行带宽

sum by \(instance\) \(irate\(node\_network\_receive\_bytes{instance="xxx",device\!~"bond.\*?|lo"}\[5m\]\)/128\)

// 下行带宽

sum by \(instance\) \(irate\(node\_network\_transmit\_bytes{instance="xxx",device\!~"bond.\*?|lo"}\[5m\]\)/128\)

插件目录：

/soft/grafana\-7.3.1/data/plugins

./grafana\-cli plugins install

## 添加node expoter grafana

左侧 \+ 号，create import

Grafana%0A\-\-\-\-\-\-\-\-\-\-\-\-%0Ahttps%3A%2F%2Fgrafana.com%2Fgrafana%2Fdownload%0A%0A%E6%96%B9%E5%BC%8F1%20%20%0A%3Ewget%20https%3A%2F%2Fdl.grafana.com%2Foss%2Frelease%2Fgrafana\-7.1.3\-1.x86\_64.rpm%20%20%0A%3Esudo%20yum%20install%20grafana\-7.1.3\-1.x86\_64.rpm%0A%0A%E6%96%B9%E5%BC%8F2%20%20%0A%3Ehttps%3A%2F%2Fdl.grafana.com%2Foss%2Frelease%2Fgrafana\-7.3.1.linux\-amd64.tar.gz%0A%E5%90%AF%E5%8A%A8%0A%3E.%2Fgrafana\-server%20%0A%0A%0A39.107.127.244%3A3000%20%20%0A%E9%BB%98%E8%AE%A4%E5%AF%86%E7%A0%81%20%20%0Aadmin%20%20%0Aadmin%20%20%0A%0A%0A%E5%BC%BA%E5%88%B6%E4%BF%AE%E6%94%B9%E5%AF%86%E7%A0%81%20%20%0Aadmin%20%20%0Amqzhifu%20%20%0A%0A%0Agrafana%0A\-\-\-\-\-\-\-\-\-\-\-%0A%0A%E5%85%88%E6%B7%BB%E5%8A%A0%E6%95%B0%E6%8D%AE%E6%BA%90%EF%BC%8C%E8%BF%99%E9%87%8C%E7%94%A8promethue%0Adata%20source%0A%0A%0Ahttp%3A%2F%2F127.0.0.1%3A9000%0A%0Asave%20%26%20test%0A%0A%0A%E6%B2%A1%E9%97%AE%E9%A2%98%E5%90%8E%EF%BC%8C%E5%BC%80%E5%A7%8B%E6%B7%BB%E5%8A%A0DashBoard%0A%E8%BF%98%E6%98%AF%20data\-course%20%E7%AA%97%E5%8F%A3%EF%BC%8C%E5%8F%B3%E4%BE%A7%E6%9C%89%E7%9B%B4%E6%8E%A5%20%E6%9C%89%E4%B8%AAtab%20%20%EF%BC%8C%E9%80%89%E6%8B%A9%20promethue%20stats%202.0%20%2Cimport%20%0A%0A%E5%9B%9E%E5%88%B0dashboard%20%E9%A1%B5%EF%BC%8C%E5%B0%B1%E4%BC%9A%E7%9C%8B%E5%88%B0%E4%BA%86%0A%0A%0A%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%BB%E8%A6%81%E7%9B%91%E6%8E%A7%E7%9A%84%E5%85%B3%E6%B3%A8%E7%82%B9%0A%0Acpu%20process%20num%20%20%3A%20cpu%E6%80%BB%E6%95%B0%0Amem%20cache%20%0Amem%20buffer%0Amem%20total%0A%E5%86%85%E5%AD%98%E4%BD%BF%E7%94%A8%E7%8E%87%EF%BC%9A%0A100%20\-%20\(\(node\_memory\_MemFree%7Binstance%3D%22xxx%22%7D%2Bnode\_memory\_Cached%7Binstance%3D%22xxx%22%7D%2Bnode\_memory\_Buffers%7Binstance%3D%22xxx%22%7D\)%2Fnode\_memory\_MemTotal\)%20\*%20100%0A%0ACPU%E4%BD%BF%E7%94%A8%E7%8E%87%0A100%20\-%20\(avg%20by%20\(instance\)%20\(irate\(node\_cpu%7Binstance%3D%22xxx%22%2C%20mode%3D%22idle%22%7D%5B5m%5D\)\)%20\*%20100\)%0A%0A%E7%A1%AC%E7%9B%98%E4%BD%BF%E7%94%A8%E7%8E%87%0A100%20\-%20node\_filesystem\_free%7Binstance%3D%22xxx%22%2Cfstype\!~%22rootfs%7Cselinuxfs%7Cautofs%7Crpc\_pipefs%7Ctmpfs%7Cudev%7Cnone%7Cdevpts%7Csysfs%7Cdebugfs%7Cfuse.\*%22%7D%20%2F%20node\_filesystem\_size%7Binstance%3D%22xxx%22%2Cfstype\!~%22rootfs%7Cselinuxfs%7Cautofs%7Crpc\_pipefs%7Ctmpfs%7Cudev%7Cnone%7Cdevpts%7Csysfs%7Cdebugfs%7Cfuse.\*%22%7D%20\*%20100%0A%0A%2F%2F%20%E4%B8%8A%E8%A1%8C%E5%B8%A6%E5%AE%BD%0Asum%20by%20\(instance\)%20\(irate\(node\_network\_receive\_bytes%7Binstance%3D%22xxx%22%2Cdevice\!~%22bond.\*%3F%7Clo%22%7D%5B5m%5D\)%2F128\)%0A%0A%2F%2F%20%E4%B8%8B%E8%A1%8C%E5%B8%A6%E5%AE%BD%0Asum%20by%20\(instance\)%20\(irate\(node\_network\_transmit\_bytes%7Binstance%3D%22xxx%22%2Cdevice\!~%22bond.\*%3F%7Clo%22%7D%5B5m%5D\)%2F128\)%0A%0A%0A%0A%E6%8F%92%E4%BB%B6%E7%9B%AE%E5%BD%95%EF%BC%9A%0A%2Fsoft%2Fgrafana\-7.3.1%2Fdata%2Fplugins%0A.%2Fgrafana\-cli%20plugins%20install%20%0A%0A%0A%E6%B7%BB%E5%8A%A0node%20expoter%20grafana%0A\-\-\-\-%0A%E5%B7%A6%E4%BE%A7%20%2B%20%E5%8F%B7%EF%BC%8Ccreate%20import%0A
