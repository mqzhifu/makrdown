# composer

参数 ： \-vvvv 显示安装过程

//第一次安装/初始化，配置好composer.json

composer install

//更新所有包

composer.phar update

//安装指定包，指定版本

composer.phar require rouchi/jy\_proxy\_client ~v1.3.4

//查找包

composer search

//列出可用包

composer show

//设置composer

composer config XXXXXX

参数 ：\-g 全局

//查看 composer 配置信息

composer config \-\-list

//清理缓存

composer clear\-cache

更改源

composer config repo.packagist composer https://mirrors.aliyun.com/composer/

composer config repo.packagist composer http://packagist.rouchi.com/

https://packagist.phpcomposer.com

https://mirrors.aliyun.com/composer/

https://packagist.laravel\-china.org

https://pkg.phpcomposer.com

\#腾讯

https://mirrors.cloud.tencent.com/composer

\#官方

https://packagist.org/

//也可以使用配置文件

"repositories": {

"packagist": {

"type": "composer",

"url": "https://packagist.rouchi.com"

}

},

不使用https

composer \-g config secure\-http false

//也可以使用配置文件

"config": {

"secure\-http": false

}

composer config \-g cache\-dir "D:\\Program Files \(x86\)\\composer\\cache"

composer config \-g data\-dir "D:\\Program Files \(x86\)\\composer\\data"

\#composer config \-g cache\-files\-dir 'D:\\Program Files \(x86\)\\composer\\cache\-files'

%E5%8F%82%E6%95%B0%20%20%EF%BC%9A%20\-vvvv%20%20%E6%98%BE%E7%A4%BA%E5%AE%89%E8%A3%85%E8%BF%87%E7%A8%8B%0A%0A%0A%2F%2F%E7%AC%AC%E4%B8%80%E6%AC%A1%E5%AE%89%E8%A3%85%2F%E5%88%9D%E5%A7%8B%E5%8C%96%EF%BC%8C%E9%85%8D%E7%BD%AE%E5%A5%BDcomposer.json%0Acomposer%20install%0A%0A%2F%2F%E6%9B%B4%E6%96%B0%E6%89%80%E6%9C%89%E5%8C%85%0Acomposer.phar%20update%0A%0A%2F%2F%E5%AE%89%E8%A3%85%E6%8C%87%E5%AE%9A%E5%8C%85%EF%BC%8C%E6%8C%87%E5%AE%9A%E7%89%88%E6%9C%AC%0Acomposer.phar%20require%20rouchi%2Fjy\_proxy\_client%20~v1.3.4%0A%0A%2F%2F%E6%9F%A5%E6%89%BE%E5%8C%85%0Acomposer%20search%0A%0A%2F%2F%E5%88%97%E5%87%BA%E5%8F%AF%E7%94%A8%E5%8C%85%0Acomposer%20show%0A%0A%2F%2F%E8%AE%BE%E7%BD%AEcomposer%0Acomposer%20config%20%20XXXXXX%0A%E5%8F%82%E6%95%B0%20%EF%BC%9A\-g%20%E5%85%A8%E5%B1%80%0A%0A%2F%2F%E6%9F%A5%E7%9C%8B%20composer%20%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%0Acomposer%20config%20\-\-list%0A%0A%20%2F%2F%E6%B8%85%E7%90%86%E7%BC%93%E5%AD%98%0Acomposer%20clear\-cache%0A%0A%0A%E6%9B%B4%E6%94%B9%E6%BA%90%20%0Acomposer%20config%20repo.packagist%20composer%20https%3A%2F%2Fmirrors.aliyun.com%2Fcomposer%2F%0A%0Acomposer%20config%20repo.packagist%20composer%20http%3A%2F%2Fpackagist.rouchi.com%2F%0A%0Ahttps%3A%2F%2Fpackagist.phpcomposer.com%0Ahttps%3A%2F%2Fmirrors.aliyun.com%2Fcomposer%2F%0Ahttps%3A%2F%2Fpackagist.laravel\-china.org%0Ahttps%3A%2F%2Fpkg.phpcomposer.com%0A%23%E8%85%BE%E8%AE%AF%0Ahttps%3A%2F%2Fmirrors.cloud.tencent.com%2Fcomposer%0A%23%E5%AE%98%E6%96%B9%0Ahttps%3A%2F%2Fpackagist.org%2F%0A%2F%2F%E4%B9%9F%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%0A%20%20%20%20%22repositories%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%22packagist%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%22type%22%3A%20%22composer%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%22url%22%3A%20%22https%3A%2F%2Fpackagist.rouchi.com%22%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%2C%0A%0A%E4%B8%8D%E4%BD%BF%E7%94%A8https%0Acomposer%20\-g%20config%20secure\-http%20false%0A%2F%2F%E4%B9%9F%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%0A%20%20%20%20%22config%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%22secure\-http%22%3A%20false%0A%20%20%20%20%7D%0A%0A%0Acomposer%20config%20\-g%20cache\-dir%20%22D%3A%5CProgram%20Files%20\(x86\)%5Ccomposer%5Ccache%22%0Acomposer%20config%20\-g%20data\-dir%20%22D%3A%5CProgram%20Files%20\(x86\)%5Ccomposer%5Cdata%22%0A%23composer%20config%20\-g%20cache\-files\-dir%20'D%3A%5CProgram%20Files%20\(x86\)%5Ccomposer%5Ccache\-files'%0A
