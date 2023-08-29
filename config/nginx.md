# nginx

## nginx.conf

```
#进程启动的用户名
user www;
#进程ID存在路径
pid logs/nginx.pid;

#woker 进程数量。建议与CPU核数对应，理想状态下：WORK 进程会分散在各个CPU上
worker_processes auto;
#WORK进程绑定CPU，避免 进程被切换到其它CPU上，更新CPU缓存
worker_cpu_affinity auto;
#错误日志位置
error_log /data/logs/nginx/error.log notice;

event{
    use epoll;
    worker_connections 1024;#每个work最大
    multi_accept on;#设置一个进程是否同时接受多个网络连接，默认为off
    accept_mutex on; #避免惊群,默认打开
}

http {
    #隐藏版本号，正常HTTP交互，header中Server值会显示nginx/18.0，还有404会直接把版本号显示在页面中
    server_tokens off;

    #标签：main ，缓存32K内存后输出到硬盘，每5秒flush一次，注：得配置log_format一起使用，不然报错
    access_log /data/logs/nginx/access.log main gzip buffer=32k flush=5s;

    #nginx读取本地文件，需要用户模式/系统模式切换，开启后，直接让CPU来读文件，0-copy
    sendfile on;
    #数据包会累计到一定大小之后才会发送，减小了额外开销(开启sendfile此参数才生效)
    tcp_nopush on;
    #禁用 Nagle 算法，只对keep-alive的TCP 连接有用，好像与tcp_nopush互斥
    tcp_delay on;
    #长连接超时时间
    keepalived_timeout 60;

    #C端最大上传文件
    client_max_body_size 2m;
    #C端上传的文件，小于如下值会保存在内存中，如果大于此值会放于文件
    client_body_buffer_size 1m;

    #缓存(传输数据)设置
    gzip on;
    gzip_comp_level 5; # 1-9
    gzip_min_length 100k;
    gzip_types application/javascript text/css text/xml;
    gzip_vary on; #告诉客户端开启了压缩

    #fastcgi php
    fastcgi_connect_timeout 60;#cgi 连接超时60s
    #fastcgi_send_timeout;#向cgi发送时间
    fastcgi_read_timeout 60;#接收php响应时间

    #申请一块区域10M，存储$binary_remote_addr(ip)变量的统计(限制IP)
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    
    #申请一块区域10M，存储$binary_remote_addr(domain)变量的统计
    limit_conn_zone $server_name zone=perserver:10m;
    
    #申请一块区域10M，存储$binary_remote_addr(ip)变量的统计，每秒只能有一个请求进来(限制一个IP的请求数)
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=1r/s;
    
    location / {
        #一个IP同一时间最多一个连接,addr 对应上面的 addr
        limit_conn addr 1;
        #每个连接限速300K,是针对连接的，如果一个IP最多可以有2个连接，那就是每个连接300K
        limit_rate 300k;
        #设置超出最大连接数的日志级别
        limit_conn_log_level error;
        #当有请求过来触发了最大请求数，由burst缓存住5个请求
        #nodelay：当请求频次 被限制burst缓存也满了，直接扔掉,return 503
        limit_req zone=mylimit burst=5 nodelay;
    }
    
    include vhost/*.conf;
    

}

```

## metrics

```

vhost_traffic_status_zone;
server {
    listen 9111;
    location /status {
    vhost_traffic_status_display;
    vhost_traffic_status_display_format html;
    }
}
```

#### server

```
server {
    autoindex off;
    access_log /data/logs/nginx/agent.xlsyfx.cn.access.log;
    server_name agent.xlsyfx.cn;

    listen 80;
    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/xlsyfx.cn/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/xlsyfx.cn/privkey.pem;

    ssl_session_timeout 5m;
    ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers "HIGH:!aNULL:!MD5 or HIGH:!aNULL:!MD5:!3DES";
    ssl_prefer_server_ciphers on;

    error_page 404 /404.html;
        location = /404.html {
        root /data/www;
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        root /data/www;
    }

    root /data/www;
        location / {
        index index.php index.html;
    }
    
    #图片映射本机硬盘地址
    location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
        root /data/www/php/zhongyuhuacai/static;
    }
    #特殊文件映射本机硬盘地址
    location ~* \.(eot|otf|ttf|woff|woff2)$ {
        root /data/www/php/zhongyuhuacai/static;
        add_header Access-Control-Allow-Origin *;
    }

    #对URI进行转换
    location ~ / {
        if (!-e $request_filename) {
            rewrite ^/(\w+)/(\w+)/(.*)$ /index.php?ctrl=$1&ac=$2&$3 last;
        }
    }

    #解析PHP
    location ~ \.php$ {
        root /data/www;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        #fastcgi_param SCRIPT_FILENAME /data/www/php/zhongyuhuacai/api$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_param HTTP_X_REAL_IP $HTTP_X_REAL_IP;
    }
}

```

## 代理/负载均衡

```
upstream my_server {
    server 127.0.0.1:8080 weight=5;
    server 10.0.0.2:8080;
    keepalive 2000;
}
server {
    listen 80;
    server_name 10.0.0.1;
    location /my/ {
    proxy_pass http://my_server/;
    proxy_set_header Host $host:$server_port;
    }
}
```

## trace\_id

```
server {  
        lua_code_cache off;
        location ~ \.php$ {
            set $first 0;
            if ( $http_x_trace_){
                set $first 1;
            }

            if ( $http_x_trace_id = 0){
                set $first 1;
            }

            if ( $http_x_trace_){
                set $first 1;
            }

            set_by_lua $span_id 'return "200000"..os.time()';

            if ( $first = 1 ){
                set_by_lua_file $trace_id /home/www/test_lua/trace_id.lua;
                set $first 1;
            }

            if ( $first = 0 ){
                set $trace_id $http_x_trace_id;
                set $parent_id $http_x_span_id;
            }

            more_set_input_headers "X-Parent-Id:$parent_id";
            more_set_input_headers "X-Trace-Id:$trace_id";
            more_set_input_headers "X-Span-Id:$span_id";
            more_set_input_headers "X-From-Service:$server_name";
            more_set_input_headers "X-First:$first";
        
        }

}
```

## VUE

```
server {

    location / {
        root /data/www/Digitaltwin/master/dist;
        try_files $uri $uri/ @router;
        index index.html;
    }

    location @router {
        rewrite ^.*$ /index.html last;
    }
}
```

## golang

```
server {
    location / {
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header  REMOTE-HOST $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;

    
        proxy_pass http://127.0.0.1:5555;

        proxy_connect_timeout 30;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
    }
}
```

nginx.conf%0A\-\-\-\-\-%0A%60%60%60%0A%23%E8%BF%9B%E7%A8%8B%E5%90%AF%E5%8A%A8%E7%9A%84%E7%94%A8%E6%88%B7%E5%90%8D%0Auser%20www%3B%0A%23%E8%BF%9B%E7%A8%8BID%E5%AD%98%E5%9C%A8%E8%B7%AF%E5%BE%84%0Apid%20logs%2Fnginx.pid%3B%0A%0A%23woker%20%E8%BF%9B%E7%A8%8B%E6%95%B0%E9%87%8F%E3%80%82%E5%BB%BA%E8%AE%AE%E4%B8%8ECPU%E6%A0%B8%E6%95%B0%E5%AF%B9%E5%BA%94%EF%BC%8C%E7%90%86%E6%83%B3%E7%8A%B6%E6%80%81%E4%B8%8B%EF%BC%9AWORK%20%E8%BF%9B%E7%A8%8B%E4%BC%9A%E5%88%86%E6%95%A3%E5%9C%A8%E5%90%84%E4%B8%AACPU%E4%B8%8A%0Aworker\_processes%20auto%3B%0A%23WORK%E8%BF%9B%E7%A8%8B%E7%BB%91%E5%AE%9ACPU%EF%BC%8C%E9%81%BF%E5%85%8D%20%E8%BF%9B%E7%A8%8B%E8%A2%AB%E5%88%87%E6%8D%A2%E5%88%B0%E5%85%B6%E5%AE%83CPU%E4%B8%8A%EF%BC%8C%E6%9B%B4%E6%96%B0CPU%E7%BC%93%E5%AD%98%0Aworker\_cpu\_affinity%20auto%3B%0A%23%E9%94%99%E8%AF%AF%E6%97%A5%E5%BF%97%E4%BD%8D%E7%BD%AE%0Aerror\_log%20%2Fdata%2Flogs%2Fnginx%2Ferror.log%20notice%3B%0A%0Aevent%7B%0A%20%20%20%20use%20epoll%3B%0A%20%20%20%20worker\_connections%201024%3B%23%E6%AF%8F%E4%B8%AAwork%E6%9C%80%E5%A4%A7%0A%20%20%20%20multi\_accept%20on%3B%23%E8%AE%BE%E7%BD%AE%E4%B8%80%E4%B8%AA%E8%BF%9B%E7%A8%8B%E6%98%AF%E5%90%A6%E5%90%8C%E6%97%B6%E6%8E%A5%E5%8F%97%E5%A4%9A%E4%B8%AA%E7%BD%91%E7%BB%9C%E8%BF%9E%E6%8E%A5%EF%BC%8C%E9%BB%98%E8%AE%A4%E4%B8%BAoff%0A%20%20%20%20accept\_mutex%20on%3B%20%23%E9%81%BF%E5%85%8D%E6%83%8A%E7%BE%A4%2C%E9%BB%98%E8%AE%A4%E6%89%93%E5%BC%80%0A%7D%0A%0Ahttp%20%7B%0A%20%20%20%20%23%E9%9A%90%E8%97%8F%E7%89%88%E6%9C%AC%E5%8F%B7%EF%BC%8C%E6%AD%A3%E5%B8%B8HTTP%E4%BA%A4%E4%BA%92%EF%BC%8Cheader%E4%B8%ADServer%E5%80%BC%E4%BC%9A%E6%98%BE%E7%A4%BAnginx%2F18.0%EF%BC%8C%E8%BF%98%E6%9C%89404%E4%BC%9A%E7%9B%B4%E6%8E%A5%E6%8A%8A%E7%89%88%E6%9C%AC%E5%8F%B7%E6%98%BE%E7%A4%BA%E5%9C%A8%E9%A1%B5%E9%9D%A2%E4%B8%AD%0A%20%20%20%20server\_tokens%20off%3B%0A%0A%20%20%20%20%23%E6%A0%87%E7%AD%BE%EF%BC%9Amain%20%EF%BC%8C%E7%BC%93%E5%AD%9832K%E5%86%85%E5%AD%98%E5%90%8E%E8%BE%93%E5%87%BA%E5%88%B0%E7%A1%AC%E7%9B%98%EF%BC%8C%E6%AF%8F5%E7%A7%92flush%E4%B8%80%E6%AC%A1%EF%BC%8C%E6%B3%A8%EF%BC%9A%E5%BE%97%E9%85%8D%E7%BD%AElog\_format%E4%B8%80%E8%B5%B7%E4%BD%BF%E7%94%A8%EF%BC%8C%E4%B8%8D%E7%84%B6%E6%8A%A5%E9%94%99%0A%20%20%20%20access\_log%20%2Fdata%2Flogs%2Fnginx%2Faccess.log%20main%20gzip%20buffer%3D32k%20flush%3D5s%3B%0A%0A%20%20%20%20%23nginx%E8%AF%BB%E5%8F%96%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6%EF%BC%8C%E9%9C%80%E8%A6%81%E7%94%A8%E6%88%B7%E6%A8%A1%E5%BC%8F%2F%E7%B3%BB%E7%BB%9F%E6%A8%A1%E5%BC%8F%E5%88%87%E6%8D%A2%EF%BC%8C%E5%BC%80%E5%90%AF%E5%90%8E%EF%BC%8C%E7%9B%B4%E6%8E%A5%E8%AE%A9CPU%E6%9D%A5%E8%AF%BB%E6%96%87%E4%BB%B6%EF%BC%8C0\-copy%0A%20%20%20%20sendfile%20on%3B%0A%20%20%20%20%23%E6%95%B0%E6%8D%AE%E5%8C%85%E4%BC%9A%E7%B4%AF%E8%AE%A1%E5%88%B0%E4%B8%80%E5%AE%9A%E5%A4%A7%E5%B0%8F%E4%B9%8B%E5%90%8E%E6%89%8D%E4%BC%9A%E5%8F%91%E9%80%81%EF%BC%8C%E5%87%8F%E5%B0%8F%E4%BA%86%E9%A2%9D%E5%A4%96%E5%BC%80%E9%94%80\(%E5%BC%80%E5%90%AFsendfile%E6%AD%A4%E5%8F%82%E6%95%B0%E6%89%8D%E7%94%9F%E6%95%88\)%0A%20%20%20%20tcp\_nopush%20on%3B%0A%20%20%20%20%23%E7%A6%81%E7%94%A8%20Nagle%20%E7%AE%97%E6%B3%95%EF%BC%8C%E5%8F%AA%E5%AF%B9keep\-alive%E7%9A%84TCP%20%E8%BF%9E%E6%8E%A5%E6%9C%89%E7%94%A8%EF%BC%8C%E5%A5%BD%E5%83%8F%E4%B8%8Etcp\_nopush%E4%BA%92%E6%96%A5%0A%20%20%20%20tcp\_delay%20on%3B%0A%20%20%20%20%23%E9%95%BF%E8%BF%9E%E6%8E%A5%E8%B6%85%E6%97%B6%E6%97%B6%E9%97%B4%0A%20%20%20%20keepalived\_timeout%2060%3B%0A%0A%20%20%20%20%23C%E7%AB%AF%E6%9C%80%E5%A4%A7%E4%B8%8A%E4%BC%A0%E6%96%87%E4%BB%B6%0A%20%20%20%20client\_max\_body\_size%202m%3B%0A%20%20%20%20%23C%E7%AB%AF%E4%B8%8A%E4%BC%A0%E7%9A%84%E6%96%87%E4%BB%B6%EF%BC%8C%E5%B0%8F%E4%BA%8E%E5%A6%82%E4%B8%8B%E5%80%BC%E4%BC%9A%E4%BF%9D%E5%AD%98%E5%9C%A8%E5%86%85%E5%AD%98%E4%B8%AD%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%A4%A7%E4%BA%8E%E6%AD%A4%E5%80%BC%E4%BC%9A%E6%94%BE%E4%BA%8E%E6%96%87%E4%BB%B6%0A%20%20%20%20client\_body\_buffer\_size%201m%3B%0A%0A%20%20%20%20%23%E7%BC%93%E5%AD%98\(%E4%BC%A0%E8%BE%93%E6%95%B0%E6%8D%AE\)%E8%AE%BE%E7%BD%AE%0A%20%20%20%20gzip%20on%3B%0A%20%20%20%20gzip\_comp\_level%205%3B%20%23%201\-9%0A%20%20%20%20gzip\_min\_length%20100k%3B%0A%20%20%20%20gzip\_types%20application%2Fjavascript%20text%2Fcss%20text%2Fxml%3B%0A%20%20%20%20gzip\_vary%20on%3B%20%23%E5%91%8A%E8%AF%89%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%BC%80%E5%90%AF%E4%BA%86%E5%8E%8B%E7%BC%A9%0A%0A%20%20%20%20%23fastcgi%20php%0A%20%20%20%20fastcgi\_connect\_timeout%2060%3B%23cgi%20%E8%BF%9E%E6%8E%A5%E8%B6%85%E6%97%B660s%0A%20%20%20%20%23fastcgi\_send\_timeout%3B%23%E5%90%91cgi%E5%8F%91%E9%80%81%E6%97%B6%E9%97%B4%0A%20%20%20%20fastcgi\_read\_timeout%2060%3B%23%E6%8E%A5%E6%94%B6php%E5%93%8D%E5%BA%94%E6%97%B6%E9%97%B4%0A%0A%0A%20%20%20%20%23%E7%94%B3%E8%AF%B7%E4%B8%80%E5%9D%97%E5%8C%BA%E5%9F%9F10M%EF%BC%8C%E5%AD%98%E5%82%A8%24binary\_remote\_addr\(ip\)%E5%8F%98%E9%87%8F%E7%9A%84%E7%BB%9F%E8%AE%A1\(%E9%99%90%E5%88%B6IP\)%0A%20%20%20%20limit\_conn\_zone%20%24binary\_remote\_addr%20zone%3Daddr%3A10m%3B%0A%20%20%20%20%0A%20%20%20%20%23%E7%94%B3%E8%AF%B7%E4%B8%80%E5%9D%97%E5%8C%BA%E5%9F%9F10M%EF%BC%8C%E5%AD%98%E5%82%A8%24binary\_remote\_addr\(domain\)%E5%8F%98%E9%87%8F%E7%9A%84%E7%BB%9F%E8%AE%A1%0A%20%20%20%20limit\_conn\_zone%20%24server\_name%20zone%3Dperserver%3A10m%3B%0A%20%20%20%20%0A%20%20%20%20%23%E7%94%B3%E8%AF%B7%E4%B8%80%E5%9D%97%E5%8C%BA%E5%9F%9F10M%EF%BC%8C%E5%AD%98%E5%82%A8%24binary\_remote\_addr\(ip\)%E5%8F%98%E9%87%8F%E7%9A%84%E7%BB%9F%E8%AE%A1%EF%BC%8C%E6%AF%8F%E7%A7%92%E5%8F%AA%E8%83%BD%E6%9C%89%E4%B8%80%E4%B8%AA%E8%AF%B7%E6%B1%82%E8%BF%9B%E6%9D%A5\(%E9%99%90%E5%88%B6%E4%B8%80%E4%B8%AAIP%E7%9A%84%E8%AF%B7%E6%B1%82%E6%95%B0\)%0A%20%20%20%20limit\_req\_zone%20%24binary\_remote\_addr%20zone%3Dmylimit%3A10m%20rate%3D1r%2Fs%3B%0A%20%20%20%20%0A%20%20%20%20location%20%2F%20%7B%0A%20%20%20%20%20%20%20%20%23%E4%B8%80%E4%B8%AAIP%E5%90%8C%E4%B8%80%E6%97%B6%E9%97%B4%E6%9C%80%E5%A4%9A%E4%B8%80%E4%B8%AA%E8%BF%9E%E6%8E%A5%2Caddr%20%E5%AF%B9%E5%BA%94%E4%B8%8A%E9%9D%A2%E7%9A%84%20addr%0A%20%20%20%20%20%20%20%20limit\_conn%20addr%201%3B%0A%20%20%20%20%20%20%20%20%23%E6%AF%8F%E4%B8%AA%E8%BF%9E%E6%8E%A5%E9%99%90%E9%80%9F300K%2C%E6%98%AF%E9%92%88%E5%AF%B9%E8%BF%9E%E6%8E%A5%E7%9A%84%EF%BC%8C%E5%A6%82%E6%9E%9C%E4%B8%80%E4%B8%AAIP%E6%9C%80%E5%A4%9A%E5%8F%AF%E4%BB%A5%E6%9C%892%E4%B8%AA%E8%BF%9E%E6%8E%A5%EF%BC%8C%E9%82%A3%E5%B0%B1%E6%98%AF%E6%AF%8F%E4%B8%AA%E8%BF%9E%E6%8E%A5300K%0A%20%20%20%20%20%20%20%20limit\_rate%20300k%3B%0A%20%20%20%20%20%20%20%20%23%E8%AE%BE%E7%BD%AE%E8%B6%85%E5%87%BA%E6%9C%80%E5%A4%A7%E8%BF%9E%E6%8E%A5%E6%95%B0%E7%9A%84%E6%97%A5%E5%BF%97%E7%BA%A7%E5%88%AB%0A%20%20%20%20%20%20%20%20limit\_conn\_log\_level%20error%3B%0A%20%20%20%20%20%20%20%20%23%E5%BD%93%E6%9C%89%E8%AF%B7%E6%B1%82%E8%BF%87%E6%9D%A5%E8%A7%A6%E5%8F%91%E4%BA%86%E6%9C%80%E5%A4%A7%E8%AF%B7%E6%B1%82%E6%95%B0%EF%BC%8C%E7%94%B1burst%E7%BC%93%E5%AD%98%E4%BD%8F5%E4%B8%AA%E8%AF%B7%E6%B1%82%0A%20%20%20%20%20%20%20%20%23nodelay%EF%BC%9A%E5%BD%93%E8%AF%B7%E6%B1%82%E9%A2%91%E6%AC%A1%20%E8%A2%AB%E9%99%90%E5%88%B6burst%E7%BC%93%E5%AD%98%E4%B9%9F%E6%BB%A1%E4%BA%86%EF%BC%8C%E7%9B%B4%E6%8E%A5%E6%89%94%E6%8E%89%2Creturn%20503%0A%20%20%20%20%20%20%20%20limit\_req%20zone%3Dmylimit%20burst%3D5%20nodelay%3B%0A%20%20%20%20%7D%0A%20%20%20%20%0A%20%20%20%20include%20vhost%2F\*.conf%3B%0A%20%20%20%20%0A%0A%7D%0A%0A%60%60%60%0Ametrics%0A\-\-\-\-\-%0A%60%60%60%60%0A%0Avhost\_traffic\_status\_zone%3B%0Aserver%20%7B%0A%20%20%20%20listen%209111%3B%0A%20%20%20%20location%20%2Fstatus%20%7B%0A%20%20%20%20vhost\_traffic\_status\_display%3B%0A%20%20%20%20vhost\_traffic\_status\_display\_format%20html%3B%0A%20%20%20%20%7D%0A%7D%0A%60%60%60%60%0A%0A%0A%0A%23%23%23%23%20server%0A%0A%0A%0A%0A%60%60%60%0Aserver%20%7B%0A%20%20%20%20autoindex%20off%3B%0A%20%20%20%20access\_log%20%2Fdata%2Flogs%2Fnginx%2Fagent.xlsyfx.cn.access.log%3B%0A%20%20%20%20server\_name%20agent.xlsyfx.cn%3B%0A%0A%0A%20%20%20%20listen%2080%3B%0A%20%20%20%20listen%20443%20ssl%3B%0A%20%20%20%20ssl\_certificate%20%2Fetc%2Fletsencrypt%2Flive%2Fxlsyfx.cn%2Ffullchain.pem%3B%0A%20%20%20%20ssl\_certificate\_key%20%2Fetc%2Fletsencrypt%2Flive%2Fxlsyfx.cn%2Fprivkey.pem%3B%0A%0A%20%20%20%20ssl\_session\_timeout%205m%3B%0A%20%20%20%20ssl\_protocols%20SSLv3%20TLSv1%20TLSv1.1%20TLSv1.2%3B%0A%20%20%20%20ssl\_ciphers%20%22HIGH%3A\!aNULL%3A\!MD5%20or%20HIGH%3A\!aNULL%3A\!MD5%3A\!3DES%22%3B%0A%20%20%20%20ssl\_prefer\_server\_ciphers%20on%3B%0A%0A%20%20%20%20error\_page%20404%20%2F404.html%3B%0A%20%20%20%20%20%20%20%20location%20%3D%20%2F404.html%20%7B%0A%20%20%20%20%20%20%20%20root%20%2Fdata%2Fwww%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20error\_page%20500%20502%20503%20504%20%2F50x.html%3B%0A%20%20%20%20%20%20%20%20location%20%3D%20%2F50x.html%20%7B%0A%20%20%20%20%20%20%20%20root%20%2Fdata%2Fwww%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20root%20%2Fdata%2Fwww%3B%0A%20%20%20%20%20%20%20%20location%20%2F%20%7B%0A%20%20%20%20%20%20%20%20index%20index.php%20index.html%3B%0A%20%20%20%20%7D%0A%20%20%20%20%0A%20%20%20%20%23%E5%9B%BE%E7%89%87%E6%98%A0%E5%B0%84%E6%9C%AC%E6%9C%BA%E7%A1%AC%E7%9B%98%E5%9C%B0%E5%9D%80%0A%20%20%20%20location%20~\*%20%5C.\(gif%7Cjpg%7Cjpeg%7Cpng%7Ccss%7Cjs%7Cico\)%24%20%7B%0A%20%20%20%20%20%20%20%20root%20%2Fdata%2Fwww%2Fphp%2Fzhongyuhuacai%2Fstatic%3B%0A%20%20%20%20%7D%0A%20%20%20%20%23%E7%89%B9%E6%AE%8A%E6%96%87%E4%BB%B6%E6%98%A0%E5%B0%84%E6%9C%AC%E6%9C%BA%E7%A1%AC%E7%9B%98%E5%9C%B0%E5%9D%80%0A%20%20%20%20location%20~\*%20%5C.\(eot%7Cotf%7Cttf%7Cwoff%7Cwoff2\)%24%20%7B%0A%20%20%20%20%20%20%20%20root%20%2Fdata%2Fwww%2Fphp%2Fzhongyuhuacai%2Fstatic%3B%0A%20%20%20%20%20%20%20%20add\_header%20Access\-Control\-Allow\-Origin%20\*%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20%23%E5%AF%B9URI%E8%BF%9B%E8%A1%8C%E8%BD%AC%E6%8D%A2%0A%20%20%20%20location%20~%20%2F%20%7B%0A%20%20%20%20%20%20%20%20if%20\(\!\-e%20%24request\_filename\)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20rewrite%20%5E%2F\(%5Cw%2B\)%2F\(%5Cw%2B\)%2F\(.\*\)%24%20%2Findex.php%3Fctrl%3D%241%26ac%3D%242%26%243%20last%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%0A%20%20%20%20%23%E8%A7%A3%E6%9E%90PHP%0A%20%20%20%20location%20~%20%5C.php%24%20%7B%0A%20%20%20%20%20%20%20%20root%20%2Fdata%2Fwww%3B%0A%20%20%20%20%20%20%20%20fastcgi\_pass%20127.0.0.1%3A9000%3B%0A%20%20%20%20%20%20%20%20fastcgi\_index%20index.php%3B%0A%20%20%20%20%20%20%20%20fastcgi\_param%20SCRIPT\_FILENAME%20%24document\_root%24fastcgi\_script\_name%3B%0A%20%20%20%20%20%20%20%20%23fastcgi\_param%20SCRIPT\_FILENAME%20%2Fdata%2Fwww%2Fphp%2Fzhongyuhuacai%2Fapi%24fastcgi\_script\_name%3B%0A%20%20%20%20%20%20%20%20include%20fastcgi\_params%3B%0A%20%20%20%20%20%20%20%20fastcgi\_param%20HTTP\_X\_REAL\_IP%20%24HTTP\_X\_REAL\_IP%3B%0A%20%20%20%20%7D%0A%7D%0A%0A%0A%60%60%60%0A%0A%E4%BB%A3%E7%90%86%2F%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%0A\-\-\-\-\-%0A%60%60%60%0Aupstream%20my\_server%20%7B%0A%20%20%20%20server%20127.0.0.1%3A8080%20weight%3D5%3B%0A%20%20%20%20server%2010.0.0.2%3A8080%3B%0A%20%20%20%20keepalive%202000%3B%0A%7D%0Aserver%20%7B%0A%20%20%20%20listen%2080%3B%0A%20%20%20%20server\_name%2010.0.0.1%3B%0A%20%20%20%20location%20%2Fmy%2F%20%7B%0A%20%20%20%20proxy\_pass%20http%3A%2F%2Fmy\_server%2F%3B%0A%20%20%20%20proxy\_set\_header%20Host%20%24host%3A%24server\_port%3B%0A%20%20%20%20%7D%0A%7D%0A%60%60%60%0A%0A%0Atrace\_id%0A\-\-\-\-\-\-%0A%60%60%60%0Aserver%20%7B%20%20%0A%20%20%20%20%20%20%20%20lua\_code\_cache%20off%3B%0A%20%20%20%20%20%20%20%20location%20~%20%5C.php%24%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20set%20%24first%200%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20if%20\(%20%24http\_x\_trace\_id%20%3D%20''\)%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20set%20%24first%201%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20if%20\(%20%24http\_x\_trace\_id%20%3D%200\)%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20set%20%24first%201%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20if%20\(%20%24http\_x\_trace\_id%20%3D%20'0'\)%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20set%20%24first%201%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20set\_by\_lua%20%24span\_id%20'return%20%22200000%22..os.time\(\)'%3B%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20if%20\(%20%24first%20%3D%201%20\)%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20set\_by\_lua\_file%20%24trace\_id%20%2Fhome%2Fwww%2Ftest\_lua%2Ftrace\_id.lua%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20set%20%24first%201%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20if%20\(%20%24first%20%3D%200%20\)%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20set%20%24trace\_id%20%24http\_x\_trace\_id%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20set%20%24parent\_id%20%24http\_x\_span\_id%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20more\_set\_input\_headers%20%22X\-Parent\-Id%3A%24parent\_id%22%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20more\_set\_input\_headers%20%22X\-Trace\-Id%3A%24trace\_id%22%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20more\_set\_input\_headers%20%22X\-Span\-Id%3A%24span\_id%22%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20more\_set\_input\_headers%20%22X\-From\-Service%3A%24server\_name%22%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20more\_set\_input\_headers%20%22X\-First%3A%24first%22%3B%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%7D%0A%0A%0A%7D%0A%60%60%60%0AVUE%0A\-\-\-\-\-%0A%60%60%60%0Aserver%20%7B%0A%0A%20%20%20%20location%20%2F%20%7B%0A%20%20%20%20%20%20%20%20root%20%2Fdata%2Fwww%2FDigitaltwin%2Fmaster%2Fdist%3B%0A%20%20%20%20%20%20%20%20try\_files%20%24uri%20%24uri%2F%20%40router%3B%0A%20%20%20%20%20%20%20%20index%20index.html%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20location%20%40router%20%7B%0A%20%20%20%20%20%20%20%20rewrite%20%5E.\*%24%20%2Findex.html%20last%3B%0A%20%20%20%20%7D%0A%7D%0A%60%60%60%0Agolang%0A\-\-\-\-\-\-%0A%60%60%60%0Aserver%20%7B%0A%20%20%20%20location%20%2F%20%7B%0A%20%20%20%20%20%20%20%20proxy\_set\_header%20%20X\-Real\-IP%20%24remote\_addr%3B%0A%20%20%20%20%20%20%20%20proxy\_set\_header%20%20REMOTE\-HOST%20%24remote\_addr%3B%0A%20%20%20%20%20%20%20%20proxy\_set\_header%20%20X\-Forwarded\-For%20%24proxy\_add\_x\_forwarded\_for%3B%0A%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20proxy\_pass%20http%3A%2F%2F127.0.0.1%3A5555%3B%0A%0A%20%20%20%20%20%20%20%20proxy\_connect\_timeout%2030%3B%0A%20%20%20%20%20%20%20%20proxy\_send\_timeout%20300%3B%0A%20%20%20%20%20%20%20%20proxy\_read\_timeout%20300%3B%0A%20%20%20%20%7D%0A%7D%0A%60%60%60%0A%0A%0A%0A%0A
