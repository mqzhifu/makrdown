# linkerd

```
admin:
  ip: 0.0.0.0
  port: 9990

usage:
  enabled: false

telemetry:
- kind: io.l5d.prometheus
- kind: io.l5d.zipkin
  ##IP地址根据环境更改
  host: 172.19.162.237
  port: 9410
  sampleRate: 1.00

namers:
  - kind: io.l5d.consul
    ##IP地址根据环境更改
    host: 172.19.162.237
    port: 8500

routers:
- protocol: http
  #admin UI label ，上线的时候把test改成online
  label: service_http_consul_test
  bindingTimeoutMs: 5000
  interpreter:
    kind: default
  compressionLevel: -1
  identifier:
    kind: io.l5d.header.token
  service:
    kind: io.l5d.global
    totalTimeoutMs: 10000
    responseClassifier:
      kind: io.l5d.http.retryableAll5XX
    retries:
      budget:
        minRetriesPerSec: 1
        percentCanRetry: 1.0
      backoff:
        kind: jittered
        minMs: 1000
        maxMs: 5000
  client:
    hostConnectionPool:
      minSize: 0
      maxSize: 2000
      idleTimeMs: 60000
      maxWaiters: 500
    failFast: false
    requeueBudget:
      percentCanRetry: 1.0
    failureAccrual:
      kind: io.l5d.successRateWindowed
      successRate: 0.95
      window: 300
      backoff:
        kind: jittered
        minMs: 5000
        maxMs: 60000
    loadBalancer:
      kind: ewma
      maxEffort: 5
      decayTimeMs: 10000
  httpAccessLog: logs/access.log
  httpAccessLogRollPolicy: daily
  httpAccessLogAppend: true
  httpAccessLogRotateCount: 7
  bindingCache:
    paths: 1000
    trees: 1000
    bounds: 1000
    clients: 1000
    idleTtlSecs: 600
  dtab: |
    ##dc名字根据环境更改
    /svc => /#/io.l5d.consul/dc-test01;
  servers:
  - port: 4140
    ip: 0.0.0.0
    addForwardedHeader:
      by: {kind: "ip:port"}
      for: {kind: ip}
```

%60%60%60%0Aadmin%3A%0A%20%20ip%3A%200.0.0.0%0A%20%20port%3A%209990%0A%0Ausage%3A%0A%20%20enabled%3A%20false%0A%0Atelemetry%3A%0A\-%20kind%3A%20io.l5d.prometheus%0A\-%20kind%3A%20io.l5d.zipkin%0A%20%20%23%23IP%E5%9C%B0%E5%9D%80%E6%A0%B9%E6%8D%AE%E7%8E%AF%E5%A2%83%E6%9B%B4%E6%94%B9%0A%20%20host%3A%20172.19.162.237%0A%20%20port%3A%209410%0A%20%20sampleRate%3A%201.00%0A%0Anamers%3A%0A%20%20\-%20kind%3A%20io.l5d.consul%0A%20%20%20%20%23%23IP%E5%9C%B0%E5%9D%80%E6%A0%B9%E6%8D%AE%E7%8E%AF%E5%A2%83%E6%9B%B4%E6%94%B9%0A%20%20%20%20host%3A%20172.19.162.237%0A%20%20%20%20port%3A%208500%0A%0Arouters%3A%0A\-%20protocol%3A%20http%0A%20%20%23admin%20UI%20label%20%EF%BC%8C%E4%B8%8A%E7%BA%BF%E7%9A%84%E6%97%B6%E5%80%99%E6%8A%8Atest%E6%94%B9%E6%88%90online%0A%20%20label%3A%20service\_http\_consul\_test%0A%20%20bindingTimeoutMs%3A%205000%0A%20%20interpreter%3A%0A%20%20%20%20kind%3A%20default%0A%20%20compressionLevel%3A%20\-1%0A%20%20identifier%3A%0A%20%20%20%20kind%3A%20io.l5d.header.token%0A%20%20service%3A%0A%20%20%20%20kind%3A%20io.l5d.global%0A%20%20%20%20totalTimeoutMs%3A%2010000%0A%20%20%20%20responseClassifier%3A%0A%20%20%20%20%20%20kind%3A%20io.l5d.http.retryableAll5XX%0A%20%20%20%20retries%3A%0A%20%20%20%20%20%20budget%3A%0A%20%20%20%20%20%20%20%20minRetriesPerSec%3A%201%0A%20%20%20%20%20%20%20%20percentCanRetry%3A%201.0%0A%20%20%20%20%20%20backoff%3A%0A%20%20%20%20%20%20%20%20kind%3A%20jittered%0A%20%20%20%20%20%20%20%20minMs%3A%201000%0A%20%20%20%20%20%20%20%20maxMs%3A%205000%0A%20%20client%3A%0A%20%20%20%20hostConnectionPool%3A%0A%20%20%20%20%20%20minSize%3A%200%0A%20%20%20%20%20%20maxSize%3A%202000%0A%20%20%20%20%20%20idleTimeMs%3A%2060000%0A%20%20%20%20%20%20maxWaiters%3A%20500%0A%20%20%20%20failFast%3A%20false%0A%20%20%20%20requeueBudget%3A%0A%20%20%20%20%20%20percentCanRetry%3A%201.0%0A%20%20%20%20failureAccrual%3A%0A%20%20%20%20%20%20kind%3A%20io.l5d.successRateWindowed%0A%20%20%20%20%20%20successRate%3A%200.95%0A%20%20%20%20%20%20window%3A%20300%0A%20%20%20%20%20%20backoff%3A%0A%20%20%20%20%20%20%20%20kind%3A%20jittered%0A%20%20%20%20%20%20%20%20minMs%3A%205000%0A%20%20%20%20%20%20%20%20maxMs%3A%2060000%0A%20%20%20%20loadBalancer%3A%0A%20%20%20%20%20%20kind%3A%20ewma%0A%20%20%20%20%20%20maxEffort%3A%205%0A%20%20%20%20%20%20decayTimeMs%3A%2010000%0A%20%20httpAccessLog%3A%20logs%2Faccess.log%0A%20%20httpAccessLogRollPolicy%3A%20daily%0A%20%20httpAccessLogAppend%3A%20true%0A%20%20httpAccessLogRotateCount%3A%207%0A%20%20bindingCache%3A%0A%20%20%20%20paths%3A%201000%0A%20%20%20%20trees%3A%201000%0A%20%20%20%20bounds%3A%201000%0A%20%20%20%20clients%3A%201000%0A%20%20%20%20idleTtlSecs%3A%20600%0A%20%20dtab%3A%20%7C%0A%20%20%20%20%23%23dc%E5%90%8D%E5%AD%97%E6%A0%B9%E6%8D%AE%E7%8E%AF%E5%A2%83%E6%9B%B4%E6%94%B9%0A%20%20%20%20%2Fsvc%20%3D%3E%20%2F%23%2Fio.l5d.consul%2Fdc\-test01%3B%0A%20%20servers%3A%0A%20%20\-%20port%3A%204140%0A%20%20%20%20ip%3A%200.0.0.0%0A%20%20%20%20addForwardedHeader%3A%0A%20%20%20%20%20%20by%3A%20%7Bkind%3A%20%22ip%3Aport%22%7D%0A%20%20%20%20%20%20for%3A%20%7Bkind%3A%20ip%7D%0A%60%60%60
