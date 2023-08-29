# php.ini优化

如果用户，在PHP脚本执行期间，结束连接，但是PHP端依然执行PHP~

ignore\_user\_abort\(FALSE\);

connection\_status\(\) \!= CONNECTION\_NORMAL

onnection\_status\(\) 函数返回当前的连接状态。

0 \- CONNECTION\_NORMAL \- 连接运行正常

1 \- CONNECTION\_ABORTED \- 连接由用户或网络错误终止

2 \- CONNECTION\_TIMEOUT \- 连接超时

3 \- CONNECTION\_ABORTED & CONNECTION\_TIMEOUT

fastcgi\_finish\_request\(\): [PHP FPM](http://php-fpm.org/)模式下，fastcgi\_finish\_request函数以前的输出是属于客户端的，而之后就不会显示了

pm.max\_requests的作用是发送多少个 请求后会重启该线

max\_children，这个是每次php\-fpm会建立多少个进程

\<\<\<:heredoc

$a = \<\<\<make

......

make;

email:发送客户端\-\>SMTP接收服务器\-\>服务端\-\>POP3/IMAP\-\>接收客户端

@extract\($\_GET,"EXTR\_OERWRITE"\);   //如果变量名，重复，覆盖    将$\_GET数据里的值，分别根据键创建变量

PHP中：&& || and or 都可以，但&& || 优先于 and or

@:屏蔽错误

extension\_loaded\('mbstring'\):判断扩展 是否 被打开 

**\[php.ini\]**

set\_time\_limit\(0\):防止超时

ini\_set\('display\_erros', false\);

error\_reporting\(0\);//禁止错误报告

error\_reporting\(2047\)  E\_ALL

log\_errors = On

disable\_functions = phpinfo,exec.. //禁止一些关键函数的使用

safe\_mode = on

short\_open\_tag = On   打开  \<? ?\>  格式

date.timezone=Asia/Shanghai

file\_uploads = On 上传开关

upload\_tmp\_dir = "c:/wamp/tmp" 上传临时路径

upload\_max\_filesize = 2M 上传文件大小 

post\_max\_size = 8M POST大小

**函数**

\[系统安全\]

escapeshellarg\(\) 或 escapeshellcmd\(\) //确保 用户输入的指令 无恶意代码

system\('ls'.escapeshellarg\($dir\)\);   //PHP 执行LINUX 脚本

stripslashes\(\): //清除 用户输入字符串中的 反斜杠

htmlentities\($str\); //将$str 中 的 \<\>等 HTML特殊标记 转换成 $nbst;格式

htmlspecialchars\($str\); //同上,可格式化汉字

strip\_tags\($str\) //去除 $str 中的 HTML标签

addslashes\(\): //对   用户输入字符串中的 引号、反斜杠 强行 加上 反斜杠

mysql\_real\_escape\_string\(\): //同上,会判断字符集，但是对PHP版本有要求  php5

mysql\_escape\_string\(\): //同上,不考虑连接的当前字符集

urlencode\(string str\):汉字和特殊字符编译成URL码

urldecode\(string str\):汉字和特殊字符URL码解码 

//判断变量类型

is\_bool is\_string is\_float is\_dobule 

is\_int is\_null is\_array is\_object is\_object

is\_numberic    检查是否为数字，或者由数字组成的字符串

settype\($variable,$type\):强制转换类型

gettype\(\):查看数据类型

header\('Refresh: 10; url=b.php'\);

header 重定向后，header 后面的PHP代码还会被执行　

header\(\)函数之前不允许有任何输出，不过，PHP默认把output\_buffering = on  浏览器缓冲打开

所以不会报错，修改 output\_buffering = off 才会出现警告

\[页面缓存\]

output\_buffering = off //缓冲开关　　一般PHP输出，会存入缓存，再打印到济览器上

ob\_start\(\);//启用output buffering机制。

ob\_flush\(\);  //将IE中缓存的数据，发送出去

flush\(\);     //打印非缓存中的数据，和ob\_flush\(\)配合着使用

ob\_implicit\_flush\(true\);  强制每当有输出的时候，即刻把输出发送到浏览器

ob\_end\_clean\(\)  就是终止缓冲。不用等到output\_buffering的索引值满，再发到浏览器上

ob\_end\_flush\(\)  就是终止缓冲。不用等到output\_buffering的索引值满，再发到浏览器上

$content = ob\_get\_contents\(\);//取得php页面输出的全部内容  

\[字符串\]

stroupper\(\):将字符串转换成大写

strtolower\(\):将字符串转换成小写

ucfirst\(\):将字符串首字母转换大写

ucwords\(\):将字符串中单词首字母大写

join\(" ",$arr\); 字符 连接 数组 并 组成新的字符串;

strrev\($str\):反序字符串  但不能处理汉字

\[oop\]

func\_get\_args\(\):动态将 参数 赋 给一个数组

func\_num\_args\(\):动态将 参数 总和 

func\_num\_arg\(\) :动态将 一个参数 

\[$\_SERVER\] 

不一一列举了

\[数组\]

implode 数组转:a,b,c,d

array\_unique   重复数组删除

array\_values   数组重新索引

in\_array\("str",$array\)  确定str是否为  数组中的值

prec\($array\):向上移动指针，一次

next\($array\):移动指针，并返回值

reset\($array\):移动指针于开始，并返回值

end\($array\):移动指针于结束，并返回值

array\_push\($str,"value"\):在数组的末尾增加元素

array\_unshift:在数组开始增加元素

array\_pop:返回数组的末尾元素

array\_shift:返回数组中的首元素

array\_unique:删除数组中重复值，并返回一个新的数组

sort\($array\):排序－升序

rsort\($array\):排序－降序

array\_merge\($array1,$array2,$array3...\):合并多个数组

array\_slice\($array,n,m\):数组截取，从第N个位置，截取M个元素

explode\("value",$STR\):将字符串，按照指定符号进行分割，并将生成新的数组

implode\("value",$array\):将数组，按照指定符号进行合并，并将生成新的字符串

range\(2,10,2\):创建一个，10以内，步长为2的，数组

array\_sum\($array\):对数组求和

key\(\):取当数据的　键名

\[文件操作\]

fileowner\(\):返回该文件所有者的ID

chown\(\):将 fileowner\(\)的值转换成字符串

filegroup\(\):返回该文件所有者的组

chgrp\(\):将 filegroup\(\)的值转换成字符串

fopen\("filename\_url",openMode\[,include\_path,context\]\)

filename\_url:http ftp 绝对 相对    \(远程，打开文件\(php.ini   allow\_url\_fopen = on\)\)

r r\+ :移到文件头截取内容为0，读  读写

w w\+ :移到文件头截取内容为0，写  读写 ，如果文件不存在尝试创建

a a\+ :移到文件头尾，写 读写，如果文件不存在尝试创建

file\_get\_contents\("filename\_url",openMode\[,include\_path,context,start,max\_length\]\):读取文件内容，到一个字符串中

unlink\(\):删除文件

fclose\(\):关闭文件

flokc\(\):锁定文件，当对文件进行写操作时，需要锁定，防止其它用户写入，损坏文件

fsize\(\):取得当前文件大小

$LastAccess = fileatime\("data.txt"\); //文件的最后存取时间

file\_exists\(\):检查文件是否存在

file\_atime\(\):文件上次访问日期

is\_file\(\):判断是否为一个正常的文件

\[目录操作\]

is\_dir\(\):检验目录合法

opendir\(\):打开目录

readdir\(\):读取目录

mkdir\(\):建立目录

closedir\(\):关闭目录

rmdir\(\):删除目录

chdir\("img"\):设置当前工作目录，类似于 cd img

getcwd\(\):输出当前路径

disk\_free\_space\("c:"\):检查当前磁盘盛余空间

drealpath\("$path"\):删除 path 中的  空格 等字符  保持  相对路径 的正确，如果 删除失败 返回 flase

basepath\("$path"\):返回 PATH 中的 文件名 部分

\[魔术方法\]

类中的魔术方法:

\_\_construct\(\)   构造函数

\_\_destruct\(\)    析构函数,执行$a = new class\(\);a=null时调用此方法

\_\_call\($func\_name,$arguments\):访问了某类中不存在的方法，调用此函数,可以实现　类似　重载的功能

\_\_callStatic\(\):访问某个不存在的静态方法

\_\_set\(\):访问某类中不存在的属性并赋值，调用此函数

\_\_get\(\):访问某类中不存在的属性，调用此函

\_\_toString \(\):$a = new class\(\);echo $a

\_\_sleep:

\_\_wakeup

\_\_clone

\_\_isset:想判断一个类中的某个私有成员变量是否被定义

\_\_unset:注销一个类中的某个私有成员变量

\_\_set\_state\(\)

魔术函数

//自动加载类文件\(类名.class.php\)

function \_\_autoload\($class\_name\)

{

require\_once\("../$class\_name.class.php"\);

}

\_\_FILE\_\_：魔术常量 ，返回当前执行PHP脚本的完整路径和文件名,包含一个绝对路径

\_\_LINE\_\_：魔术常量 ，当前行，若引用文件\(include\)则在引用文件内的该常量为引用文件的行，而不是引用它的文件行

dirname\(\_\_FILE\_\_\);     获取当前文件所在目录

dirname\(dirname\(\_\_FILE\_\_\)\); 得到的是文件上一层目录名

A instanceof B  判断A是否是B的实例

get\_class\(new className\(\)\):返回 实例化 对象的  原始 引用

get\_class\_methods\(className\):返回 类 的 方法 名

get\_class\_vars\(className\):返回 类 中 关联 的 变量 

get\_parent\_class\(className\):返回 当前 类的父类名  如没有 父类，返回 假

class\_exists\(classname\):判断 类是否 存在

class\_method\(className,method\):判断 某类中的某方法 是否存在 

\[mysql\]

net start mysql  启动MYSQL

net stop  mysql  停止MYSQL

mysql\_pconnect\(\):建立一个持久链接，mysql\_close\(\)对其无效.执行过程：首先，检查，之前是否有连接，如有，不执行

mysql\_close\(\):关闭数据库，如无参数，关闭上一个打开的数据库

mysql\_data\_seek\($quert,rows\):移动指针到某行

mysql\_error\(\):返回错误信息

$link = mysql\_connect\('mysql\_host', 'mysql\_user', 'mysql\_password'\);

mysql\_select\_db\('my\_database',$link\);

mysql\_query\($sql\);

mysql\_affected\_rows:执行一条SQL之后，所影响的行数

mysql\_fetch\_array \-\-  从结果集中取得一行作为关联数组，或数字数组，或二者兼有 

mysql\_fetch\_assoc \-\-  从结果集中取得一行作为关联数组 

mysql\_fetch\_field \-\-  从结果集中取得列信息并作为对象返回 

\[字符集\]

big5\(繁体，香港 台湾 澳门\)  ISO\-8859\-1,ISO\-8859\-2\(又称：Latin\-1,Latin\-2....西欧,中欧\)

utf\-8\(UNICODE\)  gb2312\(gbk,是对gb2312扩充\)

ord\('A'\): 65   将参数 转换成 ASCII 码值

chr\(65\):'A'    将ASCII 码值 转换成 字符

$query = array\('name'=\>' 你好','age'=\>24, 'sex'=\>'1', 'work'=\>'IT', 'company'=\>'zol'\);

http\_build\_query\($query\);//将数组转化成URL格式的字符串

decbin\(\) 十进制数转换成二进制数

iconv\("gb2312","ISO\-8859\-1","我们"\);

iconv\_strlen\("王东",'UTF\-8'\);

mb\_convert\_encoding\("妳係我的友仔", "UTF\-8", "GBK"\); 

mb\_substr\($tmpvar,0,1,'UTF8'\); //可处理汉字

mb\_strlen\($tmpvar,'UTF8'\); //可处理汉字

\[日期\]

mktime\(hour,minute,second,month,day,year,is\_dst\)

$today=date\("Y\-m\-d"\);   //今天日期

date\("Y\-m\-d H:i:s"\):取得当前日期＋时间

getdate\(\);取得当前日期＋时间，并返回于数组

checkdate\(int month,int day,int year\):检查日期是否有效

昨天日期  

date\("Y\-m\-d H:i:s",time\(\)\-\(60\*60\*24\)\);

date\("Y\-m\-d", mktime\(0, 0, 0,date\("m"\),date\("d"\)\-1,date\("Y"\)\)\);

date\("Y:m:d H:i:s",strtotime\("\-1 day"\)\);

date\('Y\-m\-d',strtotime\('last sunday'\)\); //上周日

a \- "am" 或是 "pm"

A \- "AM" 或是 "PM"

d \- 几日，二位数字，若不足二位则前面补零; 如: "01" 至 "31"

D \- 星期几，三个英文字母; 如: "Fri"

F \- 月份，英文全名; 如: "January"

h \- 12 小时制的小时; 如: "01" 至 "12"

H \- 24 小时制的小时; 如: "00" 至 "23"

g \- 12 小时制的小时，不足二位不补零; 如: "1" 至 12"

G \- 24 小时制的小时，不足二位不补零; 如: "0" 至 "23"

i \- 分钟; 如: "00" 至 "59"

j \- 几日，二位数字，若不足二位不补零; 如: "1" 至 "31"

l \- 星期几，英文全名; 如: "Friday"

m \- 月份，二位数字，若不足二位则在前面补零; 如: "01" 至 "12"

n \- 月份，二位数字，若不足二位则不补零; 如: "1" 至 "12"

M \- 月份，三个英文字母; 如: "Jan"

s \- 秒; 如: "00" 至 "59"

S \- 字尾加英文序数，二个英文字母; 如: "th"，"nd"

t \- 指定月份的天数; 如: "28" 至 "31"

U \- 总秒数

w \- 数字型的星期几，如: "0" \(星期日\) 至 "6" \(星期六\)

Y \- 年，四位数字; 如: "1999"

y \- 年，二位数字; 如: "99"

z \- 一年中的第几天; 如: "0" 至 "365" 

![448f26fcace6e7c2e5274ab77d7566d2.jpg](image/448f26fcace6e7c2e5274ab77d7566d2.jpg)
