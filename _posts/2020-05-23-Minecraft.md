---
layout: post
title: Deepin下安装我的世界
categories: 游戏
description: Deepin下安装我的世界
keywords: 游戏 我的世界
---

## <center>Deepin下安装我的世界</center>

Header: X-Github-Request-Id=2E50:1327:F4EE67:11171CC:60182AE0                                                           Header: Status=200 OK                                                                                                   Header: X-Ratelimit-Remaining=59                                                                                        Header: X-Frame-Options=deny                                                                                            Header: Strict-Transport-Security=max-age=31536000; includeSubdomains; preload                                          Header: Referrer-Policy=origin-when-cross-origin, strict-origin-when-cross-origin                                       Header: Date=Mon, 01 Feb 2021 16:22:56 GMT                                                                              Header: X-Ratelimit-Limit=60                                                                                            Header: X-Ratelimit-Used=1                                                                                              Header: X-Content-Type-Options=nosniff                                                                                  Header: Content-Security-Policy=default-src 'none'                                                                      Header: Vary=Accept-Encoding, Accept, X-Requested-With                                                                  Header: Content-Type=text/html;charset=utf-8                                                                            Header: Server=GitHub.com                                                                                               Header: Access-Control-Allow-Origin=*                                                                                   Header: X-Xss-Protection=1; mode=block                                                                                  Header: X-Commonmarker-Version=0.21.0                                                                                   Header: X-Ratelimit-Reset=1612200176                                                                                    Header: Access-Control-Expose-Headers=ETag, Link, Location, Retry-After, X-GitHub-OTP, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Used, X-RateLimit-Reset, X-OAuth-Scopes, X-Accepted-OAuth-Scopes, X-Poll-Interval, X-GitHub-Media-Type, Deprecation, Sunset                                                                                                * [layout: posttitle: Deepin下安装我的世界categories: 游戏description: Deepin下安装我的世界keywords: 游戏 我的世界](#layout-posttitle-deepin下安装我的世界categories-游戏description-deepin下安装我的世界keywords-游戏-我的世界)                * [Deepin下安装我的世界](#deepin下安装我的世界)

一、Java环境安装

1.打开终端 安装OpenJDK8和Openjfx

```
sudo apt-get install openjdk-8-jdk

sudo apt-get install openjfx
```

2.注释掉此文件的最后一行(在此使用的是vim)

```
sudo vim /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/accessibility.properties
```

![Alt text](https://i.niupic.com/images/2021/02/01/9aNe.png)

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

![Alt text](https://i.niupic.com/images/2021/02/01/9aNg.png)

![Alt text](https://i.niupic.com/images/2021/02/01/9aNh.png)

3.点击左侧栏旁边的齿轮,选择自动安装 安装Forge

![Alt text](https://i.niupic.com/images/2021/02/01/9aNi.png)

4.在游戏设置里设置好内存和分辨率

![Alt text](https://i.niupic.com/images/2021/02/01/9aNj.png)

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

![Alt text](https://i.niupic.com/images/2021/02/01/9aNk.png)

方法二

打开文件管理器显示隐藏文件夹,找到.minecraft文件夹

mods: 将mod放入其中，重启游戏生效

resourcepacks: 将材质包放入其中，重启游戏之后到设置选择。

saves: Minecraft本地地图存放在这里

shaderpacks: 光影包存放在这里，重启游戏之后到视频选项中开启。