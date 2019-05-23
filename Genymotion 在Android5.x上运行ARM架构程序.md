layout: genymotion
title: 在Android5.x上运行ARM架构程序
date: 2016-04-20 10:04:53
tags: [Genymotion, Android, ARM-Translation]
---
# Genymotion 在Android5.x上运行ARM架构程序

原生的Genymotion模拟器只支持x86架构，很多使用了.so文件的应用不支持x86架构，因此无法运行。如果想要运行，必须安装ARM转换包。

本文提供`Genymotion-ARM-Translation`、`Genymotion-ARM-Translation_v1.1`、`ARM_Translation_Lollipop`的三个版本。其中`Genymotion-ARM-Translation`、`Genymotion-ARM-Translation_v1.1`对应Android 5.x以下，个人发现4.4安装v1.1没卵用，安装前者反而有效，读者自行试验。`ARM_Translation_Lollipop`毫无疑问对应5.x系统。
> 下载地址：[https://pan.baidu.com/s/1qY0Onk4](https://pan.baidu.com/s/1qY0Onk4 "https://pan.baidu.com/s/1qY0Onk4") 密码：f44k

## 5.x的使用方法
使用方法：

1. 在 Genymotion 里面建立 5.0 or 5.1 的模拟器。

2. 开机后把 `ARM_Translation_Lollipop.zip`（请勿解压）拖到模拟器中，自动安装。

3. 先不要重启模拟器，打开CMD命令行，输入`adb shell /system/etc/houdini_patcher.sh`

4. 完成后重启模拟器。

> 方法及安装包来自二三接脚大神：[http://23pin.logdown.com/posts/294446-genymotion-use-arm-translation-on-5x-image](http://23pin.logdown.com/posts/294446-genymotion-use-arm-translation-on-5x-image)

## 5.0以下的使用方法
1. 在 Genymotion 里面建立4.x的模拟器。

2. 开机后把`Genymotion-ARM-Translation`或者`Genymotion-ARM-Translation_v1.1`（请勿解压）拖到模拟器中，自动安装。

3. 完成后重启模拟器。