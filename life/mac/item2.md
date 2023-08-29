# item2

比mac下自带的terminal更人性化些，好用！

## 改变home end 行首行尾了

打开 key

add hex key

0x01

0x05

可恨的ctrl \+ c 中断进程，也不能用了，想办法解决

stty \-a ，查看当前iterm2工具 键盘组合键 转换成的 \<发送值\>

失败......

ps:正常不安装，my\-zsh，上面需要改，这了后，发现有冲突！！！

打开全局快捷键

> sys setup \-\> keys \-\> hotKey \-\> ctrl\+空格

account \-\>添加自启

也可以设置个背影图

## 可选\-关闭鼠标选中复制

> profiles\-\>default\-\>terminal\-\> enable mouse reporting

关闭后，改用ctrl\+c复制，另外，鼠标滚动也失效，得重新配置下

## 高级配置

echo $0

正常返回是：\-zsh，如果不是，执行：chsh \-s /bin/zsh

默认会在:

/bin/zsh

/etc/shells 文件中，多一项：/bin/zsh

## 每开启一个shell，即执行下面指令

touch /Users/wangdongyan/Documents/login.sh

chmod 777 /Users/wangdongyan/Documents/login.sh

vim /Users/wangdongyan/Documents/login.sh

```
expect -c '
spawn sudo su - root
expect ":"
send "mqzhifu\r"
interact                                                    
'
```

改下配置

> profiles \> general \-\> custom shell \-\> /Users/wangdongyan/Documents/login.sh

## 开机自动执行脚本

touch /Users/wangdongyan/Documents/mount.sh

chmod 777 /Users/wangdongyan/Documents/mount.sh

vim /Users/wangdongyan/Documents/mount.sh

开机自启执行脚本:

```
expect -c '
spawn sudo su - root
expect ":"
send "mqzhifu\r"

expect "%"
send "mount -uw /\r"
interact
'
```

system \-\> 用户 \-\> 登陆 \-\> /Users/wangdongyan/Documents/mount.sh

## 开始配置主题

1. 这里有个小插曲：修改 mac 网络 DNS为：8.8.8.8 114.114.114.114
2. vim /etc/resolv.conf ,DNS为：8.8.8.8 114.114.114.114
    两种方式均可

先安装个插件\(翻墙\)

> sh \-c "$\(curl \-fsSL https://raw.githubusercontent.com/robbyrussell/oh\-my\-zsh/master/tools/install.sh\)"

会提示你，是否将此主题设置为默认模式，选择 yes

它会覆盖 ~/.zshrc

ll 一下，配色真的很爽

这个主题还有插件功能，设置开启一下：

vim ~/.zshrc

```
#这行注释打开，不然 local/bin 下的文件找不到
export PATH=$HOME/bin:/usr/local/bin:$PATH
#改一下主题配置
ZSH_THEME="agnoster"

plugins=(
  git
  bundler
  dotenv
  rake
  rbenv
  ruby
)

```

ll一下，发现：指令行的样式~乱码，开始下载字体

> https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fpowerline%2Ffonts%2Fblob%2Fmaster%2FMeslo%2520Slashed%2FMeslo%2520LG%2520M%2520Regular%2520for%2520Powerline.ttf

双击安装

更新设置

> profiles \> default \-\> text \-\> meslo LG M for powerline

优点

1. 支持快捷直接切换
2. 文件夹直接给上色了
3. 提示行里的目录也有配色
4. 提示行里，支持git
5. 命令\+tab ，列出所有文件的时候，居然还支持选择\-\>选中\-\>补全

发现汉字乱码

```
vim ~/.zshrc
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8
```

查看提示符

```
echo $PS1 
echo $PROMPT
```

提示符中的用户名/host/目录，太长，修复下

```
vim ~/.zshrc
DEFAULT_USER="root"
```

这里因为我每个shell窗口开启后，会自动切换到root

也可以换种方式修改

查看当前主题名

echo $ZSH\_THEME

vim /var/root/.oh\-my\-zsh/themes/agnoster.zsh\-theme

搜索 DEFAULT\_USER ,注释掉，prompt\_segment 这一行

整行注释，提示符就为 空了

以上的修改，都比较极端且无脑，下面，试下详细的个性

vim /var/root/.oh\-my\-zsh/themes/agnoster.zsh\-theme

到文件最后，会有如下结构体：

```
build_prompt() {
  RETVAL=$?
  prompt_status
  prompt_virtualenv
  prompt_aws
  prompt_context
  prompt_dir
  prompt_git
  prompt_bzr
  prompt_hg
  prompt_end
}
```

prompt\_status:这一行，就是提示符最前面的icon符号

prompt\_context:这一行，就会把整个username@host 删除掉,也就是第2种方法要注释掉的那一行

只留一个 %n :就是用户名

ompt\_segment black default "%n"

指令行最前面的小icon 闪电符号，查了它的文档，目前知道的有3种符号

> 1、闪电，每条指令前都有，不知道干啥的
> 
> 
> 2、删除，这个是ctrl\+c 会显示，执行错误指令也会显示
> 
> 
> 3、小圆圈，代理后台有启动进程，比如：mysql，但是切换个窗口后，这个小圈就没有了

所以，综合上述所看，这个icon 的功能没啥用，注释了

## 小麻烦

都弄完，重新登陆，开一个新shell，发现报错，遇到点小麻烦，执行如下：

> chmod 755 /usr/local/share/zsh
> 
> 
> chmod 755 /usr/local/share/zsh/site\-functions

或

```
chown -R $(whoami) /usr/local/share/zsh
```

或添加 vim ~/.zshrc

> ZSH\_DISABLE\_COMPFIX="true"

## shell指令行~高亮

输入指令的时候，有时候会出现错的情况

```
cd ~/.oh-my-zsh/custom/plugins  
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git

vim ~/.zshrc 

plugins=(git zsh-syntax-highlighting)
```

## git\-快捷操作

|Alias|Command           |
|-----|------------------|
|gaa  |git add \-\-all   |
|gba  |git branch \-a    |
|gcam |git commit \-a \-m|
|gd   |git diff          |
|gl   |git pull          |
|gp   |git push          |
|gr   |git remote        |
|gt   |git status        |
|gb   |git branch        |
|Alias|Command           |

zsh\-autosuggestions

autojump

%E6%AF%94mac%E4%B8%8B%E8%87%AA%E5%B8%A6%E7%9A%84terminal%E6%9B%B4%E4%BA%BA%E6%80%A7%E5%8C%96%E4%BA%9B%EF%BC%8C%E5%A5%BD%E7%94%A8%EF%BC%81%0A%0A%E6%94%B9%E5%8F%98home%20end%20%E8%A1%8C%E9%A6%96%E8%A1%8C%E5%B0%BE%E4%BA%86%20%20%0A\-\-\-\-\-%0A%E6%89%93%E5%BC%80%20key%20%20%0Aadd%20hex%20key%20%20%0A0x01%20%20%0A0x05%20%0A%0A%E5%8F%AF%E6%81%A8%E7%9A%84ctrl%20%2B%20c%20%E4%B8%AD%E6%96%AD%E8%BF%9B%E7%A8%8B%EF%BC%8C%E4%B9%9F%E4%B8%8D%E8%83%BD%E7%94%A8%E4%BA%86%EF%BC%8C%E6%83%B3%E5%8A%9E%E6%B3%95%E8%A7%A3%E5%86%B3%20%20%0Astty%20\-a%20%EF%BC%8C%E6%9F%A5%E7%9C%8B%E5%BD%93%E5%89%8Diterm2%E5%B7%A5%E5%85%B7%20%E9%94%AE%E7%9B%98%E7%BB%84%E5%90%88%E9%94%AE%20%E8%BD%AC%E6%8D%A2%E6%88%90%E7%9A%84%20%3C%E5%8F%91%E9%80%81%E5%80%BC%3E%20%20%0A%E5%A4%B1%E8%B4%A5......%0A%0Aps%3A%E6%AD%A3%E5%B8%B8%E4%B8%8D%E5%AE%89%E8%A3%85%EF%BC%8Cmy\-zsh%EF%BC%8C%E4%B8%8A%E9%9D%A2%E9%9C%80%E8%A6%81%E6%94%B9%EF%BC%8C%E8%BF%99%E4%BA%86%E5%90%8E%EF%BC%8C%E5%8F%91%E7%8E%B0%E6%9C%89%E5%86%B2%E7%AA%81%EF%BC%81%EF%BC%81%EF%BC%81%20%20%0A%0A%E6%89%93%E5%BC%80%E5%85%A8%E5%B1%80%E5%BF%AB%E6%8D%B7%E9%94%AE%0A%3Esys%20setup%20\-%3E%20keys%20\-%3E%20hotKey%20\-%3E%20%20ctrl%2B%E7%A9%BA%E6%A0%BC%20%0A%0Aaccount%20\-%3E%E6%B7%BB%E5%8A%A0%E8%87%AA%E5%90%AF%0A%0A%E4%B9%9F%E5%8F%AF%E4%BB%A5%E8%AE%BE%E7%BD%AE%E4%B8%AA%E8%83%8C%E5%BD%B1%E5%9B%BE%20%20%20%0A%0A%E5%8F%AF%E9%80%89\-%E5%85%B3%E9%97%AD%E9%BC%A0%E6%A0%87%E9%80%89%E4%B8%AD%E5%A4%8D%E5%88%B6%0A\-\-\-\-\-\-\-%0A%3Eprofiles\-%3Edefault\-%3Eterminal\-%3E%20enable%20mouse%20reporting%0A%0A%E5%85%B3%E9%97%AD%E5%90%8E%EF%BC%8C%E6%94%B9%E7%94%A8ctrl%2Bc%E5%A4%8D%E5%88%B6%EF%BC%8C%E5%8F%A6%E5%A4%96%EF%BC%8C%E9%BC%A0%E6%A0%87%E6%BB%9A%E5%8A%A8%E4%B9%9F%E5%A4%B1%E6%95%88%EF%BC%8C%E5%BE%97%E9%87%8D%E6%96%B0%E9%85%8D%E7%BD%AE%E4%B8%8B%0A%0A%0A%E9%AB%98%E7%BA%A7%E9%85%8D%E7%BD%AE%0A\-\-\-\-\-%0A%0Aecho%20%240%20%20%0A%E6%AD%A3%E5%B8%B8%E8%BF%94%E5%9B%9E%E6%98%AF%EF%BC%9A\-zsh%EF%BC%8C%E5%A6%82%E6%9E%9C%E4%B8%8D%E6%98%AF%EF%BC%8C%E6%89%A7%E8%A1%8C%EF%BC%9Achsh%20\-s%20%2Fbin%2Fzsh%20%20%0A%0A%E9%BB%98%E8%AE%A4%E4%BC%9A%E5%9C%A8%3A%20%20%0A%2Fbin%2Fzsh%20%20%0A%2Fetc%2Fshells%20%E6%96%87%E4%BB%B6%E4%B8%AD%EF%BC%8C%E5%A4%9A%E4%B8%80%E9%A1%B9%EF%BC%9A%2Fbin%2Fzsh%20%20%0A%0A%0A%0A%E6%AF%8F%E5%BC%80%E5%90%AF%E4%B8%80%E4%B8%AAshell%EF%BC%8C%E5%8D%B3%E6%89%A7%E8%A1%8C%E4%B8%8B%E9%9D%A2%E6%8C%87%E4%BB%A4%0A\-\-\-\-\-%0Atouch%20%2FUsers%2Fwangdongyan%2FDocuments%2Flogin.sh%0Achmod%20777%20%2FUsers%2Fwangdongyan%2FDocuments%2Flogin.sh%0Avim%20%2FUsers%2Fwangdongyan%2FDocuments%2Flogin.sh%20%0A%0A%60%60%60%0Aexpect%20\-c%20'%0Aspawn%20sudo%20su%20\-%20root%0Aexpect%20%22%3A%22%0Asend%20%22mqzhifu%5Cr%22%0Ainteract%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0A'%0A%60%60%60%0A%E6%94%B9%E4%B8%8B%E9%85%8D%E7%BD%AE%20%20%0A%3Eprofiles%20%3E%20general%20\-%3E%20custom%20shell%20\-%3E%20%20%2FUsers%2Fwangdongyan%2FDocuments%2Flogin.sh%20%20%0A%0A%E5%BC%80%E6%9C%BA%E8%87%AA%E5%8A%A8%E6%89%A7%E8%A1%8C%E8%84%9A%E6%9C%AC%20%0A\-\-\-\-\-%0Atouch%20%2FUsers%2Fwangdongyan%2FDocuments%2Fmount.sh%0Achmod%20777%20%2FUsers%2Fwangdongyan%2FDocuments%2Fmount.sh%0Avim%20%2FUsers%2Fwangdongyan%2FDocuments%2Fmount.sh%20%0A%E5%BC%80%E6%9C%BA%E8%87%AA%E5%90%AF%E6%89%A7%E8%A1%8C%E8%84%9A%E6%9C%AC%3A%20%20%0A%60%60%60%0Aexpect%20\-c%20'%0Aspawn%20sudo%20su%20\-%20root%0Aexpect%20%22%3A%22%0Asend%20%22mqzhifu%5Cr%22%0A%0Aexpect%20%22%25%22%0Asend%20%22mount%20\-uw%20%2F%5Cr%22%0Ainteract%0A'%0A%60%60%60%0A%0Asystem%20\-%3E%20%E7%94%A8%E6%88%B7%20\-%3E%20%E7%99%BB%E9%99%86%20\-%3E%20%2FUsers%2Fwangdongyan%2FDocuments%2Fmount.sh%0A%0A%E5%BC%80%E5%A7%8B%E9%85%8D%E7%BD%AE%E4%B8%BB%E9%A2%98%0A\-\-\-\-%0A%0A1.%20%E8%BF%99%E9%87%8C%E6%9C%89%E4%B8%AA%E5%B0%8F%E6%8F%92%E6%9B%B2%EF%BC%9A%E4%BF%AE%E6%94%B9%20mac%20%E7%BD%91%E7%BB%9C%20DNS%E4%B8%BA%EF%BC%9A8.8.8.8%20114.114.114.114%0A2.%20vim%20%2Fetc%2Fresolv.conf%20%2CDNS%E4%B8%BA%EF%BC%9A8.8.8.8%20114.114.114.114%0A%E4%B8%A4%E7%A7%8D%E6%96%B9%E5%BC%8F%E5%9D%87%E5%8F%AF%0A%0A%E5%85%88%E5%AE%89%E8%A3%85%E4%B8%AA%E6%8F%92%E4%BB%B6\(%E7%BF%BB%E5%A2%99\)%0A%3Esh%20\-c%20%22%24\(curl%20\-fsSL%20https%3A%2F%2Fraw.githubusercontent.com%2Frobbyrussell%2Foh\-my\-zsh%2Fmaster%2Ftools%2Finstall.sh\)%22%0A%0A%0A%E4%BC%9A%E6%8F%90%E7%A4%BA%E4%BD%A0%EF%BC%8C%E6%98%AF%E5%90%A6%E5%B0%86%E6%AD%A4%E4%B8%BB%E9%A2%98%E8%AE%BE%E7%BD%AE%E4%B8%BA%E9%BB%98%E8%AE%A4%E6%A8%A1%E5%BC%8F%EF%BC%8C%E9%80%89%E6%8B%A9%20yes%20%20%0A%E5%AE%83%E4%BC%9A%E8%A6%86%E7%9B%96%20~%2F.zshrc%20%0A%0All%20%E4%B8%80%E4%B8%8B%EF%BC%8C%E9%85%8D%E8%89%B2%E7%9C%9F%E7%9A%84%E5%BE%88%E7%88%BD%20%0A%0A%E8%BF%99%E4%B8%AA%E4%B8%BB%E9%A2%98%E8%BF%98%E6%9C%89%E6%8F%92%E4%BB%B6%E5%8A%9F%E8%83%BD%EF%BC%8C%E8%AE%BE%E7%BD%AE%E5%BC%80%E5%90%AF%E4%B8%80%E4%B8%8B%EF%BC%9A%20%0Avim%20~%2F.zshrc%20%20%0A%0A%60%60%60%0A%23%E8%BF%99%E8%A1%8C%E6%B3%A8%E9%87%8A%E6%89%93%E5%BC%80%EF%BC%8C%E4%B8%8D%E7%84%B6%20local%2Fbin%20%E4%B8%8B%E7%9A%84%E6%96%87%E4%BB%B6%E6%89%BE%E4%B8%8D%E5%88%B0%0Aexport%20PATH%3D%24HOME%2Fbin%3A%2Fusr%2Flocal%2Fbin%3A%24PATH%0A%23%E6%94%B9%E4%B8%80%E4%B8%8B%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE%0AZSH\_THEME%3D%22agnoster%22%0A%0Aplugins%3D\(%0A%20%20git%0A%20%20bundler%0A%20%20dotenv%0A%20%20rake%0A%20%20rbenv%0A%20%20ruby%0A\)%0A%0A%0A%0A%60%60%60%0A%0All%E4%B8%80%E4%B8%8B%EF%BC%8C%E5%8F%91%E7%8E%B0%EF%BC%9A%E6%8C%87%E4%BB%A4%E8%A1%8C%E7%9A%84%E6%A0%B7%E5%BC%8F~%E4%B9%B1%E7%A0%81%EF%BC%8C%E5%BC%80%E5%A7%8B%E4%B8%8B%E8%BD%BD%E5%AD%97%E4%BD%93%20%20%0A%3Ehttps%3A%2F%2Flinks.jianshu.com%2Fgo%3Fto%3Dhttps%253A%252F%252Fgithub.com%252Fpowerline%252Ffonts%252Fblob%252Fmaster%252FMeslo%252520Slashed%252FMeslo%252520LG%252520M%252520Regular%252520for%252520Powerline.ttf%0A%0A%E5%8F%8C%E5%87%BB%E5%AE%89%E8%A3%85%0A%0A%E6%9B%B4%E6%96%B0%E8%AE%BE%E7%BD%AE%20%20%0A%3Eprofiles%20%3E%20default%20\-%3E%20text%20\-%3E%20meslo%20LG%20M%20for%20powerline%0A%0A%E4%BC%98%E7%82%B9%20%20%0A1.%20%E6%94%AF%E6%8C%81%E5%BF%AB%E6%8D%B7%E7%9B%B4%E6%8E%A5%E5%88%87%E6%8D%A2%0A2.%20%E6%96%87%E4%BB%B6%E5%A4%B9%E7%9B%B4%E6%8E%A5%E7%BB%99%E4%B8%8A%E8%89%B2%E4%BA%86%0A3.%20%E6%8F%90%E7%A4%BA%E8%A1%8C%E9%87%8C%E7%9A%84%E7%9B%AE%E5%BD%95%E4%B9%9F%E6%9C%89%E9%85%8D%E8%89%B2%0A4.%20%E6%8F%90%E7%A4%BA%E8%A1%8C%E9%87%8C%EF%BC%8C%E6%94%AF%E6%8C%81git%0A5.%20%E5%91%BD%E4%BB%A4%2Btab%20%EF%BC%8C%E5%88%97%E5%87%BA%E6%89%80%E6%9C%89%E6%96%87%E4%BB%B6%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E5%B1%85%E7%84%B6%E8%BF%98%E6%94%AF%E6%8C%81%E9%80%89%E6%8B%A9\-%3E%E9%80%89%E4%B8%AD\-%3E%E8%A1%A5%E5%85%A8%0A%0A%0A%E5%8F%91%E7%8E%B0%E6%B1%89%E5%AD%97%E4%B9%B1%E7%A0%81%20%20%0A%0A%60%60%60%0Avim%20~%2F.zshrc%0Aexport%20LC\_ALL%3Den\_US.UTF\-8%0Aexport%20LANG%3Den\_US.UTF\-8%0A%60%60%60%0A%0A%E6%9F%A5%E7%9C%8B%E6%8F%90%E7%A4%BA%E7%AC%A6%20%20%0A%60%60%60%0Aecho%20%24PS1%20%0Aecho%20%24PROMPT%0A%60%60%60%0A%0A%E6%8F%90%E7%A4%BA%E7%AC%A6%E4%B8%AD%E7%9A%84%E7%94%A8%E6%88%B7%E5%90%8D%2Fhost%2F%E7%9B%AE%E5%BD%95%EF%BC%8C%E5%A4%AA%E9%95%BF%EF%BC%8C%E4%BF%AE%E5%A4%8D%E4%B8%8B%0A%60%60%60%0Avim%20~%2F.zshrc%0ADEFAULT\_USER%3D%22root%22%0A%60%60%60%0A%E8%BF%99%E9%87%8C%E5%9B%A0%E4%B8%BA%E6%88%91%E6%AF%8F%E4%B8%AAshell%E7%AA%97%E5%8F%A3%E5%BC%80%E5%90%AF%E5%90%8E%EF%BC%8C%E4%BC%9A%E8%87%AA%E5%8A%A8%E5%88%87%E6%8D%A2%E5%88%B0root%0A%0A%E4%B9%9F%E5%8F%AF%E4%BB%A5%E6%8D%A2%E7%A7%8D%E6%96%B9%E5%BC%8F%E4%BF%AE%E6%94%B9%0A%0A%E6%9F%A5%E7%9C%8B%E5%BD%93%E5%89%8D%E4%B8%BB%E9%A2%98%E5%90%8D%20%20%0Aecho%20%24ZSH\_THEME%0A%0Avim%20%2Fvar%2Froot%2F.oh\-my\-zsh%2Fthemes%2Fagnoster.zsh\-theme%20%20%0A%E6%90%9C%E7%B4%A2%20DEFAULT\_USER%20%20%2C%E6%B3%A8%E9%87%8A%E6%8E%89%EF%BC%8Cprompt\_segment%20%E8%BF%99%E4%B8%80%E8%A1%8C%0A%0A%E6%95%B4%E8%A1%8C%E6%B3%A8%E9%87%8A%EF%BC%8C%E6%8F%90%E7%A4%BA%E7%AC%A6%E5%B0%B1%E4%B8%BA%20%E7%A9%BA%E4%BA%86%0A%0A%E4%BB%A5%E4%B8%8A%E7%9A%84%E4%BF%AE%E6%94%B9%EF%BC%8C%E9%83%BD%E6%AF%94%E8%BE%83%E6%9E%81%E7%AB%AF%E4%B8%94%E6%97%A0%E8%84%91%EF%BC%8C%E4%B8%8B%E9%9D%A2%EF%BC%8C%E8%AF%95%E4%B8%8B%E8%AF%A6%E7%BB%86%E7%9A%84%E4%B8%AA%E6%80%A7%0A%0A%0Avim%20%2Fvar%2Froot%2F.oh\-my\-zsh%2Fthemes%2Fagnoster.zsh\-theme%20%20%0A%E5%88%B0%E6%96%87%E4%BB%B6%E6%9C%80%E5%90%8E%EF%BC%8C%E4%BC%9A%E6%9C%89%E5%A6%82%E4%B8%8B%E7%BB%93%E6%9E%84%E4%BD%93%EF%BC%9A%20%20%0A%60%60%60%0Abuild\_prompt\(\)%20%7B%0A%20%20RETVAL%3D%24%3F%0A%20%20prompt\_status%0A%20%20prompt\_virtualenv%0A%20%20prompt\_aws%0A%20%20prompt\_context%0A%20%20prompt\_dir%0A%20%20prompt\_git%0A%20%20prompt\_bzr%0A%20%20prompt\_hg%0A%20%20prompt\_end%0A%7D%0A%60%60%60%0A%0Aprompt\_status%3A%E8%BF%99%E4%B8%80%E8%A1%8C%EF%BC%8C%E5%B0%B1%E6%98%AF%E6%8F%90%E7%A4%BA%E7%AC%A6%E6%9C%80%E5%89%8D%E9%9D%A2%E7%9A%84icon%E7%AC%A6%E5%8F%B7%20%20%0Aprompt\_context%3A%E8%BF%99%E4%B8%80%E8%A1%8C%EF%BC%8C%E5%B0%B1%E4%BC%9A%E6%8A%8A%E6%95%B4%E4%B8%AAusername%40host%20%E5%88%A0%E9%99%A4%E6%8E%89%2C%E4%B9%9F%E5%B0%B1%E6%98%AF%E7%AC%AC2%E7%A7%8D%E6%96%B9%E6%B3%95%E8%A6%81%E6%B3%A8%E9%87%8A%E6%8E%89%E7%9A%84%E9%82%A3%E4%B8%80%E8%A1%8C%20%20%0A%0A%E5%8F%AA%E7%95%99%E4%B8%80%E4%B8%AA%20%25n%20%3A%E5%B0%B1%E6%98%AF%E7%94%A8%E6%88%B7%E5%90%8D%0Aompt\_segment%20black%20default%20%22%25n%22%0A%0A%E6%8C%87%E4%BB%A4%E8%A1%8C%E6%9C%80%E5%89%8D%E9%9D%A2%E7%9A%84%E5%B0%8Ficon%20%E9%97%AA%E7%94%B5%E7%AC%A6%E5%8F%B7%EF%BC%8C%E6%9F%A5%E4%BA%86%E5%AE%83%E7%9A%84%E6%96%87%E6%A1%A3%EF%BC%8C%E7%9B%AE%E5%89%8D%E7%9F%A5%E9%81%93%E7%9A%84%E6%9C%893%E7%A7%8D%E7%AC%A6%E5%8F%B7%0A%3E1%E3%80%81%E9%97%AA%E7%94%B5%EF%BC%8C%E6%AF%8F%E6%9D%A1%E6%8C%87%E4%BB%A4%E5%89%8D%E9%83%BD%E6%9C%89%EF%BC%8C%E4%B8%8D%E7%9F%A5%E9%81%93%E5%B9%B2%E5%95%A5%E7%9A%84%20%20%0A%3E2%E3%80%81%E5%88%A0%E9%99%A4%EF%BC%8C%E8%BF%99%E4%B8%AA%E6%98%AFctrl%2Bc%20%E4%BC%9A%E6%98%BE%E7%A4%BA%EF%BC%8C%E6%89%A7%E8%A1%8C%E9%94%99%E8%AF%AF%E6%8C%87%E4%BB%A4%E4%B9%9F%E4%BC%9A%E6%98%BE%E7%A4%BA%20%20%0A%3E3%E3%80%81%E5%B0%8F%E5%9C%86%E5%9C%88%EF%BC%8C%E4%BB%A3%E7%90%86%E5%90%8E%E5%8F%B0%E6%9C%89%E5%90%AF%E5%8A%A8%E8%BF%9B%E7%A8%8B%EF%BC%8C%E6%AF%94%E5%A6%82%EF%BC%9Amysql%EF%BC%8C%E4%BD%86%E6%98%AF%E5%88%87%E6%8D%A2%E4%B8%AA%E7%AA%97%E5%8F%A3%E5%90%8E%EF%BC%8C%E8%BF%99%E4%B8%AA%E5%B0%8F%E5%9C%88%E5%B0%B1%E6%B2%A1%E6%9C%89%E4%BA%86%0A%0A%E6%89%80%E4%BB%A5%EF%BC%8C%E7%BB%BC%E5%90%88%E4%B8%8A%E8%BF%B0%E6%89%80%E7%9C%8B%EF%BC%8C%E8%BF%99%E4%B8%AAicon%20%E7%9A%84%E5%8A%9F%E8%83%BD%E6%B2%A1%E5%95%A5%E7%94%A8%EF%BC%8C%E6%B3%A8%E9%87%8A%E4%BA%86%0A%0A%E5%B0%8F%E9%BA%BB%E7%83%A6%0A\-\-\-\-%0A%E9%83%BD%E5%BC%84%E5%AE%8C%EF%BC%8C%E9%87%8D%E6%96%B0%E7%99%BB%E9%99%86%EF%BC%8C%E5%BC%80%E4%B8%80%E4%B8%AA%E6%96%B0shell%EF%BC%8C%E5%8F%91%E7%8E%B0%E6%8A%A5%E9%94%99%EF%BC%8C%E9%81%87%E5%88%B0%E7%82%B9%E5%B0%8F%E9%BA%BB%E7%83%A6%EF%BC%8C%E6%89%A7%E8%A1%8C%E5%A6%82%E4%B8%8B%EF%BC%9A%0A%0A%3Echmod%20755%20%2Fusr%2Flocal%2Fshare%2Fzsh%0A%3Echmod%20755%20%2Fusr%2Flocal%2Fshare%2Fzsh%2Fsite\-functions%0A%0A%E6%88%96%20%0A%60%60%60%0Achown%20\-R%20%24\(whoami\)%20%2Fusr%2Flocal%2Fshare%2Fzsh%0A%60%60%60%0A%E6%88%96%E6%B7%BB%E5%8A%A0%20vim%20~%2F.zshrc%0A%3EZSH\_DISABLE\_COMPFIX%3D%22true%22%0A%0Ashell%E6%8C%87%E4%BB%A4%E8%A1%8C~%E9%AB%98%E4%BA%AE%0A\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-%0A%E8%BE%93%E5%85%A5%E6%8C%87%E4%BB%A4%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E6%9C%89%E6%97%B6%E5%80%99%E4%BC%9A%E5%87%BA%E7%8E%B0%E9%94%99%E7%9A%84%E6%83%85%E5%86%B5%20%20%0A%0A%60%60%60%0Acd%20~%2F.oh\-my\-zsh%2Fcustom%2Fplugins%20%20%0Agit%20clone%20https%3A%2F%2Fgithub.com%2Fzsh\-users%2Fzsh\-syntax\-highlighting.git%0A%0Avim%20~%2F.zshrc%20%0A%0Aplugins%3D\(git%20zsh\-syntax\-highlighting\)%0A%60%60%60%0A%0A%0A%20%0A%0A%0Agit\-%E5%BF%AB%E6%8D%B7%E6%93%8D%E4%BD%9C%0A\-\-\-\-\-%0A%0A%0A%7C%20%20Alias%20%20%20%7C%20Command%20%20%7C%0A%7C%20%20\-\-\-\-%20%20%7C%20\-\-\-\-%20%20%7C%0A%7C%20gaa%20%20%7C%20git%20add%20\-\-all%20%7C%0A%7C%20gba%20%20%7C%20git%20branch%20\-a%20%7C%0A%7C%20%20gcam%20%20%20%7C%20git%20commit%20\-a%20\-m%20%20%7C%0A%7C%20%20gd%20%20%20%7C%20git%20diff%20%20%7C%0A%7C%20%20gl%20%20%20%7C%20git%20pull%20%20%7C%0A%7C%20%20gp%20%20%20%7C%20git%20push%20%20%7C%0A%7C%20%20gr%20%20%20%7C%20git%20remote%20%20%7C%0A%7C%20%20gt%20%20%20%7C%20git%20status%20%20%7C%0A%7C%20%20gb%20%20%20%7C%20git%20branch%20%20%7C%0A%7C%20%20Alias%20%20%20%7C%20Command%20%20%7C%0A%0A%0A%0Azsh\-autosuggestions%20%0A%0Aautojump
