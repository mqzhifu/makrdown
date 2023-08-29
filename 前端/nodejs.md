# nodejs

## nodejs/npm 安装

官网

> https://nodejs.org/en

有：win mac linux 版本

现在是 20 版本，稳定版好像是18

安装都挺简单

安装完成后，记得看下是否加到 环境变量中

查看 node 版本：

> node \-v

```
v18.16.1
```

node 安全成功 后，会自带 npm

> npm \-v

```
9.5.1
```

查看 npm 配置信息

> npm config ls

更改 安装目录

> npm config set prefix "D:\\nodejs\\node\_global"

查看已安装的包：

> npm install webpack \-\-save\-dev

## 创建一个新项目

cd /data/nodejs

mkdir projceName

npm init

然后就是一顿回车即可

会在根目录多一个 package.json

vim package.json

```
{
    "name": "zjsframword",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "type": "module",
    "scripts": {
    "   test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "",
    "license": "ISC",
        "dependencies": {
        "crypto-js": "^4.1.1",
        "protobufjs": "^7.2.4",
        "ws": "^8.13.0",
        "yamljs": "^0.3.0"
    }
}
```

分析：

1. 大概就是项目的基础
2. 依赖哪些库
3. scripts 带些脚本，如：npm run build / npm run serve 等

## webpack

npm i @types/node \-\-save\-dev

npm install stream\-http

npm install path\-browserify

> 上面这3个包，因为 webpack 打包时报错

npm install \-g webpack

npm install \-g webpack\-cli

看下版本信息

> webpack \-v

创建配置文件

> touch webpack.config.js

编辑配置文件

> vim webpack.config.js

创建目录/文件

```
mkdir src
cd src
touch index.js
```

简单测试一下：

> webpack.cmd ./src/index.js \-\-output\-path ./dist \-\-mode=development

nodejs%2Fnpm%20%E5%AE%89%E8%A3%85%0A\-\-\-\-%0A%E5%AE%98%E7%BD%91%0A%3Ehttps%3A%2F%2Fnodejs.org%2Fen%0A%0A%E6%9C%89%EF%BC%9Awin%20mac%20linux%20%E7%89%88%E6%9C%AC%0A%0A%E7%8E%B0%E5%9C%A8%E6%98%AF%2020%20%E7%89%88%E6%9C%AC%EF%BC%8C%E7%A8%B3%E5%AE%9A%E7%89%88%E5%A5%BD%E5%83%8F%E6%98%AF18%0A%0A%E5%AE%89%E8%A3%85%E9%83%BD%E6%8C%BA%E7%AE%80%E5%8D%95%0A%0A%E5%AE%89%E8%A3%85%E5%AE%8C%E6%88%90%E5%90%8E%EF%BC%8C%E8%AE%B0%E5%BE%97%E7%9C%8B%E4%B8%8B%E6%98%AF%E5%90%A6%E5%8A%A0%E5%88%B0%20%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E4%B8%AD%0A%0A%E6%9F%A5%E7%9C%8B%20node%20%E7%89%88%E6%9C%AC%EF%BC%9A%0A%3Enode%20\-v%0A%60%60%60%0Av18.16.1%0A%60%60%60%0Anode%20%E5%AE%89%E5%85%A8%E6%88%90%E5%8A%9F%20%E5%90%8E%EF%BC%8C%E4%BC%9A%E8%87%AA%E5%B8%A6%20npm%0A%3Enpm%20\-v%0A%60%60%60%0A9.5.1%0A%60%60%60%0A%E6%9F%A5%E7%9C%8B%20npm%20%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%0A%3Enpm%20config%20ls%0A%0A%E6%9B%B4%E6%94%B9%20%E5%AE%89%E8%A3%85%E7%9B%AE%E5%BD%95%0A%3Enpm%20config%20set%20prefix%20%22D%3A%5Cnodejs%5Cnode\_global%22%0A%0A%E6%9F%A5%E7%9C%8B%E5%B7%B2%E5%AE%89%E8%A3%85%E7%9A%84%E5%8C%85%EF%BC%9A%0A%3Enpm%20install%20webpack%20\-\-save\-dev%0A%0A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E6%96%B0%E9%A1%B9%E7%9B%AE%0A\-\-\-%0Acd%20%2Fdata%2Fnodejs%0Amkdir%20projceName%0Anpm%20init%20%0A%0A%E7%84%B6%E5%90%8E%E5%B0%B1%E6%98%AF%E4%B8%80%E9%A1%BF%E5%9B%9E%E8%BD%A6%E5%8D%B3%E5%8F%AF%0A%0A%E4%BC%9A%E5%9C%A8%E6%A0%B9%E7%9B%AE%E5%BD%95%E5%A4%9A%E4%B8%80%E4%B8%AA%20package.json%0Avim%20package.json%0A%60%60%60%0A%7B%0A%20%20%20%20%22name%22%3A%20%22zjsframword%22%2C%0A%20%20%20%20%22version%22%3A%20%221.0.0%22%2C%0A%20%20%20%20%22description%22%3A%20%22%22%2C%0A%20%20%20%20%22main%22%3A%20%22index.js%22%2C%0A%20%20%20%20%22type%22%3A%20%22module%22%2C%0A%20%20%20%20%22scripts%22%3A%20%7B%0A%20%20%20%20%22%20%20%20test%22%3A%20%22echo%20%5C%22Error%3A%20no%20test%20specified%5C%22%20%26%26%20exit%201%22%0A%20%20%20%20%7D%2C%0A%20%20%20%20%22author%22%3A%20%22%22%2C%0A%20%20%20%20%22license%22%3A%20%22ISC%22%2C%0A%20%20%20%20%20%20%20%20%22dependencies%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%22crypto\-js%22%3A%20%22%5E4.1.1%22%2C%0A%20%20%20%20%20%20%20%20%22protobufjs%22%3A%20%22%5E7.2.4%22%2C%0A%20%20%20%20%20%20%20%20%22ws%22%3A%20%22%5E8.13.0%22%2C%0A%20%20%20%20%20%20%20%20%22yamljs%22%3A%20%22%5E0.3.0%22%0A%20%20%20%20%7D%0A%7D%0A%60%60%60%0A%E5%88%86%E6%9E%90%EF%BC%9A%0A1.%20%E5%A4%A7%E6%A6%82%E5%B0%B1%E6%98%AF%E9%A1%B9%E7%9B%AE%E7%9A%84%E5%9F%BA%E7%A1%80%0A2.%20%E4%BE%9D%E8%B5%96%E5%93%AA%E4%BA%9B%E5%BA%93%0A3.%20scripts%20%E5%B8%A6%E4%BA%9B%E8%84%9A%E6%9C%AC%EF%BC%8C%E5%A6%82%EF%BC%9Anpm%20run%20build%20%2F%20npm%20run%20serve%20%E7%AD%89%0A%0Awebpack%0A\-\-\-\-%0Anpm%20i%20%40types%2Fnode%20\-\-save\-dev%0Anpm%20install%20stream\-http%0Anpm%20install%20path\-browserify%0A%3E%E4%B8%8A%E9%9D%A2%E8%BF%993%E4%B8%AA%E5%8C%85%EF%BC%8C%E5%9B%A0%E4%B8%BA%20webpack%20%E6%89%93%E5%8C%85%E6%97%B6%E6%8A%A5%E9%94%99%0A%0Anpm%20install%20\-g%20webpack%0Anpm%20install%20\-g%20webpack\-cli%0A%0A%E7%9C%8B%E4%B8%8B%E7%89%88%E6%9C%AC%E4%BF%A1%E6%81%AF%0A%3Ewebpack%20\-v%0A%0A%E5%88%9B%E5%BB%BA%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%0A%3Etouch%20webpack.config.js%0A%0A%E7%BC%96%E8%BE%91%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%0A%3Evim%20webpack.config.js%0A%0A%0A%E5%88%9B%E5%BB%BA%E7%9B%AE%E5%BD%95%2F%E6%96%87%E4%BB%B6%0A%60%60%60%0Amkdir%20src%0Acd%20src%0Atouch%20index.js%0A%60%60%60%0A%0A%E7%AE%80%E5%8D%95%E6%B5%8B%E8%AF%95%E4%B8%80%E4%B8%8B%EF%BC%9A%0A%3Ewebpack.cmd%20%20.%2Fsrc%2Findex.js%20\-\-output\-path%20.%2Fdist%20\-\-mode%3Ddevelopment
