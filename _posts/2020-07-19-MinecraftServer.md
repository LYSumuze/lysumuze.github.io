---
layout: post
title: Linux下搭建我的世界服务器(Forge版)
categories: Linux 游戏
description: Linux Minecraft
keywords: Linux下搭建我的世界服务器(Forge版)
---

## <center> Linux下搭建我的世界服务器(Forge版) </center>

前言

为了戒网瘾，将自己的工作学习环境转移到了Linux操作系统上，耐不住无聊，Linux上游戏又很少，就想到玩Minecraft。玩久了世界就一个人，便萌生自己做Minecraft的服务器的念头。

趁着假期，自己查了很多资料，由于没有经验，从第一个纯净版服务端到Forge版，从终端管理到网页管理，中间零零散散的问题，终于在经过一周的熬夜下完成了。

最开始是部署在笔记本上，用frp穿透，期间发现阿里爸爸的的学生活动，答题领到了一个2核，4G的ECS服务器一年的使用期，又重新部署在了服务器上。

将这些记录下来，希望可以帮助到你们搭建自己的服务端，可以免去查大量资料，避一些坑，快速的解决一些问题。

整合的服务端里有着大佬们很多很棒的mod，并不希望其作为二次创作后的盈利方式。

一位modder维护了数年的项目，点击量下载量几万，十几万，甚至数百万，仅几人赞助的例子很常见，最后还可能因侵权等其他问题所放弃。

游戏因为有了mod的存在，有了众多无数热爱的游戏的modder存在，用爱发电，才使其有了顽强的生命力。

最后，向每一位热爱游戏的modder致敬!

---

一、准备工作

1.服务器介质：阿里云ECS/笔记本 

2.操作系统： Ubuntu 18.04 64位/Deepin 15.11 64位

3.游戏服务器文件：

[Minecraft1.7.10(Forge版终端后台)下载](https://cloud.189.cn/t/rMZNnqnUZ3yi)  (访问码：no0h) 

优点：配置简单 熟悉终端操作的同学会很舒适 缺点：ssh管理不便 不熟悉终端操作的同学有点困难 

PS：iOS上有一款很强大的软件叫Remote Desktop Manager 有ssh、VNC、RDP等等功能非常方便 

[Minecraft1.7.10(Forge版网页后台)下载](https://cloud.189.cn/t/QBZJBbQrYVjm)  （访问码：dxe1）

这个后台版是我打包好的，已经有一个世界添加了一个城市地图，和一些基本mod。特殊mod添加了一个半条命mod、食品mod、家具mod、载具mod、海洋世界mod等。也可以跟着后面的教程自己配置

优点：网页后台 管理方便 几乎不需要终端操作 缺点：配置有一点点复杂

[基本Mods下载](https://cloud.189.cn/t/UZvYr2IjqqI3)（访问码：07su）

[Windows客户端下载](https://cloud.189.cn/t/iIFNNzBvMNvy)（访问码：iy66）

[Linux客户端下载](https://cloud.189.cn/t/eInYfqiaa6Jv)（访问码：phd4） 

Linux安装我的世界可参考 https://www.sumuze.cn/2020/05/23/Minecraft/

二、搭建环境 

终端管理版：

1.ssh登录 (使用电脑做服务器可以跳过此步骤)

```
ssh root@IP
```

2.安装screen 

```
sudo apt-get install screen
```

介绍：screen是一个可以在多个进程（通常是交互式shell）之间复用一个物理终端的全屏幕窗口管理器。即linux下使用多窗口

常用screen参数

```
screen -S session_name          # 新建一个叫session_name的session
```

```
screen -ls（或者screen -list）   # 列出当前所有的session
```

```
screen -r session_name          # 回到session_name这个session
```

```
screen -d session_name          # 远程detach某个session
```

```
screen -d -r session_name       # 结束当前session并回到session_name这个session
```

进入screen窗口后，想暂时退出（等会还想连接这个screen窗口）

crtl+a+d

退出当前screen窗口，结束当前screen窗口，不想再连接回来（即杀死会话）

exit或者ctrl+d

3.安装open-jdk-8

```
sudo apt install openjdk-8-jre-headless
```

使用vim或nano注释掉最后一行，在最后一行开头添加一个#号

```
sudo vim /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/accessibility.properties
```

4.切换到home目录，新建文件夹

```
cd 

mkdir
```

5.使用scp命令将游戏服务器文件上传到刚刚新建的文件夹 (使用电脑做服务器可以跳过此步骤)

使用方法： (如果使用终端管理会经常使用scp)

(1) 上传本地文件到服务器 

```
scp /path/filename username@servername:/path/
```

例如scp /var/tale.sql root@120.79.212.177:/var/把本机/var/目录下的tale.sql文件上传到120.79.212.177这台服务器上的/var/目录中

(2)从服务器上下载文件

下载文件我们经常使用wget，但是如果没有http服务，如何从服务器上下载文件呢？

```
scp username@servername:/path/filename /local_dir（本地目录）
```

例如scp root@120.79.212.177:/var/test.txt /web把120.79.212.177上的/var/test.txt 的文件下载到/web（本地目录）

(3)从服务器下载整个目录

```
scp -r username@servername:/var/www/remote_dir/（远程目录） /var/www/local_dir（本地目录）
```

例如:scp -r root@120.79.212.177:/var/www/test /var/www/

(4)上传目录到服务器

```
scp -r local_dir username@servername:remote_dir
```

例如：scp -r test root@120.79.212.177:/var/www/ 把当前目录下的test目录上传到服务器的/var/www/ 目录

6.切换到刚刚新建的文件夹，解压游戏服务器压缩包

```
tar -xvf file.tar
```

7.切换到游戏服务器文件夹，使用vim或nano编辑Minecraft.sh文件(如果没有使用touch命令自行创建)

```
sudo vim Minecraft.sh

java -Xms1024M -Xmx1024M -jar minecraft_server.1.7.10.jar nogui  (大小自行分配)
```

保存后使用chmod命令赋予权限

```
sudo chmod +x Minecraft.sh
```

8.使用vim命令编辑server.properties文件

server-port 为游戏服务器端口号默认是25565 可自己更改

server-ip 为游戏服务器IP，一般留空即可。以电脑作为服务器可设置为127.0.0.1

white-list 为游戏服务器白名单，true表示开启，false默认为关闭，建议开启

这三项较为关键，其他几项可在 https://minecraft-zh.gamepedia.com/Server.properties 中参考查阅

9.Sakura frp (非电脑做服务器可跳过)

Sakura frp是一个简单免费的内网穿透网站，注册后点击创建隧道，根据提示完成创建。 PS：建议使用TCP IP填写127.0.0.1 端口填写server.properties文件中的即可

下载Sakura frp的可执行文件到本机电脑或服务器上，使用chmod命令赋予权限

```
sudo chmod +x filesname
```

10.阿里云安全组 (电脑做服务器可跳过)

阿里云需要额外配置一下安全组，貌似腾讯云等不需要。

格式：

网卡类型：内网

规则方向：入方向

授权策略：允许

协议类型：自定义TCP

端口范围：25565/25565 填你自己的游戏服务器端口

优先级：1

授权类型：IPv4地址段访问

授权对象：0.0.0.0/0

保存即可

到此步骤就已经完成了终端管理版的所有环境配置

网页管理版：步骤和终端管理版一致，但需要一些额外的配置

1.安装node.js

```
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash 

apt-get install -y nodejs
```

输入 node -v 测试下是否输出版本号，有输出则安装成功

2.安装McServerManager (可以下载我已经打包好的也可以自己搭建，使用打包文件可以跳过此步骤)

```
git clone https://github.com/Suwings/MCSManager.git  #使用git把MCSM拉下来
```

```
cd MCSManager  #进入目录

npm install --production  #开始安装
```

3.阿里云安全组

也和终端管理版一样，但需要在出方向新添加一条规则

格式：

网卡类型：内网

规则方向：出方向

授权策略：允许

协议类型：全部

端口范围：-1/-1

优先级：1

授权类型：IPv4地址段访问

授权对象：0.0.0.0/0

到此步骤就已经完成了网页管理版的所有环境配置


三、运行游戏服务器

终端管理版：

1.使用screen命令新建终端会话，防止关闭终端后结束进程

```
screen -S name
```

2.运行启动脚本

在游戏服务器目录下输入

```
./Minecraft.sh
```

添加op 白名单 权限等命令在此输入即可

Ctrl a+d 返回 使用screen再新建一个会话，启动frp

```
sudo screen -S name2

./frpc_linux_amd64
```

网页管理版：

1.和终端管理版的步骤1相同

2.切换到MCSManager目录 输入命令启动服务

```
sudo node app.js 

或 

sudo npm start
```

打开浏览器输入你的IP:23333进入后台登录页面  (23333是默认端口可修改)

管理员账户为：#master  初始密码为：123456

进入后台根据引导创建服务器，可使用它给的服务器目录也可以自己选择，服务端文件名填游戏文件夹里的.jar文件名称即可，然后将游戏服务器文件夹上传或剪切到刚刚的服务器目录

保存后启动服务器 完成

---

注：如果终端管理版或者网页管理版正常配置后无法启动游戏服务器，可关闭防火墙

在Ubuntu中 我们使用sudo ufw enable命令来开发防火墙 通过sudo ufw status命令查看开启防火墙后的状态为active 说明防火墙开启成功。

也可以使用sudo ufw allow 端口号 sudo ufw delete allow 端口号开启/关闭相应的端口

MCSManager服务启动失败或者显示端口占用可以用lsof -i查看端口使用情况 kill -9 PID 结束占用


我的世界服务器常用命令:

/op add name

/op delete name

/whitelist add name

/whitelist remove name

/gamerule keepInventory true 死亡不掉落

/manuaddp 用户名 essentials.* 给权限

/gamerule mobGriefing false 生物破坏

doFireTick：启用/禁用火的蔓延

doMobLoot：启用/禁用生物掉落物

doMobSpawning：启用/禁用生物生成(刷怪蛋和刷怪箱不受影响)

doTileDroPS：启用/禁用方块被坏时掉落物品(包括TNT破坏的)

mobGriefing：启用/禁用怪物对方块的破坏(TNT不在此限)

naturalRegeneration：启用/禁用自然生命恢复

doDaylightCycle：启用/禁用日夜交替(关闭的话时间会停止)。

参考文档：

https://minecraft-zh.gamepedia.com/Server.properties (Server.properties的参考文档)

https://minecraft-zh.gamepedia.com/%E5%91%BD%E4%BB%A4/gamerule (gamerule的参考文档)

https://bbs.mcplugin.cn/thread-626-1-1.html (这个是将箱子变成刷一个随机物品的奖励箱mod的使用指南)