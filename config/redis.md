# redis

```
bind 127.0.0.1
port 6379
daemonize yes
timeout 60
pidfile /var/run/redis_6379.pid
loglevel notice
logfile "/data/logs/redis/redis.log"

maxmemory 256MB

#持久化-快照模式 ，系统触发机制 ,bgsave 开新纯种后台异步
save 900 1：表示900 秒内如果至少有 1 个 key 的值变化，则保存
save 300 10：表示300 秒内如果至少有 10 个 key 的值变化，则保存
save 60 10000：表示60 秒内如果至少有 10000 个 key 的值变化，则保存

rdbcompression no  #不要压缩RDB，占CPU
dbfilename "6379.rdb"
dir /data/logs/redis
stop-writes-on-bgsave-error yes #bgsave持久化rdb时，如果失败了，redis就自动停掉

#异步删除
lazyfree-lazy-eviction yes
lazyfree-lazy-expire yes
lazyfree-lazy-server-del yes
replica-lazy-flush yes

slowlog-log-slower-than 5000
slowlog-max-len 1000 #慢日志，存储多少条

```

%0A%60%60%60%0Abind%20127.0.0.1%0Aport%206379%0Adaemonize%20yes%0Atimeout%2060%0Apidfile%20%2Fvar%2Frun%2Fredis\_6379.pid%0Aloglevel%20notice%0Alogfile%20%22%2Fdata%2Flogs%2Fredis%2Fredis.log%22%0A%0A%0Amaxmemory%20256MB%0A%0A%23%E6%8C%81%E4%B9%85%E5%8C%96\-%E5%BF%AB%E7%85%A7%E6%A8%A1%E5%BC%8F%20%EF%BC%8C%E7%B3%BB%E7%BB%9F%E8%A7%A6%E5%8F%91%E6%9C%BA%E5%88%B6%20%2Cbgsave%20%E5%BC%80%E6%96%B0%E7%BA%AF%E7%A7%8D%E5%90%8E%E5%8F%B0%E5%BC%82%E6%AD%A5%0Asave%20900%201%EF%BC%9A%E8%A1%A8%E7%A4%BA900%20%E7%A7%92%E5%86%85%E5%A6%82%E6%9E%9C%E8%87%B3%E5%B0%91%E6%9C%89%201%20%E4%B8%AA%20key%20%E7%9A%84%E5%80%BC%E5%8F%98%E5%8C%96%EF%BC%8C%E5%88%99%E4%BF%9D%E5%AD%98%0Asave%20300%2010%EF%BC%9A%E8%A1%A8%E7%A4%BA300%20%E7%A7%92%E5%86%85%E5%A6%82%E6%9E%9C%E8%87%B3%E5%B0%91%E6%9C%89%2010%20%E4%B8%AA%20key%20%E7%9A%84%E5%80%BC%E5%8F%98%E5%8C%96%EF%BC%8C%E5%88%99%E4%BF%9D%E5%AD%98%0Asave%2060%2010000%EF%BC%9A%E8%A1%A8%E7%A4%BA60%20%E7%A7%92%E5%86%85%E5%A6%82%E6%9E%9C%E8%87%B3%E5%B0%91%E6%9C%89%2010000%20%E4%B8%AA%20key%20%E7%9A%84%E5%80%BC%E5%8F%98%E5%8C%96%EF%BC%8C%E5%88%99%E4%BF%9D%E5%AD%98%0A%0Ardbcompression%20no%20%20%23%E4%B8%8D%E8%A6%81%E5%8E%8B%E7%BC%A9RDB%EF%BC%8C%E5%8D%A0CPU%0Adbfilename%20%226379.rdb%22%0Adir%20%2Fdata%2Flogs%2Fredis%0Astop\-writes\-on\-bgsave\-error%20yes%20%23bgsave%E6%8C%81%E4%B9%85%E5%8C%96rdb%E6%97%B6%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%A4%B1%E8%B4%A5%E4%BA%86%EF%BC%8Credis%E5%B0%B1%E8%87%AA%E5%8A%A8%E5%81%9C%E6%8E%89%0A%0A%23%E5%BC%82%E6%AD%A5%E5%88%A0%E9%99%A4%0Alazyfree\-lazy\-eviction%20yes%0Alazyfree\-lazy\-expire%20yes%0Alazyfree\-lazy\-server\-del%20yes%0Areplica\-lazy\-flush%20yes%0A%0Aslowlog\-log\-slower\-than%205000%0Aslowlog\-max\-len%201000%20%23%E6%85%A2%E6%97%A5%E5%BF%97%EF%BC%8C%E5%AD%98%E5%82%A8%E5%A4%9A%E5%B0%91%E6%9D%A1%0A%0A%60%60%60
