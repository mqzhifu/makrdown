# id-list


TG-工作号:
>+63 9082542571

git 地址: 
```
https://repo.xdev.stream/CustomerService/wcs
nash_go 
mqzhifu
```


git  分支 ：
>N7Q_master

产品需求
https://y9wbg2.axshare.com/#id=gfrm3i&p=%E5%90%8D%E8%AF%8D%E8%A7%A3%E9%87%8A&g=1

# 代码部署

git 代码仓库 ：
```
https://repo.xdev.stream/CustomerService/wcs.git

https://repo.xdev.stream/CustomerService/WebSdkTools.git

https://repo.xdev.stream/CustomerService/WebSdk.git

https://repo.xdev.stream/CustomerService/ServiceDocs.git
```

wcs 是主要代码仓，先进入：
>cd /e/project/tg/wcs/

启动部署脚本：
>./up.sh

win-兼容问题：
```
vi /e/project/tg/wcs/docs/db/.env_db.sh
//添加如下：
DAPR_WORKDIR_WIN=${DAPR_WORKDIR//\//\/\/}
winpty
sed -i 's/\r//' $tmp_file

```


export https_proxy=http://127.0.0.1:11346;
export http_proxy=http://127.0.0.1:11346;
export all_proxy=socks5://127.0.0.1:11346


这里问题：
- docker exec 进入容器时，路径问题，前置还得加上 winpty
- 生成临时文件换行符问题

git 下 ：win换行符问题：
```
git config --global core.autocrlf input 

git config --global core.filemode
git config --global core.eol lf
```

# ./up.sh

#### /dapr/.env_default.sh：
设置所有系统环境变量 ->   /dapr/.env_local.sh 

#### ./start_dapr.sh:
- 下包 gcache mysql dapr gox sentry dbml2go  resolv.conf
- /dapr/docker-compose.yaml
	- nats：队列
	- astools：KV-DB
	- aerospike：KV-DB
	- scylla：KV-DB
	- consul
	- mysql
	- kafka
	- prometheus
	- dapr
	- zipkin
	- redis
	- 设置网络
./gox dao
./gox pb
./gox --start

>gox gcache sentry 这3个是编译后的文件，看不到源码
# 微服务

core：所有的API逻辑均在这里，包括发消息
gateway：长连接，但交不直接发消息，所有请求都转后后端core服务的短连接请求。更像是个状态连接，实际的消息在kafka中处理.
asset：代码很少，不确定用处
sink：代码很少，不确定用处




# PROTOBUF

编译使用的是容器：
protobuf:3.19.2
protoc-gen-go-grpc 1.2.0



go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.27.0
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2.0


```
C:\protoc.exe  --proto_path=.  --go_out=e:\test api\asset\*.proto  api\common\*.proto  api\common\*.proto api\core\*.proto

C:\protoc.exe  --proto_path=.  --go_out=e:\test --go-grpc_out=e:\test api\asset\*.proto  api\common\*.proto  api\common\*.proto api\core\*.proto


C:\protoc.exe  --proto_path=.  --swagger_out=e:\test api\asset\*.proto  api\common\*.proto  api\common\*.proto api\core\*.proto

C:\protoc.exe  --proto_path=. --go-grpc_out=e:\test  --swagger_out=e:\test -  *.proto




go install github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger@latest
go install github.com/grpc-ecosystem/grpc-gateway/protoc-gen-grpc-gateway@latest


```
