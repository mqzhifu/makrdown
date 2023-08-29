# 重装GVA

## gin\-admin\-vue

一个可视化管理台后\(web 浏览器 BS结构\)，带一些基础功能。简化重复造轮子。

主体是：GO\+VUE

实现了：前后端分离、MVC

官网

> https://www.gin\-vue\-admin.com/docs/first\_master

下代码：

> git clone https://github.com/flipped\-aurora/gin\-vue\-admin.git

代码结构主要分为两个部分：server 和 web

## 下载

官方地址：

> git clone https://github.com/flipped\-aurora/gin\-vue\-admin.git

自己项目的地址:

> git http://192.168.1.22:40080/frontend/seed\_admin\_vue.git
> 
> 
> git http://192.168.1.22:40080/backend/seed\_admin\_go.git

下载完成后，修改下权限：

> chown \-R wangdongyan:\_www /data/www/golang/seed\_admin\_vue
> 
> 
> chown \-R wangdongyan:\_www /data/www/golang/seed\_admin\_go

把cicd脚本复制进去：

> cp cicd.toml /data/www/golang/seed\_admin\_vue
> 
> 
> cp cicd.toml /data/www/golang/seed\_admin\_go

## 安装前端

cd seed\_admin\_vue

> npm install \-\-unsafe\-perm=true \-\-allow\-root

安装不成功的话，换个淘宝的源：

> npm install \-g cnpm \-\-registry=https://registry.npm.taobao.org
> 
> 
> cnpm install \-\-unsafe\-perm=true \-\-allow\-root

两种方式均可，官方更推荐npm，但速度没有cnpm快

启动:

> npm server

查看下端口号：

lsof \-i:8080

配置文件修改：

```
vim .env.product
>VITE_BASE_API = http://adminapi.seedreality.com
```

vim vite.config.js

> outDir: '../dist'

产出目录,修改编译JS输出目录

## 安装后端

修改 mod 的包名，也可以不改，它这个就是有点长...

> github.com/flipped\-aurora/gin\-vue\-admin/server \> Zwebuigo

mod下包，并启动后端:

```
cd seed_admin_go
go mod tidy
go run main
```

修改一下自动生成代码的目录:

vim config.yaml

```
autocode->server->/seed_admin_go
autocode->web->/seed_admin_vue/src
```

## 初始化DB

> http://127.0.0.1:8080

点击：初始化按钮，输入MYSQL配置信息，执行即可

## 页面修改

#### 删除默认密码/始化按钮

view/login/login.vue

168行：

删除admin 和 123456

#### 换LOGO

#### 删除尾部

#### 个人信息，修改头部 手机号 地址 邮箱 等等

## DIY\-前端代码操作

#### 添加2个自定义常用的函数

vim web/src/utils/format.js

```
export const formatUnixDate = (time) => {
  if (time !== null && time !== '' && time > 0) {
    time = time + "000"
    time = parseInt(time);
    var date = new Date(time)
    return formatTimeToStr(date, 'yyyy-MM-dd hh:mm:ss')
  } else {
    return '--'
  }
}
```

date字段标识添加:formatter="substrContent"

```
const substrContent = (val) => {
  var v = val.headerCommon
  if( v.length > 40){
    return v.substr(0,40)
  }else{
    return v
  }
}
```

## DIY\-后端代码

#### 修改配置文件，增加一个DB配置项：

vim config.yaml

```
mysql-business:
  path: 8.142.177.235
  port: "3306"
  config: charset=utf8mb4&parseTime=True&loc=Local
  db-name: "seed_pre"
  username: root
  password: ''
  max-idle-conns: 10
  max-open-conns: 100
  log-mode: error
  log-zap: false
```

给DB增加公共配置，DB选项

vim config/config.go

```
MysqlBusiness  Mysql `mapstructure:"mysql-business" json:"mysql_business" yaml:"mysql"`
```

#### 增加DB控制器代码

touch initialize/business.go

vim initialize/business.go

```
package initialize

import (
	"github.com/flipped-aurora/gin-vue-admin/server/global"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/schema"
	"fmt"
)

func  BusinessDb() (gormDb *gorm.DB, err error) {
	m := global.GVA_CONFIG.MysqlBusiness
	//dns := "root" + ":" + "mqzhifu" + "@tcp(" + "8.142.177.235" + ":" + m.Port + ")/" + "test" + "?" + m.Config
	dns := m.Username + ":" + m.Password + "@tcp(" + m.Path + ":" + m.Port + ")/" + m.Dbname + "?" + m.Config
	fmt.Println("mysql link info: " + dns)
	mysqlConfig := mysql.Config{
		DSN:                       dns,   // DSN data source name
		DefaultStringSize:         191,   // string 类型字段的默认长度
		DisableDatetimePrecision:  true,  // 禁用 datetime 精度，MySQL 5.6 之前的数据库不支持
		DontSupportRenameIndex:    true,  // 重命名索引时采用删除并新建的方式，MySQL 5.7 之前的数据库和 MariaDB 不支持重命名索引
		DontSupportRenameColumn:   true,  // 用 `change` 重命名列，MySQL 8 之前的数据库和 MariaDB 不支持重命名列
		SkipInitializeWithVersion: false, // 根据版本自动配置
	}
	diyConfig := &gorm.Config{DisableForeignKeyConstraintWhenMigrating: true, NamingStrategy: schema.NamingStrategy{SingularTable: true}}
	db, err := gorm.Open(mysql.New(mysqlConfig), diyConfig)
	fmt.Println("gorm.Open er  info:", err)
	if err != nil {
		return nil, err
	}
	global.GVA_DBList = make(map[string]*gorm.DB)
	global.GVA_DBList["business"] = db
	return db, nil
}

```

#### main添加启动新DB的入口

vim main.go

```
initialize.BusinessDb()
```

#### 自带的时间格式是基于DB datetime类型，而日常使用的不依赖DB，就是个INT值

touch model/my\_model.go

vim model/my\_model.go

```
package model

import "gorm.io/gorm"

type DIY_MODEL struct {
	ID        uint           `gorm:"primarykey"` // 主键ID
	CreatedAt int            // 创建时间
	UpdatedAt int            // 更新时间
	DeletedAt gorm.DeletedAt `gorm:"index" json:"-"` // 删除时间
}

```

#### 打开跨域

vim initialize/router.go

打开跨域注释：

> Router.Use\(middleware.Cors\(\)\)

## 自动生成页面代码

#### 导入常量值

> 自动生成页面时，有些字段时是枚举，为了减少代码量，从原项目中拿到该值，并直接生成sql query，直接插入到DB中

请求接口拿到sql:

> http://127.0.0.1:1111/tools/const/init/db

导入 gva DB 中

```
delete from sys_dictionaries where id >= 10;
delete from sys_dictionary_details where sys_dictionary_id >= 10 
```

#### 创建一个包：

> 所有的页面，得给一个公共包

1. 系统工具\-\>自动化package\-\>创建一个包:op 运维 运维日常维护使用的一些表格数据工具
2. 系统工具\-\>生成代码器

#### 创建根菜单：

> 目前先把所有的页面统一放在这个新的菜单中，后续再考虑如何放置位置

超级管理员\-\>菜单管理\-\>新建根菜单\-\>运维\-\>op

给新创建的根菜单，添加一个索引页面，用于包含子菜单的代码:

> touch view/op/index.html

新建子菜单：

> view/op/statisticsLog.vue

#### 自动创建代码：

系统工具\-\>代码生成器\-\>数据库名\-\>表名\-\>生成代码

需要创建的页面：

```
statistic_log
project
instance
server
```

调整字段的时候注意下：tinyint 它是给转成了bool 得手动改成int

将刚刚生成的模板文件，转移到指定目录下：

> mkdir web/src/view/op
> 
> 
> mv web/src/view/statisticsLog/\* web/src/view/op

修改model

```
model.DIY_MODEL
```

后端代码修改：

vim server/service/op/statistic\_log.go

replace:

```
global.GVA_DB
global.GetGlobalDBByDBName("business")
```

给角色设置新权限，退出 ，重新登陆即可。

#### 导出DB

/soft/mysql/bin/mysqldump \-uroot \-h8.142.177.235 \-p'\#\#\#\#\#' gva \> /data/www/gva.sql

## 自有安装

> git http://192.168.1.22:40080/frontend/seed\_admin\_vue.git
> 
> 
> git http://192.168.1.22:40080/backend/seed\_admin\_go.git

> 以上是属于空项目安装，下面是：直接下载已经部署好的项目

下载代码：

> chown \-R wangdongyan:\_www /data/www/golang/seed\_admin\_vue
> 
> 
> chown \-R wangdongyan:\_www /data/www/golang/seed\_admin\_go

后端密码：admin 123456

## 安装前端

cd seed\_admin\_vue

> npm install \-\-unsafe\-perm=true \-\-allow\-root

安装不成功的话，换个淘宝的源：

> cnpm install \-\-unsafe\-perm=true \-\-allow\-root

两种方式均可，官方更推荐npm，但速度没有cnpm快

启动:

> npm server

查看下端口号：

lsof \-i:8080

配置文件修改：

```
vim .env.product
>VITE_BASE_API = http://adminapi.seedreality.com
```

vim vite.config.js

> outDir: '../dist'

产出目录,修改编译JS输出目录

## 安装后端

#### 把DB导入到本地：gva.sql

source gva.sql

mod下包，并启动后端:

```
cd seed_admin_go
go mod tidy
go run main
```

修改配置信息：

vim config.yaml

1. DB\-MYSQL配置，一个GVA自带的，另一个连接到业务库
2. 自动生成代码的根目录，root: /data/www/golang

自动生成代码：

1. 先确定放在哪个菜单下，如果是已存在的根目录就忽略，如果是新的根目录，那得新建一个
2. 创建一个子菜单
3. 自动生成的代码，得有个包名\(package\)，这里如果已经存在就忽略，不存在 手动创建一个
4. 系统工具\-》代码生成器\-》数据库名\-》表名\-》下面就根据表的字段生成了table
5. 选择包名
6. 对每个字段进行编辑
    1. 是否添加搜索功能
    2. 它仅支持varchar int ，像：tinyint 他是给转成了bool，这里注意下，改回int
    3. 如有枚举字段，记得要选择具体的常量列表
7. 点击确认后，它会生成前端、后端代码
8. 前端代码在：view/packageName下面，你转移到自己想放的位置，然后修改你上面刚刚创建的子菜单，把这个文件地址复制进去
9. 后端代码略多，不详细写了，具体，你就需要修改：
    1. model/packageName/文件 ，把它的 global.GVA\_MODEL 改成 model.DIY\_MODEL
    2. 如果不是GVA库的表，是业务表， service/packageName/文件 ，把它的 global.GVA\_DB 统一换秘 global.GetGlobalDBByDBName\("business"\)

#### 任务1：

页面修改

#### 删除默认密码/始化按钮

view/login/login.vue

168行：

删除admin 和 123456

#### 换LOGO

#### 删除尾部

#### 个人信息，修改头部 手机号 地址 邮箱 等等

等等吧，把这个开源项目自带的一些东西，都弄掉吧。

任务2：

自己创建一个页面，如：声网回调这张表

两个任务的核心是熟悉这个后台项目的代码

gin\-admin\-vue%0A\-\-\-\-\-\-\-\-\-\-\-\-%0A%E4%B8%80%E4%B8%AA%E5%8F%AF%E8%A7%86%E5%8C%96%E7%AE%A1%E7%90%86%E5%8F%B0%E5%90%8E\(web%20%E6%B5%8F%E8%A7%88%E5%99%A8%20BS%E7%BB%93%E6%9E%84\)%EF%BC%8C%E5%B8%A6%E4%B8%80%E4%BA%9B%E5%9F%BA%E7%A1%80%E5%8A%9F%E8%83%BD%E3%80%82%E7%AE%80%E5%8C%96%E9%87%8D%E5%A4%8D%E9%80%A0%E8%BD%AE%E5%AD%90%E3%80%82%0A%E4%B8%BB%E4%BD%93%E6%98%AF%EF%BC%9AGO%2BVUE%0A%E5%AE%9E%E7%8E%B0%E4%BA%86%EF%BC%9A%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E3%80%81MVC%0A%0A%0A%E5%AE%98%E7%BD%91%0A%3Ehttps%3A%2F%2Fwww.gin\-vue\-admin.com%2Fdocs%2Ffirst\_master%0A%3E%0A%E4%B8%8B%E4%BB%A3%E7%A0%81%EF%BC%9A%0A%3Egit%20clone%20https%3A%2F%2Fgithub.com%2Fflipped\-aurora%2Fgin\-vue\-admin.git%0A%0A%E4%BB%A3%E7%A0%81%E7%BB%93%E6%9E%84%E4%B8%BB%E8%A6%81%E5%88%86%E4%B8%BA%E4%B8%A4%E4%B8%AA%E9%83%A8%E5%88%86%EF%BC%9Aserver%20%E5%92%8C%20web%0A%0A%0A%E4%B8%8B%E8%BD%BD%0A\-\-\-\-%0A%E5%AE%98%E6%96%B9%E5%9C%B0%E5%9D%80%EF%BC%9A%0A%3Egit%20clone%20https%3A%2F%2Fgithub.com%2Fflipped\-aurora%2Fgin\-vue\-admin.git%0A%0A%E8%87%AA%E5%B7%B1%E9%A1%B9%E7%9B%AE%E7%9A%84%E5%9C%B0%E5%9D%80%3A%0A%3Egit%20http%3A%2F%2F192.168.1.22%3A40080%2Ffrontend%2Fseed\_admin\_vue.git%0A%3Egit%20http%3A%2F%2F192.168.1.22%3A40080%2Fbackend%2Fseed\_admin\_go.git%0A%0A%E4%B8%8B%E8%BD%BD%E5%AE%8C%E6%88%90%E5%90%8E%EF%BC%8C%E4%BF%AE%E6%94%B9%E4%B8%8B%E6%9D%83%E9%99%90%EF%BC%9A%0A%3Echown%20\-R%20wangdongyan%3A\_www%20%2Fdata%2Fwww%2Fgolang%2Fseed\_admin\_vue%0A%3Echown%20\-R%20wangdongyan%3A\_www%20%2Fdata%2Fwww%2Fgolang%2Fseed\_admin\_go%0A%0A%E6%8A%8Acicd%E8%84%9A%E6%9C%AC%E5%A4%8D%E5%88%B6%E8%BF%9B%E5%8E%BB%EF%BC%9A%0A%3Ecp%20cicd.toml%20%2Fdata%2Fwww%2Fgolang%2Fseed\_admin\_vue%0A%3Ecp%20cicd.toml%20%2Fdata%2Fwww%2Fgolang%2Fseed\_admin\_go%0A%0A%0A%E5%AE%89%E8%A3%85%E5%89%8D%E7%AB%AF%0A\-\-\-\-%0A%0Acd%20seed\_admin\_vue%0A%3Enpm%20%20install%20\-\-unsafe\-perm%3Dtrue%20\-\-allow\-root%20%0A%0A%E5%AE%89%E8%A3%85%E4%B8%8D%E6%88%90%E5%8A%9F%E7%9A%84%E8%AF%9D%EF%BC%8C%E6%8D%A2%E4%B8%AA%E6%B7%98%E5%AE%9D%E7%9A%84%E6%BA%90%EF%BC%9A%0A%3Enpm%20install%20\-g%20cnpm%20\-\-registry%3Dhttps%3A%2F%2Fregistry.npm.taobao.org%0A%3Ecnpm%20install%20\-\-unsafe\-perm%3Dtrue%20\-\-allow\-root%20%0A%0A%E4%B8%A4%E7%A7%8D%E6%96%B9%E5%BC%8F%E5%9D%87%E5%8F%AF%EF%BC%8C%E5%AE%98%E6%96%B9%E6%9B%B4%E6%8E%A8%E8%8D%90npm%EF%BC%8C%E4%BD%86%E9%80%9F%E5%BA%A6%E6%B2%A1%E6%9C%89cnpm%E5%BF%AB%0A%0A%E5%90%AF%E5%8A%A8%3A%0A%3Enpm%20server%0A%0A%E6%9F%A5%E7%9C%8B%E4%B8%8B%E7%AB%AF%E5%8F%A3%E5%8F%B7%EF%BC%9A%0Alsof%20\-i%3A8080%20%0A%0A%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%BF%AE%E6%94%B9%EF%BC%9A%0A%60%60%60%0Avim%20.env.product%0A%3EVITE\_BASE\_API%20%3D%20http%3A%2F%2Fadminapi.seedreality.com%0A%60%60%60%0A%0Avim%20vite.config.js%0A%3EoutDir%3A%20'..%2Fdist'%0A%0A%E4%BA%A7%E5%87%BA%E7%9B%AE%E5%BD%95%2C%E4%BF%AE%E6%94%B9%E7%BC%96%E8%AF%91JS%E8%BE%93%E5%87%BA%E7%9B%AE%E5%BD%95%0A%0A%0A%E5%AE%89%E8%A3%85%E5%90%8E%E7%AB%AF%0A\-\-\-\-%0A%E4%BF%AE%E6%94%B9%20mod%20%E7%9A%84%E5%8C%85%E5%90%8D%EF%BC%8C%E4%B9%9F%E5%8F%AF%E4%BB%A5%E4%B8%8D%E6%94%B9%EF%BC%8C%E5%AE%83%E8%BF%99%E4%B8%AA%E5%B0%B1%E6%98%AF%E6%9C%89%E7%82%B9%E9%95%BF...%0A%3Egithub.com%2Fflipped\-aurora%2Fgin\-vue\-admin%2Fserver%20%3E%20Zwebuigo%0A%0Amod%E4%B8%8B%E5%8C%85%EF%BC%8C%E5%B9%B6%E5%90%AF%E5%8A%A8%E5%90%8E%E7%AB%AF%3A%0A%60%60%60%0Acd%20seed\_admin\_go%0Ago%20mod%20tidy%0Ago%20run%20main%0A%60%60%60%0A%0A%E4%BF%AE%E6%94%B9%E4%B8%80%E4%B8%8B%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90%E4%BB%A3%E7%A0%81%E7%9A%84%E7%9B%AE%E5%BD%95%3A%0A%0Avim%20config.yaml%0A%60%60%60%0Aautocode\-%3Eserver\-%3E%2Fseed\_admin\_go%0Aautocode\-%3Eweb\-%3E%2Fseed\_admin\_vue%2Fsrc%0A%60%60%60%0A%0A%E5%88%9D%E5%A7%8B%E5%8C%96DB%0A\-\-\-%0A%3Ehttp%3A%2F%2F127.0.0.1%3A8080%0A%0A%E7%82%B9%E5%87%BB%EF%BC%9A%E5%88%9D%E5%A7%8B%E5%8C%96%E6%8C%89%E9%92%AE%EF%BC%8C%E8%BE%93%E5%85%A5MYSQL%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%EF%BC%8C%E6%89%A7%E8%A1%8C%E5%8D%B3%E5%8F%AF%0A%0A%0A%E9%A1%B5%E9%9D%A2%E4%BF%AE%E6%94%B9%0A\-\-\-%0A%23%23%23%23%20%E5%88%A0%E9%99%A4%E9%BB%98%E8%AE%A4%E5%AF%86%E7%A0%81%2F%E5%A7%8B%E5%8C%96%E6%8C%89%E9%92%AE%0A%0Aview%2Flogin%2Flogin.vue%0A168%E8%A1%8C%EF%BC%9A%0A%E5%88%A0%E9%99%A4admin%20%E5%92%8C%20123456%0A%0A%23%23%23%23%20%E6%8D%A2LOGO%0A%23%23%23%23%20%E5%88%A0%E9%99%A4%E5%B0%BE%E9%83%A8%0A%23%23%23%23%20%E4%B8%AA%E4%BA%BA%E4%BF%A1%E6%81%AF%EF%BC%8C%E4%BF%AE%E6%94%B9%E5%A4%B4%E9%83%A8%20%E6%89%8B%E6%9C%BA%E5%8F%B7%20%E5%9C%B0%E5%9D%80%20%E9%82%AE%E7%AE%B1%20%E7%AD%89%E7%AD%89%0A%0A%0ADIY\-%E5%89%8D%E7%AB%AF%E4%BB%A3%E7%A0%81%E6%93%8D%E4%BD%9C%0A\-\-\-\-%0A%0A%23%23%23%23%20%E6%B7%BB%E5%8A%A02%E4%B8%AA%E8%87%AA%E5%AE%9A%E4%B9%89%E5%B8%B8%E7%94%A8%E7%9A%84%E5%87%BD%E6%95%B0%0A%0Avim%20web%2Fsrc%2Futils%2Fformat.js%0A%0A%60%60%60%0Aexport%20const%20formatUnixDate%20%3D%20\(time\)%20%3D%3E%20%7B%0A%20%20if%20\(time%20\!%3D%3D%20null%20%26%26%20time%20\!%3D%3D%20''%20%26%26%20time%20%3E%200\)%20%7B%0A%20%20%20%20time%20%3D%20time%20%2B%20%22000%22%0A%20%20%20%20time%20%3D%20parseInt\(time\)%3B%0A%20%20%20%20var%20date%20%3D%20new%20Date\(time\)%0A%20%20%20%20return%20formatTimeToStr\(date%2C%20'yyyy\-MM\-dd%20hh%3Amm%3Ass'\)%0A%20%20%7D%20else%20%7B%0A%20%20%20%20return%20'\-\-'%0A%20%20%7D%0A%7D%0A%60%60%60%0Adate%E5%AD%97%E6%AE%B5%E6%A0%87%E8%AF%86%E6%B7%BB%E5%8A%A0%3Aformatter%3D%22substrContent%22%20%0A%60%60%60%0Aconst%20substrContent%20%3D%20\(val\)%20%3D%3E%20%7B%0A%20%20var%20v%20%3D%20val.headerCommon%0A%20%20if\(%20v.length%20%3E%2040\)%7B%0A%20%20%20%20return%20v.substr\(0%2C40\)%0A%20%20%7Delse%7B%0A%20%20%20%20return%20v%0A%20%20%7D%0A%7D%0A%60%60%60%0A%0ADIY\-%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81%0A\-\-\-\-\-%0A%23%23%23%23%20%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%EF%BC%8C%E5%A2%9E%E5%8A%A0%E4%B8%80%E4%B8%AADB%E9%85%8D%E7%BD%AE%E9%A1%B9%EF%BC%9A%0Avim%20config.yaml%0A%60%60%60%0Amysql\-business%3A%0A%20%20path%3A%208.142.177.235%0A%20%20port%3A%20%223306%22%0A%20%20config%3A%20charset%3Dutf8mb4%26parseTime%3DTrue%26loc%3DLocal%0A%20%20db\-name%3A%20%22seed\_pre%22%0A%20%20username%3A%20root%0A%20%20password%3A%20''%0A%20%20max\-idle\-conns%3A%2010%0A%20%20max\-open\-conns%3A%20100%0A%20%20log\-mode%3A%20error%0A%20%20log\-zap%3A%20false%0A%60%60%60%0A%0A%E7%BB%99DB%E5%A2%9E%E5%8A%A0%E5%85%AC%E5%85%B1%E9%85%8D%E7%BD%AE%EF%BC%8CDB%E9%80%89%E9%A1%B9%0Avim%20config%2Fconfig.go%0A%60%60%60%0AMysqlBusiness%20%20Mysql%20%60mapstructure%3A%22mysql\-business%22%20json%3A%22mysql\_business%22%20yaml%3A%22mysql%22%60%0A%60%60%60%0A%0A%23%23%23%23%20%E5%A2%9E%E5%8A%A0DB%E6%8E%A7%E5%88%B6%E5%99%A8%E4%BB%A3%E7%A0%81%0Atouch%20initialize%2Fbusiness.go%0Avim%20initialize%2Fbusiness.go%0A%60%60%60%0Apackage%20initialize%0A%0Aimport%20\(%0A%09%22github.com%2Fflipped\-aurora%2Fgin\-vue\-admin%2Fserver%2Fglobal%22%0A%09%22gorm.io%2Fdriver%2Fmysql%22%0A%09%22gorm.io%2Fgorm%22%0A%09%22gorm.io%2Fgorm%2Fschema%22%0A%09%22fmt%22%0A\)%0A%0Afunc%20%20BusinessDb\(\)%20\(gormDb%20\*gorm.DB%2C%20err%20error\)%20%7B%0A%09m%20%3A%3D%20global.GVA\_CONFIG.MysqlBusiness%0A%09%2F%2Fdns%20%3A%3D%20%22root%22%20%2B%20%22%3A%22%20%2B%20%22mqzhifu%22%20%2B%20%22%40tcp\(%22%20%2B%20%228.142.177.235%22%20%2B%20%22%3A%22%20%2B%20m.Port%20%2B%20%22\)%2F%22%20%2B%20%22test%22%20%2B%20%22%3F%22%20%2B%20m.Config%0A%09dns%20%3A%3D%20m.Username%20%2B%20%22%3A%22%20%2B%20m.Password%20%2B%20%22%40tcp\(%22%20%2B%20m.Path%20%2B%20%22%3A%22%20%2B%20m.Port%20%2B%20%22\)%2F%22%20%2B%20m.Dbname%20%2B%20%22%3F%22%20%2B%20m.Config%0A%09fmt.Println\(%22mysql%20link%20info%3A%20%22%20%2B%20dns\)%0A%09mysqlConfig%20%3A%3D%20mysql.Config%7B%0A%09%09DSN%3A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20dns%2C%20%20%20%2F%2F%20DSN%20data%20source%20name%0A%09%09DefaultStringSize%3A%20%20%20%20%20%20%20%20%20191%2C%20%20%20%2F%2F%20string%20%E7%B1%BB%E5%9E%8B%E5%AD%97%E6%AE%B5%E7%9A%84%E9%BB%98%E8%AE%A4%E9%95%BF%E5%BA%A6%0A%09%09DisableDatetimePrecision%3A%20%20true%2C%20%20%2F%2F%20%E7%A6%81%E7%94%A8%20datetime%20%E7%B2%BE%E5%BA%A6%EF%BC%8CMySQL%205.6%20%E4%B9%8B%E5%89%8D%E7%9A%84%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%8D%E6%94%AF%E6%8C%81%0A%09%09DontSupportRenameIndex%3A%20%20%20%20true%2C%20%20%2F%2F%20%E9%87%8D%E5%91%BD%E5%90%8D%E7%B4%A2%E5%BC%95%E6%97%B6%E9%87%87%E7%94%A8%E5%88%A0%E9%99%A4%E5%B9%B6%E6%96%B0%E5%BB%BA%E7%9A%84%E6%96%B9%E5%BC%8F%EF%BC%8CMySQL%205.7%20%E4%B9%8B%E5%89%8D%E7%9A%84%E6%95%B0%E6%8D%AE%E5%BA%93%E5%92%8C%20MariaDB%20%E4%B8%8D%E6%94%AF%E6%8C%81%E9%87%8D%E5%91%BD%E5%90%8D%E7%B4%A2%E5%BC%95%0A%09%09DontSupportRenameColumn%3A%20%20%20true%2C%20%20%2F%2F%20%E7%94%A8%20%60change%60%20%E9%87%8D%E5%91%BD%E5%90%8D%E5%88%97%EF%BC%8CMySQL%208%20%E4%B9%8B%E5%89%8D%E7%9A%84%E6%95%B0%E6%8D%AE%E5%BA%93%E5%92%8C%20MariaDB%20%E4%B8%8D%E6%94%AF%E6%8C%81%E9%87%8D%E5%91%BD%E5%90%8D%E5%88%97%0A%09%09SkipInitializeWithVersion%3A%20false%2C%20%2F%2F%20%E6%A0%B9%E6%8D%AE%E7%89%88%E6%9C%AC%E8%87%AA%E5%8A%A8%E9%85%8D%E7%BD%AE%0A%09%7D%0A%09diyConfig%20%3A%3D%20%26gorm.Config%7BDisableForeignKeyConstraintWhenMigrating%3A%20true%2C%20NamingStrategy%3A%20schema.NamingStrategy%7BSingularTable%3A%20true%7D%7D%0A%09db%2C%20err%20%3A%3D%20gorm.Open\(mysql.New\(mysqlConfig\)%2C%20diyConfig\)%0A%09fmt.Println\(%22gorm.Open%20er%20%20info%3A%22%2C%20err\)%0A%09if%20err%20\!%3D%20nil%20%7B%0A%09%09return%20nil%2C%20err%0A%09%7D%0A%09global.GVA\_DBList%20%3D%20make\(map%5Bstring%5D\*gorm.DB\)%0A%09global.GVA\_DBList%5B%22business%22%5D%20%3D%20db%0A%09return%20db%2C%20nil%0A%7D%0A%0A%60%60%60%0A%23%23%23%23%20main%E6%B7%BB%E5%8A%A0%E5%90%AF%E5%8A%A8%E6%96%B0DB%E7%9A%84%E5%85%A5%E5%8F%A3%0Avim%20main.go%0A%60%60%60%0Ainitialize.BusinessDb\(\)%0A%60%60%60%0A%23%23%23%23%20%E8%87%AA%E5%B8%A6%E7%9A%84%E6%97%B6%E9%97%B4%E6%A0%BC%E5%BC%8F%E6%98%AF%E5%9F%BA%E4%BA%8EDB%20datetime%E7%B1%BB%E5%9E%8B%EF%BC%8C%E8%80%8C%E6%97%A5%E5%B8%B8%E4%BD%BF%E7%94%A8%E7%9A%84%E4%B8%8D%E4%BE%9D%E8%B5%96DB%EF%BC%8C%E5%B0%B1%E6%98%AF%E4%B8%AAINT%E5%80%BC%0Atouch%20model%2Fmy\_model.go%0Avim%20model%2Fmy\_model.go%0A%60%60%60%0Apackage%20model%0A%0Aimport%20%22gorm.io%2Fgorm%22%0A%0Atype%20DIY\_MODEL%20struct%20%7B%0A%09ID%20%20%20%20%20%20%20%20uint%20%20%20%20%20%20%20%20%20%20%20%60gorm%3A%22primarykey%22%60%20%2F%2F%20%E4%B8%BB%E9%94%AEID%0A%09CreatedAt%20int%20%20%20%20%20%20%20%20%20%20%20%20%2F%2F%20%E5%88%9B%E5%BB%BA%E6%97%B6%E9%97%B4%0A%09UpdatedAt%20int%20%20%20%20%20%20%20%20%20%20%20%20%2F%2F%20%E6%9B%B4%E6%96%B0%E6%97%B6%E9%97%B4%0A%09DeletedAt%20gorm.DeletedAt%20%60gorm%3A%22index%22%20json%3A%22\-%22%60%20%2F%2F%20%E5%88%A0%E9%99%A4%E6%97%B6%E9%97%B4%0A%7D%0A%0A%60%60%60%0A%0A%23%23%23%23%20%E6%89%93%E5%BC%80%E8%B7%A8%E5%9F%9F%0Avim%20initialize%2Frouter.go%0A%E6%89%93%E5%BC%80%E8%B7%A8%E5%9F%9F%E6%B3%A8%E9%87%8A%EF%BC%9A%0A%3ERouter.Use\(middleware.Cors\(\)\)%20%0A%0A%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90%E9%A1%B5%E9%9D%A2%E4%BB%A3%E7%A0%81%0A\-\-\-\-\-%0A%0A%23%23%23%23%20%E5%AF%BC%E5%85%A5%E5%B8%B8%E9%87%8F%E5%80%BC%0A%3E%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90%E9%A1%B5%E9%9D%A2%E6%97%B6%EF%BC%8C%E6%9C%89%E4%BA%9B%E5%AD%97%E6%AE%B5%E6%97%B6%E6%98%AF%E6%9E%9A%E4%B8%BE%EF%BC%8C%E4%B8%BA%E4%BA%86%E5%87%8F%E5%B0%91%E4%BB%A3%E7%A0%81%E9%87%8F%EF%BC%8C%E4%BB%8E%E5%8E%9F%E9%A1%B9%E7%9B%AE%E4%B8%AD%E6%8B%BF%E5%88%B0%E8%AF%A5%E5%80%BC%EF%BC%8C%E5%B9%B6%E7%9B%B4%E6%8E%A5%E7%94%9F%E6%88%90sql%20query%EF%BC%8C%E7%9B%B4%E6%8E%A5%E6%8F%92%E5%85%A5%E5%88%B0DB%E4%B8%AD%0A%0A%E8%AF%B7%E6%B1%82%E6%8E%A5%E5%8F%A3%E6%8B%BF%E5%88%B0sql%3A%0A%3Ehttp%3A%2F%2F127.0.0.1%3A1111%2Ftools%2Fconst%2Finit%2Fdb%0A%0A%E5%AF%BC%E5%85%A5%20gva%20DB%20%E4%B8%AD%0A%60%60%60%0Adelete%20from%20sys\_dictionaries%20where%20id%20%3E%3D%2010%3B%0Adelete%20from%20sys\_dictionary\_details%20where%20sys\_dictionary\_id%20%3E%3D%2010%20%0A%60%60%60%0A%0A%23%23%23%23%20%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E5%8C%85%EF%BC%9A%0A%3E%E6%89%80%E6%9C%89%E7%9A%84%E9%A1%B5%E9%9D%A2%EF%BC%8C%E5%BE%97%E7%BB%99%E4%B8%80%E4%B8%AA%E5%85%AC%E5%85%B1%E5%8C%85%0A%0A1.%20%E7%B3%BB%E7%BB%9F%E5%B7%A5%E5%85%B7\-%3E%E8%87%AA%E5%8A%A8%E5%8C%96package\-%3E%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E5%8C%85%3Aop%20%E8%BF%90%E7%BB%B4%20%E8%BF%90%E7%BB%B4%E6%97%A5%E5%B8%B8%E7%BB%B4%E6%8A%A4%E4%BD%BF%E7%94%A8%E7%9A%84%E4%B8%80%E4%BA%9B%E8%A1%A8%E6%A0%BC%E6%95%B0%E6%8D%AE%E5%B7%A5%E5%85%B7%0A2.%20%E7%B3%BB%E7%BB%9F%E5%B7%A5%E5%85%B7\-%3E%E7%94%9F%E6%88%90%E4%BB%A3%E7%A0%81%E5%99%A8%0A%0A%23%23%23%23%20%E5%88%9B%E5%BB%BA%E6%A0%B9%E8%8F%9C%E5%8D%95%EF%BC%9A%0A%3E%E7%9B%AE%E5%89%8D%E5%85%88%E6%8A%8A%E6%89%80%E6%9C%89%E7%9A%84%E9%A1%B5%E9%9D%A2%E7%BB%9F%E4%B8%80%E6%94%BE%E5%9C%A8%E8%BF%99%E4%B8%AA%E6%96%B0%E7%9A%84%E8%8F%9C%E5%8D%95%E4%B8%AD%EF%BC%8C%E5%90%8E%E7%BB%AD%E5%86%8D%E8%80%83%E8%99%91%E5%A6%82%E4%BD%95%E6%94%BE%E7%BD%AE%E4%BD%8D%E7%BD%AE%0A%0A%E8%B6%85%E7%BA%A7%E7%AE%A1%E7%90%86%E5%91%98\-%3E%E8%8F%9C%E5%8D%95%E7%AE%A1%E7%90%86\-%3E%E6%96%B0%E5%BB%BA%E6%A0%B9%E8%8F%9C%E5%8D%95\-%3E%E8%BF%90%E7%BB%B4\-%3Eop%0A%0A%E7%BB%99%E6%96%B0%E5%88%9B%E5%BB%BA%E7%9A%84%E6%A0%B9%E8%8F%9C%E5%8D%95%EF%BC%8C%E6%B7%BB%E5%8A%A0%E4%B8%80%E4%B8%AA%E7%B4%A2%E5%BC%95%E9%A1%B5%E9%9D%A2%EF%BC%8C%E7%94%A8%E4%BA%8E%E5%8C%85%E5%90%AB%E5%AD%90%E8%8F%9C%E5%8D%95%E7%9A%84%E4%BB%A3%E7%A0%81%3A%0A%3Etouch%20view%2Fop%2Findex.html%0A%0A%E6%96%B0%E5%BB%BA%E5%AD%90%E8%8F%9C%E5%8D%95%EF%BC%9A%0A%3Eview%2Fop%2FstatisticsLog.vue%0A%0A%23%23%23%23%20%E8%87%AA%E5%8A%A8%E5%88%9B%E5%BB%BA%E4%BB%A3%E7%A0%81%EF%BC%9A%0A%E7%B3%BB%E7%BB%9F%E5%B7%A5%E5%85%B7\-%3E%E4%BB%A3%E7%A0%81%E7%94%9F%E6%88%90%E5%99%A8\-%3E%E6%95%B0%E6%8D%AE%E5%BA%93%E5%90%8D\-%3E%E8%A1%A8%E5%90%8D\-%3E%E7%94%9F%E6%88%90%E4%BB%A3%E7%A0%81%0A%0A%E9%9C%80%E8%A6%81%E5%88%9B%E5%BB%BA%E7%9A%84%E9%A1%B5%E9%9D%A2%EF%BC%9A%0A%60%60%60%0Astatistic\_log%0Aproject%0Ainstance%0Aserver%0A%60%60%60%0A%E8%B0%83%E6%95%B4%E5%AD%97%E6%AE%B5%E7%9A%84%E6%97%B6%E5%80%99%E6%B3%A8%E6%84%8F%E4%B8%8B%EF%BC%9Atinyint%20%E5%AE%83%E6%98%AF%E7%BB%99%E8%BD%AC%E6%88%90%E4%BA%86bool%20%E5%BE%97%E6%89%8B%E5%8A%A8%E6%94%B9%E6%88%90int%0A%0A%0A%E5%B0%86%E5%88%9A%E5%88%9A%E7%94%9F%E6%88%90%E7%9A%84%E6%A8%A1%E6%9D%BF%E6%96%87%E4%BB%B6%EF%BC%8C%E8%BD%AC%E7%A7%BB%E5%88%B0%E6%8C%87%E5%AE%9A%E7%9B%AE%E5%BD%95%E4%B8%8B%EF%BC%9A%0A%3Emkdir%20web%2Fsrc%2Fview%2Fop%0A%3Emv%20web%2Fsrc%2Fview%2FstatisticsLog%2F\*%20web%2Fsrc%2Fview%2Fop%0A%0A%E4%BF%AE%E6%94%B9model%0A%60%60%60%0Amodel.DIY\_MODEL%0A%60%60%60%0A%0A%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81%E4%BF%AE%E6%94%B9%EF%BC%9A%0Avim%20server%2Fservice%2Fop%2Fstatistic\_log.go%0Areplace%3A%0A%60%60%60%0Aglobal.GVA\_DB%0Aglobal.GetGlobalDBByDBName\(%22business%22\)%0A%60%60%60%0A%0A%E7%BB%99%E8%A7%92%E8%89%B2%E8%AE%BE%E7%BD%AE%E6%96%B0%E6%9D%83%E9%99%90%EF%BC%8C%E9%80%80%E5%87%BA%20%EF%BC%8C%E9%87%8D%E6%96%B0%E7%99%BB%E9%99%86%E5%8D%B3%E5%8F%AF%E3%80%82%0A%0A%0A%23%23%23%23%20%E5%AF%BC%E5%87%BADB%0A%2Fsoft%2Fmysql%2Fbin%2Fmysqldump%20\-uroot%20\-h8.142.177.235%20\-p'%23%23%23%23%23'%20gva%20%3E%20%2Fdata%2Fwww%2Fgva.sql%0A%0A%0A%E8%87%AA%E6%9C%89%E5%AE%89%E8%A3%85%0A\-\-\-\-\-%0A%3Egit%20http%3A%2F%2F192.168.1.22%3A40080%2Ffrontend%2Fseed\_admin\_vue.git%0Agit%20http%3A%2F%2F192.168.1.22%3A40080%2Fbackend%2Fseed\_admin\_go.git%0A%0A%3E%E4%BB%A5%E4%B8%8A%E6%98%AF%E5%B1%9E%E4%BA%8E%E7%A9%BA%E9%A1%B9%E7%9B%AE%E5%AE%89%E8%A3%85%EF%BC%8C%E4%B8%8B%E9%9D%A2%E6%98%AF%EF%BC%9A%E7%9B%B4%E6%8E%A5%E4%B8%8B%E8%BD%BD%E5%B7%B2%E7%BB%8F%E9%83%A8%E7%BD%B2%E5%A5%BD%E7%9A%84%E9%A1%B9%E7%9B%AE%0A%0A%E4%B8%8B%E8%BD%BD%E4%BB%A3%E7%A0%81%EF%BC%9A%0A%3Echown%20\-R%20wangdongyan%3A\_www%20%2Fdata%2Fwww%2Fgolang%2Fseed\_admin\_vue%0A%3Echown%20\-R%20wangdongyan%3A\_www%20%2Fdata%2Fwww%2Fgolang%2Fseed\_admin\_go%0A%0A%E5%90%8E%E7%AB%AF%E5%AF%86%E7%A0%81%EF%BC%9Aadmin%20123456%0A%0A%E5%AE%89%E8%A3%85%E5%89%8D%E7%AB%AF%0A\-\-\-\-%0A%0Acd%20seed\_admin\_vue%0A%3Enpm%20%20install%20\-\-unsafe\-perm%3Dtrue%20\-\-allow\-root%20%0A%0A%E5%AE%89%E8%A3%85%E4%B8%8D%E6%88%90%E5%8A%9F%E7%9A%84%E8%AF%9D%EF%BC%8C%E6%8D%A2%E4%B8%AA%E6%B7%98%E5%AE%9D%E7%9A%84%E6%BA%90%EF%BC%9A%0A%3Ecnpm%20install%20\-\-unsafe\-perm%3Dtrue%20\-\-allow\-root%20%0A%0A%E4%B8%A4%E7%A7%8D%E6%96%B9%E5%BC%8F%E5%9D%87%E5%8F%AF%EF%BC%8C%E5%AE%98%E6%96%B9%E6%9B%B4%E6%8E%A8%E8%8D%90npm%EF%BC%8C%E4%BD%86%E9%80%9F%E5%BA%A6%E6%B2%A1%E6%9C%89cnpm%E5%BF%AB%0A%0A%E5%90%AF%E5%8A%A8%3A%0A%3Enpm%20server%0A%0A%E6%9F%A5%E7%9C%8B%E4%B8%8B%E7%AB%AF%E5%8F%A3%E5%8F%B7%EF%BC%9A%0Alsof%20\-i%3A8080%20%0A%0A%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%BF%AE%E6%94%B9%EF%BC%9A%0A%60%60%60%0Avim%20.env.product%0A%3EVITE\_BASE\_API%20%3D%20http%3A%2F%2Fadminapi.seedreality.com%0A%60%60%60%0A%0Avim%20vite.config.js%0A%3EoutDir%3A%20'..%2Fdist'%0A%0A%E4%BA%A7%E5%87%BA%E7%9B%AE%E5%BD%95%2C%E4%BF%AE%E6%94%B9%E7%BC%96%E8%AF%91JS%E8%BE%93%E5%87%BA%E7%9B%AE%E5%BD%95%0A%0A%0A%E5%AE%89%E8%A3%85%E5%90%8E%E7%AB%AF%0A\-\-\-\-%0A%23%23%23%23%20%E6%8A%8ADB%E5%AF%BC%E5%85%A5%E5%88%B0%E6%9C%AC%E5%9C%B0%EF%BC%9Agva.sql%0Asource%20gva.sql%0A%0A%0A%0Amod%E4%B8%8B%E5%8C%85%EF%BC%8C%E5%B9%B6%E5%90%AF%E5%8A%A8%E5%90%8E%E7%AB%AF%3A%0A%60%60%60%0Acd%20seed\_admin\_go%0Ago%20mod%20tidy%0Ago%20run%20main%0A%60%60%60%0A%0A%0A%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%EF%BC%9A%0Avim%20config.yaml%0A1.%20DB\-MYSQL%E9%85%8D%E7%BD%AE%EF%BC%8C%E4%B8%80%E4%B8%AAGVA%E8%87%AA%E5%B8%A6%E7%9A%84%EF%BC%8C%E5%8F%A6%E4%B8%80%E4%B8%AA%E8%BF%9E%E6%8E%A5%E5%88%B0%E4%B8%9A%E5%8A%A1%E5%BA%93%0A2.%20%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90%E4%BB%A3%E7%A0%81%E7%9A%84%E6%A0%B9%E7%9B%AE%E5%BD%95%EF%BC%8Croot%3A%20%2Fdata%2Fwww%2Fgolang%0A%0A%0A%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90%E4%BB%A3%E7%A0%81%EF%BC%9A%0A1.%20%E5%85%88%E7%A1%AE%E5%AE%9A%E6%94%BE%E5%9C%A8%E5%93%AA%E4%B8%AA%E8%8F%9C%E5%8D%95%E4%B8%8B%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%98%AF%E5%B7%B2%E5%AD%98%E5%9C%A8%E7%9A%84%E6%A0%B9%E7%9B%AE%E5%BD%95%E5%B0%B1%E5%BF%BD%E7%95%A5%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%98%AF%E6%96%B0%E7%9A%84%E6%A0%B9%E7%9B%AE%E5%BD%95%EF%BC%8C%E9%82%A3%E5%BE%97%E6%96%B0%E5%BB%BA%E4%B8%80%E4%B8%AA%0A2.%20%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E5%AD%90%E8%8F%9C%E5%8D%95%0A3.%20%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90%E7%9A%84%E4%BB%A3%E7%A0%81%EF%BC%8C%E5%BE%97%E6%9C%89%E4%B8%AA%E5%8C%85%E5%90%8D\(package\)%EF%BC%8C%E8%BF%99%E9%87%8C%E5%A6%82%E6%9E%9C%E5%B7%B2%E7%BB%8F%E5%AD%98%E5%9C%A8%E5%B0%B1%E5%BF%BD%E7%95%A5%EF%BC%8C%E4%B8%8D%E5%AD%98%E5%9C%A8%20%E6%89%8B%E5%8A%A8%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%0A3.%20%E7%B3%BB%E7%BB%9F%E5%B7%A5%E5%85%B7\-%E3%80%8B%E4%BB%A3%E7%A0%81%E7%94%9F%E6%88%90%E5%99%A8\-%E3%80%8B%E6%95%B0%E6%8D%AE%E5%BA%93%E5%90%8D\-%E3%80%8B%E8%A1%A8%E5%90%8D\-%E3%80%8B%E4%B8%8B%E9%9D%A2%E5%B0%B1%E6%A0%B9%E6%8D%AE%E8%A1%A8%E7%9A%84%E5%AD%97%E6%AE%B5%E7%94%9F%E6%88%90%E4%BA%86table%0A4.%20%E9%80%89%E6%8B%A9%E5%8C%85%E5%90%8D%0A5.%20%E5%AF%B9%E6%AF%8F%E4%B8%AA%E5%AD%97%E6%AE%B5%E8%BF%9B%E8%A1%8C%E7%BC%96%E8%BE%91%0A%20%20%20%201.%20%E6%98%AF%E5%90%A6%E6%B7%BB%E5%8A%A0%E6%90%9C%E7%B4%A2%E5%8A%9F%E8%83%BD%0A%20%20%20%202.%20%E5%AE%83%E4%BB%85%E6%94%AF%E6%8C%81varchar%20int%20%EF%BC%8C%E5%83%8F%EF%BC%9Atinyint%20%E4%BB%96%E6%98%AF%E7%BB%99%E8%BD%AC%E6%88%90%E4%BA%86bool%EF%BC%8C%E8%BF%99%E9%87%8C%E6%B3%A8%E6%84%8F%E4%B8%8B%EF%BC%8C%E6%94%B9%E5%9B%9Eint%0A%20%20%20%203.%20%E5%A6%82%E6%9C%89%E6%9E%9A%E4%B8%BE%E5%AD%97%E6%AE%B5%EF%BC%8C%E8%AE%B0%E5%BE%97%E8%A6%81%E9%80%89%E6%8B%A9%E5%85%B7%E4%BD%93%E7%9A%84%E5%B8%B8%E9%87%8F%E5%88%97%E8%A1%A8%0A6.%20%E7%82%B9%E5%87%BB%E7%A1%AE%E8%AE%A4%E5%90%8E%EF%BC%8C%E5%AE%83%E4%BC%9A%E7%94%9F%E6%88%90%E5%89%8D%E7%AB%AF%E3%80%81%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81%0A7.%20%E5%89%8D%E7%AB%AF%E4%BB%A3%E7%A0%81%E5%9C%A8%EF%BC%9Aview%2FpackageName%E4%B8%8B%E9%9D%A2%EF%BC%8C%E4%BD%A0%E8%BD%AC%E7%A7%BB%E5%88%B0%E8%87%AA%E5%B7%B1%E6%83%B3%E6%94%BE%E7%9A%84%E4%BD%8D%E7%BD%AE%EF%BC%8C%E7%84%B6%E5%90%8E%E4%BF%AE%E6%94%B9%E4%BD%A0%E4%B8%8A%E9%9D%A2%E5%88%9A%E5%88%9A%E5%88%9B%E5%BB%BA%E7%9A%84%E5%AD%90%E8%8F%9C%E5%8D%95%EF%BC%8C%E6%8A%8A%E8%BF%99%E4%B8%AA%E6%96%87%E4%BB%B6%E5%9C%B0%E5%9D%80%E5%A4%8D%E5%88%B6%E8%BF%9B%E5%8E%BB%0A6.%20%E5%90%8E%E7%AB%AF%E4%BB%A3%E7%A0%81%E7%95%A5%E5%A4%9A%EF%BC%8C%E4%B8%8D%E8%AF%A6%E7%BB%86%E5%86%99%E4%BA%86%EF%BC%8C%E5%85%B7%E4%BD%93%EF%BC%8C%E4%BD%A0%E5%B0%B1%E9%9C%80%E8%A6%81%E4%BF%AE%E6%94%B9%EF%BC%9A%0A%20%20%20%201.%20model%2FpackageName%2F%E6%96%87%E4%BB%B6%20%20%EF%BC%8C%E6%8A%8A%E5%AE%83%E7%9A%84%20%20global.GVA\_MODEL%20%E6%94%B9%E6%88%90%20model.DIY\_MODEL%0A%20%20%20%202.%20%E5%A6%82%E6%9E%9C%E4%B8%8D%E6%98%AFGVA%E5%BA%93%E7%9A%84%E8%A1%A8%EF%BC%8C%E6%98%AF%E4%B8%9A%E5%8A%A1%E8%A1%A8%EF%BC%8C%20service%2FpackageName%2F%E6%96%87%E4%BB%B6%20%EF%BC%8C%E6%8A%8A%E5%AE%83%E7%9A%84%20global.GVA\_DB%20%E7%BB%9F%E4%B8%80%E6%8D%A2%E7%A7%98%20%20global.GetGlobalDBByDBName\(%22business%22\)%0A%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%0A%23%23%23%23%20%E4%BB%BB%E5%8A%A11%EF%BC%9A%0A%0A%E9%A1%B5%E9%9D%A2%E4%BF%AE%E6%94%B9%0A%23%23%23%23%20%E5%88%A0%E9%99%A4%E9%BB%98%E8%AE%A4%E5%AF%86%E7%A0%81%2F%E5%A7%8B%E5%8C%96%E6%8C%89%E9%92%AE%0A%0Aview%2Flogin%2Flogin.vue%0A168%E8%A1%8C%EF%BC%9A%0A%E5%88%A0%E9%99%A4admin%20%E5%92%8C%20123456%0A%0A%23%23%23%23%20%E6%8D%A2LOGO%0A%23%23%23%23%20%E5%88%A0%E9%99%A4%E5%B0%BE%E9%83%A8%0A%23%23%23%23%20%E4%B8%AA%E4%BA%BA%E4%BF%A1%E6%81%AF%EF%BC%8C%E4%BF%AE%E6%94%B9%E5%A4%B4%E9%83%A8%20%E6%89%8B%E6%9C%BA%E5%8F%B7%20%E5%9C%B0%E5%9D%80%20%E9%82%AE%E7%AE%B1%20%E7%AD%89%E7%AD%89%0A%E7%AD%89%E7%AD%89%E5%90%A7%EF%BC%8C%E6%8A%8A%E8%BF%99%E4%B8%AA%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE%E8%87%AA%E5%B8%A6%E7%9A%84%E4%B8%80%E4%BA%9B%E4%B8%9C%E8%A5%BF%EF%BC%8C%E9%83%BD%E5%BC%84%E6%8E%89%E5%90%A7%E3%80%82%0A%0A%E4%BB%BB%E5%8A%A12%EF%BC%9A%0A%E8%87%AA%E5%B7%B1%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E9%A1%B5%E9%9D%A2%EF%BC%8C%E5%A6%82%EF%BC%9A%E5%A3%B0%E7%BD%91%E5%9B%9E%E8%B0%83%E8%BF%99%E5%BC%A0%E8%A1%A8%0A%0A%E4%B8%A4%E4%B8%AA%E4%BB%BB%E5%8A%A1%E7%9A%84%E6%A0%B8%E5%BF%83%E6%98%AF%E7%86%9F%E6%82%89%E8%BF%99%E4%B8%AA%E5%90%8E%E5%8F%B0%E9%A1%B9%E7%9B%AE%E7%9A%84%E4%BB%A3%E7%A0%81
