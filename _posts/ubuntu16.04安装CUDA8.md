---
layout:     post
title:      ubuntu16.04安装CUDA8
subtitle:   
date:       2021-01-30
author:     Pallas Cat
header-img: img/post-bg-recitewords.jpg
catalog: false
tags:
    - CUDA
---

# 一、准备工作


```
查看GPU信息
lspci | grep -i nvidia
查看当前使用的系统内核版本
uname -r
查看gcc版本
gcc --version
查看所有内核
sudo dpkg --get-selections|grep linux-image
```


![图片.png](https://cdn.nlark.com/yuque/0/2020/png/702655/1597303740875-48d323c2-76db-47d4-8002-beacdfc65ad0.png#align=left&display=inline&height=297&margin=%5Bobject%20Object%5D&name=%E5%9B%BE%E7%89%87.png&originHeight=297&originWidth=735&size=75048&status=done&style=none&width=735)

CUDA8.0最高仅支持4.4版本内核，因此第一步工作就是更换系统内核。如你的内核不高于4.4，可以跳过整个该步骤。实际上我用的这个镜像安装完内核版本上4.14，高于cuda8要求打4.4，因此必须更换系统内核。

**sudo apt-get install linux-image-4.4.0-75-generic**
**sudo apt-get remove linux-image-4.13.0-75-generic**




更换完了内核后要禁用nouveau，这是ubuntu自带的显卡驱动，和CUDA冲突，所以要禁用，不禁用的话可能会导致黑屏或循环登陆。终端中运行：$  lsmod | grep nouveau，如果有输出则代表nouveau正在加载。需要我们手动禁掉nouveau。Ubuntu的nouveau禁用方法：

a、在/etc/modprobe.d中创建文件blacklist-nouveau.conf
输入命令：$  sudo vi /etc/modprobe.d/blacklist-nouveau.conf                       （利用vi编辑器编辑和保存文件）
在文件中输入一下内容：
blacklist nouveau
options nouveau modeset=0


b、在blacklist.conf文件末尾添加一行 blacklist nouveau


c、执行：
$ sudo update-initramfs –u


d、再执行：
$  lsmod | grep nouveau
若无内容输出，则禁用成功，若仍有内容输出，请检查操作，并重复上述操作。


重启电脑，发现界面的分辨率，图标大小等变了，这就是禁用 nouveau的效果，等安装完NVIDIA驱动后会恢复正常。
输入 sudo service lightdm stop关闭图形界面，按ctrl+alt+f1或者f2登录帐号


# 二、安装显卡驱动，CUDA，CUDNN


CUDA提供两种安装方式：package manager安装和runfile安装，  package manager  安装方式相对简单一些，但是我在阅读别人博客的过程中发现选择这种方式在安装过程中问题可能多一点，失败的概率较大。为了减少不必要的麻烦我选择runfile安装方式。
下载cuda安装包：cuda官网下载，根据系统信息选择对应的版本，runfile安装的话最后一项要选择 runfile文件。


安装CUDA前要安装显卡驱动，在官网找到对应的驱动下载。显卡驱动文件名：NVIDIA-Linux-x86_64-450.57.run。下载CUDA8的安装文件，官网有好几种，我们选择runfile，下载文件，得到cuda_8.0.61_375.26_linux.run。


安装驱动
sudo sh NVIDIA-Linux-x86_64-450.57.run
在有些系统里不加-no-x-check -no-nouveau-check -no-opengl-files图形界面会循环登录，最好加上
sudo sh NVIDIA-Linux-x86_64-381.22.run -no-x-check -no-nouveau-check -no-opengl-files
![image.png](https://cdn.nlark.com/yuque/0/2020/png/702655/1599643163515-86c015cb-f03c-4910-960a-beb6b847ff12.png#align=left&display=inline&height=343&margin=%5Bobject%20Object%5D&name=image.png&originHeight=343&originWidth=746&size=17353&status=done&style=none&width=746)
![image.png](https://cdn.nlark.com/yuque/0/2020/png/702655/1599643179838-6f491df1-e271-472d-8e22-b3c5b543815a.png#align=left&display=inline&height=436&margin=%5Bobject%20Object%5D&name=image.png&originHeight=436&originWidth=729&size=16579&status=done&style=none&width=729)
![image.png](https://cdn.nlark.com/yuque/0/2020/png/702655/1599643199929-1c16f61a-7ae0-4580-b27a-98d8c7997e38.png#align=left&display=inline&height=303&margin=%5Bobject%20Object%5D&name=image.png&originHeight=303&originWidth=741&size=20147&status=done&style=none&width=741)

在终端中输入：$  sudo apt-get install linux-headers-$(uname -r)
可以安装对应kernel版本的kernel header和package development
安装了linux-headers-4.4.0-75 linux-headers-4.4.0-75-generic

![image.png](https://cdn.nlark.com/yuque/0/2020/png/702655/1599643223427-0f975ee8-4033-48c3-b275-2e734b907386.png#align=left&display=inline&height=436&margin=%5Bobject%20Object%5D&name=image.png&originHeight=436&originWidth=772&size=19743&status=done&style=none&width=772)

正确情况：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/702655/1600161075548-25749070-860e-4ffc-9294-5d86bb65ae72.png#align=left&display=inline&height=147&margin=%5Bobject%20Object%5D&name=image.png&originHeight=294&originWidth=1049&size=10721&status=done&style=none&width=524.5)
![image.png](https://cdn.nlark.com/yuque/0/2020/png/702655/1600161076274-5ff5ff1b-75d3-4732-b8cb-f3764f03bee1.png#align=left&display=inline&height=155&margin=%5Bobject%20Object%5D&name=image.png&originHeight=309&originWidth=1054&size=25199&status=done&style=none&width=527)


![image.png](https://cdn.nlark.com/yuque/0/2020/png/702655/1600161098828-bde7816f-9427-4d8a-8a0b-4bcb362a4e5e.png#align=left&display=inline&height=125&margin=%5Bobject%20Object%5D&name=image.png&originHeight=250&originWidth=1042&size=20049&status=done&style=none&width=521)
![image.png](https://cdn.nlark.com/yuque/0/2020/png/702655/1600161129952-802344dd-431d-401d-ab13-941f4311c65e.png#align=left&display=inline&height=109&margin=%5Bobject%20Object%5D&name=image.png&originHeight=217&originWidth=1005&size=18484&status=done&style=none&width=502.5)


如果报错要进bios将secure boot改成disable
安装完重启


安装cuda，并且禁用opengl，据说是为了防止和系统原有的opengl冲突
sudo sh cuda_8.0.61_375.26_linux.run --no-opengl-lib
安装过程中问是否安装nvidia driver，选no，我们之前已经安装啦
问是否创建 symbolic link，如果是第一次安装就选yes


这样应该就顺利安装完了，重启，sudo service lightdm start开启图形界面
然后就能检测cuda是否安装成功了。
网上有很多博客教怎么设置环境变量。我安装完没有设置，输入 nvcc --version就能看到版本号，运行sample也没问题。

![图片.png](https://cdn.nlark.com/yuque/0/2020/png/702655/1597306177955-464b364f-cfd8-4da9-a0b0-436f1686153b.png#align=left&display=inline&height=900&margin=%5Bobject%20Object%5D&name=%E5%9B%BE%E7%89%87.png&originHeight=900&originWidth=1600&size=679403&status=done&style=none&width=1600)

终端中输入 $ sudo gedit /etc/profile
在打开的文件末尾，添加以下两行。
64位系统：
export PATH=/usr/local/cuda-9.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64\ ${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}


source /etc/profile

安装cudnn
下载cudnn [https://developer.nvidia.com/rdp/cudnn-archive ](https://developer.nvidia.com/rdp/cudnn-archive)
官网下载要注册，填问卷等，知乎上一个答案有百度网盘链接，下载就行，然后解压
```
• sudo cp cuda/include/cudnn.h /usr/local/cuda/include/
• 
• sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64/
• 
• sudo chmod a+r /usr/local/cuda/include/cudnn.h
• 
• sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
```


查看cudnn版本
cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
![图片.png](https://cdn.nlark.com/yuque/0/2020/png/702655/1597309628199-fd1d6492-364d-49b6-9884-04f74f38ddc7.png#align=left&display=inline&height=173&margin=%5Bobject%20Object%5D&name=%E5%9B%BE%E7%89%87.png&originHeight=173&originWidth=729&size=34094&status=done&style=none&width=729)






# 三、测试CUDA是否安装成功
转到sample保存目录，像我在安装过程中用的默认保存位置，因此它们在我的~/NVIDIA_CUDA-8.0_Samples/目录下，在这个目录下，进入./1_Utilities/deviceQuery/，在这个目录下make然后执行./deviceQuery。如果出现如下画面说明CUDA安装成功。
![图片.png](https://cdn.nlark.com/yuque/0/2020/png/702655/1597304343298-d4fc850f-3f26-40d6-b127-d7feecd2ecd2.png#align=left&display=inline&height=806&margin=%5Bobject%20Object%5D&name=%E5%9B%BE%E7%89%87.png&originHeight=806&originWidth=733&size=181598&status=done&style=none&width=733)

![图片.png](https://cdn.nlark.com/yuque/0/2020/png/702655/1597303496557-6e51181b-4aad-4b83-b00b-cb8596e65e51.png#align=left&display=inline&height=108&margin=%5Bobject%20Object%5D&name=%E5%9B%BE%E7%89%87.png&originHeight=108&originWidth=727&size=27637&status=done&style=none&width=727)
![图片.png](https://cdn.nlark.com/yuque/0/2020/png/702655/1597304822165-8526be40-d41e-4350-a009-f3b96937b8d9.png#align=left&display=inline&height=81&margin=%5Bobject%20Object%5D&name=%E5%9B%BE%E7%89%87.png&originHeight=81&originWidth=723&size=20697&status=done&style=none&width=723)

# 四、参考博客
[https://blog.csdn.net/qlulibin/article/details/78714596](https://blog.csdn.net/qlulibin/article/details/78714596)
[https://www.cnblogs.com/left4back/p/10952845.html](https://www.cnblogs.com/left4back/p/10952845.html)
[https://blog.csdn.net/wanzhen4330/article/details/81699769](https://blog.csdn.net/wanzhen4330/article/details/81699769)
[https://blog.csdn.net/xunan003/article/details/81665835](https://blog.csdn.net/xunan003/article/details/81665835)
[https://blog.csdn.net/sinat_30545761/article/details/107709468](https://blog.csdn.net/sinat_30545761/article/details/107709468)
[https://blog.csdn.net/lb838315586/article/details/82495804](https://blog.csdn.net/lb838315586/article/details/82495804)


