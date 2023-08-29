# promehtues报警规则

节点的运行时间

> time\(\) \- node\_boot\_time\_seconds{}

系统15分钟负载

> node\_load1

//==========cpu

节点的cpu 百分比\(system\)

> \(avg by \(environment,instance\) \(irate\(node\_cpu\_seconds\_total{job="node\_206",mode="system"}\[5m\]\)\)\) \* 100

//=======内存

节点的内存总量

> node\_memory\_MemTotal\_bytes{job="node\_206"}

节点的剩余内存量

> node\_memory\_MemFree\_bytes{job="node\_206"}

节点的已使用内存量

> node\_memory\_MemTotal\_bytes{job="node\_206"} \- node\_memory\_MemFree\_bytes{job="node\_206"}

节点的内存使用百分比

> \(\(node\_memory\_MemAvailable\_bytes{job="node\_206"} / \(node\_memory\_MemTotal\_bytes{job="node\_206"}\)\)\)\* 100

节点的内存剩余百分比

> \(1\-\(node\_memory\_MemAvailable\_bytes{job="node\_206"} / \(node\_memory\_MemTotal\_bytes{job="node\_206"}\)\)\)\* 100

//=======磁盘

节点的磁盘总量

> node\_filesystem\_size\_bytes{job="node\_206" ,fstype=~"ext4|xfs"}

节点的磁盘剩余空间

> node\_filesystem\_avail\_bytes{job="node\_206",fstype=~"ext4|xfs"}

节点的磁盘使用的空间

> node\_filesystem\_size\_bytes{job="node\-exporter",fstype=~"ext4|xfs"} \- node\_filesystem\_avail\_bytes{job="node\-exporter",fstype=~"ext4|xfs"}

节点的磁盘的使用百分比

> \(1 \- node\_filesystem\_avail\_bytes{job="node\-exporter",fstype=~"ext4|xfs"} / node\_filesystem\_size\_bytes{job="node\-exporter",fstype=~"ext4|xfs"}\) \* 100

//=====网络

节点当前established的个数

> node\_tcp\_connection\_states{job="node\-exporter", state="established"}

节点timewait的连接数

> node\_tcp\_connection\_states{job="node\-exporter", state="time\_wait"}

节点tcp连接总数

> sum by \(environment,instance\) \(node\_tcp\_connection\_states{job="node\-exporter"}\)

节点网卡eth0每秒接收的比特数

> avg by \(environment,instance,device\) \(irate\(node\_network\_receive\_bytes\_total{device=~"eth0|eth1|ens33|ens37"}\[1m\]\)\)

节点网卡eth0每秒发送的比特数

> avg by \(environment,instance,device\) \(irate\(node\_network\_transmit\_bytes\_total{device=~"eth0|eth1|ens33|ens37"}\[1m\]\)\)

//================================================

实例丢失

up{job="node\-exporter"} == 0

summary: "服务器实例 {{ $labels.instance }} 丢失"

description: "{{ $labels.instance }} 上的任务 {{ $labels.job }} 已经停止了 1 分钟已上了"

磁盘容量小于 5%

100 \- \(\(node\_filesystem\_avail\_bytes{job="node\-exporter",mountpoint=~"._",fstype=~"ext4|xfs|ext2|ext3"} \* 100\) / node\_filesystem\_size\_bytes {job="node\-exporter",mountpoint=~"._",fstype=~"ext4|xfs|ext2|ext3"}\) \> 95

summary: "服务器实例 {{ $labels.instance }} 磁盘不足 告警通知"

description: "{{ $labels.instance }}磁盘 {{ $labels.device }} 资源 已不足 5%, 当前值: {{ $value }}"

内存容量小于 20%

\(\(node\_memory\_MemTotal\_bytes \- node\_memory\_MemFree\_bytes \- node\_memory\_Buffers\_bytes \- node\_memory\_Cached\_bytes\) / \(node\_memory\_MemTotal\_bytes \)\) \* 100 \> 80

summary: "服务器实例 {{ $labels.instance }} 内存不足 告警通知"

description: "{{ $labels.instance }}内存资源已不足 20%,当前值: {{ $value }}"

CPU 平均负载大于 4 个"

node\_load5 \> 4

sumary: "服务器实例 {{ $labels.instance }} CPU 负载 告警通知"

description: "{{ $labels.instance }}CPU 平均负载\(5 分钟\) 已超过 4 ,当前值: {{ $value }}"

磁盘读 I/O 超过 30MB/s"

irate\(node\_disk\_read\_bytes\_total{device="sda"}\[1m\]\) \> 30000000

sumary: "服务器实例 {{ $labels.instance }} I/O 读负载 告警通知"

description: "{{ $labels.instance }}I/O 每分钟读已超过 30MB/s,当前值: {{ $value }}"

磁盘写 I/O 超过 30MB/s"

irate\(node\_disk\_written\_bytes\_total{device="sda"}\[1m\]\) \> 30000000

sumary: "服务器实例 {{ $labels.instance }} I/O 写负载 告警通知"

description: "{{ $labels.instance }}I/O 每分钟写已超过 30MB/s,当前值: {{ $value }}"

网卡流出速率大于 10MB/s"

\(irate\(node\_network\_transmit\_bytes\_total{device\!~"lo"}\[1m\]\) / 1000\) \> 1000000

sumary: "服务器实例 {{ $labels.instance }} 网卡流量负载 告警通知"

description: "{{ $labels.instance }}网卡 {{ $labels.device }} 流量已经超过 10MB/s, 当前值: {{ $value }}"

CPU 使用率大于 90%"

100 \- \(\(avg by \(instance,job,env\)\(irate\(node\_cpu\_seconds\_total{mode="idle"}\[30s\]\)\)\) \*100\) \> 90

sumary: "服务器实例 {{ $labels.instance }} CPU 使用率 告警通知"

description: "{{ $labels.instance }}CPU 使用率已超过 90%, 当前值: {{ $value }}

%E8%8A%82%E7%82%B9%E7%9A%84%E8%BF%90%E8%A1%8C%E6%97%B6%E9%97%B4%0A%3Etime\(\)%20\-%20node\_boot\_time\_seconds%7B%7D%0A%0A%E7%B3%BB%E7%BB%9F10%E5%88%86%E9%92%9F%E8%B4%9F%E8%BD%BD%0A%3Enode\_load1%0A%0A%0A%2F%2F%3D%3D%3D%3D%3D%3D%3D%3D%3D%3Dcpu%0A%E8%8A%82%E7%82%B9%E7%9A%84cpu%20%E7%99%BE%E5%88%86%E6%AF%94\(system\)%0A%3E\(avg%20by%20\(environment%2Cinstance\)%20\(irate\(node\_cpu\_seconds\_total%7Bjob%3D%22node\_206%22%2Cmode%3D%22system%22%7D%5B5m%5D\)\)\)%20%20\*%20100%0A%0A%2F%2F%3D%3D%3D%3D%3D%3D%3D%E5%86%85%E5%AD%98%0A%E8%8A%82%E7%82%B9%E7%9A%84%E5%86%85%E5%AD%98%E6%80%BB%E9%87%8F%0A%3Enode\_memory\_MemTotal\_bytes%7Bjob%3D%22node\_206%22%7D%0A%0A%E8%8A%82%E7%82%B9%E7%9A%84%E5%89%A9%E4%BD%99%E5%86%85%E5%AD%98%E9%87%8F%0A%3Enode\_memory\_MemFree\_bytes%7Bjob%3D%22node\_206%22%7D%0A%20%0A%E8%8A%82%E7%82%B9%E7%9A%84%E5%B7%B2%E4%BD%BF%E7%94%A8%E5%86%85%E5%AD%98%E9%87%8F%0A%3Enode\_memory\_MemTotal\_bytes%7Bjob%3D%22node\_206%22%7D%20\-%20node\_memory\_MemFree\_bytes%7Bjob%3D%22node\_206%22%7D%0A%0A%E8%8A%82%E7%82%B9%E7%9A%84%E5%86%85%E5%AD%98%E4%BD%BF%E7%94%A8%E7%99%BE%E5%88%86%E6%AF%94%0A%3E\(\(node\_memory\_MemAvailable\_bytes%7Bjob%3D%22node\_206%22%7D%20%2F%20\(node\_memory\_MemTotal\_bytes%7Bjob%3D%22node\_206%22%7D\)\)\)\*%20100%0A%0A%0A%E8%8A%82%E7%82%B9%E7%9A%84%E5%86%85%E5%AD%98%E5%89%A9%E4%BD%99%E7%99%BE%E5%88%86%E6%AF%94%0A%3E\(1\-\(node\_memory\_MemAvailable\_bytes%7Bjob%3D%22node\_206%22%7D%20%2F%20\(node\_memory\_MemTotal\_bytes%7Bjob%3D%22node\_206%22%7D\)\)\)\*%20100%0A%20%0A%2F%2F%3D%3D%3D%3D%3D%3D%3D%E7%A3%81%E7%9B%98%0A%0A%E8%8A%82%E7%82%B9%E7%9A%84%E7%A3%81%E7%9B%98%E6%80%BB%E9%87%8F%0A%3Enode\_filesystem\_size\_bytes%7Bjob%3D%22node\_206%22%20%2Cfstype%3D~%22ext4%7Cxfs%22%7D%0A%20%0A%E8%8A%82%E7%82%B9%E7%9A%84%E7%A3%81%E7%9B%98%E5%89%A9%E4%BD%99%E7%A9%BA%E9%97%B4%0A%3Enode\_filesystem\_avail\_bytes%7Bjob%3D%22node\_206%22%2Cfstype%3D~%22ext4%7Cxfs%22%7D%0A%0A%E8%8A%82%E7%82%B9%E7%9A%84%E7%A3%81%E7%9B%98%E4%BD%BF%E7%94%A8%E7%9A%84%E7%A9%BA%E9%97%B4%0A%3Enode\_filesystem\_size\_bytes%7Bjob%3D%22node\-exporter%22%2Cfstype%3D~%22ext4%7Cxfs%22%7D%20\-%20node\_filesystem\_avail\_bytes%7Bjob%3D%22node\-exporter%22%2Cfstype%3D~%22ext4%7Cxfs%22%7D%0A%0A%E8%8A%82%E7%82%B9%E7%9A%84%E7%A3%81%E7%9B%98%E7%9A%84%E4%BD%BF%E7%94%A8%E7%99%BE%E5%88%86%E6%AF%94%0A%3E\(1%20\-%20node\_filesystem\_avail\_bytes%7Bjob%3D%22node\-exporter%22%2Cfstype%3D~%22ext4%7Cxfs%22%7D%20%2F%20node\_filesystem\_size\_bytes%7Bjob%3D%22node\-exporter%22%2Cfstype%3D~%22ext4%7Cxfs%22%7D\)%20\*%20100%20%0A%20%0A%2F%2F%3D%3D%3D%3D%3D%E7%BD%91%E7%BB%9C%0A%20%0A%E8%8A%82%E7%82%B9%E5%BD%93%E5%89%8Destablished%E7%9A%84%E4%B8%AA%E6%95%B0%0A%3Enode\_tcp\_connection\_states%7Bjob%3D%22node\-exporter%22%2C%20state%3D%22established%22%7D%0A%20%0A%E8%8A%82%E7%82%B9timewait%E7%9A%84%E8%BF%9E%E6%8E%A5%E6%95%B0%0A%3Enode\_tcp\_connection\_states%7Bjob%3D%22node\-exporter%22%2C%20state%3D%22time\_wait%22%7D%0A%20%0A%E8%8A%82%E7%82%B9tcp%E8%BF%9E%E6%8E%A5%E6%80%BB%E6%95%B0%0A%3Esum%20by%20\(environment%2Cinstance\)%20\(node\_tcp\_connection\_states%7Bjob%3D%22node\-exporter%22%7D\)%0A%20%0A%20%0A%E8%8A%82%E7%82%B9%E7%BD%91%E5%8D%A1eth0%E6%AF%8F%E7%A7%92%E6%8E%A5%E6%94%B6%E7%9A%84%E6%AF%94%E7%89%B9%E6%95%B0%0A%3Eavg%20by%20\(environment%2Cinstance%2Cdevice\)%20\(irate\(node\_network\_receive\_bytes\_total%7Bdevice%3D~%22eth0%7Ceth1%7Cens33%7Cens37%22%7D%5B1m%5D\)\)%0A%20%0A%20%0A%E8%8A%82%E7%82%B9%E7%BD%91%E5%8D%A1eth0%E6%AF%8F%E7%A7%92%E5%8F%91%E9%80%81%E7%9A%84%E6%AF%94%E7%89%B9%E6%95%B0%0A%3Eavg%20by%20\(environment%2Cinstance%2Cdevice\)%20\(irate\(node\_network\_transmit\_bytes\_total%7Bdevice%3D~%22eth0%7Ceth1%7Cens33%7Cens37%22%7D%5B1m%5D\)\)%0A%0A%0A%0A%2F%2F%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%0A%E5%AE%9E%E4%BE%8B%E4%B8%A2%E5%A4%B1%0Aup%7Bjob%3D%22node\-exporter%22%7D%20%3D%3D%200%0Asummary%3A%20%22%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%9E%E4%BE%8B%20%7B%7B%20%24labels.instance%20%7D%7D%20%E4%B8%A2%E5%A4%B1%22%0Adescription%3A%20%22%7B%7B%20%24labels.instance%20%7D%7D%20%E4%B8%8A%E7%9A%84%E4%BB%BB%E5%8A%A1%20%7B%7B%20%24labels.job%20%7D%7D%20%E5%B7%B2%E7%BB%8F%E5%81%9C%E6%AD%A2%E4%BA%86%201%20%E5%88%86%E9%92%9F%E5%B7%B2%E4%B8%8A%E4%BA%86%22%0A%20%0A%E7%A3%81%E7%9B%98%E5%AE%B9%E9%87%8F%E5%B0%8F%E4%BA%8E%205%25%0A100%20\-%20\(\(node\_filesystem\_avail\_bytes%7Bjob%3D%22node\-exporter%22%2Cmountpoint%3D~%22.\*%22%2Cfstype%3D~%22ext4%7Cxfs%7Cext2%7Cext3%22%7D%20\*%20100\)%20%2F%20node\_filesystem\_size\_bytes%20%7Bjob%3D%22node\-exporter%22%2Cmountpoint%3D~%22.\*%22%2Cfstype%3D~%22ext4%7Cxfs%7Cext2%7Cext3%22%7D\)%20%3E%2095%0Asummary%3A%20%22%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%9E%E4%BE%8B%20%7B%7B%20%24labels.instance%20%7D%7D%20%E7%A3%81%E7%9B%98%E4%B8%8D%E8%B6%B3%20%E5%91%8A%E8%AD%A6%E9%80%9A%E7%9F%A5%22%0Adescription%3A%20%22%7B%7B%20%24labels.instance%20%7D%7D%E7%A3%81%E7%9B%98%20%7B%7B%20%24labels.device%20%7D%7D%20%E8%B5%84%E6%BA%90%20%E5%B7%B2%E4%B8%8D%E8%B6%B3%205%25%2C%20%E5%BD%93%E5%89%8D%E5%80%BC%3A%20%7B%7B%20%24value%20%7D%7D%22%0A%20%0A%E5%86%85%E5%AD%98%E5%AE%B9%E9%87%8F%E5%B0%8F%E4%BA%8E%2020%25%0A\(\(node\_memory\_MemTotal\_bytes%20\-%20node\_memory\_MemFree\_bytes%20\-%20node\_memory\_Buffers\_bytes%20\-%20node\_memory\_Cached\_bytes\)%20%2F%20\(node\_memory\_MemTotal\_bytes%20\)\)%20\*%20100%20%3E%2080%0Asummary%3A%20%22%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%9E%E4%BE%8B%20%7B%7B%20%24labels.instance%20%7D%7D%20%E5%86%85%E5%AD%98%E4%B8%8D%E8%B6%B3%20%E5%91%8A%E8%AD%A6%E9%80%9A%E7%9F%A5%22%0Adescription%3A%20%22%7B%7B%20%24labels.instance%20%7D%7D%E5%86%85%E5%AD%98%E8%B5%84%E6%BA%90%E5%B7%B2%E4%B8%8D%E8%B6%B3%2020%25%2C%E5%BD%93%E5%89%8D%E5%80%BC%3A%20%7B%7B%20%24value%20%7D%7D%22%0A%20%0ACPU%20%E5%B9%B3%E5%9D%87%E8%B4%9F%E8%BD%BD%E5%A4%A7%E4%BA%8E%204%20%E4%B8%AA%22%0Anode\_load5%20%3E%204%0Asumary%3A%20%22%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%9E%E4%BE%8B%20%7B%7B%20%24labels.instance%20%7D%7D%20CPU%20%E8%B4%9F%E8%BD%BD%20%E5%91%8A%E8%AD%A6%E9%80%9A%E7%9F%A5%22%0Adescription%3A%20%22%7B%7B%20%24labels.instance%20%7D%7DCPU%20%E5%B9%B3%E5%9D%87%E8%B4%9F%E8%BD%BD\(5%20%E5%88%86%E9%92%9F\)%20%E5%B7%B2%E8%B6%85%E8%BF%87%204%20%2C%E5%BD%93%E5%89%8D%E5%80%BC%3A%20%7B%7B%20%24value%20%7D%7D%22%0A%20%0A%E7%A3%81%E7%9B%98%E8%AF%BB%20I%2FO%20%E8%B6%85%E8%BF%87%2030MB%2Fs%22%0Airate\(node\_disk\_read\_bytes\_total%7Bdevice%3D%22sda%22%7D%5B1m%5D\)%20%3E%2030000000%0Asumary%3A%20%22%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%9E%E4%BE%8B%20%7B%7B%20%24labels.instance%20%7D%7D%20I%2FO%20%E8%AF%BB%E8%B4%9F%E8%BD%BD%20%E5%91%8A%E8%AD%A6%E9%80%9A%E7%9F%A5%22%0Adescription%3A%20%22%7B%7B%20%24labels.instance%20%7D%7DI%2FO%20%E6%AF%8F%E5%88%86%E9%92%9F%E8%AF%BB%E5%B7%B2%E8%B6%85%E8%BF%87%2030MB%2Fs%2C%E5%BD%93%E5%89%8D%E5%80%BC%3A%20%7B%7B%20%24value%20%7D%7D%22%0A%20%0A%E7%A3%81%E7%9B%98%E5%86%99%20I%2FO%20%E8%B6%85%E8%BF%87%2030MB%2Fs%22%0Airate\(node\_disk\_written\_bytes\_total%7Bdevice%3D%22sda%22%7D%5B1m%5D\)%20%3E%2030000000%0Asumary%3A%20%22%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%9E%E4%BE%8B%20%7B%7B%20%24labels.instance%20%7D%7D%20I%2FO%20%E5%86%99%E8%B4%9F%E8%BD%BD%20%E5%91%8A%E8%AD%A6%E9%80%9A%E7%9F%A5%22%0Adescription%3A%20%22%7B%7B%20%24labels.instance%20%7D%7DI%2FO%20%E6%AF%8F%E5%88%86%E9%92%9F%E5%86%99%E5%B7%B2%E8%B6%85%E8%BF%87%2030MB%2Fs%2C%E5%BD%93%E5%89%8D%E5%80%BC%3A%20%7B%7B%20%24value%20%7D%7D%22%0A%20%0A%E7%BD%91%E5%8D%A1%E6%B5%81%E5%87%BA%E9%80%9F%E7%8E%87%E5%A4%A7%E4%BA%8E%2010MB%2Fs%22%0A\(irate\(node\_network\_transmit\_bytes\_total%7Bdevice\!~%22lo%22%7D%5B1m%5D\)%20%2F%201000\)%20%3E%201000000%0Asumary%3A%20%22%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%9E%E4%BE%8B%20%7B%7B%20%24labels.instance%20%7D%7D%20%E7%BD%91%E5%8D%A1%E6%B5%81%E9%87%8F%E8%B4%9F%E8%BD%BD%20%E5%91%8A%E8%AD%A6%E9%80%9A%E7%9F%A5%22%0Adescription%3A%20%22%7B%7B%20%24labels.instance%20%7D%7D%E7%BD%91%E5%8D%A1%20%7B%7B%20%24labels.device%20%7D%7D%20%E6%B5%81%E9%87%8F%E5%B7%B2%E7%BB%8F%E8%B6%85%E8%BF%87%2010MB%2Fs%2C%20%E5%BD%93%E5%89%8D%E5%80%BC%3A%20%7B%7B%20%24value%20%7D%7D%22%0A%20%0ACPU%20%E4%BD%BF%E7%94%A8%E7%8E%87%E5%A4%A7%E4%BA%8E%2090%25%22%0A100%20\-%20\(\(avg%20by%20\(instance%2Cjob%2Cenv\)\(irate\(node\_cpu\_seconds\_total%7Bmode%3D%22idle%22%7D%5B30s%5D\)\)\)%20\*100\)%20%3E%2090%0Asumary%3A%20%22%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%9E%E4%BE%8B%20%7B%7B%20%24labels.instance%20%7D%7D%20CPU%20%E4%BD%BF%E7%94%A8%E7%8E%87%20%E5%91%8A%E8%AD%A6%E9%80%9A%E7%9F%A5%22%0Adescription%3A%20%22%7B%7B%20%24labels.instance%20%7D%7DCPU%20%E4%BD%BF%E7%94%A8%E7%8E%87%E5%B7%B2%E8%B6%85%E8%BF%87%2090%25%2C%20%E5%BD%93%E5%89%8D%E5%80%BC%3A%20%7B%7B%20%24value%20%7D%7D%0A
