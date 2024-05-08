

rep -R '该商户已搬家' /e/project/tg/wcs/*



v1/tenant/worker/transfer

登陆：
 curl -X POST -H 'dapr-app-id:core' -H "Content-type: application/json" -d '{"account":"qladmin","password":"mBJjB^h@41YSDC%2"}'  http://127.0.0.1:8887//v1/tenant/login

 curl -X POST -H 'dapr-app-id:core' -H "Content-type: application/json" -d '{"account":"dev_jeff","password":"aaaaaaaaaaa"}'  http://127.0.0.1:8887/v1/tenant/login


 curl -X POST -H 'dapr-app-id:core' -H "Content-type: application/json" -d '{"account":"dev_bowen","password":"aaaaaaaaaaa"}'  http://127.0.0.1:8887/v1/tenant/login
 
 curl -X POST -H 'dapr-app-id:core' -H "Content-type: application/json" -d '{"account":"lucky01","password":"aaaaaaaaaaa"}'  http://127.0.0.1:8887/v1/tenant/login





创建：租户

curl -X POST -H 'dapr-app-id:core' -H "Content-type: application/json" -H 'X-Token: "COcHEAYY5gcg5Qco3_OBiO8x.NPl2ui_nzQ9BLvtXFoo1S9zDBjt6xmP3A9ICBU0hwob_g_PLjBeSRxa5YSIa0HNOpz65JKKkXa7AHdkmh8c_Bg' -d '{"name":"xiaoz" , "account":"xiaoxiaoz", "password":"123456","worker_capacity":"100","daily_service_capacity":100 }'  http://127.0.0.1:8887/v1/tenant/create

登陆刚刚创建的-租户

 curl -X POST -H 'dapr-app-id:core' -H "Content-type: application/json" -d '{"account":"32kf5","password":"123456"}'  http://127.0.0.1:8887//v1/tenant/login


创建一个 worker

curl -X POST -H 'dapr-app-id:core' -H "Content-type: application/json" -H 'X-Token: CAEQAxgBIAIo-qPkie8x.EsmDHt3bcpCLruyNHRCTfgJKl99GaIrjYoMhfqPyv7X5bFKTATCt0-2X4-RSFyXiwjVRPYeOVxjWCx6VDDRIBg'  -d '{ "account":"worker_one",  "password":"m123456" , "group_ids":[],"perm_mask":4,"name":"worker_one","avatar":"","bneednim":false,"avatarurl":"","group_cids":[],"tips":"" }'  http://127.0.0.1:8887/v1/tenant/worker/create


创建一个咨询

curl -X POST -H 'dapr-app-id:core' -H "Content-type: application/json" -H 'X-Token: CAEQAxgBIAIo-qPkie8x.EsmDHt3bcpCLruyNHRCTfgJKl99GaIrjYoMhfqPyv7X5bFKTATCt0-2X4-RSFyXiwjVRPYeOVxjWCx6VDDRIBg'  -d  '{ "consult":{ "name":"who nice?",    "guide":"that girls",    "worker_group_ids":[],    "worker_ids":[3],    "worker_group_names":"whowho",    "worker_names":"zzzzz" }}'  http://127.0.0.1:8887/v1/tenant/consult/create-consult


创建一个入口用户

curl -X POST -H 'dapr-app-id:core' -H "Content-type: application/json" -H 'X-Token: CAEQAxgBIAIo-qPkie8x.EsmDHt3bcpCLruyNHRCTfgJKl99GaIrjYoMhfqPyv7X5bFKTATCt0-2X4-RSFyXiwjVRPYeOVxjWCx6VDDRIBg'  -d '{ "entrances":{"name":"xiaoz",   "nick":"daZ","avatar":"",   "guide":"imguide",   "consult_ids":[1],   "change_default_time":"2018-12-12",   "h5_link":"",   "web_link":"",   "internal_parameters":"",   "app_parameters":"",   "access_link":"",  "default_consult_id":1, "certificate":""  }}
'      http://127.0.0.1:8887/v1/tenant/entrance/create-entrance



创建匿名用户

curl -X POST -H 'dapr-app-id:core' -H "Content-type: application/json" -H 'X-Token: CAEQAxgBIAIo-qPkie8x.EsmDHt3bcpCLruyNHRCTfgJKl99GaIrjYoMhfqPyv7X5bFKTATCt0-2X4-RSFyXiwjVRPYeOVxjWCx6VDDRIBg' -d '{"tenant_id":1, "entrance_id":1,   "ip":"127.0.0.1"}	'  http://127.0.0.1:8887/v1/api/create-anon-user




 curl -X POST -H 'dapr-app-id:core' -H "Content-type: application/json" -H 'X-Token: COcHEAYY5gcg5Qco6rrY5O4x.IrxGn7rCwrNtTCVvTizaW93h_BCd0KfudsEDvmr-M9NvDC1gXUpNHkoGdhqfJnoBcduqKJoz7qpEkkXCKcznAQ' -d '{"chat_id":1}'  http://127.0.0.1:8887/v1/tenant/worker/transfer


快速回复

 curl -X POST -H 'dapr-app-id:core' -H "Content-type: application/json" -H 'X-Token: CH0QAxgBIO0GKJDPwqrvMQ.ECw9ECmi32CDsl1I3INVUl-mscKMZW5hpziyCnnqLlcILmP1RvehBmn4lSH6f4NNEaj-BVCYm4Nz5GtKj_JbCQ' -d '{"page":1,"limit":5}'  http://127.0.0.1:8887/v1/tenant/quick-reply/query-reply


快速回复2

 curl -X POST -H 'dapr-app-id:core' -H "Content-type: application/json" -H 'X-Token: CH0QAxgBIO0GKJDPwqrvMQ.ECw9ECmi32CDsl1I3INVUl-mscKMZW5hpziyCnnqLlcILmP1RvehBmn4lSH6f4NNEaj-BVCYm4Nz5GtKj_JbCQ' -d '{"page":1,"limit":5}'  http://127.0.0.1:8887/v1/tenant/auto-reply/tenant


查询组

curl -X POST -H 'dapr-app-id:core' -H "Content-type: application/json;charset=utf-8" -H 'X-Token: CAEQAxgBILoBKITQuIbwMQ.rFCnW6zYQig51jOYpvRbnEx9DUf28yNGCNes5JVw7rToGkEjSZctLP2csJoz8NxFDdjuB3EtyOxTGkOVdYTlBw' -d '{"group_child_id":1,"group_id":0,"batch":{"offset":1,"limit":5}}'  http://127.0.0.1:8887/v1/tenant/worker-group/query

查询入口：
curl -X POST -H 'dapr-app-id:core' -H "Content-type: application/json;charset=utf-8" -H 'X-Token: CAEQAxgBILoBKITQuIbwMQ.rFCnW6zYQig51jOYpvRbnEx9DUf28yNGCNes5JVw7rToGkEjSZctLP2csJoz8NxFDdjuB3EtyOxTGkOVdYTlBw' -d '{"group_child_id":1,"page":1,"limit":5}'  http://127.0.0.1:8887/v1/tenant/entrance/query-entrance

 




export http_proxy=http://127.0.0.1:11346  ; export   https_proxy=http://127.0.0.1:11346


代码问题
1. 很多泛类型
2. 变量名只有一两个字符:  w(websocket)    l(lanch)     tm(tanlantModel)     wm(workerModel)    v at(autToken)
3. 代码没有注释
4. 很多地方用了反射机制 event loop ，函数被注册到两个队列中 sync async 中
6. 包里各种大量的 ：init   函数  var 全局变量


商户  -> 创建咨询 -> 设置预接待 worker

商户 -> 创建 worker

entrances  custom  匿名账号




grep -R 'limit' /e/project/tg/wcs/*


println("step 1 =========================== OwnerId:", at.OwnerId, " TenantId:", at.TenantId)