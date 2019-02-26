---
layout: post
title: ArchLinux安装及配置
categories: Arch
description: ArchLinux安装及配置
keywords: Arch, Linux
---

## <center>ArchLinux安装及配置</center>

### 【系统安装篇】 

#### 磁盘空间准备  
<span style="color:#ff0000;">注意：安装单系统的跳过此步骤(例如：我)</span>   
Windows：参考[如何删除磁盘分区](https://jingyan.baidu.com/article/1876c852b7f88f890a13765e.html);空出一块空闲空间即可  
Linux：使用[`fdisk`](http://www.runoob.com/linux/linux-comm-fdisk.html)命令，简单粗暴
  
#### 启动盘制作  
首先去163的镜像源下载镜像文件[archlinux-2019.01.01-x86_64.iso       ](http://mirrors.163.com/archlinux/iso/2019.01.01/)  
**Windows制作方式:** 推荐使用[usbwriter](https://sourceforge.net/projects/usbwriter/)这款轻量级的工具  
**Linux制作方式:**  
```bash
fdisk -l   #查看U盘设备(例如：/dev/sdb)
dd if=archlinux-2019.01.01-x86_64.iso  of=/dev/sdb bs=1440k     #使用dd命令制作启动盘
```
#### 修改BIOS引导  
参考百度经验——[联想小新笔记本设置U盘启动教程](https://zhinan.sogou.com/guide/detail/?id=316512718960)  
在BIOS Setup中的Security选项卡中把Secure Boot设置为Disable  
在Boot选项卡中把Boot Mode改成Legac Support；Boot Priority改成UEFI First  
<span style="color:#ff0000;">注意：如果没有UEFI选项，请选择BIOS引导(请记住你的引导方式：本教程采用UEFI方式)</span>      
保存退出重启  
<center>【BIOS引导界面】</center>  
  
![BIOS](https://img.vim-cn.com/20/1663b6d3931b86638ffba3a551c2efb44313b8.gif)  
    
<center>【UEFI引导界面】</center>
  
![UEFI](https://img.vim-cn.com/d1/b60cac95a1b029dfc4076d4d7d1796adb072c3.gif)  
  
当屏幕上出现命令行提示符及闪烁的光标时即启动完毕  
![](https://img.vim-cn.com/c9/8526fbde50ec9d88220dc3c03bd52aa6676671.gif)  
  
由于我之前系统是Ubuntu，所以Boot Menu上还有Ubuntu的引导按钮，需要[删除启动菜单多余选项](https://jingyan.baidu.com/article/67508eb426c8ad9ccb1ce445.html)  
  
#### 再次确认引导方式  
在命令提示符下执行 
```bash
ls /sys/firmware/efi/efivars    #如果出现大量信息，则为UEFI引导
```
如果出现`ls: cannot access '/sys/firmware/efi/efivars': No such file or directory
`则为BIOS引导  
  
#### 检查网络连接  
虚拟机环境下可能是有网络的  
```bash
ping www.baidu.com  #看是否出现连续的ip信息
```
![ping](https://img.vim-cn.com/94/cd38bc6a84fcaeceb62cecac2d4b7d4c9bccfc.gif)  
  
如果没有出现连续信息，那就是网络不通  
#### 连接网络  
方法一：使用 WiFi 连接，请使用 wifi-menu 命令  
```bash
wifi-menu   #选择wifi，输入密码连接
```
方法二：使用 ADSL 宽带连接  
```bash
pppoe-setup     #配置
systemctl start adsl    #连接ADSL
```
  
#### 更新系统时间  
```bash
timedatectl set-ntp true    #设置时钟
```
  
####  修改源文件  
```bash
vim /etc/pacman.d/mirrorlist    #使用vim编辑源文件
```
在没有`#`注释的第一行添加一下内容，输入`:wq`保存退出  
```bash
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch   #使用中科大的源
```
使修改后的源文件生效  
```bash
pacman -Sy
```
  
#### 分区  
<span style="color:#ff0000;">注意：分区操作涉及到SSD和机械硬盘，务必想清楚再下手</span>  
这里，我先介绍我目前磁盘情况  
> 128G固态硬盘+500G机械硬盘
  
查看目前硬盘情况  
```bash
lsblk   #查看硬盘情况  
```
结果如下：  
```bash
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0       7:0    0 476.7M 1 loop /rn/archiso/sfs/airootfs      #iso镜像
sda           8:0    0 465.8G  0 disk   #465.8G的机械硬盘sda 
└─sda1        8:1    0 465.8G  0 part /mnt/home #sda下的第一个分区sda1
sdb           8:16   1  14.6G  0  disk  #16G的启动盘
├─sdb1    8:17    1  588M  0 part  /runarchiso/bootmnt
└─sda2    8:18    1  64M  0  part
nvme0n1     259:0    0 119.2G  0 disk   #119.2G固态硬盘nvme0n1 
├─nvme0n1p1 259:1    0   100G  0 part /mnt  #nme0n1第一个分区nvme0n1p1
└─nvme0n1p2 259:2 0 19.2G   0  part  /mnt/boot/efi  #nme0n1第二个分区nvme0n1p2
```
首先，我们确定要格式化的硬盘有机械硬盘`sda`和固态硬盘`nvme0n1`  
那么，目前不论俩个硬盘下的分区如何，都需要被格式化掉  
假设俩个硬盘格式化好了，那么如何来分区呢？  
  
推荐以下分区方案  
> 固态硬盘(nvme0n1) ：119.2GB分三个区  
> nvme0n1p1  100GB  Linux Filesystem  根/分区  
> nvme0n1p2  9.2GB  EFI System  EFI分区  
> nvme0n1p3  10GB  Linux Swap   交换区  
> 
> 机械硬盘(sda)：465.8GB分一个区就好  
> sda1 465.8GB Linux Filesystem  HOME分区  
  
虚拟机用户只有一个盘就不需要机械硬盘分区方案  
接下来，就要开始撸命令啦  
```bash
cfdisk /dev/nvme0n1  #对固态硬盘进行分区
```
使用delete按钮删除原来的分区，然后重新new新分区，分区大小按照上面方案设置，在Type中选择分区类型(例如：EFI System)，new出三个分区后，选择write按钮，输入yes保存，选择Quit按钮退出  
```bash
cfdisk /dev/sda #对机械硬盘分区
```
方法和方案同上，分好可以通过`lsblk`查看分区情况确认无误  
  
#### 格式化分区  
<span style="color:#ff0000;">注意：格式化是对硬盘分区操作的,不是硬盘，别看错了</span>  
```bash
mkfs.ext4 /dev/nvme0n1p1  #格式化根分区，类型为ext4
mkfs.vfat /dev/nvme0n1p2  #格式化EFI分区，类型为vfa
mkswap -f /dev/nvme0n1p3 #格式化Swap分区
swapon /dev/nvme0n1p3  #使用swapon命令将Swap分区打开
mkfs.ext4 /dev/sda1  #格式化HOME分区，类型为ext4
```
  
#### 挂载分区  
```bash
mount /dev/nvme0n1p1 /mnt  #根分区挂载到/mnt

mkdir /mnt/home  #创建home文件夹
mkdir -p /mnt/boot/EFI   #创建boot文件夹及其子文件夹EFI

mount /dev/sda1 /mnt/home  #HOME分区挂载到/mnt/home
mount /dev/nvme0n1p2 /mnt/boot/EFI  #EFI分区挂载到/mnt/boot/EFI
```
挂载好可以输入`lsblk`查看下挂载结果  
```bash
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda           8:0    0 465.8G  0 disk 
└─sda1        8:1    0 465.8G  0 part /home
nvme0n1     259:0    0 119.2G  0 disk 
├─nvme0n1p1 259:1    0   100G  0 part /
├─nvme0n1p2 259:2    0   9.2G  0 part /boot/EFI
└─nvme0n1p3 259:3    0    10G  0 part [SWAP]
```
  
#### 安装系统  
```bash
pacstrap /mnt base    ​#安装base组件包到/mnt
pacstrap /mnt base-devel    ​#安装base-devel 开发组件包到/mnt
```

#### 分区挂载情况写入到fstab中  
```bash
genfstab -U /mnt >> /mnt/etc/fstab 
vim /mnt/etc/fstab  #检查一下
```
  
#### 切换到安装的系统  
```bash
arch-chroot /mnt    #进入/mnt下的系统
```
这时候，我们就要对/mnt下安装的新系统进行必要的配置了  
  
#### 设置时间  
```bash
ln -sf /usr/share/zoninfo/Asia/Shanghai /etc/localtime  #设为上海时间
hwclock --systohc --utc     #设置硬件时间
```
  
#### 修改编码格式  
由于新系统没有vim，所以要安装一下  
```bash
pacman -S vim   #安装vim
```
使用vim编辑locale.gen文件  
```bash
vim /etc/locale.gen
```
找到`en_US.UTF-8 UTF-8`和`zh_CN.UTF-8 UTF-8`，把前面的`#`号删除，保存退出  
```bash
locale-gen  #重新生成locale
echo LANG=en_US.UTF-8 > /etc/locale.conf  #生成locale.conf文件
cat /etc/locale.conf    #查看一下
```
#### 创建主机名  
```bash
echo Arch > /etc/hostname   #这里的Arch就是自定义的主机名
vim /etc/hosts  #编辑hosts文件
```
添加以下配置：  
```bash
127.0.0.1   localhost.localdomain   localhost
::1                 localhost.localdomain   localhost
127.0.1.1   Arch.localdomain    Arch
```
  
#### 安装网络连接组件  
无线网络组件：
```bash
pacman -S iw wpa_supplicant dialog  #无线网络
```
后面重启系统后可以使用`wifi-menu`命令连网  
  
有线网络组件：  
```bash
systemctl enable dhcpcd     #进入系统自动连网
systemctl start dhcpcd      #重启后执行此命令启动网络服务
```
  
其他网络组件：
```bash
pacman -S rp-pppoe pppoe-setup 
systemctl start adsl    #启动ADSL
```
  
#### 设置ROOT用户  
```bash
passwd  #设置root密码
```
  
#### 安装Intel-ucode(CPU非Intel跳过)  
```bash
pacman -S intel-ucode
```
#### EFI/GPT引导  
```bash
pacman -S grub efibootmgr   #安装grub与efibootmgr两个包
grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=ArchLinux #部署grub
grub-mkconfig -o /boot/grub/grub.cfg    #生成配置文件
```
  
#### 重启  
```bash
exit    #退出新系统
umount -R /mnt #取消/mnt挂载
reboot #重启
```
  
  
### 【桌面安装篇】  
通过上面的操作，不出意外的话，Arch是成功装好了(U盘可以拔了)，但还是命令行界面，接下来我们就要为Arch安装桌面以及配置一些必要参数  
#### 添加用户  
如果这里不添加，安装完桌面后，登录界面没有用户出现(root用户不出现在登录窗口)  
```bash
useradd -m -g users -s /bin/bash teaper #添加teaper用户，用户名你自定义
passwd teaper #为teaper用户设置密码
```
配置用户sudo权限  
```
vim /etc/sudoers #编辑配置文件
```
在`root ALL = (ALL) ALL`下添加`teaper ALL = (ALL) ALL`;输入`:wq!`强制保存退出vim  
  
#### 安装显卡驱动  
```bash
lspci | grep VGA    #查看显卡型号(例如：Intel Corporation HD Graphics 530)
00:02.0 VGA compatible controller: Intel Corporation HD Graphics 530 (rev 06)   
lspci | grep 3D     #查看独显型号(例如：NVIDIA Corporation GM107M [GeForce GTX 950M])
01:00.0 3D controller: NVIDIA Corporation GM107M [GeForce GTX 950M] (rev a2)    #(rev a2) 表示正在运行，如果是ff则未运行


pacman -S 驱动包名字
```
参照下图根据你的显卡类型，选择相应驱动包  
```bash
xf86-video-resa   #——通用的
xf86-video-intel    #——Intel
xf86-video-nouveau  #——Geforce7
xf86-video-304xx #——Geforce6/7
```
![显卡类型](https://teaper.github.io/media/img/blog/vga.png)  
如果你还有独显`NVIDIA`可以使用,可以一并安装`pacman -S xf86-video-intel nvidia lib32-nvidia-utils`  
  
<span style="color:#fc572f;">**双显卡配置**</span>  
参考博客——[【Arch双显卡问题】](https://dclunatic.github.io/2018/09/10/%E7%8B%AC%E6%98%BE/)
  
> 什么情况需要配置双显卡？  
> 系统中有 Intel 集显和 NVIDIA 独显  
> 观察BIOS中的启用的[显卡模式](http://iknow.lenovo.com/detail/dc_102471.html) `Graphic Device` 和上方命令运行结果有没有出现集显/独显出现`(rev ff)`未运行情况。例如我的BIOS中的显卡模式有 `UMA only`（只用集显）和默认 `Discrete`（集显独显分开运行）；上方命令结果也没有出现 ` (rev ff)`显卡未启动的情况， 所以不需要进行下方双显卡配置
  
使用Linux下第三方程序 `Bumblebee` 来实现类似于显卡模式 `ptimus` （同时使用Intel VGA及Nvidia VGA）的功能；它通过 `virtualGL` 或者 `primus` 来实现独显渲染，集显显示的功能；独立显卡在空闲的时候会被禁用掉
  
安装命令`pacman -S bumblebee mesa bbswitch`；其中 `bumblebee` 提供是主要程序实现的包，`mesa` 提供开源的 `OpenGL` 实现，而 `bbswtich` 负责切换显卡  
  
> 如果你的系统是 32 位的，那么你需要启用 `Multilib`，要安装 `lib32-virtualgl` 和 `lib32-nvidia-utils` 或 `lib32-nvidia-340xx-utils` 来和 64 位的对应,具体参照上图  
  
修改配置文件 `/etc/bumblebee/bumblebee.conf`，将 `Driver` 的值设置为 `nvidia`，来让其使用 `nvidia` 驱动，其次将 `PMMethod` 的值设置为 `bbswitch`，让它使用刚刚安装的 `bbswitch` 来进行显卡的切换。修改配置文件 `/etc/modprobe.d/bbswitch.conf`，添加 `options bbswitch load_state=0 unload_state=0 ` 来设置 `bbswitch` 的状态。使用命令 `modprobe bbswitch` 来加载这个模块  
  
将要使用 `Bumblebee` 的用户添加到 `bumblebee` 组中，`gpasswd -a username bumblebee`，并将 `bumblebeed` 服务设为开机启动，`systemctl enable bumblebeed`，启用它 `systemctl start bumblebeed`  
  
#### 安装X窗口系统  
```bash
pacman -S xorg #安装xorg
pacman -S xf86-input-synaptics #安装触摸板驱动
pacman -S ttf-dejavu wqy-microhei   #安装文泉驿米黑字体
pacman -S ttf-dejavu wqy-zenhei wqy-microhei    #安装常用字体
```
  
####  安装Gnome桌面  
```bash
pacman -S gnome  #安装gnome
pacman -S gnome-tweak-tool #gnome桌面优化工具
pacman -S alacarte  #桌面菜单编辑器

systemctl enable gdm    #启用gnome窗口管理器服务
systemctl enable NetworkManager     #启用网络管理

reboot #重启
```
  
  
### 【桌面美化篇】
至此，Arch已经有图形界面了，不过有些丑，还需要配置一下  
在设置->设备->键盘处配置快捷键   
```
名称                      命令                         快捷键
终端            gnome-terminal          Super+R
系统监视器  gnome-system-monitor    Ctrl+Alt+Backspace
文件管理    nautilus                        Super+E
```
  
使用快捷键打开系统监视器，右击进程`gnome-shell`选择 > Change Priority > Very High，将`gnome-shell`进程优先级设为最高  
  
由于gnome3右键没有创建文件的快捷菜单，所以需要手动在`~/Templates`目录下创建模板文件，下次我们右击创建文件的时候，类似于从该文件夹下复制文件，所以你可以把一些编程模板也加进去  
```bash
cd ~/Templates  #进入模板文件夹

gedit text #创建简单文本
gedit markdown.md   #Markdown文件
```
除了简单文本文档，最好在其他新建的脚本内加上一行头代码，例如:在 markdown.md 中加入 `# markdown`，js 文件中加入 `var a = 0`，Python 文件中加入 `import os`
  
为了更好操作，先切换成root用户,当然homia  
```bash
su  #切换root用户
```
#### 更新源  
```bash
vim /etc/pacman.conf    #编辑源文件
```
在末尾追加以下内容：  
```bash
[archlinuxcn]
SigLevel = Optional TrustAll
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch

[multilib]
Include = /etc/pacman.d/mirrorlist      #不添加此项无法安装32位依赖程序
```
使修改后的源文件生效  
```bash
pacman -Sy
```
  
#### 安装AUR软件包    
```bash
sudo pacman -S yaourt    #可以使用aur中的软件，使用方法同pacman一样
sudo pacman -S archlinuxcn-keyring  #安装archlinuxcn-keyring 包以导入 GPG key
```
  
#### 安装Git及SSH  
```bash
sudo pacman -S git #安装git
git config --global user.name "Your Name"   #配置git用户名
git config --global user.email "youremail@example.com"      #配置git邮箱
git config --list   #查看

sudo pacman -S openssh   #安装ssh服务
ssh-keygen -t rsa -C "youremail@example.com"    #一路按Enter
cd ~/.ssh   #进入.ssh隐藏文件夹
gedit id_rsa.pub       #打开公钥并复制id_rsa.pub文件中的内容
```
进入你自己的[github](https://github.com) 
进入Settings > SSH and GPG keys > New SSH key  
新建一个key，名字随便，把复制的内容粘贴上去  
  
配置一些git命令别名  
```bash
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.br branch

git config --global alias.psm 'push origin master'
git config --global alias.plm 'pull origin master'

git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative"
```
  
#### 安装yay  
```bash
cd /opt
git clone https://aur.archlinux.org/yay.git
cd yay
sudo chmod a＋w /opt/yay
makepkg -si
```
  
#### 下载numix图标  
```bash
yaourt numix-circle
```
选择第一个安装，注意导入PGP KEY选择Y；然后打开Tweak->外观->icons中选择你觉得好看的  
#### 安装主题  
```bash
mkdir ~/.themes     #创建主题隐藏文件夹
cd ~/.themes #进入主题文件夹
git clone https://gitlab.com/ZhuHan/GNOME-OSX-II-Theme.git  #克隆主题文件
```
进入Tweak->扩展中启用Application menu；再到窗口标题栏选项卡中启用maximize和minimize以及修改称Left按钮  
  
#### 安装Dock  
```bash
yaourt dash-to-dock
```
选择第一个，安装成功后进入tweak的extension启用
  
#### 安装oh-my-zsh  
```bash
pacman -S zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
右击终端->配置文件首选项，进行颜色配置  
  
修改[ZSH主题](https://github.com/robbyrussell/oh-my-zsh/wiki/External-themes)  
```bash
ls ~/.oh-my-zsh/themes   #列出所有主题
gedit ~/.zshrc  #编辑ZSH配置文件
```
修改`ZSH_THEME="mortalscumbag"`为你想要的主题`mortalscumbag.zsh-theme`  
  
有些主题会出现字符乱码，需要下载主题[修补字体](https://github.com/powerline/fonts)  
```bash
git clone git@github.com:powerline/fonts.git --depth = 1 
cd fonts    #进入这个字体文件夹
./install.sh    #安装字体
```
其他主题：[alien](https://github.com/eendroroy/alien)<span style="color:#ff0000;">(强烈推荐)</span>  
```bash
cd ~/.oh-my-zsh/themes  #进入主题文件夹

git clone https://github.com/eendroroy/alien.git
cd alien
git submodule update --init --recursive

gedit ~/.zshrc #编写配置文件
```
在`~/.zshrc`中`source $ZSH/oh-my-zsh.sh`后一行添加`source ~/.oh-my-zsh/themes/alien/alien.zsh`就可以看到效果了
  
此外,alien 提供了多种不同的配色方案(默认为蓝色)  
只需要在`~/.zshrc`中的`ZSH_THEME="mortalscumbag"`后一行添加以下任意一行
```bash
export ALIEN_THEME="blue"  #蓝色
export ALIEN_THEME="green" #绿色
export ALIEN_THEME="red"    #红色
export ALIEN_THEME="soft"   #淡蓝色
export ALIEN_THEME="gruvbox"    #灰褐色
```

### 【软件安装篇】  
#### 谷歌浏览器  
```bash
pacman -S google-chrome #直接安装chrome
```
不过安装之后不能使用flash,所以  
```bash
pacman -S flashplugin   #flash插件
```
  
#### 火狐浏览器  
```bash
pacman -S firefox   #安装火狐浏览器
```
  
#### 安装搜狗输入法  
由于搜狗拼音输入法依赖于Fcitx，在安装搜狗拼音输入法之前，需要先行安装Fcitx  
```bash
pacman -S fcitx
pacman -S fcitx-configtool
pacman -S fcitx-gtk2 fcitx-gtk3 fcitx-qt4 fcitx-qt5
 
pacman -S fcitx-sogoupinyin
```
用文本编辑器打开~/.xprofile;在其末尾添加以下几行:  
```bash
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
# fcitx &后请替换为你的桌面环境的启动命令
fcitx & gnome-session
```
若你使用的桌面环境比较特殊，可能需要在`/etc/environmenet`后方也加入  
```bash
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```
  
#### 互联网小飞机
富强、民主、文明、和谐、自由、平等、公正、法治、爱国、敬业、诚信、友善  
下载[electron-ssr-0.2.4.pacman ](https://github.com/erguotou520/electron-ssr/releases)
```bash
pacman -U electron-ssr-0.2.4.pacman #安装本地包
```
启动后提示安装SSR，确定安装，然后配置服务器信息，如果没有出现配置界面，使用编辑器编辑`~/.config/electron-ssr/gui-config.json`文件手动设置参数  
```json
{
	"configs": [
		{
			"enable": true,
			"group": "",
			"id": "D02C9C64178F5BAD5F5009DFACC33907",
			"method": "aes-256-cfb",  #加密方式
			"obfs": "plain",  #混淆方式
			"obfsparam": "",
			"password": "pawwsord",  #密码
			"protocol": "auth_sha1_v4",   #协议
			"protocolparam": "",
			"remarks": "",
			"remarks_base64": "",
			"server": "139.175.195.66",  #服务器
			"server_port": 2334   #远程端口
		}
	],
	"index": 0,
	"enable": true,
	"autoLaunch": false,
	"shareOverLan": false,
	"localPort": 1080,
	"ssrPath": "/home/teaper/.config/electron-ssr/shadowsocksr/shadowsocks",
	"pacPort": 2333,
	"sysProxyMode": 1,  #代理方式(1：PAC代理 2：全局 3：不代理)
	"serverSubscribes": [],
	"httpProxyEnable": true,
	"globalShortcuts": {
		"toggleWindow": {
			"key": "Ctrl+Shift+W",
			"enable": true
		},
		"switchSystemProxy": {
			"key": "",
			"enable": false
		}
	},
	"windowShortcuts": {
		"toggleMenu": {
			"key": "Ctrl+Shift+B",
			"enable": true
		}
	},
	"httpProxyPort": 12333,
	"autoUpdateSubscribes": true,
	"subscribeUpdateInterval": 24
}
```
  
#### 网易云音乐  
```bash
pacman -S netease-cloud-music
```
  
#### 蓝牙配置  
之前我们只安装了网络工具，没有配蓝牙驱动，现在安装一下  
```bash
pacman -S bluez bluez-utils bluez-firmware pulseaudio-bluetooth pavucontrol pulseaudio-alsa #全装
```
> bluez软件包提供蓝牙协议栈  
> bluez-utils软件包提供bluetoothctl工具  
> pulseaudio-bluetooth则为bluez提供了PulseAudio音频服务,若没有安装则蓝牙设备在配对 完成后,连接会失败,提示  
> pavucontrol则提供了pulseaudio的图形化控制界面  
> pulseaudio-alsa(可选)则使pulseaudio和alsa协同使用，之后就可以用alsamixer来管理蓝牙音频了    
  
启动蓝牙服务  
```bash
systemctl enable bluetooth
systemctl start bluetooth
```
启动pulseaudio服务(使用非root执行此命令)  
```bash
pulseaudio -k                   # 确保没有pulseaudio启动
pulseaudio --start              # 启动pulseaudio服务
```
将用户加入lp用户组  
```bash
usermod -a -G lp $USER
```
默认情况下，蓝牙仅为 lp 用户组中的用户启用 bnep0 设备。如果想要加入蓝牙系统，需确认已将用户加入该组。可以修改/etc/dbus-1/system.d/bluetooth.conf文件中相应的组配置来实现  
现在就可以使用系统中自带的蓝牙进行连接，连接之后  
  
配置蓝牙  
> 启动bluetoothctl交互命令.可以输入 help 列出所有有效的命令  
> 
> 输入 power on 命令打开控制器电源。默认是关闭的  
> 输入 devices 命令获取要配对设备的 MAC 地址  
> 如果设备未在清单中列出，输入 scan on 命令设置设备发现模式  
> 输入 agent on 命令打开代理  
> 输入 pair $MAC 开始配对(支持 tab 键补全)  
> 如果使用无 PIN 码设备，再次连接可能需要手工认证。输入 trust $MAC 命令  
> 用 connect $MAC 命令建立连接  
  
以下为操作示例
```bash
[teaper@Arch ~]$ bluetoothctl 
Agent registered
[CHG] Device 5C:7E:6A:EF:81:CB RSSI: -52
[CHG] Device 5C:7E:6A:EF:81:CB RSSI: -30
[小爱音箱-9708]# agent KeyboardOnly 
Agent is already registered
[CHG] Device B8:98:F7:5D:8F:54 RSSI: -86
[CHG] Device B8:98:F7:5D:8F:54 RSSI: -77
[小爱音箱-9708]# default-agent 
Default agent request successful
[小爱音箱-9708]# scan on
Discovery started
[CHG] Device B8:98:F7:5D:8F:54 RSSI: -85
[CHG] Device B8:98:F7:5D:8F:54 RSSI: -76
[CHG] Device 5C:7E:6A:EF:81:CB RSSI: -52
[CHG] Device B8:98:F7:5D:8F:54 RSSI: -85
[CHG] Device 5C:7E:6A:EF:81:CB RSSI: -30
[CHG] Device B8:98:F7:5D:8F:54 RSSI: -76
[CHG] Device B8:98:F7:5D:8F:54 RSSI: -86
[CHG] Device B8:98:F7:5D:8F:54 RSSI: -76
[CHG] Device B8:98:F7:5D:8F:54 RSSI: -85
[CHG] Device B8:98:F7:5D:8F:54 RSSI: -76
[CHG] Device B8:98:F7:5D:8F:54 RSSI: -84
[小爱音箱-9708]# pair B8:98:F7:5D:8F:54
Attempting to pair with B8:98:F7:5D:8F:54
[CHG] Device B8:98:F7:5D:8F:54 Connected: yes
Request confirmation
[agent] Confirm passkey 853334 (yes/no): yes
Failed to pair: org.bluez.Error.AuthenticationFailed
[CHG] Device B8:98:F7:5D:8F:54 Connected: no
[CHG] Device B8:98:F7:5D:8F:54 RSSI: -76
[CHG] Device B8:98:F7:5D:8F:54 RSSI: -85
[NEW] Device 4D:87:74:CB:11:18 小爱音箱-9708
[CHG] Device 4D:87:74:CB:11:18 RSSI: -52
[小爱音箱-9708]# connect B8:98:F7:5D:8F:54
Attempting to connect to B8:98:F7:5D:8F:54
Failed to connect: org.bluez.Error.Failed
[CHG] Device 4D:87:74:CB:11:18 RSSI: -30
[CHG] Device B8:98:F7:5D:8F:54 RSSI: -76
[小爱音箱-9708]# 
```
设置自动启动蓝牙  
将 `/etc/bluetooth/main.conf `最后的 `AutoEnable` 值修改为 `true`
```bash
vim /etc/bluetooth/main.conf        #使用root用户修改，：wq!强制保存退出
```
指定使用蓝牙音频输出  
通过 `pavucontrol `的`Playback`和`Pecording`标签页重定向音频的输入和输出
    
#### 安装TIM/QQ
```bash
sudo pacman -S deepin.com.qq.office  #TIM
sudo pacman -S deepin.com.qq.im  #QQ
```
  
#### 安装微信  
```bash
yay -S deepin-wechat  #一路回车
```
  
#### 安装多线程下载工具
```bash
pacman -S wget
pacman -S axel  
```
  
#### BaiduPCS  
百度网盘shell版，开玩笑的，下载请戳[BaiduPCS-Go-v3.5.6-linux-amd64.zip
](https://github.com/iikira/BaiduPCS-Go/releases)  
提取到本地  
```bash
mv BaiduPCS-Go-v3.5.6-linux-amd64 /opt/
cd /opt/BaiduPCS-Go-v3.5.6-linux-amd64
./BaiduPCS-Go   #终端运行该文件

login -bduss=BDUSS值   #使用BDUSS登录
```
BDUSS值获取方式戳[这里](https://github.com/iikira/BaiduPCS-Go/wiki/%E5%85%B3%E4%BA%8E-%E8%8E%B7%E5%8F%96%E7%99%BE%E5%BA%A6-BDUSS)  
登录成功之后就可以进行下载等操作
```
cd  文件夹名        #切换目录
config set -savedir /目录     #自定义下载目录
d   [文件或目录1]  [文件或目录2]  #下载
exit        #退出BaiduPCS-Go

sudo ln -s /opt/BaiduPCS-Go-v3.5.6-linux-amd64/BaiduPCS-Go /usr/bin/BaiduPCS    #添加到终端命令,输入BaiduPCS即可打开
```
  
#### 录像/直播 
```bash
pacman -S obs-studio    #OBS
```
启动后，obs 会根据你电脑性能自动配置出一套默认设置，但是默认设置下的视频清晰度达不到我们的要求，所以需要在此基础上修改一下  
> 进入 file > settings 打开设置面板，点击左边列表的 Output 选项卡  
> 设置 Video Bitrate 比特率为 `5000`   
> Encoder选择带 `NVENC` 的N卡驱动  
> 勾选 `Enable Advanced Encoder Settings` 复选框  
> 点击左边列表的 Video 选项卡,设置`Output(Scaled)Resolution`输出像素为`1920×1080`  
  
<span style="color:#ff0000;">注意：如果OBS无法录制电脑桌面，请在终端使用`sudo obs`命令启动一次</span>
  
#### Teamviewer远程工具
```bash
yay -S teamviewer
```
启动后如果出现，未就绪，请检查网络连接，则运行  
```bash
sudo systemctl enable --now teamviewerd
```
  
#### 安装WPS  
```bash
yay -S wps-office
```
  
#### 安装jdk  
```bash
yay -S jdk8     #安装jdk8
yay -S jdk11     #安装jdk11
archlinux-java status   #列出所有jdk
sudo archlinux-java set java-11-jdk #切换默认jdk,记得切回来
```
  
#### 安装xmind  
```bash
yay -S xmind        #一路回车
```
如果出现打不开，则编辑`XMind.ini`
```bash
gedit /usr/share/xmind/XMind/XMind.ini
#最后一行--add-modules=java.se.ee修改为
-javaagent:/usr/share/xmind/XMind/XMindCrack.jar
```
保存修改，下载破解jar包[XMindCrack.jar](https://stormxing.oss-cn-beijing.aliyuncs.com/files/XMindCrack.jar)；将其移动到XMind.ini同级目录  
```bash
mv XMindCrack.jar /usr/share/xmind/XMind/
```
打开XMind, 点击帮助——序列号，然后输入以下序列号，邮箱随便填，可以填自己的  
```bash
XAka34A2rVRYJ4XBIU35UZMUEEF64CMMIYZCK2FZZUQNODEKUHGJLFMSLIQMQUCUBX
RENLK6NZL37JXP4PZXQFILMQ2RG5R7G4QNDO3PSOEUBOCDRYSSXZGRARV6MGA33TN2
AMUBHEL4FXMWYTTJDEINJXUAV4BAYKBDCZQWVF3LWYXSDCXY546U3NBGOI3ZPAP2SO
3CSQFNB7VVIY123456789012345
```
#### 安装eclipse  
```bash
yay -S  eclipse-jee     #安装社区版
```
  
#### Maven安装及配置  
```bash
yay -S maven
```
检查一下`~/.m2`文件夹是否存在，不存在则执行  
```bash
mvn help:system     #执行该命令的过程中，会生成~/.m2文件夹,并且会从Maven官网下载必要的依赖包
```
复制settings.xml到`.m2`文件夹  
```bash
cp /opt/maven/conf/settings.xml ~/.m2        
gedit ~/.m2/settings.xml     #配置maven仓库
```
配置信息如下：  
```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
<!-- 本地仓库地址 -->
<localRepository>${user.home}/.m2/repository</localRepository>
 
<!-- 阿里云镜像地址 -->
<mirrors>
    <mirror>
      <id>aliyunmaven</id>
      <mirrorOf>*</mirrorOf>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
</mirrors>
 
<!-- 配置maven的jdk版本 -->
<profile>
    <id>jdk-1.8</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>
 
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
</profile>
```
#### Tomcat安装及配置  
下载[apache-tomcat-9.0.14.tar.gz](https://tomcat.apache.org/download-90.cgi)  
找到压缩包右击提取到此处  
```bash
sudo chmod a+x apache-tomcat-9.0.16          #添加权限
sudo mv apache-tomcat-9.0.16 /opt
cd /opt/apache-tomcat-9.0.16/bin/
sudo chmod 777 *.sh  #给所有.sh文件加777权限(解决Idea中Tomcat权限错误)
sudo gedit /etc/profile     #配置环境变量

export TOMCAT_HOME=/opt/apache-tomcat-9.0.16
export PATH=$TOMCAT_HOME/bin:$PATH

sudo nano /etc/profile   #使环境变量生效
sudo /opt/apache-tomcat-9.0.16/bin/startup.sh   #启动tomcat测试一下
```
测试端口：[ http://localhost:8080/ ]( http://localhost:8080/ )  
  
#### 安装MySQL/MariaDB  
Archlinux的MySQL被称为MariaDB  
```bash
yay -S mariadb      #安装mariadb及其依赖包
```
根据shell提示初始化数据目录  
```bash
sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```
启动MariaDB  
```bash
sudo systemctl start mysqld
```
为root用户设置一个新密码  
```bash
mysqladmin -u root password '123456'    #设置root密码为123456
```
尝试登录以下  
```bash
mysql -u root -p123456
```
如果想要MariaDb开机自动启动，那么就运行以下命令  
```bash
sudo systemctl enable mysqld
```
  
#### 数据库管理神器Dbeaver  
Dbeaver是一个免费的多平台数据库管理工具  
它支持流行的数据库，如MySQL，MariaDB，PostgreSQL，SQLite，Oracle  
```bash
yay -S dbeaver
```
打开软件会出现引导，类型选择`MariaDB`，它会在线下载驱动包，然后填写数据库名`mysql`，用户名及密码即可登录  
  
#### 数据库管理神器Navicat Premium  
**安装**  
Linux下的Navica运行是需要wine的，因为之前安装过deepin-tim，wine已经默认作为依赖包安装好了，除此之外，Navicat安装包中也默认封装了一套wine组件  

找到Linux的Navicat Premium，随便找个下载线路，点击[下载](https://www.navicat.com/en/products)，在Chrome中的下载列表复制下载文件的下载路径`(例如：http://download3.navicat.com/download/navicat121_premium_en_x64.tar.gz)`  
```bash
axel -n 10 http://download3.navicat.com/download/navicat121_premium_en_x64.tar.gz  #使用axel多线程(10线程)下载Navicat安装包
```
右击`navicat121_premium_en_x64.tar.gz`提取到本地  
```bash
sudo mv navicat121_premium_en_x64 /opt
cd /opt/navicat121_premium_en_x64/
gedit start_navicat  #编辑start_navicat文件，解决中文乱码
```
把`export LANG="en_US.UTF-8"` 改为 `export LANG="zh_CN.UTF-8"`可以解决中文系统下乱码问题
```bash
./start_navicat #开始安装，会提示安装wine Mono，点安装
```
打开之后，点左边Trial试用，右边是注册；这时候发现字体好小，进工具 > 选项改字体；找张[图片](https://teaper.github.io/media/img/blog/icon_navicat.png)做LOGO；修改图片名称为icon_navicat.png  
```bash
sudo mv icon_navicat.png /opt/navicat121_premium_en_x64/
gedit navicat.desktop   #创建快捷方式
```
`navicat.desktop`中填入一下内容(注意版本信息相应替换)
```
[Desktop Entry]
Encoding=UTF-8
Name=navicat Premium
Comment=The Smarter Way to manage dadabase
Exec=/bin/sh "/opt/navicat121_premium_en_x64/start_navicat"
Icon=/opt/navicat121_premium_en_x64/icon_navicat.png
Categories=Application;Database;MySQL;navicat
Version=1.0
Type=Application
Terminal=0
```
附加运行权限  
```bash
sudo chmod a+x navicat.desktop  #添加运行权限
sudo mv navicat.desktop /usr/share/applications/  #复制到快捷方式文件夹
```
  
**破解**  
破解方法参考——[【Ubuntu的安装的Navicat正版永久使用方法】](https://yq.aliyun.com/ziliao/5468)  
  
事先约定下环境，在你的文件夹下确认是否拥有这些文件  
> navicat配置文件夹路径：~/.navicat64  
> 原始文件user.reg：~/.navicat64/user.reg #navicat读取配置使用
> 备份文件user_backup.reg: ~/.navicat64/user_backup.reg     #稍后我们自己创建
  
使用创建好的快捷图标打开navicat，设置数据库连接信息,后面要记录信息并存放到创建的备份文件中，到时候修改数据库连接信息就会比较麻烦  
```bash
cd ~/.navicat64/ && ls -a  #进入文件夹
gedit user.reg  #复制里面的内容
gedit user_backup.reg #新建user_backup.reg文件，把user.reg中的内容粘贴进去
```
把每个`[]`后的`数字`和重复性的摸块`(以空行为分隔，重复性的小段代码块)`这样的重复代码块全部删除,保存  
最终效果类似于如下示例：   
```bash
WINE REGISTRY Version 2
;; All keys relative to \\User\\S-1-5-21-0-0-0-1000

#arch=win64

[AppEvents\\Schemes\\Apps\\Explorer\\Navigating\\.Current] 
#time=1d4b06f4ffe9cec
@=""

[Control Panel\\Accessibility\\Blind Access] 
#time=1d4b06cc6848bf4
"On"="0"

[Control Panel\\Accessibility\\Keyboard Preference] 
#time=1d4b06cc684826c
"On"="1"

[Control Panel\\Accessibility\\ShowSounds] 
#time=1d4b06cc6848df2
"On"="0"

[Control Panel\\Colors] 
#time=1d4b06cc68cc918

[Control Panel\\Desktop] 
#time=1d4b06fa2eb5364
"ActiveWndTrackTimeout"=dword:00000000
"BlockSendInputResets"="0"
"CaretWidth"=dword:00000001
"ClickLockTime"=dword:000004b0
"DoubleClickHeight"="4"
"DoubleClickWidth"="4"
"DragFullWindows"="0"
"DragHeight"="4"
"DragWidth"="4"
"FocusBorderHeight"=dword:00000001
"FocusBorderWidth"=dword:00000001
"FontSmoothing"="2"
"FontSmoothingGamma"=dword:00000578
"FontSmoothingOrientation"=dword:00000001
"FontSmoothingType"=dword:00000002
"ForegroundFlashCount"=dword:00000003
"ForegroundLockTimeout"=dword:00000000
"IconTitleWrap"="1"
"LowPowerActive"="0"
"MenuShowDelay"="400"
"UserPreferencesMask"=hex:30,00,00,80,10,00,00,00
"Wallpaper"=""
"WheelScrollChars"="3"
"WheelScrollLines"="3"

[Control Panel\\International] 
#time=1d4b06f8b78a9b6
"iCalendarType"="1"
"iCountry"="86"
"iCurrDigits"="2"
"iCurrency"="0"
"iDate"="2"
"iDigits"="2"
"iFirstDayOfWeek"="6"
"iFirstWeekOfYear"="0"
"iLDate"="2"
"iLZero"="0"
"iMeasure"="0"
"iNegCurr"="2"
"iNegNumber"="1"
"iPaperSize"="9"
"iTime"="1"
"iTimePrefix"="1"
"iTLZero"="0"
"LC_CTYPE"="00000804"
"LC_MEASUREMENT"="00000804"
"LC_MONETARY"="00000804"
"LC_NUMERIC"="00000804"
"LC_PAPER"="00000804"
"LC_TELEPHONE"="00000804"
"LC_TIME"="00000804"
"Locale"="00000804"
"Numshape"="1"
"s1159"="\x4e0a\x5348"
"s2359"="\x4e0b\x5348"
"sCountry"="People's Republic of China"
"sCurrency"="\xffe5"
"sDate"="-"
"sDecimal"="."
"sGrouping"="3;0"
"sLanguage"="CHS"
"sList"=","
"sLongDate"="yyyy'\x5e74'M'\x6708'd'\x65e5'"
"sMonDecimalSep"="."
"sMonGrouping"="3;0"
"sMonThousandSep"=","
"sNativeDigits"="0123456789"
"sNegativeSign"="-"
"sPositiveSign"=""
"sShortDate"="yyyy-M-d"
"sThousand"=","
"sTime"=":"
"sTimeFormat"="H:mm:ss"
"sYearMonth"="yyyy'\x5e74'M'\x6708'"

[Control Panel\\Keyboard] 
#time=1d4b06cc684830c
"KeyboardDelay"="1"
"KeyboardSpeed"="31"

[Control Panel\\Mouse] 
#time=1d4b06cc6848fa0
"ActiveWindowTracking"=dword:00000000
"DoubleClickHeight"="4"
"DoubleClickSpeed"="500"
"DoubleClickWidth"="4"
"MouseHoverHeight"="4"
"MouseHoverTime"="400"
"MouseHoverWidth"="4"
"MouseSensitivity"="10"
"MouseSpeed"="1"
"MouseThreshold1"="6"
"MouseThreshold2"="10"
"SnapToDefaultButton"="0"
"SwapMouseButtons"="0"

[Control Panel\\Sound] 
#time=1d4b06cc6846ed0
"Beep"="Yes"

[Environment] 
#time=1d4b06cc7108bfe
"TEMP"="C:\\users\\teaper\\Temp"
"TMP"="C:\\users\\teaper\\Temp"

[Keyboard Layout\\Preload] 
#time=1d4b06cc68b598e
"1"="00000409"

[Software\\PremiumSoft\\Navicat\\Servers\\MySQL] 
#time=1d4b47a4056b0cc
"AllowInvalidHostName"=dword:00000000
"AltHTTPProxyUserName"=""
"AltHTTPUserName"=""
"AltSSHUserName"=""
"AltUserName"=""
"AskPassword"="false"
"AutoConnect"=dword:00000000
"CACert"=""
"Cipher"=""
"ClientCert"=""
"ClientKey"=""
"ClientKeyPassword"=""
"Codepage"=dword:0000fde9
"Host"="localhost"
"HTTP_Authen"=dword:00000000
"HTTP_CACert"=""
"HTTP_CertAuth"=dword:00000000
"HTTP_ClientCert"=""
"HTTP_ClientKey"=""
"HTTP_EncodeBase64"=dword:00000000
"HTTP_Passphrase"=""
"HTTP_Password"=""
"HTTP_Proxy"=dword:00000000
"HTTP_ProxyHost"=""
"HTTP_ProxyPassword"=""
"HTTP_ProxyPort"=dword:00001f90
"HTTP_ProxyUsername"=""
"HTTP_URL"=""
"HTTP_Username"=""
"HttpProxySavePassword"=dword:00000001
"HttpSavePassword"=dword:00000001
"NamedPipeSocket"=""
"NSYDirtyFlag"=dword:00000000
"NSYHash"=""
"NSYID"=""
"NSYProjectUUID"=""
"NSYSyncHTTPProxyUserName"=dword:00000001
"NSYSyncHTTPUserName"=dword:00000001
"NSYSyncSSHUserName"=dword:00000001
"NSYSyncUserName"=dword:00000001
"PGSSLCRL"=""
"PGSSLMode"="smRequire"
"PingInterval"=dword:000000f0
"Port"=dword:00000cea
"Pwd"="15057D7BA390"
"QuerySavePath"="C:\\users\\teaper\\\x6211\x7684\x6587\x6863\\Navicat\\MySQL\\Servers\\MySQL"
"RootCert"=""
"SaveClientKeyPassword"=dword:00000000
"ServerVersion"=dword:00018729
"ServerVersionStr"="10.1.37-MariaDB"
"ServiceProvider"="spDefault"
"SessionLimit"=dword:00000000
"SSH_AuthenMethod"="saPassword"
"SSH_Host"=""
"SSH_Passphrase"=""
"SSH_Password"=""
"SSH_Port"=dword:00000016
"SSH_PrivateKey"=""
"SSH_SavePassphrase"=dword:00000001
"SSH_SavePassword"=dword:00000001
"SSH_UserName"=""
"UseAdvSettings"="false"
"UseCharacterSet"=dword:00000001
"UseCompression"=dword:00000000
"UseHTTP"=dword:00000000
"UseNamedPipe"=dword:00000000
"UsePingInterval"=dword:00000000
"UserName"="root"
"UseSSH"=dword:00000000
"UseSSL"=dword:00000000
"UseSSLAuthen"=dword:00000000
"VerifyCACert"=dword:00000000
"WeakCertValidation"=dword:00000000

[Software\\PremiumSoft\\NavicatPremium] 
#time=1d4b479b4fb125c
"AlreadyShowNavicateV121WelcomeScreen"=dword:00000001
"FirstFileAssociation"=dword:00000001

[Software\\Wine\\Fonts] 
#time=1d4b479b3066690
"Codepages"="936,936"
"LogPixels"=dword:00000060

[Software\\Wine\\Fonts\\External Fonts] 
#time=1d4b479b3066690

[Software\\Wine\\MenuFiles] 
#time=1d4b479b40c2732
```
  
创建开机自动替换文件的脚本  
```bash
gedit reset_navicat.sh
```
内容如下,`teaper`是我的系统用户名,你自己替换为你自己的  
```bash
#!/bin/sh
#！每次开机删除user.reg
rm /home/teaper/.navicat64/user.reg
#！将备份脚本中的内容换成navicat读取的配置
cp /home/teaper/.navicat64/user_backup.reg /home/teaper/.navicat64/user.reg
```
给文件添加权限  
```bash
sudo chmod a+x reset_navicat.sh
/home/teaper/.navicat64/reset_navicat.sh    #24小时后运行一下试试有没有推迟试用时间
```
以后增加数据库链接，只需在user.reg中把新增加的数据库链接添加到user_backup.reg文件中即可  
  
**添加开机自启服务**  
如果24小时候运行`reset_navicat.sh`没问题，就可以把`reset_navicat.sh`添加到开机自启的service中  
```bash
cd /etc/systemd/system  #进入文件夹
su #切换为root用户
gedit autonavicat.service #创建autonavicat.service文件
```
`autonavicat.service`中内容如下  
```bash
[Unit]
Description=autonavicat 
[Service]
ExecStart=/home/teaper/.navicat64/reset_navicat.sh
[Install]
WantedBy=multi-user.target
```
添加权限  
```bash
chmod 644 autonavicat.service
```
设置开机自启  
```bash
systemctl start autonavicat.service  #启动一下试试
systemctl enable autonavicat.service #添加到开机自启服务
```
  
#### 安装IntelliJ IDEA  
```bash
yay -S intellij-idea-ultimate-edition
```
> ---------------------------------安装配置-------------------------------------  
> Do not import settings　　　ok  
> 拉到最下面　　　Accept  
> Send Usage Statististics  
> username: teapers　　　password:Zxcvbnm9　　　Activate  
> 　　　　　　Next:Desktop Entry  
> 　　　　　　Next:Launcher Script  
> 自定义路径　　　Next:Default plugins  
> 
> ---------------------------------功能选择--------------------------------------  
> Java Frameworks: Hibernate Spring Struts JavaEE Velocity  
> Build Tools: Maven  
> Web Developent: HTML JavaScript CSS  
> Version Controls: Git GitHub  
> Test Tools: JUnit  
> Application Servers: Tomcat and TomEE  
> Clouds : DisableAll (一个都不要)  
> Swing: Disable  
> Android: Disable  
> Other Tools: UML  
> 
> 　　　　　　Next:Featured plugins  
> 　　　　　　Start using IntelliJ IDEA  
  
破解方法，在`/etc/hosts`文件中加入`0.0.0.0 account.jetbrains.com`，在idea中加入下方激活码  
```
K71U8DBPNE-eyJsaWNlbnNlSWQiOiJLNzFVOERCUE5FIiwibGljZW5zZWVOYW1lIjoibGFuIHl1IiwiYXNzaWduZWVOYW1lIjoiIiwiYXNzaWduZWVFbWFpbCI6IiIsImxpY2Vuc2VSZXN0cmljdGlvbiI6IkZvciBlZHVjYXRpb25hbCB1c2Ugb25seSIsImNoZWNrQ29uY3VycmVudFVzZSI6ZmFsc2UsInByb2R1Y3RzIjpbeyJjb2RlIjoiSUkiLCJwYWlkVXBUbyI6IjIwMTktMDUtMDQifSx7ImNvZGUiOiJSUzAiLCJwYWlkVXBUbyI6IjIwMTktMDUtMDQifSx7ImNvZGUiOiJXUyIsInBhaWRVcFRvIjoiMjAxOS0wNS0wNCJ9LHsiY29kZSI6IlJEIiwicGFpZFVwVG8iOiIyMDE5LTA1LTA0In0seyJjb2RlIjoiUkMiLCJwYWlkVXBUbyI6IjIwMTktMDUtMDQifSx7ImNvZGUiOiJEQyIsInBhaWRVcFRvIjoiMjAxOS0wNS0wNCJ9LHsiY29kZSI6IkRCIiwicGFpZFVwVG8iOiIyMDE5LTA1LTA0In0seyJjb2RlIjoiUk0iLCJwYWlkVXBUbyI6IjIwMTktMDUtMDQifSx7ImNvZGUiOiJETSIsInBhaWRVcFRvIjoiMjAxOS0wNS0wNCJ9LHsiY29kZSI6IkFDIiwicGFpZFVwVG8iOiIyMDE5LTA1LTA0In0seyJjb2RlIjoiRFBOIiwicGFpZFVwVG8iOiIyMDE5LTA1LTA0In0seyJjb2RlIjoiR08iLCJwYWlkVXBUbyI6IjIwMTktMDUtMDQifSx7ImNvZGUiOiJQUyIsInBhaWRVcFRvIjoiMjAxOS0wNS0wNCJ9LHsiY29kZSI6IkNMIiwicGFpZFVwVG8iOiIyMDE5LTA1LTA0In0seyJjb2RlIjoiUEMiLCJwYWlkVXBUbyI6IjIwMTktMDUtMDQifSx7ImNvZGUiOiJSU1UiLCJwYWlkVXBUbyI6IjIwMTktMDUtMDQifV0sImhhc2giOiI4OTA4Mjg5LzAiLCJncmFjZVBlcmlvZERheXMiOjAsImF1dG9Qcm9sb25nYXRlZCI6ZmFsc2UsImlzQXV0b1Byb2xvbmdhdGVkIjpmYWxzZX0=-Owt3/+LdCpedvF0eQ8635yYt0+ZLtCfIHOKzSrx5hBtbKGYRPFDrdgQAK6lJjexl2emLBcUq729K1+ukY9Js0nx1NH09l9Rw4c7k9wUksLl6RWx7Hcdcma1AHolfSp79NynSMZzQQLFohNyjD+dXfXM5GYd2OTHya0zYjTNMmAJuuRsapJMP9F1z7UTpMpLMxS/JaCWdyX6qIs+funJdPF7bjzYAQBvtbz+6SANBgN36gG1B2xHhccTn6WE8vagwwSNuM70egpahcTktoHxI7uS1JGN9gKAr6nbp+8DbFz3a2wd+XoF3nSJb/d2f/6zJR8yJF8AOyb30kwg3zf5cWw==-MIIEPjCCAiagAwIBAgIBBTANBgkqhkiG9w0BAQsFADAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBMB4XDTE1MTEwMjA4MjE0OFoXDTE4MTEwMTA4MjE0OFowETEPMA0GA1UEAwwGcHJvZDN5MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxcQkq+zdxlR2mmRYBPzGbUNdMN6OaXiXzxIWtMEkrJMO/5oUfQJbLLuMSMK0QHFmaI37WShyxZcfRCidwXjot4zmNBKnlyHodDij/78TmVqFl8nOeD5+07B8VEaIu7c3E1N+e1doC6wht4I4+IEmtsPAdoaj5WCQVQbrI8KeT8M9VcBIWX7fD0fhexfg3ZRt0xqwMcXGNp3DdJHiO0rCdU+Itv7EmtnSVq9jBG1usMSFvMowR25mju2JcPFp1+I4ZI+FqgR8gyG8oiNDyNEoAbsR3lOpI7grUYSvkB/xVy/VoklPCK2h0f0GJxFjnye8NT1PAywoyl7RmiAVRE/EKwIDAQABo4GZMIGWMAkGA1UdEwQCMAAwHQYDVR0OBBYEFGEpG9oZGcfLMGNBkY7SgHiMGgTcMEgGA1UdIwRBMD+AFKOetkhnQhI2Qb1t4Lm0oFKLl/GzoRykGjAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBggkA0myxg7KDeeEwEwYDVR0lBAwwCgYIKwYBBQUHAwEwCwYDVR0PBAQDAgWgMA0GCSqGSIb3DQEBCwUAA4ICAQC9WZuYgQedSuOc5TOUSrRigMw4/+wuC5EtZBfvdl4HT/8vzMW/oUlIP4YCvA0XKyBaCJ2iX+ZCDKoPfiYXiaSiH+HxAPV6J79vvouxKrWg2XV6ShFtPLP+0gPdGq3x9R3+kJbmAm8w+FOdlWqAfJrLvpzMGNeDU14YGXiZ9bVzmIQbwrBA+c/F4tlK/DV07dsNExihqFoibnqDiVNTGombaU2dDup2gwKdL81ua8EIcGNExHe82kjF4zwfadHk3bQVvbfdAwxcDy4xBjs3L4raPLU3yenSzr/OEur1+jfOxnQSmEcMXKXgrAQ9U55gwjcOFKrgOxEdek/Sk1VfOjvS+nuM4eyEruFMfaZHzoQiuw4IqgGc45ohFH0UUyjYcuFxxDSU9lMCv8qdHKm+wnPRb0l9l5vXsCBDuhAGYD6ss+Ga+aDY6f/qXZuUCEUOH3QUNbbCUlviSz6+GiRnt1kA9N2Qachl+2yBfaqUqr8h7Z2gsx5LcIf5kYNsqJ0GavXTVyWh7PYiKX4bs354ZQLUwwa/cG++2+wNWP+HtBhVxMRNTdVhSm38AknZlD+PTAsWGu9GyLmhti2EnVwGybSD2Dxmhxk3IPCkhKAK+pl0eWYGZWG3tJ9mZ7SowcXLWDFAk0lRJnKGFMTggrWjV8GYpw5bq23VmIqqDLgkNzuoog==
```
> 激活码失效，戳[这里](http://idea.lanyus.com/)  
> 配置IDEA，戳[这里](https://www.jetbrains.com/help/idea/guided-tour-around-the-user-interface.html)  
  
#### 安装虚拟机 VMware  
```bash
yay vmware  #选择vmware-workstation 15.0.2-1 输入编号，一路回车
```
安装比较慢，不要以为是卡住了，安装成功启动输入以下序列号  
```bash
GV7N2-DQZ00-4897Y-27ZNX-NV0TD
```
  
#### 安装虚拟机 Virtualbox  
```bash
sudo pacman -S virtualbox  
正在解决依赖关系...  
      :: 有 2 个软件包可提供 VIRTUALBOX-HOST-MODULES ：  
      :: 软件库 community  
       1) virtualbox-host-dkms  2) virtualbox-host-modules-arch  

      输入某个数字 ( 默认=1 ): 2  
```
执行上面安装命令后，有两个选择，按照[Arch wiki](https://wiki.archlinux.org/index.php/VirtualBox_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))的解释：  
如果在用默认的 linux 内核，建议安装virtualbox-host-modules-arch  
如果用了其它的内核，需要安装 virtualbox-host-dkms  
  
第一种virtualbox-host-dkms安装：输入1的时候,等待安装完成  
```bash
sudo pacman -S virtualbox   #再安装virtualbox
sudo pacman -S linux-headers    #安装Linux头文件内核
```
  
第二种virtualbox-host-modules-arch安装：输入2的时候，也就是我目前安装的方式  
```bash
sudo pacman -S virtualbox   #也要安装virtualbox
```
  
<span style="color:#ff0000;">注意：如果再创建虚拟机后启动虚拟机时报错，大意是： 不是模块没有加载，就是有权限许可问题，用 /sbin/vboxconfig 解决问题！但是，在 /sbin 目录里面根本没有这条命令，那么可以使用下面这条命令重新加载模块</span>  
```bash
sudo vboxreload
```
  
#### 安装 Visual Studio Code  
```bash
yay vscode
```
选择一个下载量最高的`aur/visual-studio-code-bin 1.30.1-1 (+781 28.27%)  `，输入编号，一路回车  
> <center>插件列表</center>
>
> [Material](#)　　　#一款冷门主题    
> [One Dark Pro](#)　　　#源于Atom的主题  
> [Power Mode](#)　　　#打字泡沫  
> 
> [vscode-icon](#)　　　#树目录加上图标  
> [Path Intellisense](#)　　　#默认路径补全  
> [Document this](#)　　　#快速注释  
> [Project Manager](#)　　　#多个项目切换  
> [vscode-fileheader](#)　　　#顶部注释模板  
> [filesize](#)　　　#底部显示文件大小  
> 
> [open-in-browser](#)　　　#打开浏览器插件  
> [view-in-browser](#)     #打开浏览器插件  
>  
> [Atuo Rename Tag](#)　　　#同时修改html标签首
  
#### 安装Node.js及GitBook  
安装Node.js及npm包  
```bash
yay -S nodejs npm       #安装
  
node -v     #查看node.js版本
npm -v      #查看npm版本

sudo npm config set registry https://registry.npm.taobao.org    #更换npn镜像为淘宝镜像
sudo npm config list    #查看下配置是否生效
```
安装gitbook  
```bash
sudo npm install gitbook-cli -g      #使用npm安装,前提是你先安装了Node.js
```
gitbook使用方式可以使用`gitbook -help`命令查看  
  
安装PDF输出  
```bash
sudo -v && wget -nv -O- https://download.calibre-ebook.com/linux-installer.sh | sudo sh /dev/stdin  #安装PDF阅读器calibre-ebook
sudo npm install gitbook-pdf -g  #安装gitbook pdf插件
```
使用 `gibook pdf` 命令输出为PDF文件  
  
评论插件——[【disqus】](https://plugins.gitbook.com/plugin/disqus)  
    
### 【Mac美化篇】
#### 窗口主题
主题一：[VimixDark-Gtk-Theme](https://github.com/vinceliuice/vimix-gtk-themes)(推荐)  
下载之后解压，进入目录有一个shell执行文件，进入终端输入安装命令：  
```bash
./Vimix-installer.sh
```
安装成功，可以在gnome-tweak-tool中的外观处选择主题  
  
主题二：[Xenlism极简主义](https://github.com/xenlism/minimalism)
```bash
gedit /etc/pacman.conf  #编辑源文件
```
将这些行添加到文件中：  
```bash
[xenlism-arch]
SigLevel = Optional TrustAll
Server = http://downloads.sourceforge.net/project/xenlism-minimalism/repo/arch
```
运行命令：  
```bash
sudo pacman -Syyu
sudo pacman -Sy xenlism-minimalism-theme
```
<span style="color:#ff0000;">注意：改主题安装成功之后，记得把`/etc/pacman.conf`中的内容删除，否则会引起数据源异常，无法安装后面的aur软件</span>  
  
主题三：其他Mac主题[Mac OS Mojave ](https://github.com/vinceliuice/Mojave-gtk-theme);安装方法同上，下载之后解压到`~/.themes`中即可
  
 
#### 图标主题  
主题一：[La Capitaine](https://github.com/keeferrourke/la-capitaine-icon-theme)
```bash
cd /usr/share/icons
git clone https://github.com/keeferrourke/la-capitaine-icon-theme.git
```
gnome-tweak-tool中的外观处选择图标  
  
主题二：MacOS[图标](https://git.opendesktop.org/umayanga/Cupertino-macOS-iCons)
```bash
cd /usr/share/icons
git clone https://git.opendesktop.org/umayanga/Cupertino-macOS-iCons.git
```
  
####  安装gnome-shell-extensions
```bash
pacman -S gnome-shell-extensions
 sudo pacman -S chrome-gnome-shell
```
然后在谷歌商店直接搜`Gnome Shell Integration`进行安装，需要更多美化插件，可以通过[Gnome Shell Integration](https://extensions.gnome.org)下载和安装  
> [状态栏天气插件](https://extensions.gnome.org/extension/750/openweather/)配置[坐标](https://lbs.qq.com/tool/getpoint/index.html)  
> [状态栏系统监测插件](https://extensions.gnome.org/extension/120/system-monitor/)
  
#### GRUB主题  
GRUB 是什么？GRUB 是引导程序，负责引导操作系统，开机时那个选择系统的画面  
  
主题一：[Breeze GRUB2主题](https://www.opendesktop.org/p/1000140/)  
下载主题包并解压  
```bash
sudo mv grub2-theme-breeze-5.13.1 /boot/grub/themes/    #解压文件夹内的 breeze 文件夹复制到 /boot/grub/themes/
cd /boot/grub/themes/grub2-theme-breeze-5.13.1/breeze/  #观察是否有theme.txt文件

sudo gedit /etc/default/grub     #修改该文件
```
将`#GRUB_THEME="/path/to/gfxtheme"`改为`GRUB_THEME="/boot/grub/themes/grub2-theme-breeze-5.13.1/breeze/theme.txt"`  
最后还要重新生成grub.cfg文件才能让背景或者主题生效  
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
  
主题二：[蛴螬主题vimix](https://www.opendesktop.org/p/1009236/)  
下载主题包并解压  
```bash
sudo ./Install  #使用root用户运行里面的安装脚步
```
  
### 【游戏安装篇】 
#### 安装Steam  
安装steam之前，确保已经安装了32位的N卡驱动`lib32-nvidia-utils`，否则会出现无法启动问题
```bash
sudo pacman -S steam    #安装steam
```
另外，由于我教程的`/home`目录属于机械硬盘，机械盘不适合运行软件，所以需要在`/opt`目录新建`steam`文件夹  
```bash
sudo mkdir /opt/steam   #创建文件夹
sudo chmod a+w /opt/steam   #加权限
```
然后在启动steam之后，进入steam设置 > 下载 > steam库文件中添加`/opt/steam`路径，并且右击设置为默认路径，以后在steam中下载的游戏就会自动进入该文件夹  
> STEAM 游戏推荐  
> [Northgard](https://store.steampowered.com/app/466560/Northgard/)（北境之地）  
> [Dota 2](https://store.steampowered.com/app/570/Dota_2/)  
> [文明 VI](https://store.steampowered.com/app/289070/Sid_Meiers_Civilization_VI/)  

