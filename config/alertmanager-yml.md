# alertmanager.yml

```
global:
  resolve_timeout: 5m

  smtp_from: '78878296@qq.com'
  smtp_smarthost: 'smtp.qq.com:465'
  smtp_auth_username: '78878296@qq.com'
  smtp_auth_password: 'glnteewafftmcaje'
  smtp_require_tls: false
  smtp_hello: 'qq.com'

route:
  #这里先做一次大分组,下面的routers会再细分分组
  group_by: ['alertname']
  group_wait: 1s
  group_interval: 10s
  repeat_interval: 10s
  receiver: 'op'
  routes:
  - match:
      severity: 'critical'
      receiver: op
  - match:
      service: "frame_sync"
      receiver: backend
  - match:
      service: "app"
      receiver: front

receivers:
- name: 'op'
  email_configs:
  - to: 'mqzhifu@sina.com'
    send_resolved: true

- name: 'front'
  email_configs:
  - to: 'mqzhifu@sina.com'
    send_resolved: true
  webhook_configs:
  - url: 'http://127.0.0.1:8088/alert.php'

- name: 'backend'
  webhook_configs:
  - url: 'http://127.0.0.1:8088/alert.php'

#inhibit_rules:
#  - source_match:
#      severity: 'critical'
#    target_match:
#      severity: 'warning'
#    equal: ['alertname', 'dev', 'instance']
```

%60%60%60%0Aglobal%3A%0A%20%20resolve\_timeout%3A%205m%0A%0A%20%20smtp\_from%3A%20'78878296%40qq.com'%0A%20%20smtp\_smarthost%3A%20'smtp.qq.com%3A465'%0A%20%20smtp\_auth\_username%3A%20'78878296%40qq.com'%0A%20%20smtp\_auth\_password%3A%20'glnteewafftmcaje'%0A%20%20smtp\_require\_tls%3A%20false%0A%20%20smtp\_hello%3A%20'qq.com'%0A%0Aroute%3A%0A%20%20%23%E8%BF%99%E9%87%8C%E5%85%88%E5%81%9A%E4%B8%80%E6%AC%A1%E5%A4%A7%E5%88%86%E7%BB%84%2C%E4%B8%8B%E9%9D%A2%E7%9A%84routers%E4%BC%9A%E5%86%8D%E7%BB%86%E5%88%86%E5%88%86%E7%BB%84%0A%20%20group\_by%3A%20%5B'alertname'%5D%0A%20%20group\_wait%3A%201s%0A%20%20group\_interval%3A%2010s%0A%20%20repeat\_interval%3A%2010s%0A%20%20receiver%3A%20'op'%0A%20%20routes%3A%0A%20%20\-%20match%3A%0A%20%20%20%20%20%20severity%3A%20'critical'%0A%20%20%20%20%20%20receiver%3A%20op%0A%20%20\-%20match%3A%0A%20%20%20%20%20%20service%3A%20%22frame\_sync%22%0A%20%20%20%20%20%20receiver%3A%20backend%0A%20%20\-%20match%3A%0A%20%20%20%20%20%20service%3A%20%22app%22%0A%20%20%20%20%20%20receiver%3A%20front%0A%0A%0Areceivers%3A%0A\-%20name%3A%20'op'%0A%20%20email\_configs%3A%0A%20%20\-%20to%3A%20'mqzhifu%40sina.com'%0A%20%20%20%20send\_resolved%3A%20true%0A%0A%0A\-%20name%3A%20'front'%0A%20%20email\_configs%3A%0A%20%20\-%20to%3A%20'mqzhifu%40sina.com'%0A%20%20%20%20send\_resolved%3A%20true%0A%20%20webhook\_configs%3A%0A%20%20\-%20url%3A%20'http%3A%2F%2F127.0.0.1%3A8088%2Falert.php'%0A%0A\-%20name%3A%20'backend'%0A%20%20webhook\_configs%3A%0A%20%20\-%20url%3A%20'http%3A%2F%2F127.0.0.1%3A8088%2Falert.php'%0A%0A%0A%23inhibit\_rules%3A%0A%23%20%20\-%20source\_match%3A%0A%23%20%20%20%20%20%20severity%3A%20'critical'%0A%23%20%20%20%20target\_match%3A%0A%23%20%20%20%20%20%20severity%3A%20'warning'%0A%23%20%20%20%20equal%3A%20%5B'alertname'%2C%20'dev'%2C%20'instance'%5D%0A%60%60%60
