# PCL

## filter

过滤掉 点云数据中的 噪音

## feature

特征

## keyword point

关键点

## registration

配准

## kd\-tree

## oc\-tree

## segmentation

分割

## sample consensus

拟合

## surface

## recognition

识别

## io

处理：PCD 格式点云数据文件

处理：IO设备的数据

## viszualization

可视化

## 安装\-ubuntu18.0

前置回顾：

1. mac 安装失败
2. cantos7 centos8 centos\-strem 均安装失败

最后只能用ubuntu了

包安装：

```
apt-get install llvm
apt-get install clang clang-format-10
apt-get install git build-essential linux-libc-dev
apt-get install cmake cmake-gui
apt-get install libusb-1.0-0-dev libusb-dev libudev-dev
apt-get install mpi-default-dev openmpi-bin openmpi-common 
apt-get install libeigen3-dev
apt-get install libboost-all-dev
apt-get install libflann1.9 libflann-dev
apt-get install libqhull* libgtest-dev 
apt-get install libx11-doc mesa-utils   
apt-get install mono-complete
apt-get install libpcap-dev
apt-get install libopencv-dev
apt-get install libopenni-dev libopenni2-dev 

#下面几个包循环依赖这个包，但这个包在不同os版本下数字不太一样，注意下：
apt-get install libx11-6=2:1.6.9-2ubuntu1.2
apt-get install libx11-6=2:1.6.4-3ubuntu0.4

apt-get install libxmu-dev libxi-dev
apt-get install freeglut3-dev pkg-config
apt-get install libvtk7.1p libvtk7.1p-qt libvtk7-dev libvtk7-qt-dev vtk7 
apt-get install qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools

```

```
wget https://www.coin-or.org/download/source/metslib/metslib-0.5.2.zip
unzip metslib-0.5.2.zip 
cd metslib-0.5.2
./configure
make
make install
```

PCL安装：

```
#67MB，而且是github，速度很慢，可以从本机传上去一上
wget wget https://github.com/PointCloudLibrary/pcl/releases/download/pcl-1.12.0/source.zip
unzip source.zip
cd pcl
mkdir build
cd build

cmake  -D CMAKE_BUILD_TYPE=None  -D BUILD_GPU=ON  -D BUILD_apps=ON  -D BUILD_examples=ON ..

#看一眼，build后的信息，是不是都正常，有 not found 的，继续安装

make & make install
```

## 安装\-ubuntu\-UI

sudo apt\-get update && sudo apt\-get upgrade

sudo apt\-get install lightdm

sudo systemctl start lightdm

sudo apt\-get install tasksel

sudo tasksel install xubuntu\-desktop

sudo apt\-get install ubuntu\-desktop

filter%0A\-\-\-%0A%E8%BF%87%E6%BB%A4%E6%8E%89%20%E7%82%B9%E4%BA%91%E6%95%B0%E6%8D%AE%E4%B8%AD%E7%9A%84%20%E5%99%AA%E9%9F%B3%0A%0Afeature%0A\-\-\-\-%0A%E7%89%B9%E5%BE%81%0A%0Akeyword%20point%0A\-\-\-\-%0A%E5%85%B3%E9%94%AE%E7%82%B9%0A%0Aregistration%0A\-\-\-\-\-%0A%E9%85%8D%E5%87%86%0A%0Akd\-tree%0A\-\-\-\-%0A%0Aoc\-tree%0A\-\-\-\-%0A%0A%0Asegmentation%0A\-\-\-\-\-%0A%E5%88%86%E5%89%B2%0A%0A%0Asample%20consensus%0A\-\-\-%0A%E6%8B%9F%E5%90%88%0A%0Asurface%0A\-\-\-\-%0A%0Arecognition%0A\-\-\-%0A%E8%AF%86%E5%88%AB%0A%0Aio%0A\-\-\-\-\-%0A%E5%A4%84%E7%90%86%EF%BC%9APCD%20%E6%A0%BC%E5%BC%8F%E7%82%B9%E4%BA%91%E6%95%B0%E6%8D%AE%E6%96%87%E4%BB%B6%0A%E5%A4%84%E7%90%86%EF%BC%9AIO%E8%AE%BE%E5%A4%87%E7%9A%84%E6%95%B0%E6%8D%AE%0A%0A%0Aviszualization%0A\-\-\-\-%0A%E5%8F%AF%E8%A7%86%E5%8C%96%0A%0A%0A%0A%E5%AE%89%E8%A3%85\-ubuntu18.0%0A\-\-\-%0A%E5%89%8D%E7%BD%AE%E5%9B%9E%E9%A1%BE%EF%BC%9A%0A1.%20mac%20%E5%AE%89%E8%A3%85%E5%A4%B1%E8%B4%A5%0A2.%20cantos7%20centos8%20centos\-strem%20%E5%9D%87%E5%AE%89%E8%A3%85%E5%A4%B1%E8%B4%A5%0A%0A%E6%9C%80%E5%90%8E%E5%8F%AA%E8%83%BD%E7%94%A8ubuntu%E4%BA%86%0A%0A%E5%8C%85%E5%AE%89%E8%A3%85%EF%BC%9A%0A%0A%60%60%60%0Aapt\-get%20install%20llvm%0Aapt\-get%20install%20clang%20clang\-format\-10%0Aapt\-get%20install%20git%20build\-essential%20linux\-libc\-dev%0Aapt\-get%20install%20cmake%20cmake\-gui%0Aapt\-get%20install%20libusb\-1.0\-0\-dev%20libusb\-dev%20libudev\-dev%0Aapt\-get%20install%20mpi\-default\-dev%20openmpi\-bin%20openmpi\-common%20%0Aapt\-get%20install%20libeigen3\-dev%0Aapt\-get%20install%20libboost\-all\-dev%0Aapt\-get%20install%20libflann1.9%20libflann\-dev%0Aapt\-get%20install%20libqhull\*%20libgtest\-dev%20%0Aapt\-get%20install%20libx11\-doc%20mesa\-utils%20%20%20%0Aapt\-get%20install%20mono\-complete%0Aapt\-get%20install%20libpcap\-dev%0Aapt\-get%20install%20libopencv\-dev%0Aapt\-get%20install%20libopenni\-dev%20libopenni2\-dev%20%0A%0A%0A%23%E4%B8%8B%E9%9D%A2%E5%87%A0%E4%B8%AA%E5%8C%85%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96%E8%BF%99%E4%B8%AA%E5%8C%85%EF%BC%8C%E4%BD%86%E8%BF%99%E4%B8%AA%E5%8C%85%E5%9C%A8%E4%B8%8D%E5%90%8Cos%E7%89%88%E6%9C%AC%E4%B8%8B%E6%95%B0%E5%AD%97%E4%B8%8D%E5%A4%AA%E4%B8%80%E6%A0%B7%EF%BC%8C%E6%B3%A8%E6%84%8F%E4%B8%8B%EF%BC%9A%0Aapt\-get%20install%20libx11\-6%3D2%3A1.6.9\-2ubuntu1.2%0Aapt\-get%20install%20libx11\-6%3D2%3A1.6.4\-3ubuntu0.4%0A%0Aapt\-get%20install%20libxmu\-dev%20libxi\-dev%0Aapt\-get%20install%20freeglut3\-dev%20pkg\-config%0Aapt\-get%20install%20libvtk7.1p%20libvtk7.1p\-qt%20libvtk7\-dev%20libvtk7\-qt\-dev%20vtk7%20%0Aapt\-get%20install%20qtbase5\-dev%20qtchooser%20qt5\-qmake%20qtbase5\-dev\-tools%0A%0A%60%60%60%0A%60%60%60%0Awget%20https%3A%2F%2Fwww.coin\-or.org%2Fdownload%2Fsource%2Fmetslib%2Fmetslib\-0.5.2.zip%0Aunzip%20metslib\-0.5.2.zip%20%0Acd%20metslib\-0.5.2%0A.%2Fconfigure%0Amake%0Amake%20install%0A%60%60%60%0APCL%E5%AE%89%E8%A3%85%EF%BC%9A%0A%60%60%60%0A%2367MB%EF%BC%8C%E8%80%8C%E4%B8%94%E6%98%AFgithub%EF%BC%8C%E9%80%9F%E5%BA%A6%E5%BE%88%E6%85%A2%EF%BC%8C%E5%8F%AF%E4%BB%A5%E4%BB%8E%E6%9C%AC%E6%9C%BA%E4%BC%A0%E4%B8%8A%E5%8E%BB%E4%B8%80%E4%B8%8A%0Awget%20wget%20https%3A%2F%2Fgithub.com%2FPointCloudLibrary%2Fpcl%2Freleases%2Fdownload%2Fpcl\-1.12.0%2Fsource.zip%0Aunzip%20source.zip%0Acd%20pcl%0Amkdir%20build%0Acd%20build%0A%0Acmake%20%20\-D%20CMAKE\_BUILD\_TYPE%3DNone%20%20\-D%20BUILD\_GPU%3DON%20%20\-D%20BUILD\_apps%3DON%20%20\-D%20BUILD\_examples%3DON%20..%0A%0A%23%E7%9C%8B%E4%B8%80%E7%9C%BC%EF%BC%8Cbuild%E5%90%8E%E7%9A%84%E4%BF%A1%E6%81%AF%EF%BC%8C%E6%98%AF%E4%B8%8D%E6%98%AF%E9%83%BD%E6%AD%A3%E5%B8%B8%EF%BC%8C%E6%9C%89%20not%20found%20%E7%9A%84%EF%BC%8C%E7%BB%A7%E7%BB%AD%E5%AE%89%E8%A3%85%E4%BE%9D%E8%B5%96%E5%8C%85%0A%0Amake%20%26%20make%20install%0A%60%60%60%0A%0A%E5%AE%89%E8%A3%85\-ubuntu\-UI%0A\-\-\-\-\-\-%0Asudo%20apt\-get%20update%20%26%26%20sudo%20apt\-get%20upgrade%0Asudo%20apt\-get%20install%20lightdm%0Asudo%20systemctl%20start%20lightdm%0Asudo%20apt\-get%20install%20tasksel%0Asudo%20tasksel%20install%20xubuntu\-desktop%0Asudo%20apt\-get%20install%20ubuntu\-desktop%0A%0A%0A
