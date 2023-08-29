# httpd.conf

```
liseten 80
User www
Group www

ErrorLog "/data/logs/apache/error_log"
CustomLog "/data/logs/apache/access_log" common

ServerName 39.106.65.76:80

DocumentRoot "/data/www"
DirectoryIndex index.htm index.html index.php
AddType application/x-httpd-php .php .phtml .php3 .inc 

#关闭掉apache目录显示
Options Indexes FollowSymLinks

#允许rewrite .ht文件
rewrite_module modules/mod_rewrite.so

AllowOverride Index #这行代码会有问题
AllowOverride All

Include conf/extra/httpd-info.conf

支付特殊字体类型请求

<FilesMatch "\.(ttf|otf|eot|woff|svg|woff2)$">
  <IfModule mod_headers.c>
    Header set Access-Control-Allow-Origin "*"
  </IfModule>
</FilesMatch>

<VirtualHost *:80>
ServerName majiang.com
DocumentRoot D:\www\z
</VirtualHost>
```

> 

## https

> 打开注释
> 
> 
> LoadModule ssl\_module modules/mod\_ssl.so
> 
> 
> LoadModule socache\_shmcb\_module modules/mod\_socache\_shmcb.so
> 
> 
> Include conf/extra/httpd\-ssl.conf

cd /soft/apache/conf/extra/

vim httpd\-ssl.conf

SSLCertificateFile "/etc/letsencrypt/live/xlsyfx.cn/fullchain.pem"

SSLCertificateKeyFile "/etc/letsencrypt/live/xlsyfx.cn/privkey.pem"

%60%60%60%0Aliseten%2080%0AUser%20www%0AGroup%20www%0A%0AErrorLog%20%22%2Fdata%2Flogs%2Fapache%2Ferror\_log%22%0ACustomLog%20%22%2Fdata%2Flogs%2Fapache%2Faccess\_log%22%20common%0A%0AServerName%2039.106.65.76%3A80%0A%0ADocumentRoot%20%22%2Fdata%2Fwww%22%0ADirectoryIndex%20index.htm%20index.html%20index.php%0AAddType%20application%2Fx\-httpd\-php%20.php%20.phtml%20.php3%20.inc%20%0A%0A%23%E5%85%B3%E9%97%AD%E6%8E%89apache%E7%9B%AE%E5%BD%95%E6%98%BE%E7%A4%BA%0AOptions%20Indexes%20FollowSymLinks%0A%0A%23%E5%85%81%E8%AE%B8rewrite%20.ht%E6%96%87%E4%BB%B6%0Arewrite\_module%20modules%2Fmod\_rewrite.so%0A%0AAllowOverride%20Index%20%23%E8%BF%99%E8%A1%8C%E4%BB%A3%E7%A0%81%E4%BC%9A%E6%9C%89%E9%97%AE%E9%A2%98%0AAllowOverride%20All%0A%0AInclude%20conf%2Fextra%2Fhttpd\-info.conf%0A%0A%E6%94%AF%E4%BB%98%E7%89%B9%E6%AE%8A%E5%AD%97%E4%BD%93%E7%B1%BB%E5%9E%8B%E8%AF%B7%E6%B1%82%0A%0A%3CFilesMatch%20%22%5C.\(ttf%7Cotf%7Ceot%7Cwoff%7Csvg%7Cwoff2\)%24%22%3E%0A%20%20%3CIfModule%20mod\_headers.c%3E%0A%20%20%20%20Header%20set%20Access\-Control\-Allow\-Origin%20%22\*%22%0A%20%20%3C%2FIfModule%3E%0A%3C%2FFilesMatch%3E%0A%0A%3CVirtualHost%20\*%3A80%3E%0AServerName%20majiang.com%0ADocumentRoot%20D%3A%5Cwww%5Cz%0A%3C%2FVirtualHost%3E%0A%60%60%60%0A%3E%0A%0Ahttps%0A\-\-\-\-\-%0A%3E%E6%89%93%E5%BC%80%E6%B3%A8%E9%87%8A%0ALoadModule%20ssl\_module%20modules%2Fmod\_ssl.so%20%20%0ALoadModule%20socache\_shmcb\_module%20modules%2Fmod\_socache\_shmcb.so%20%20%0AInclude%20conf%2Fextra%2Fhttpd\-ssl.conf%20%20%0A%0Acd%20%2Fsoft%2Fapache%2Fconf%2Fextra%2F%20%20%0Avim%20httpd\-ssl.conf%0A%0A%0ASSLCertificateFile%20%22%2Fetc%2Fletsencrypt%2Flive%2Fxlsyfx.cn%2Ffullchain.pem%22%0ASSLCertificateKeyFile%20%22%2Fetc%2Fletsencrypt%2Flive%2Fxlsyfx.cn%2Fprivkey.pem%22%0A%0A
