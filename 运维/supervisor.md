# supervisor

http://supervisord.org/plugins.html?highlight=cesi

## 安装

官方给的方法:

> pip install supervisor

我用的：它依赖了一个py3的包

> yum install supervisor

yum 挺方便，完成后，会有两个指令:

supervisord:守护进程/主进程

supervisorctl:管理 supervisord

etc目录下会有一个新文件和一个新的目录：

/etc/supervisord.conf:核心配置文件

/etc/supervisord.d:空目录

supervisord.conf 主配置文件

```
loglevel=info
logfile=/var/log/supervisor/supervisord.log 
pidfile=/var/run/supervisord.pid
serverurl=unix:///var/run/supervisor/supervisor.sock
[include]
files = supervisord.d/*.ini
```

新版unix\_http\_server必须得打开uname \+ ps

> 主配置文件中，会包含supervisord.d目录下的子配置文件

创建一个demo文件

> touch /etc/supervisord.d/demoone.ini

```
[program:demoone]   ;脚本名称
command=/www/accountServer/accountServer ;执行的启动指令
directory=/www/accountServer/   ;脚本工具目录
autorstart=true     ;supervisor启动后，自动启动该脚本
autorestart=true    ;脚本进程挂了后，自动拉起
startsecs=3
startretries=3      ;脚本进程挂了后，自动拉起重试次数
stdout_logfile=/www/server/panel/plugin/supervisor/log/login.out.log
stderr_logfile=/www/server/panel/plugin/supervisor/log/login.err.log
stdout_logfile_maxbytes=2MB
stderr_logfile_maxbytes=2MB
user=root
priority=999
numprocs=1
process_name=%(program_name)s_%(process_num)02d

```

启动

> supervisord \-c /etc/supervisord.conf

进程定义名：可能出现两部分，如果DIY配置文件中的process\_name，也就是不等于默认%\(program\_name\)s,再简单点进程名与demoone不一样，即两分钟：demoone:\[diy process\_name\]

## 日常操作指令

\#管理supervisor中的应用进程

supervisorctl status 进程定义名

supervisorctl stop 进程定义名

supervisorctl restart 进程定义名

\#管理supervisord守护进程

supervisorctl reread

supervisorctl reload

supervisorctl update

supervisorctl shutdown

## web\-ui

自带的：

```
[inet_http_server]      ; 
port=0.0.0.0:5555       ; 
username=admin           ; 
password=123456          ;
```

丑是丑了点，但核心功能都有...基本上能覆盖主要的命令行指令,官网还提供了几个UI的:

Nodervisor:nodejs，URL地址都是错的...

Django\-Dashvisor:py,2015年停更

SupervisorUI:php

Supvisors:py

supervisorui:php 2012年就停更了

multivisor:py

cesi:py

Supervisord\-Monitor:php，2017年3月停更

算下来就两个还靠点谱吧：Supervisord\-Monitor 和cesi

cesi是比较好看的，无奈PY依赖包安装不上，放弃了...

官方自带的只能监控单机节点，3方包可监控多节点

%0Ahttp%3A%2F%2Fsupervisord.org%2Fplugins.html%3Fhighlight%3Dcesi%0A%E5%AE%89%E8%A3%85%0A\-\-\-%0A%E5%AE%98%E6%96%B9%E7%BB%99%E7%9A%84%E6%96%B9%E6%B3%95%3A%0A%3Epip%20install%20supervisor%0A%0A%E6%88%91%E7%94%A8%E7%9A%84%EF%BC%9A%E5%AE%83%E4%BE%9D%E8%B5%96%E4%BA%86%E4%B8%80%E4%B8%AApy3%E7%9A%84%E5%8C%85%0A%3Eyum%20install%20supervisor%0A%0Ayum%20%E6%8C%BA%E6%96%B9%E4%BE%BF%EF%BC%8C%E5%AE%8C%E6%88%90%E5%90%8E%EF%BC%8C%E4%BC%9A%E6%9C%89%E4%B8%A4%E4%B8%AA%E6%8C%87%E4%BB%A4%3A%0Asupervisord%3A%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B%2F%E4%B8%BB%E8%BF%9B%E7%A8%8B%0Asupervisorctl%3A%E7%AE%A1%E7%90%86%20supervisord%0A%0Aetc%E7%9B%AE%E5%BD%95%E4%B8%8B%E4%BC%9A%E6%9C%89%E4%B8%80%E4%B8%AA%E6%96%B0%E6%96%87%E4%BB%B6%E5%92%8C%E4%B8%80%E4%B8%AA%E6%96%B0%E7%9A%84%E7%9B%AE%E5%BD%95%EF%BC%9A%0A%2Fetc%2Fsupervisord.conf%3A%E6%A0%B8%E5%BF%83%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%0A%2Fetc%2Fsupervisord.d%3A%E7%A9%BA%E7%9B%AE%E5%BD%95%0A%0Asupervisord.conf%20%E4%B8%BB%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%0A%60%60%60%0Aloglevel%3Dinfo%0Alogfile%3D%2Fvar%2Flog%2Fsupervisor%2Fsupervisord.log%20%0Apidfile%3D%2Fvar%2Frun%2Fsupervisord.pid%0Aserverurl%3Dunix%3A%2F%2F%2Fvar%2Frun%2Fsupervisor%2Fsupervisor.sock%0A%5Binclude%5D%0Afiles%20%3D%20supervisord.d%2F\*.ini%0A%60%60%60%0A%0A%E6%96%B0%E7%89%88unix\_http\_server%E5%BF%85%E9%A1%BB%E5%BE%97%E6%89%93%E5%BC%80uname%20%2B%20ps%0A%0A%3E%E4%B8%BB%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%B8%AD%EF%BC%8C%E4%BC%9A%E5%8C%85%E5%90%ABsupervisord.d%E7%9B%AE%E5%BD%95%E4%B8%8B%E7%9A%84%E5%AD%90%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%0A%0A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AAdemo%E6%96%87%E4%BB%B6%0A%3Etouch%20%2Fetc%2Fsupervisord.d%2Fdemoone.ini%0A%0A%60%60%60%0A%5Bprogram%3Ademoone%5D%20%20%20%3B%E8%84%9A%E6%9C%AC%E5%90%8D%E7%A7%B0%0Acommand%3D%2Fwww%2FaccountServer%2FaccountServer%20%3B%E6%89%A7%E8%A1%8C%E7%9A%84%E5%90%AF%E5%8A%A8%E6%8C%87%E4%BB%A4%0Adirectory%3D%2Fwww%2FaccountServer%2F%20%20%20%3B%E8%84%9A%E6%9C%AC%E5%B7%A5%E5%85%B7%E7%9B%AE%E5%BD%95%0Aautorstart%3Dtrue%20%20%20%20%20%3Bsupervisor%E5%90%AF%E5%8A%A8%E5%90%8E%EF%BC%8C%E8%87%AA%E5%8A%A8%E5%90%AF%E5%8A%A8%E8%AF%A5%E8%84%9A%E6%9C%AC%0Aautorestart%3Dtrue%20%20%20%20%3B%E8%84%9A%E6%9C%AC%E8%BF%9B%E7%A8%8B%E6%8C%82%E4%BA%86%E5%90%8E%EF%BC%8C%E8%87%AA%E5%8A%A8%E6%8B%89%E8%B5%B7%0Astartsecs%3D3%0Astartretries%3D3%20%20%20%20%20%20%3B%E8%84%9A%E6%9C%AC%E8%BF%9B%E7%A8%8B%E6%8C%82%E4%BA%86%E5%90%8E%EF%BC%8C%E8%87%AA%E5%8A%A8%E6%8B%89%E8%B5%B7%E9%87%8D%E8%AF%95%E6%AC%A1%E6%95%B0%0Astdout\_logfile%3D%2Fwww%2Fserver%2Fpanel%2Fplugin%2Fsupervisor%2Flog%2Flogin.out.log%0Astderr\_logfile%3D%2Fwww%2Fserver%2Fpanel%2Fplugin%2Fsupervisor%2Flog%2Flogin.err.log%0Astdout\_logfile\_maxbytes%3D2MB%0Astderr\_logfile\_maxbytes%3D2MB%0Auser%3Droot%0Apriority%3D999%0Anumprocs%3D1%0Aprocess\_name%3D%25\(program\_name\)s\_%25\(process\_num\)02d%0A%0A%60%60%60%0A%0A%0A%E5%90%AF%E5%8A%A8%0A%3Esupervisord%20\-c%20%2Fetc%2Fsupervisord.conf%0A%0A%0A%E8%BF%9B%E7%A8%8B%E5%AE%9A%E4%B9%89%E5%90%8D%EF%BC%9A%E5%8F%AF%E8%83%BD%E5%87%BA%E7%8E%B0%E4%B8%A4%E9%83%A8%E5%88%86%EF%BC%8C%E5%A6%82%E6%9E%9CDIY%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%B8%AD%E7%9A%84process\_name%EF%BC%8C%E4%B9%9F%E5%B0%B1%E6%98%AF%E4%B8%8D%E7%AD%89%E4%BA%8E%E9%BB%98%E8%AE%A4%25\(program\_name\)s%2C%E5%86%8D%E7%AE%80%E5%8D%95%E7%82%B9%E8%BF%9B%E7%A8%8B%E5%90%8D%E4%B8%8Edemoone%E4%B8%8D%E4%B8%80%E6%A0%B7%EF%BC%8C%E5%8D%B3%E4%B8%A4%E5%88%86%E9%92%9F%EF%BC%9Ademoone%3A%5Bdiy%20process\_name%5D%0A%0A%0A%E6%97%A5%E5%B8%B8%E6%93%8D%E4%BD%9C%E6%8C%87%E4%BB%A4%0A\-\-\-\-\-%0A%23%E7%AE%A1%E7%90%86supervisor%E4%B8%AD%E7%9A%84%E5%BA%94%E7%94%A8%E8%BF%9B%E7%A8%8B%0Asupervisorctl%20status%20%E8%BF%9B%E7%A8%8B%E5%AE%9A%E4%B9%89%E5%90%8D%0Asupervisorctl%20stop%20%E8%BF%9B%E7%A8%8B%E5%AE%9A%E4%B9%89%E5%90%8D%0Asupervisorctl%20restart%20%E8%BF%9B%E7%A8%8B%E5%AE%9A%E4%B9%89%E5%90%8D%0A%0A%23%E7%AE%A1%E7%90%86supervisord%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B%0Asupervisorctl%20reload%0Asupervisorctl%20update%0Asupervisorctl%20shutdown%0Aweb\-ui%0A\-\-\-\-\-%0A%E8%87%AA%E5%B8%A6%E7%9A%84%EF%BC%9A%0A%60%60%60%0A%5Binet\_http\_server%5D%20%20%20%20%20%20%3B%20%0Aport%3D0.0.0.0%3A5555%20%20%20%20%20%20%20%3B%20%0Ausername%3Dadmin%20%20%20%20%20%20%20%20%20%20%20%3B%20%0Apassword%3D123456%20%20%20%20%20%20%20%20%20%20%3B%0A%60%60%60%0A%0A%E4%B8%91%E6%98%AF%E4%B8%91%E4%BA%86%E7%82%B9%EF%BC%8C%E4%BD%86%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD%E9%83%BD%E6%9C%89...%E5%9F%BA%E6%9C%AC%E4%B8%8A%E8%83%BD%E8%A6%86%E7%9B%96%E4%B8%BB%E8%A6%81%E7%9A%84%E5%91%BD%E4%BB%A4%E8%A1%8C%E6%8C%87%E4%BB%A4%2C%E5%AE%98%E7%BD%91%E8%BF%98%E6%8F%90%E4%BE%9B%E4%BA%86%E5%87%A0%E4%B8%AAUI%E7%9A%84%3A%0A%0A%0ANodervisor%3Anodejs%EF%BC%8CURL%E5%9C%B0%E5%9D%80%E9%83%BD%E6%98%AF%E9%94%99%E7%9A%84...%0ADjango\-Dashvisor%3Apy%2C2015%E5%B9%B4%E5%81%9C%E6%9B%B4%0ASupervisorUI%3Aphp%0ASupvisors%3Apy%0Asupervisorui%3Aphp%202012%E5%B9%B4%E5%B0%B1%E5%81%9C%E6%9B%B4%E4%BA%86%0Amultivisor%3Apy%0Acesi%3Apy%20%0ASupervisord\-Monitor%3Aphp%EF%BC%8C2017%E5%B9%B43%E6%9C%88%E5%81%9C%E6%9B%B4%0A%0A%0A%E7%AE%97%E4%B8%8B%E6%9D%A5%E5%B0%B1%E4%B8%A4%E4%B8%AA%E8%BF%98%E9%9D%A0%E7%82%B9%E8%B0%B1%E5%90%A7%EF%BC%9ASupervisord\-Monitor%20%E5%92%8Ccesi%0Acesi%E6%98%AF%E6%AF%94%E8%BE%83%E5%A5%BD%E7%9C%8B%E7%9A%84%EF%BC%8C%E6%97%A0%E5%A5%88PY%E4%BE%9D%E8%B5%96%E5%8C%85%E5%AE%89%E8%A3%85%E4%B8%8D%E4%B8%8A%EF%BC%8C%E6%94%BE%E5%BC%83%E4%BA%86...%0A%0A%0A%0A%E5%AE%98%E6%96%B9%E8%87%AA%E5%B8%A6%E7%9A%84%E5%8F%AA%E8%83%BD%E7%9B%91%E6%8E%A7%E5%8D%95%E6%9C%BA%E8%8A%82%E7%82%B9%EF%BC%8C3%E6%96%B9%E5%8C%85%E5%8F%AF%E7%9B%91%E6%8E%A7%E5%A4%9A%E8%8A%82%E7%82%B9%0A
