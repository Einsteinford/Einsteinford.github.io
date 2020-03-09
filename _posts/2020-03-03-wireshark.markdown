---
layout: post
title: Wireshark 在MacOS10.15.3 系统无法显示网卡的解决方法
date: 2020-03-03 16:07:24.000000000 +08:00
---

###### 这问题困扰了我好几个月，为什么的我MacbookPro最新系统运行不了Wireshark，运行后只会显示如下的页面

![Wireshark界面](https://upload-images.jianshu.io/upload_images/2527601-95d2dcebe4cbedff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


en0网卡呢？去了stackoverflow和软件的官网都没法真的找到解决办法，总的来说就要我修改 /dev/bpf* 的权限来解决这个问题。

其实在安装软件时，正常安装了 ChmodBPF.pkg 后，所有bpf接口都是属于access_bpf用户的，这是正确的，不需要修改了。
![字颜色难看请忽略](https://upload-images.jianshu.io/upload_images/2527601-61092421956b290f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

直到我偶然发现
![关闭了Wi-Fi](https://upload-images.jianshu.io/upload_images/2527601-710bc28ae1bca712.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我一直用有线网，所以关闭了Wi-Fi，也可以避免蓝牙信号与Wi-Fi信号的干扰，可是竟然打开Wi-Fi才能够解决这个问题？！！

![问题解决](https://upload-images.jianshu.io/upload_images/2527601-0ae3226c4506158a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



