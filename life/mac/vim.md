# vim

## vim

vim的配置，真是可以用\<哲学\>来形容了，因为是最早的文本IDE，牵扯的配件实在是太TMD多了，如下：

1. nerdtree 目录结构树，分栏显示
2. php语法高亮
3. php自动补全
4. bundle 管理插件的工具
5. 函数点击跳转/函数结尾跳转到函数顶部
6. 自动补全小括号、大括号等
7. 文件中的函数列表
8. 代码/函数 折叠
9. 底部\-状态栏
10. ....\(太多了\)

反正吧，挺有意思，比如：高亮是高亮，函数提示是函数提示，全要单独配置。可能是被一些高级IDE惯坏了，或者不是早期程序员，也不写C\+\+，对于纯手工配置真的是挺头疼的。

nerdtree

> wget http://www.vim.org/scripts/download\_script.php?src\_id=17123 \-O nerdtree.zip

```
mkdir nerdtree
mv nerdtree.zip nerdtree
cd nerdtree
unzip nerdtree.zip
mkdir -p ~/.vim/{plugin,doc}
cp plugin/NERD_tree.vim ~/.vim/plugin/
cp doc/NERD_tree.txt ~/.vim/doc/
```

vim中输入:NERDTree

若想打开vim自动进入NERDTree,则在~/.vimrc

touch ~/.vimrc

文件中添加下面内容

```
autocmd VimEnter * NERDTree  
wincmd w
autocmd VimEnter * wincmd w
let NERDTreeWinPos=1
autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTreeType") &&b:NERDTreeType == "primary") | q | endif

map <F2> :NERDTreeMirror<CR>
map <F2> :NERDTreeToggle<CR>
```

## vim 状态栏

powerline\-status

查看py是否已安装

> python \-\-version

安装一个py的插件pip

> easy\_install pip

安装 powerline

> pip install powerline\-status

查看安装目录

> pip show powerline\-status

vimrc 中添加

```
set rtp+=/Library/Python/2.7/site-packages/powerline/bindings/vim
set laststatus=2
set t_Co=256
```

## PHP高亮插件

下载 php.tar 包，解压

```
syntax/php.vim
mv syntax ~/.vim/
vim ~./.vimrc
syntax  on
```

autocmd FileType php set omnifunc=phpcomplete\#CompletePHP

```
set autoindent  #自动缩进，回车后继承上一行的缩进
set ts=4        #设置tab 的长度为4
set expandtab   #设置tab 为空格

#以下是 自动补齐
inoremap ' ''<ESC>i
inoremap " ""<ESC>i
inoremap ( ()<ESC>i
inoremap [ []<ESC>i
inoremap { {<CR>}<ESC>O

set ignorecase  #搜索时忽略大小写

```

## 用户/组

useradd groupadd 都没有

\#查看所有组列表

> dscl . \-list /Groups

\#查看所有组的ID

> dscl . \-list /Groups PrimaryGroupID

\#创建一个组rsync,但不指定ID\( 参数PrimaryGroupID将查不到 \)

> dscl . \-create /groups/rsync

\#创建一个组rsync，并指定ID为 888

> dscl . \-create /Groups/rsync gid 888

\#删除一个组rsync

> dscl . \-delete /Groups/rsync

将某用户从某组中删除

> dscl . \-delete /Groups/某组 GroupMembership 用户名

\#查看所有用户

> dscl . \-list /Users

\#创建一个用户rsync

> dscl . \-create /Users/rsync

\#将某用户添加到某组中

> dscl . \-append /Groups/rsync GroupMembership rsync

vim%0A\-\-\-\-\-%0A%0Avim%E7%9A%84%E9%85%8D%E7%BD%AE%EF%BC%8C%E7%9C%9F%E6%98%AF%E5%8F%AF%E4%BB%A5%E7%94%A8%3C%E5%93%B2%E5%AD%A6%3E%E6%9D%A5%E5%BD%A2%E5%AE%B9%E4%BA%86%EF%BC%8C%E5%9B%A0%E4%B8%BA%E6%98%AF%E6%9C%80%E6%97%A9%E7%9A%84%E6%96%87%E6%9C%ACIDE%EF%BC%8C%E7%89%B5%E6%89%AF%E7%9A%84%E9%85%8D%E4%BB%B6%E5%AE%9E%E5%9C%A8%E6%98%AF%E5%A4%AATMD%E5%A4%9A%E4%BA%86%EF%BC%8C%E5%A6%82%E4%B8%8B%EF%BC%9A%0A%0A1.%20%20nerdtree%20%20%20%20%20%20%20%20%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84%E6%A0%91%EF%BC%8C%E5%88%86%E6%A0%8F%E6%98%BE%E7%A4%BA%0A2.%20%20php%E8%AF%AD%E6%B3%95%E9%AB%98%E4%BA%AE%0A3.%20%20php%E8%87%AA%E5%8A%A8%E8%A1%A5%E5%85%A8%0A3.%20%20bundle%20%20%20%20%20%20%20%20%20%20%E7%AE%A1%E7%90%86%E6%8F%92%E4%BB%B6%E7%9A%84%E5%B7%A5%E5%85%B7%0A4.%20%20%E5%87%BD%E6%95%B0%E7%82%B9%E5%87%BB%E8%B7%B3%E8%BD%AC%2F%E5%87%BD%E6%95%B0%E7%BB%93%E5%B0%BE%E8%B7%B3%E8%BD%AC%E5%88%B0%E5%87%BD%E6%95%B0%E9%A1%B6%E9%83%A8%0A5.%20%20%E8%87%AA%E5%8A%A8%E8%A1%A5%E5%85%A8%E5%B0%8F%E6%8B%AC%E5%8F%B7%E3%80%81%E5%A4%A7%E6%8B%AC%E5%8F%B7%E7%AD%89%0A6.%20%20%E6%96%87%E4%BB%B6%E4%B8%AD%E7%9A%84%E5%87%BD%E6%95%B0%E5%88%97%E8%A1%A8%0A7.%20%20%E4%BB%A3%E7%A0%81%2F%E5%87%BD%E6%95%B0%20%20%E6%8A%98%E5%8F%A0%0A8.%20%20%E5%BA%95%E9%83%A8\-%E7%8A%B6%E6%80%81%E6%A0%8F%20%20%20%20%20%20%0A7.%20%20....\(%E5%A4%AA%E5%A4%9A%E4%BA%86\)%0A%0A%0A%E5%8F%8D%E6%AD%A3%E5%90%A7%EF%BC%8C%E6%8C%BA%E6%9C%89%E6%84%8F%E6%80%9D%EF%BC%8C%E6%AF%94%E5%A6%82%EF%BC%9A%E9%AB%98%E4%BA%AE%E6%98%AF%E9%AB%98%E4%BA%AE%EF%BC%8C%E5%87%BD%E6%95%B0%E6%8F%90%E7%A4%BA%E6%98%AF%E5%87%BD%E6%95%B0%E6%8F%90%E7%A4%BA%EF%BC%8C%E5%85%A8%E8%A6%81%E5%8D%95%E7%8B%AC%E9%85%8D%E7%BD%AE%E3%80%82%E5%8F%AF%E8%83%BD%E6%98%AF%E8%A2%AB%E4%B8%80%E4%BA%9B%E9%AB%98%E7%BA%A7%20IDE%20%E6%83%AF%E5%9D%8F%E4%BA%86%EF%BC%8C%E6%88%96%E8%80%85%E4%B8%8D%E6%98%AF%E6%97%A9%E6%9C%9F%E7%A8%8B%E5%BA%8F%E5%91%98%EF%BC%8C%E4%B9%9F%E4%B8%8D%E5%86%99C%2B%2B%EF%BC%8C%E5%AF%B9%E4%BA%8E%E7%BA%AF%E6%89%8B%E5%B7%A5%E9%85%8D%E7%BD%AE%E7%9C%9F%E7%9A%84%E6%98%AF%E6%8C%BA%E5%A4%B4%E7%96%BC%E7%9A%84%E3%80%82%0A%0Anerdtree%20%20%0A%3Ewget%20http%3A%2F%2Fwww.vim.org%2Fscripts%2Fdownload\_script.php%3Fsrc\_id%3D17123%20\-O%20nerdtree.zip%0A%0A%60%60%60%0Amkdir%20nerdtree%0Amv%20nerdtree.zip%20nerdtree%0Acd%20nerdtree%0Aunzip%20nerdtree.zip%0Amkdir%20\-p%20~%2F.vim%2F%7Bplugin%2Cdoc%7D%0Acp%20plugin%2FNERD\_tree.vim%20~%2F.vim%2Fplugin%2F%0Acp%20doc%2FNERD\_tree.txt%20~%2F.vim%2Fdoc%2F%0A%60%60%60%0Avim%E4%B8%AD%E8%BE%93%E5%85%A5%3ANERDTree%20%20%0A%0A%E8%8B%A5%E6%83%B3%E6%89%93%E5%BC%80vim%E8%87%AA%E5%8A%A8%E8%BF%9B%E5%85%A5NERDTree%2C%E5%88%99%E5%9C%A8~%2F.vimrc%20%20%0Atouch%20~%2F.vimrc%20%20%0A%E6%96%87%E4%BB%B6%E4%B8%AD%E6%B7%BB%E5%8A%A0%E4%B8%8B%E9%9D%A2%E5%86%85%E5%AE%B9%20%0A%60%60%60%0Aautocmd%20VimEnter%20\*%20NERDTree%20%20%0Awincmd%20w%0Aautocmd%20VimEnter%20\*%20wincmd%20w%0Alet%20NERDTreeWinPos%3D1%0Aautocmd%20bufenter%20\*%20if%20\(winnr\(%22%24%22\)%20%3D%3D%201%20%26%26%20exists\(%22b%3ANERDTreeType%22\)%20%26%26b%3ANERDTreeType%20%3D%3D%20%22primary%22\)%20%7C%20q%20%7C%20endif%0A%0Amap%20%3CF2%3E%20%3ANERDTreeMirror%3CCR%3E%0Amap%20%3CF2%3E%20%3ANERDTreeToggle%3CCR%3E%0A%60%60%60%0A%0A%0Avim%20%E7%8A%B6%E6%80%81%E6%A0%8F%0A\-\-\-\-\-\-\-\-%0A%0Apowerline\-status%0A%0A%E6%9F%A5%E7%9C%8Bpy%E6%98%AF%E5%90%A6%E5%B7%B2%E5%AE%89%E8%A3%85%0A%3Epython%20\-\-version%20%0A%0A%E5%AE%89%E8%A3%85%E4%B8%80%E4%B8%AApy%E7%9A%84%E6%8F%92%E4%BB%B6pip%0A%3Eeasy\_install%20pip%20%20%0A%0A%E5%AE%89%E8%A3%85%20powerline%0A%3Epip%20install%20powerline\-status%20%20%0A%0A%E6%9F%A5%E7%9C%8B%E5%AE%89%E8%A3%85%E7%9B%AE%E5%BD%95%0A%3Epip%20show%20powerline\-status%20%20%0A%0Avimrc%20%E4%B8%AD%E6%B7%BB%E5%8A%A0%0A%60%60%60%0Aset%20rtp%2B%3D%2FLibrary%2FPython%2F2.7%2Fsite\-packages%2Fpowerline%2Fbindings%2Fvim%0Aset%20laststatus%3D2%0Aset%20t\_Co%3D256%0A%60%60%60%0A%0APHP%E9%AB%98%E4%BA%AE%E6%8F%92%E4%BB%B6%20%0A\-\-\-\-\-%0A%E4%B8%8B%E8%BD%BD%20%20php.tar%20%E5%8C%85%EF%BC%8C%E8%A7%A3%E5%8E%8B%0A%60%60%60%0Asyntax%2Fphp.vim%0Amv%20syntax%20~%2F.vim%2F%0Avim%20~.%2F.vimrc%0Asyntax%20%20on%0A%60%60%60%0Aautocmd%20FileType%20php%20set%20omnifunc%3Dphpcomplete%23CompletePHP%0A%0A%60%60%60%0Aset%20autoindent%20%20%23%E8%87%AA%E5%8A%A8%E7%BC%A9%E8%BF%9B%EF%BC%8C%E5%9B%9E%E8%BD%A6%E5%90%8E%E7%BB%A7%E6%89%BF%E4%B8%8A%E4%B8%80%E8%A1%8C%E7%9A%84%E7%BC%A9%E8%BF%9B%0Aset%20ts%3D4%20%20%20%20%20%20%20%20%23%E8%AE%BE%E7%BD%AEtab%20%E7%9A%84%E9%95%BF%E5%BA%A6%E4%B8%BA4%0Aset%20expandtab%20%20%20%23%E8%AE%BE%E7%BD%AEtab%20%E4%B8%BA%E7%A9%BA%E6%A0%BC%0A%0A%23%E4%BB%A5%E4%B8%8B%E6%98%AF%20%E8%87%AA%E5%8A%A8%E8%A1%A5%E9%BD%90%0Ainoremap%20'%20''%3CESC%3Ei%0Ainoremap%20%22%20%22%22%3CESC%3Ei%0Ainoremap%20\(%20\(\)%3CESC%3Ei%0Ainoremap%20%5B%20%5B%5D%3CESC%3Ei%0Ainoremap%20%7B%20%7B%3CCR%3E%7D%3CESC%3EO%0A%0A%0Aset%20ignorecase%20%20%23%E6%90%9C%E7%B4%A2%E6%97%B6%E5%BF%BD%E7%95%A5%E5%A4%A7%E5%B0%8F%E5%86%99%0A%0A%0A%60%60%60%0A%0A%E7%94%A8%E6%88%B7%2F%E7%BB%84%0A\-\-\-\-\-%0Auseradd%20groupadd%20%E9%83%BD%E6%B2%A1%E6%9C%89%20%20%20%0A%23%E6%9F%A5%E7%9C%8B%E6%89%80%E6%9C%89%E7%BB%84%E5%88%97%E8%A1%A8%20%20%0A%3Edscl%20.%20\-list%20%2FGroups%20%20%0A%0A%23%E6%9F%A5%E7%9C%8B%E6%89%80%E6%9C%89%E7%BB%84%E7%9A%84ID%20%20%0A%3Edscl%20.%20\-list%20%2FGroups%20PrimaryGroupID%20%20%0A%0A%23%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E7%BB%84rsync%2C%E4%BD%86%E4%B8%8D%E6%8C%87%E5%AE%9AID\(%20%E5%8F%82%E6%95%B0PrimaryGroupID%E5%B0%86%E6%9F%A5%E4%B8%8D%E5%88%B0%20\)%20%20%0A%0A%3Edscl%20.%20\-create%20%2Fgroups%2Frsync%20%20%0A%0A%23%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E7%BB%84rsync%EF%BC%8C%E5%B9%B6%E6%8C%87%E5%AE%9AID%E4%B8%BA%20888%20%20%0A%0A%3Edscl%20.%20\-create%20%2FGroups%2Frsync%20gid%20888%20%20%0A%0A%23%E5%88%A0%E9%99%A4%E4%B8%80%E4%B8%AA%E7%BB%84rsync%20%20%0A%0A%3Edscl%20.%20\-delete%20%2FGroups%2Frsync%20%20%0A%0A%E5%B0%86%E6%9F%90%E7%94%A8%E6%88%B7%E4%BB%8E%E6%9F%90%E7%BB%84%E4%B8%AD%E5%88%A0%E9%99%A4%20%20%0A%0A%3Edscl%20.%20\-delete%20%2FGroups%2F%E6%9F%90%E7%BB%84%20GroupMembership%20%E7%94%A8%E6%88%B7%E5%90%8D%20%20%0A%0A%23%E6%9F%A5%E7%9C%8B%E6%89%80%E6%9C%89%E7%94%A8%E6%88%B7%0A%0A%3Edscl%20.%20\-list%20%2FUsers%0A%0A%23%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E7%94%A8%E6%88%B7rsync%20%20%0A%0A%3Edscl%20.%20\-create%20%2FUsers%2Frsync%20%20%0A%0A%23%E5%B0%86%E6%9F%90%E7%94%A8%E6%88%B7%E6%B7%BB%E5%8A%A0%E5%88%B0%E6%9F%90%E7%BB%84%E4%B8%AD%0A%0A%3Edscl%20.%20\-append%20%2FGroups%2Frsync%20GroupMembership%20rsync%0A%0A%0A
