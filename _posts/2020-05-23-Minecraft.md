---
layout: post
title: Deepin下安装我的世界
categories: 游戏
description: Deepin下安装我的世界
keywords: 游戏 我的世界
---

## <center>Deepin下安装我的世界</center>

一、Java环境安装

1.打开终端 安装OpenJDK8和Openjfx

```
sudo apt-get install openjdk8

sudo apt-get install openjfx
```

2.注释掉此文件的最后一行(在此使用的是vim)

```
sudo vim /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/accessibility.properties
```

![Alt text](https://sbimg.cn/image/openjdk1.pZdHD)

二、HMCL下载

抵制某du w盘从我做起

```
https://cloud.189.cn/t/fUz6b2EVzYfi（访问码：asl7）
```

三、开始游戏

```
# 进入HMCL的下载位置

cd Download

# 启动HMCL

java -jar HMCL-3.2.130.jar  (文件名称可能有所不同，请自行补全)
```

1.选择离线模式，填好信息

2.点击游戏列表，选择一个游戏版本安装，太老版本的可能安装失败。我自己安装的是1.7.10

![Alt text](https://sbimg.cn/image/you-xi-lie-biao.pZcAe)

![Alt text](https://sbimg.cn/image/xuan-ze-you-xi-ban-ben.pZAv6)

3.点击左侧栏旁边的齿轮,选择自动安装 安装Forge

![Alt text](https://sbimg.cn/image/zi-dong-an-zhuang.pZBfO)

4.在游戏设置里设置好内存和分辨率

![Alt text](https://sbimg.cn/image/you-xi-guan-li.pZgSN)

5.点击返回,在主页启动游戏即可

四、拓展 Mod 光影安装

方法一

例:安装G键合成表和地图坐标

```
# G键合成表(1.7.10版本)

https://cloud.189.cn/t/Z7FFRfz2In6v（访问码：3uxy）

# 地图坐标

https://cloud.189.cn/t/qyQrE3rMVZz2（访问码：ubn7）
```

下载完成后在模组管理中进行导入

![Alt text](https://sbimg.cn/image/zi-dong-an-zhuang.pZBfO)

方法二

打开文件管理器显示隐藏文件夹,找到.minecraft文件夹

mods: 将mod放入其中，重启游戏生效

resourcepacks: 将材质包放入其中，重启游戏之后到设置选择。

saves: Minecraft本地地图存放在这里

shaderpacks: 光影包存放在这里，重启游戏之后到视频选项中开启。