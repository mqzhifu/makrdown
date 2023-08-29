# SRS

官网

> https://ossrs.net

## 安装

下载代码

> git clone \-b 4.0release https://gitee.com/ossrs/srs.git

编译

```
cd srs/trunk
./configure
make
```

启动：

> ./objs/srs \-c conf/srs.conf

检查：

> http://localhost:8080/

查看SRS的状态

> ./etc/init.d/srs status

者看SRS的日志

> tail \-n 30 \-f ./objs/srs.log

停止

> ./etc/init.d/srs stop

大概看一下，主配置文件中端口号，vim conf/srs.conf:

|端口号|协议      |描述                  |备注                                                                    |
|------|----------|----------------------|------------------------------------------------------------------------|
|1935  |RMTP      |推流                  |                                                                        |
|8080  |HTTP      |一个可视化网页管理后台|                                                                        |
|1985  |HTTP\-api |没搞懂                |没太理解为毛又分拆了一个端口，完全可以共用8080。我猜可能是后端分拆了模块|
|8088  |https     |同8080一样            |webRtc浏览器测试必须得是HTTPS                                           |
|1990  |HTTPS\-api|同1985一样            |webRtc浏览器测试必须得是HTTPS                                           |

测试一下用 ffmpeg 本地推流：

## 测试直播

前置条件：

1. 准备一个视频文件，如果是 FLV 比较好，MP4的话得转一下 FLV 格式

> /soft/ffmpeg/bin/ffmpeg \-i test.mp4 \-c copy test.flv

1. ffmpeg

> 服务器先安装 ffmpeg，centos/mac 下一键搞定，ubutu 很麻烦，源码编译安装....

#### 开始推流

> /soft/ffmpeg/bin/ffmpeg \-re \-i /install/test.mp4 \-c copy \-f flv rtmp://localhost/live/livestream

#### 拉流

1. H5\(HLS\)，这个最简单浏览器直接打开地址即可\(浏览器可能还得安装插件\)
    > http://8.142.177.235:8080/live/livestream.m3u8
2. H5\(HTTP\-FLV\)，这个略麻烦点分成两种情况吧：客户端和浏览器
    1. ffmpeg ，这个直接可以用
    > ffplay \-i "http://8.142.177.235:8080/live/livestream.flv" \-fflags nobuffer
    1. 写点JS代码，用flv.js库，最后填写服务器 URL 地址即可：
    > https://github.com/bilibili/flv.js
    > 
    > 
    > http://8.142.177.235:8080/live/livestream.flv
    > 
    > 
    > 早期是浏览器安装 flash\_player 插件，可以直接播放。之后flash 被放弃，浏览器还有些其它插件播放，后期基本上这些3方插件也被谢谢了，只有用JS库。
3. RTMP: rtmp://8.142.177.235//live/livestream

> VLC是个客户端软件\(播放器\)，去官网下一个，然后打开，里面配置一个这个URL地址即可。

## webrtc

这个略恶心人，浏览器 强制要求必须得HTTPS，所以它就得单搞：

修改配置文件：https.rtc.conf

```
candidate 8.142.177.235;
listen 8000;#UDP 

key /data/www/srs/9245472_webrtc-push.seedreality.com.key;
cert /data/www/srs/9245472_webrtc-push.seedreality.com.pem;

```

启动：\(这里换了个配置文件\)

> ./objs/srs \-c conf/https.rtc.conf

测试\-浏览器推流：

> https://webrtc\-push.seedreality.com:8088/players/rtc\_publisher.html?autostart=true&stream=livestream&api=1990&schema=https

测试\-浏览器播放：

?\>https://webrtc\-push.seedreality.com:8088/players/rtc\_player.html?autostart=true&stream=livestream&api=1990&schema=https

## ubutu安装ffmpeg

下载地址：

> https://ffmpeg.org/download.html
> 
> 
> https://launchpad.net/ubuntu/\+archive/primary/\+sourcefiles/ffmpeg/7:5.1.1\-1ubuntu1/ffmpeg\_5.1.1.orig.tar.xz

```
xz -dk ffmpeg_5.1.1.orig.tar.xz
tar xvf ffmpeg_5.1.1.orig.tar
cd ffmpeg-5.1.1/
```

```
安装ffplay需要的依赖
sudo apt-get install libx11-dev xorg-dev libsdl2-2.0 libsdl2-dev
sudo apt install yasm pkg-config libopencore-amrnb-dev libopencore-amrwb-dev
```

mkdir \-p /soft/ffmpeg

./configure \-\-prefix=/soft/ffmpeg \-\-enable\-gpl \-\-enable\-version3 \-\-enable\-nonfree \-\-enable\-ffplay \-\-enable\-ffprobe \-\-enable\-debug

下面3个找不到，先不管了

\-\-enable\-libfdk\-aac \-\-enable\-libx264 \-\-enable\-libx265

%E5%AE%98%E7%BD%91%0A%3Ehttps%3A%2F%2Fossrs.net%0A%20%0A%0A%E5%AE%89%E8%A3%85%0A\-\-\-\-\-%0A%E4%B8%8B%E8%BD%BD%E4%BB%A3%E7%A0%81%0A%3E%20git%20clone%20\-b%204.0release%20https%3A%2F%2Fgitee.com%2Fossrs%2Fsrs.git%0A%20%0A%E7%BC%96%E8%AF%91%20%0A%20%60%60%60%0A%20cd%20srs%2Ftrunk%0A.%2Fconfigure%0Amake%0A%20%60%60%60%0A%0A%E5%90%AF%E5%8A%A8%EF%BC%9A%0A%3E.%2Fobjs%2Fsrs%20\-c%20conf%2Fsrs.conf%0A%0A%E6%A3%80%E6%9F%A5%EF%BC%9A%0A%3E%20http%3A%2F%2Flocalhost%3A8080%2F%0A%0A%E6%9F%A5%E7%9C%8BSRS%E7%9A%84%E7%8A%B6%E6%80%81%0A%3E.%2Fetc%2Finit.d%2Fsrs%20status%0A%0A%E8%80%85%E7%9C%8BSRS%E7%9A%84%E6%97%A5%E5%BF%97%0A%3Etail%20\-n%2030%20\-f%20.%2Fobjs%2Fsrs.log%0A%0A%E5%81%9C%E6%AD%A2%0A%3E%20.%2Fetc%2Finit.d%2Fsrs%20stop%0A%0A%E5%A4%A7%E6%A6%82%E7%9C%8B%E4%B8%80%E4%B8%8B%EF%BC%8C%E4%B8%BB%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%B8%AD%E7%AB%AF%E5%8F%A3%E5%8F%B7%EF%BC%8Cvim%20conf%2Fsrs.conf%3A%0A%0A%0A%0A%7C%20%20%E7%AB%AF%E5%8F%A3%E5%8F%B7%7C%20%E5%8D%8F%E8%AE%AE%20%7C%E6%8F%8F%E8%BF%B0%20%20%7C%20%E5%A4%87%E6%B3%A8%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C1935%20%20%7CRMTP%20%20%7C%E6%8E%A8%E6%B5%81%20%20%7C%20%20%7C%0A%7C%208080%20%7C%20HTTP%20%7C%20%E4%B8%80%E4%B8%AA%E5%8F%AF%E8%A7%86%E5%8C%96%E7%BD%91%E9%A1%B5%E7%AE%A1%E7%90%86%E5%90%8E%E5%8F%B0%20%7C%20%20%7C%0A%7C%201985%20%7C%20HTTP\-api%20%7C%E6%B2%A1%E6%90%9E%E6%87%82%20%20%7C%20%E6%B2%A1%E5%A4%AA%E7%90%86%E8%A7%A3%E4%B8%BA%E6%AF%9B%E5%8F%88%E5%88%86%E6%8B%86%E4%BA%86%E4%B8%80%E4%B8%AA%E7%AB%AF%E5%8F%A3%EF%BC%8C%E5%AE%8C%E5%85%A8%E5%8F%AF%E4%BB%A5%E5%85%B1%E7%94%A88080%E3%80%82%E6%88%91%E7%8C%9C%E5%8F%AF%E8%83%BD%E6%98%AF%E5%90%8E%E7%AB%AF%E5%88%86%E6%8B%86%E4%BA%86%E6%A8%A1%E5%9D%97%20%7C%0A%7C%208088%20%7Chttps%20%20%7C%20%E5%90%8C8080%E4%B8%80%E6%A0%B7%20%7C%20webRtc%E6%B5%8F%E8%A7%88%E5%99%A8%E6%B5%8B%E8%AF%95%E5%BF%85%E9%A1%BB%E5%BE%97%E6%98%AFHTTPS%20%7C%0A%7C1990%20%20%7C%20HTTPS\-api%20%7C%20%20%E5%90%8C1985%E4%B8%80%E6%A0%B7%7C%20%20webRtc%E6%B5%8F%E8%A7%88%E5%99%A8%E6%B5%8B%E8%AF%95%E5%BF%85%E9%A1%BB%E5%BE%97%E6%98%AFHTTPS%7C%0A%0A%0A%E6%B5%8B%E8%AF%95%E4%B8%80%E4%B8%8B%E7%94%A8%20ffmpeg%20%E6%9C%AC%E5%9C%B0%E6%8E%A8%E6%B5%81%EF%BC%9A%0A%0A%0A%E6%B5%8B%E8%AF%95%E7%9B%B4%E6%92%AD%0A\-\-\-\-%0A%E5%89%8D%E7%BD%AE%E6%9D%A1%E4%BB%B6%EF%BC%9A%0A1.%20%E5%87%86%E5%A4%87%E4%B8%80%E4%B8%AA%E8%A7%86%E9%A2%91%E6%96%87%E4%BB%B6%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%98%AF%20FLV%20%E6%AF%94%E8%BE%83%E5%A5%BD%EF%BC%8CMP4%E7%9A%84%E8%AF%9D%E5%BE%97%E8%BD%AC%E4%B8%80%E4%B8%8B%20FLV%20%E6%A0%BC%E5%BC%8F%0A%3E%2Fsoft%2Fffmpeg%2Fbin%2Fffmpeg%20\-i%20test.mp4%20\-c%20copy%20test.flv%0A2.%20ffmpeg%0A%3E%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%85%88%E5%AE%89%E8%A3%85%20ffmpeg%EF%BC%8Ccentos%2Fmac%20%E4%B8%8B%E4%B8%80%E9%94%AE%E6%90%9E%E5%AE%9A%EF%BC%8Cubutu%20%E5%BE%88%E9%BA%BB%E7%83%A6%EF%BC%8C%E6%BA%90%E7%A0%81%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85....%0A%0A%23%23%23%23%20%E5%BC%80%E5%A7%8B%E6%8E%A8%E6%B5%81%0A%3E%2Fsoft%2Fffmpeg%2Fbin%2Fffmpeg%20\-re%20\-i%20%2Finstall%2Ftest.mp4%20%20%20\-c%20copy%20\-f%20flv%20rtmp%3A%2F%2Flocalhost%2Flive%2Flivestream%0A%0A%23%23%23%23%20%E6%8B%89%E6%B5%81%0A1.%20H5\(HLS\)%EF%BC%8C%E8%BF%99%E4%B8%AA%E6%9C%80%E7%AE%80%E5%8D%95%E6%B5%8F%E8%A7%88%E5%99%A8%E7%9B%B4%E6%8E%A5%E6%89%93%E5%BC%80%E5%9C%B0%E5%9D%80%E5%8D%B3%E5%8F%AF\(%E6%B5%8F%E8%A7%88%E5%99%A8%E5%8F%AF%E8%83%BD%E8%BF%98%E5%BE%97%E5%AE%89%E8%A3%85%E6%8F%92%E4%BB%B6\)%0A%20%20%20%20%3E%20http%3A%2F%2F8.142.177.235%3A8080%2Flive%2Flivestream.m3u8%0A2.%20H5\(HTTP\-FLV\)%EF%BC%8C%E8%BF%99%E4%B8%AA%E7%95%A5%E9%BA%BB%E7%83%A6%E7%82%B9%E5%88%86%E6%88%90%E4%B8%A4%E7%A7%8D%E6%83%85%E5%86%B5%E5%90%A7%EF%BC%9A%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%92%8C%E6%B5%8F%E8%A7%88%E5%99%A8%0A%20%20%20%201.%20ffmpeg%20%EF%BC%8C%E8%BF%99%E4%B8%AA%E7%9B%B4%E6%8E%A5%E5%8F%AF%E4%BB%A5%E7%94%A8%0A%20%20%20%20%3Effplay%20\-i%20%22http%3A%2F%2F8.142.177.235%3A8080%2Flive%2Flivestream.flv%22%20\-fflags%20nobuffer%0A%20%20%20%202.%20%E5%86%99%E7%82%B9JS%E4%BB%A3%E7%A0%81%EF%BC%8C%E7%94%A8flv.js%E5%BA%93%EF%BC%8C%E6%9C%80%E5%90%8E%E5%A1%AB%E5%86%99%E6%9C%8D%E5%8A%A1%E5%99%A8%20URL%20%E5%9C%B0%E5%9D%80%E5%8D%B3%E5%8F%AF%EF%BC%9A%0A%20%20%20%20%3Ehttps%3A%2F%2Fgithub.com%2Fbilibili%2Fflv.js%0A%20%20%20%20%3Ehttp%3A%2F%2F8.142.177.235%3A8080%2Flive%2Flivestream.flv%0A%20%20%20%20%3E%E6%97%A9%E6%9C%9F%E6%98%AF%E6%B5%8F%E8%A7%88%E5%99%A8%E5%AE%89%E8%A3%85%20flash\_player%20%E6%8F%92%E4%BB%B6%EF%BC%8C%E5%8F%AF%E4%BB%A5%E7%9B%B4%E6%8E%A5%E6%92%AD%E6%94%BE%E3%80%82%E4%B9%8B%E5%90%8Eflash%20%E8%A2%AB%E6%94%BE%E5%BC%83%EF%BC%8C%E6%B5%8F%E8%A7%88%E5%99%A8%E8%BF%98%E6%9C%89%E4%BA%9B%E5%85%B6%E5%AE%83%E6%8F%92%E4%BB%B6%E6%92%AD%E6%94%BE%EF%BC%8C%E5%90%8E%E6%9C%9F%E5%9F%BA%E6%9C%AC%E4%B8%8A%E8%BF%99%E4%BA%9B3%E6%96%B9%E6%8F%92%E4%BB%B6%E4%B9%9F%E8%A2%AB%E8%B0%A2%E8%B0%A2%E4%BA%86%EF%BC%8C%E5%8F%AA%E6%9C%89%E7%94%A8JS%E5%BA%93%E3%80%82%0A3.%20RTMP%3A%20rtmp%3A%2F%2F8.142.177.235%2F%2Flive%2Flivestream%0A%3EVLC%E6%98%AF%E4%B8%AA%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%BD%AF%E4%BB%B6\(%E6%92%AD%E6%94%BE%E5%99%A8\)%EF%BC%8C%E5%8E%BB%E5%AE%98%E7%BD%91%E4%B8%8B%E4%B8%80%E4%B8%AA%EF%BC%8C%E7%84%B6%E5%90%8E%E6%89%93%E5%BC%80%EF%BC%8C%E9%87%8C%E9%9D%A2%E9%85%8D%E7%BD%AE%E4%B8%80%E4%B8%AA%E8%BF%99%E4%B8%AAURL%E5%9C%B0%E5%9D%80%E5%8D%B3%E5%8F%AF%E3%80%82%0A%0Awebrtc%0A\-\-\-\-%0A%E8%BF%99%E4%B8%AA%E7%95%A5%E6%81%B6%E5%BF%83%E4%BA%BA%EF%BC%8C%E6%B5%8F%E8%A7%88%E5%99%A8%20%E5%BC%BA%E5%88%B6%E8%A6%81%E6%B1%82%E5%BF%85%E9%A1%BB%E5%BE%97HTTPS%EF%BC%8C%E6%89%80%E4%BB%A5%E5%AE%83%E5%B0%B1%E5%BE%97%E5%8D%95%E6%90%9E%EF%BC%9A%0A%0A%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%EF%BC%9Ahttps.rtc.conf%0A%60%60%60%0Acandidate%208.142.177.235%3B%0Alisten%208000%3B%23UDP%20%0A%0Akey%20%2Fdata%2Fwww%2Fsrs%2F9245472\_webrtc\-push.seedreality.com.key%3B%0Acert%20%2Fdata%2Fwww%2Fsrs%2F9245472\_webrtc\-push.seedreality.com.pem%3B%0A%0A%60%60%60%0A%0A%E5%90%AF%E5%8A%A8%EF%BC%9A\(%E8%BF%99%E9%87%8C%E6%8D%A2%E4%BA%86%E4%B8%AA%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6\)%0A%3E.%2Fobjs%2Fsrs%20\-c%20conf%2Fhttps.rtc.conf%0A%0A%0A%E6%B5%8B%E8%AF%95\-%E6%B5%8F%E8%A7%88%E5%99%A8%E6%8E%A8%E6%B5%81%EF%BC%9A%0A%3E%20https%3A%2F%2Fwebrtc\-push.seedreality.com%3A8088%2Fplayers%2Frtc\_publisher.html%3Fautostart%3Dtrue%26stream%3Dlivestream%26api%3D1990%26schema%3Dhttps%0A%0A%0A%0A%E6%B5%8B%E8%AF%95\-%E6%B5%8F%E8%A7%88%E5%99%A8%E6%92%AD%E6%94%BE%EF%BC%9A%0A%3Ehttps%3A%2F%2Fwebrtc\-push.seedreality.com%3A8088%2Fplayers%2Frtc\_player.html%3Fautostart%3Dtrue%26stream%3Dlivestream%26api%3D1990%26schema%3Dhttps%0A%0A%0A%0A%0A%0A%0A%0Aubutu%E5%AE%89%E8%A3%85ffmpeg%0A\-\-\-%0A%E4%B8%8B%E8%BD%BD%E5%9C%B0%E5%9D%80%EF%BC%9A%0A%3Ehttps%3A%2F%2Fffmpeg.org%2Fdownload.html%0A%20%20%20%20%3Ehttps%3A%2F%2Flaunchpad.net%2Fubuntu%2F%2Barchive%2Fprimary%2F%2Bsourcefiles%2Fffmpeg%2F7%3A5.1.1\-1ubuntu1%2Fffmpeg\_5.1.1.orig.tar.xz%0A%60%60%60%20%20%20%0Axz%20\-dk%20ffmpeg\_5.1.1.orig.tar.xz%0Atar%20xvf%20ffmpeg\_5.1.1.orig.tar%0Acd%20ffmpeg\-5.1.1%2F%0A%60%60%60%0A%0A%60%60%60%0A%E5%AE%89%E8%A3%85ffplay%E9%9C%80%E8%A6%81%E7%9A%84%E4%BE%9D%E8%B5%96%0Asudo%20apt\-get%20install%20libx11\-dev%20xorg\-dev%20libsdl2\-2.0%20libsdl2\-dev%0Asudo%20apt%20install%20yasm%20pkg\-config%20libopencore\-amrnb\-dev%20libopencore\-amrwb\-dev%0A%60%60%60%0A%0Amkdir%20\-p%20%2Fsoft%2Fffmpeg%0A%0A%0A.%2Fconfigure%20\-\-prefix%3D%2Fsoft%2Fffmpeg%20\-\-enable\-gpl%20\-\-enable\-version3%20\-\-enable\-nonfree%20\-\-enable\-ffplay%20\-\-enable\-ffprobe%20%20\-\-enable\-debug%0A%0A%E4%B8%8B%E9%9D%A23%E4%B8%AA%E6%89%BE%E4%B8%8D%E5%88%B0%EF%BC%8C%E5%85%88%E4%B8%8D%E7%AE%A1%E4%BA%86%0A\-\-enable\-libfdk\-aac%20\-\-enable\-libx264%20\-\-enable\-libx265%20
