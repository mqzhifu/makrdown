# 自动化测试/selenium

1. 性能测试
    jemeter loadRunner,locust
2. 功能测试
    robotFramework
    AirTest
    Playwright
    seleniumIDE
    cypress\(弃了，nodejs编写\)

自动化测试是针对部分业务，而不是全链路，其主要是为：核心业务。并且可标准化流程的，或者人为操作即期复杂的情况。

自动化测试的衡量标准是：覆盖系统的比重是多少

大的分层

1. UI
2. SERVER
3. UNIT

绝对不是线性代码形态，以代码与数据分离形态来实现。关键字驱动\+POM

selenium4 \+ webDrive \+ seleniumGrid

chrom 最好别自动 更新，webDrive依赖

> pip install selenium

web drive

https://selenium.dev/

下包，解压，CP

> cp /data/www/python/zpy/chromedriver /usr/local/Cellar/python@3.9/3.9.6/Frameworks/Python.framework/Versions/3.9

chown wangdongyan:staff chromedriver

这里不CP也行，在PY代码里指定文件目录也行

也可以本地启动试试：

./chromedriver

web drive:是个代理（服务），应该是接收python的操作，再调用浏览器API。

数据驱动

就是把所有配置信息放在一个文件中，日常维护这个文档即可，程序写一次就够了。

配置文件中包含，如：

域名

API\-\>URL

1.%20%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95%0A%20%20%20%20jemeter%20loadRunner%2Clocust%0A2.%20%E5%8A%9F%E8%83%BD%E6%B5%8B%E8%AF%95%0A%20%20%20%20robotFramework%20%0A%20%20%20%20AirTest%20%0A%20%20%20%20Playwright%20%0A%20%20%20%20seleniumIDE%20%0A%20%20%20%20cypress\(%E5%BC%83%E4%BA%86%EF%BC%8Cnodejs%E7%BC%96%E5%86%99\)%0A%20%20%20%20%0A%20%20%20%20%0A%E8%87%AA%E5%8A%A8%E5%8C%96%E6%B5%8B%E8%AF%95%E6%98%AF%E9%92%88%E5%AF%B9%E9%83%A8%E5%88%86%E4%B8%9A%E5%8A%A1%EF%BC%8C%E8%80%8C%E4%B8%8D%E6%98%AF%E5%85%A8%E9%93%BE%E8%B7%AF%EF%BC%8C%E5%85%B6%E4%B8%BB%E8%A6%81%E6%98%AF%E4%B8%BA%EF%BC%9A%E6%A0%B8%E5%BF%83%E4%B8%9A%E5%8A%A1%E3%80%82%E5%B9%B6%E4%B8%94%E5%8F%AF%E6%A0%87%E5%87%86%E5%8C%96%E6%B5%81%E7%A8%8B%E7%9A%84%EF%BC%8C%E6%88%96%E8%80%85%E4%BA%BA%E4%B8%BA%E6%93%8D%E4%BD%9C%E5%8D%B3%E6%9C%9F%E5%A4%8D%E6%9D%82%E7%9A%84%E6%83%85%E5%86%B5%E3%80%82%0A%0A%E8%87%AA%E5%8A%A8%E5%8C%96%E6%B5%8B%E8%AF%95%E7%9A%84%E8%A1%A1%E9%87%8F%E6%A0%87%E5%87%86%E6%98%AF%EF%BC%9A%E8%A6%86%E7%9B%96%E7%B3%BB%E7%BB%9F%E7%9A%84%E6%AF%94%E9%87%8D%E6%98%AF%E5%A4%9A%E5%B0%91%0A%0A%0A%E5%A4%A7%E7%9A%84%E5%88%86%E5%B1%82%0A1.%20UI%0A2.%20SERVER%0A3.%20UNIT%0A%0A%E7%BB%9D%E5%AF%B9%E4%B8%8D%E6%98%AF%E7%BA%BF%E6%80%A7%E4%BB%A3%E7%A0%81%E5%BD%A2%E6%80%81%EF%BC%8C%E4%BB%A5%E4%BB%A3%E7%A0%81%E4%B8%8E%E6%95%B0%E6%8D%AE%E5%88%86%E7%A6%BB%E5%BD%A2%E6%80%81%E6%9D%A5%E5%AE%9E%E7%8E%B0%E3%80%82%E5%85%B3%E9%94%AE%E5%AD%97%E9%A9%B1%E5%8A%A8%2BPOM%0A%0A%0Aselenium4%20%2B%20webDrive%20%2B%20seleniumGrid%0A%0Achrom%20%E6%9C%80%E5%A5%BD%E5%88%AB%E8%87%AA%E5%8A%A8%20%E6%9B%B4%E6%96%B0%EF%BC%8CwebDrive%E4%BE%9D%E8%B5%96%0A%0A%3Epip%20install%20selenium%0A%0Aweb%20drive%0Ahttps%3A%2F%2Fselenium.dev%2F%0A%E4%B8%8B%E5%8C%85%EF%BC%8C%E8%A7%A3%E5%8E%8B%EF%BC%8CCP%0A%0A%3Ecp%20%2Fdata%2Fwww%2Fpython%2Fzpy%2Fchromedriver%20%2Fusr%2Flocal%2FCellar%2Fpython%403.9%2F3.9.6%2FFrameworks%2FPython.framework%2FVersions%2F3.9%0A%0Achown%20wangdongyan%3Astaff%20chromedriver%0A%0A%E8%BF%99%E9%87%8C%E4%B8%8DCP%E4%B9%9F%E8%A1%8C%EF%BC%8C%E5%9C%A8PY%E4%BB%A3%E7%A0%81%E9%87%8C%E6%8C%87%E5%AE%9A%E6%96%87%E4%BB%B6%E7%9B%AE%E5%BD%95%E4%B9%9F%E8%A1%8C%0A%0A%E4%B9%9F%E5%8F%AF%E4%BB%A5%E6%9C%AC%E5%9C%B0%E5%90%AF%E5%8A%A8%E8%AF%95%E8%AF%95%EF%BC%9A%0A.%2Fchromedriver%0A%0Aweb%20drive%3A%E6%98%AF%E4%B8%AA%E4%BB%A3%E7%90%86%EF%BC%88%E6%9C%8D%E5%8A%A1%EF%BC%89%EF%BC%8C%E5%BA%94%E8%AF%A5%E6%98%AF%E6%8E%A5%E6%94%B6python%E7%9A%84%E6%93%8D%E4%BD%9C%EF%BC%8C%E5%86%8D%E8%B0%83%E7%94%A8%E6%B5%8F%E8%A7%88%E5%99%A8API%E3%80%82%0A%0A%0A%E6%95%B0%E6%8D%AE%E9%A9%B1%E5%8A%A8%0A%E5%B0%B1%E6%98%AF%E6%8A%8A%E6%89%80%E6%9C%89%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%E6%94%BE%E5%9C%A8%E4%B8%80%E4%B8%AA%E6%96%87%E4%BB%B6%E4%B8%AD%EF%BC%8C%E6%97%A5%E5%B8%B8%E7%BB%B4%E6%8A%A4%E8%BF%99%E4%B8%AA%E6%96%87%E6%A1%A3%E5%8D%B3%E5%8F%AF%EF%BC%8C%E7%A8%8B%E5%BA%8F%E5%86%99%E4%B8%80%E6%AC%A1%E5%B0%B1%E5%A4%9F%E4%BA%86%E3%80%82%0A%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%B8%AD%E5%8C%85%E5%90%AB%EF%BC%8C%E5%A6%82%EF%BC%9A%0A%E5%9F%9F%E5%90%8D%0AAPI\-%3EURL%0A%E8%AF%B7%E6%B1%82%E5%8F%82%E6%95%B0%0A%E6%96%AD%E8%A8%80%0A......
