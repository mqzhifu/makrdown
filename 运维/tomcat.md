# tomcat

cd /install

wget https://downloads.apache.org/tomcat/tomcat\-8/v8.5.81/bin/apache\-tomcat\-8.5.81.zip

unzip apache\-tomcat\-8.5.81.zip

mv apache\-tomcat\-8.5.81 /soft/tomcat

cd /soft/tomcat

bin/startup.sh

bin/shutdown.sh

vim conf/tomcat\-users.xml

```
role rolename="manager-gui"/>
<user password="ckckarar" roles="manager-gui" username="ckadmin"/>
```

vim conf/server.xml

```
 <Host name="localhost"  appBase="/data/www/tomcat_webapps" unpackWARs="true" autoDeploy="true">
```

vim tomcat\_webapps/manager/META\-INF/context.xml

```
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
          allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|\d+\.\d+\.\d+\.\d+" />
```

wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx\_prometheus\_javaagent/0.17.0/jmx\_prometheus\_javaagent\-0.17.0.jar

wget https://github.com/prometheus/jmx\_exporter/blob/master/example\_configs/tomcat.yml

cp jmx\_prometheus\_javaagent\-0.17.0.jar tomcat.yml /soft/tomcat/bin

vim bin/catalina.sh

```
JAVA_OPTS="-javaagent:/soft/tomcat/bin/jmx_prometheus_javaagent-0.17.0.jar=30018:/soft/tomcat/bin/tomcat.yaml"
```

添加prometheus 配置

添加grafana图形

cd%20%2Finstall%0Awget%20https%3A%2F%2Fdownloads.apache.org%2Ftomcat%2Ftomcat\-8%2Fv8.5.81%2Fbin%2Fapache\-tomcat\-8.5.81.zip%0A%0Aunzip%20apache\-tomcat\-8.5.81.zip%0Amv%20%20apache\-tomcat\-8.5.81%20%2Fsoft%2Ftomcat%0Acd%20%2Fsoft%2Ftomcat%0Abin%2Fstartup.sh%0Abin%2Fshutdown.sh%0A%0A%0Avim%20conf%2Ftomcat\-users.xml%0A%60%60%60%0Arole%20rolename%3D%22manager\-gui%22%2F%3E%0A%3Cuser%20password%3D%22ckckarar%22%20roles%3D%22manager\-gui%22%20username%3D%22ckadmin%22%2F%3E%0A%60%60%60%0A%0Avim%20conf%2Fserver.xml%0A%60%60%60%0A%20%3CHost%20name%3D%22localhost%22%20%20appBase%3D%22%2Fdata%2Fwww%2Ftomcat\_webapps%22%20unpackWARs%3D%22true%22%20autoDeploy%3D%22true%22%3E%0A%60%60%60%0A%0Avim%20tomcat\_webapps%2Fmanager%2FMETA\-INF%2Fcontext.xml%0A%60%60%60%0A%20%20%3CValve%20className%3D%22org.apache.catalina.valves.RemoteAddrValve%22%0A%20%20%20%20%20%20%20%20%20%20allow%3D%22127%5C.%5Cd%2B%5C.%5Cd%2B%5C.%5Cd%2B%7C%3A%3A1%7C0%3A0%3A0%3A0%3A0%3A0%3A0%3A1%7C%5Cd%2B%5C.%5Cd%2B%5C.%5Cd%2B%5C.%5Cd%2B%22%20%2F%3E%0A%60%60%60%0A%0Awget%20https%3A%2F%2Frepo1.maven.org%2Fmaven2%2Fio%2Fprometheus%2Fjmx%2Fjmx\_prometheus\_javaagent%2F0.17.0%2Fjmx\_prometheus\_javaagent\-0.17.0.jar%0Awget%20https%3A%2F%2Fgithub.com%2Fprometheus%2Fjmx\_exporter%2Fblob%2Fmaster%2Fexample\_configs%2Ftomcat.yml%0A%0Acp%20jmx\_prometheus\_javaagent\-0.17.0.jar%20tomcat.yml%20%2Fsoft%2Ftomcat%2Fbin%0Avim%20bin%2Fcatalina.sh%0A%60%60%60%0AJAVA\_OPTS%3D%22\-javaagent%3A%2Fsoft%2Ftomcat%2Fbin%2Fjmx\_prometheus\_javaagent\-0.17.0.jar%3D30018%3A%2Fsoft%2Ftomcat%2Fbin%2Ftomcat.yaml%22%0A%60%60%60%0A%0A%E6%B7%BB%E5%8A%A0prometheus%20%E9%85%8D%E7%BD%AE%0A%0A%E6%B7%BB%E5%8A%A0grafana%E5%9B%BE%E5%BD%A2%E6%A8%A1%E6%9D%BF
