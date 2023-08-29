# prometheus.yml

```
# my global config                       
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.                                              
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.                           
  # scrape_timeout is set to the global default (10s). 
# Alertmanager configuration                                                                               
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['127.0.0.1:9093']                                   
      #- "127.0.0.1:9093"                                                 
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.                                                                                 
rule_files:
  - "first_rules.yml"
  # - "second_rules.yml"
  # A scrape configuration containing exactly one endpoint to scrape:
  # Here it's Prometheus itself.
  
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.                                                                                          
  - job_name: 'prometheus'
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.                                          
    
    static_configs:
    - targets: ['localhost:9090']
    
  - job_name: 'centos_node'
    static_configs:
      - targets: ['39.106.65.76:9100']
      
  - job_name: 'apache_node'
    static_configs:
      - targets: ['39.106.65.76:9117']
      
  - job_name: 'nginx-vts'
    static_configs:
      - targets: ['39.106.65.76:9913']
      
  - job_name: 'php-fpm-exporter'
    static_configs:
     - targets: ['39.106.65.76:9190']
     
  - job_name: 'mysqld-exporter'
    static_configs:
     - targets: ['39.106.65.76:9104']
     
  - job_name: 'redis-exporter'
    static_configs:
     - targets: ['39.106.65.76:27000']
     
  - job_name: 'rabbitmq_prometheus'
    static_configs:
     - targets: ['39.106.65.76:27000']
     
```

%60%60%60%0A%23%20my%20global%20config%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0Aglobal%3A%0A%20%20scrape\_interval%3A%20%20%20%20%2015s%20%23%20Set%20the%20scrape%20interval%20to%20every%2015%20seconds.%20Default%20is%20every%201%20minute.%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20evaluation\_interval%3A%2015s%20%23%20Evaluate%20rules%20every%2015%20seconds.%20The%20default%20is%20every%201%20minute.%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%23%20scrape\_timeout%20is%20set%20to%20the%20global%20default%20\(10s\).%20%0A%23%20Alertmanager%20configuration%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0Aalerting%3A%0A%20%20alertmanagers%3A%0A%20%20\-%20static\_configs%3A%0A%20%20%20%20\-%20targets%3A%20%5B'127.0.0.1%3A9093'%5D%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%23\-%20%22127.0.0.1%3A9093%22%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0A%23%20Load%20rules%20once%20and%20periodically%20evaluate%20them%20according%20to%20the%20global%20'evaluation\_interval'.%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0Arule\_files%3A%0A%20%20\-%20%22first\_rules.yml%22%0A%20%20%23%20\-%20%22second\_rules.yml%22%0A%20%20%23%20A%20scrape%20configuration%20containing%20exactly%20one%20endpoint%20to%20scrape%3A%0A%20%20%23%20Here%20it's%20Prometheus%20itself.%0A%20%20%0Ascrape\_configs%3A%0A%20%20%23%20The%20job%20name%20is%20added%20as%20a%20label%20%60job%3D%3Cjob\_name%3E%60%20to%20any%20timeseries%20scraped%20from%20this%20config.%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20\-%20job\_name%3A%20'prometheus'%0A%20%20%20%20%23%20metrics\_path%20defaults%20to%20'%2Fmetrics'%0A%20%20%20%20%23%20scheme%20defaults%20to%20'http'.%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20static\_configs%3A%0A%20%20%20%20\-%20targets%3A%20%5B'localhost%3A9090'%5D%0A%20%20%20%20%0A%20%20\-%20job\_name%3A%20'centos\_node'%0A%20%20%20%20static\_configs%3A%0A%20%20%20%20%20%20\-%20targets%3A%20%5B'39.106.65.76%3A9100'%5D%0A%20%20%20%20%20%20%0A%20%20\-%20job\_name%3A%20'apache\_node'%0A%20%20%20%20static\_configs%3A%0A%20%20%20%20%20%20\-%20targets%3A%20%5B'39.106.65.76%3A9117'%5D%0A%20%20%20%20%20%20%0A%20%20\-%20job\_name%3A%20'nginx\-vts'%0A%20%20%20%20static\_configs%3A%0A%20%20%20%20%20%20\-%20targets%3A%20%5B'39.106.65.76%3A9913'%5D%0A%20%20%20%20%20%20%0A%20%20\-%20job\_name%3A%20'php\-fpm\-exporter'%0A%20%20%20%20static\_configs%3A%0A%20%20%20%20%20\-%20targets%3A%20%5B'39.106.65.76%3A9190'%5D%0A%20%20%20%20%20%0A%20%20\-%20job\_name%3A%20'mysqld\-exporter'%0A%20%20%20%20static\_configs%3A%0A%20%20%20%20%20\-%20targets%3A%20%5B'39.106.65.76%3A9104'%5D%0A%20%20%20%20%20%0A%20%20\-%20job\_name%3A%20'redis\-exporter'%0A%20%20%20%20static\_configs%3A%0A%20%20%20%20%20\-%20targets%3A%20%5B'39.106.65.76%3A27000'%5D%0A%20%20%20%20%20%0A%20%20\-%20job\_name%3A%20'rabbitmq\_prometheus'%0A%20%20%20%20static\_configs%3A%0A%20%20%20%20%20\-%20targets%3A%20%5B'39.106.65.76%3A27000'%5D%0A%20%20%20%20%20%0A%60%60%60
