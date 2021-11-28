---
title: ArchLinux安装
date: 2021-11-15 19:00:00
categories: Linux
tags: 
    - ArchLinux
    - Linux
---

Archlinux yyds！全面的wiki，强大的AUR，值得拥有！

首先通过安装介质启动到live环境。选第一个

![boot选项](https://klpages-imgs.oss-cn-qingdao.aliyuncs.com/img/boot%E9%80%89%E9%A1%B9.jpg)

会以root身份进入一个虚拟控制台中，默认的shell是zsh。

### 验证引导模式

`ls /sys/firmware/efi/efivars` 如果结果显示了目录且没有报告错误，则系统是以 UEFI 模式引导的。

![启动方式确认](https://klpages-imgs.oss-cn-qingdao.aliyuncs.com/img/%E5%90%AF%E5%8A%A8%E6%96%B9%E5%BC%8F%E7%A1%AE%E8%AE%A4.jpg)
### 连接到因特网

#### 判断无线网卡是否被锁

```shell
# rfkill list  
--------------
0: phy0: Wireless LAN
	Soft blocked: yes
	Hard blocked: yes
```

如果出现以上内容，可以调节网卡开关打开它。如果没有开关，那就使用以下命令：

```shell
# rfkill unblock wifi
```

#### 连接网络

```shell
$ iwctl   //会进入联网模式
[iwd]# help    //可以查看帮助
[iwd]# device list    //列出你的无线设备名称，一般以wlan0命名
[iwd]# station <device> scan    //扫描当前环境下的网络
[iwd]# station <device> get-networks    //会显示你扫描到的所有网络
[iwd]# station <device> connect <network name>
password:输入密码
[iwd]# exit    //退出当前模式，回到安装模式
```

测试网络是否连通：

```
# ping baidu.com
```

### 更新为国内镜像源

`reflector --country China --age 72 --sort rat.e --protocol ave /etc/pacman.d/mirrorlist`
已将最新的镜像源更新为国内的，保存在/etc/pacman.d/mirrorlist目录下

### 更新系统时间

` timedatectl set-ntp true ` 之后可以使用 ` timedatectl status `检查服务状态

### 通过ssh链接当前主机（可选）

可以通过ssh在另外一台可以正常使用的设备上进行安装，方便命令复制

1. 在当前安装环境下输入`passwd` 为当前环境设置一个密码，不用太长，两三位就可以了。输入时不会显示。

2. 执行`ip -brief address`查看当前ip地址，一般是192.168.<...>.<...> IP地址后面斜杠之后的掩码位不用哦

3. 准备另外一台设备，应当使该设备与安装主机在同一局域网内，就是俩设备都连着同一个WiFi

   几种设备的连接方式：

   ​	Windows：在终端输入：`ssh -o StrictHostKeyChecking=no root@<刚刚查看的IP地址>`

   ​	Linux&macOS：在终端输入：`ssh -o StrictHostKeyChecking=no root@<刚刚查看的IP地址>`

   ​	当然也可以使用Android和ios设备，这里不说了，因为那玩意太小了，用来输入命令实在鸡肋


### 磁盘分区

安装archlinux所需分区

```
EFI分区		300 MB
swap分区		4GB
root分区		剩余空间
```

使用cfdisk工具分区 `cfdisk <install disk name >` 比如我的： `cfdisk /dev/sda` 。之后会进入如下界面，选择gpt分区表：

![分区表类型](https://klpages-imgs.oss-cn-qingdao.aliyuncs.com/img/%E5%88%86%E5%8C%BA%E8%A1%A8%E7%B1%BB%E5%9E%8B.jpg)
之后就开始正式分区了，首先EFI分区，点击new新建：

![新建分区new](https://klpages-imgs.oss-cn-qingdao.aliyuncs.com/img/%E6%96%B0%E5%BB%BA%E5%88%86%E5%8C%BAnew.jpg)
这里输入300M，之后回车，就回到上面的界面了。

![EFI分区300M](https://klpages-imgs.oss-cn-qingdao.aliyuncs.com/img/EFI%E5%88%86%E5%8C%BA300M.jpg)
在建立下一个分区之前，先对第一个EFI分区的类型做一个修改，选择type选项

![更改EFI分区类型](https://klpages-imgs.oss-cn-qingdao.aliyuncs.com/img/%E6%9B%B4%E6%94%B9EFI%E5%88%86%E5%8C%BA%E7%B1%BB%E5%9E%8B.jpg)
重复之前的步骤，建立swap分区和root分区，完成之后如下图：

![分完区之后](https://klpages-imgs.oss-cn-qingdao.aliyuncs.com/img/%E5%88%86%E5%AE%8C%E5%8C%BA%E4%B9%8B%E5%90%8E.jpg)
### 格式化分区

#### EFI分区格式化

```
mkfs.vfat /dev/sda1
```

#### root分区格式化

```
mkfs.xfs -f /dev/sda3
# 强制分区为xfs
```

#### 创建swap分区

```
mkswap /dev/sda2
```

可以使用`lsblk -f`  查看磁盘分区情况

### 挂载分区

```
mount /dev/sda3 /mnt
mkdir -p /mnt/boot/efi
mount /dev/vda1 /mnt/boot/efi
swapon /dev/sda2
lsblk -f    ## 查看分区g情况 
```

### 安装系统

```
pacstrap /mnt linux linux-firmware linux-headers base base-devel vim git bash-completion
```

### 生成文件系统的表文件

```
genfstab -U /mnt >> /mnt/etc/fstab
```
```
cat /mnt/etc/fstab
```

### 进入新系统

```
arch-chroot /mnt
```

### 设置时区

```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
```
hwclock --systohc
```

### 本地化

#### 设置系统语言

```
vim /etc/locale.gen
```

将以下两行取消注释(删除前面的井号)

```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
```

#### 生成本地语言信息

```
locale-gen
```

#### 设置本地语言环境变量
```
/etc/locale.conf
-------------------------
LANG=en_US.UTF-8
```

### 网络配置

#### 主机设置

```
vim /etc/hostname
---------------------
archlinux(也可以写成你想要的主机名)
```

#### 生成对应的hosts

```
vim /etc/hosts
--------------------
127.0.0.1	localhost
::1			localhost
127.0.1.1	archlinux.localdomain archlinux   # 这里的archlinux是主机名
```

### 安装相关包

```
pacman -S grub efibootmgr efivar networkmanager intel-ucode
```

### 配置grub

```
grub-install /dev/sda
```
```
grub-mkconfig -o /boot/grub/grub.cfg
```

### 激活启用NetworkManager

```
systemctl enable NetworkManager
```

### 给root用户建立密码

```
passwd
输入密码
```

## 重启

```
exit

umount /mnt/boot/efi
umount /mnt

reboot
```

## 基本配置

### 再次联网

输入`nmtui` 选择 “Activate a connection” 回车进入，选择你需要的网络，连接后back返回即可

### 再次ssh连接

#### 安装openssh

```
pacman -S openssh
```

#### 启动ssh服务

```
systemctl start sshd
```

#### 查看当前ip地址

```
ip -brief address
```

#### 修改ssh配置

ssh默认无法连接root用户，需要修改其配置文件使其支持

```
vim /etc/ssh/sshd_config
--------------------------------------
# 将下列的语句值改为yes
PermitRootLogin yes
```

![sshd_config_p](https://klpages-imgs.oss-cn-qingdao.aliyuncs.com/img/sshd_config_p.jpg)

#### 在其他端进行连接

```
ssh -o StrictHostKeyChecking=no root@<刚刚查看的IP地址>
```

### 配置bash shell环境变量

```
vim /etc/skel/.bashrc
-------------------------------
# 在alias行上面添加
export EDITOR=vim

# 继续添加
alias grep='grep --color=auto'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'

# 在alias 之后添加脚本
[ ! -e ~/.dircolors ] && eval $(dircolors -p > ~/.dircolors)
[ -e /bin/dircolors ] && eval $(dircolors -b ~/.dircolors)
# 保存退出
```

随后将上述文件复制到home目录下

```
cp /etc/skel/.bashrc ~
```

### 添加标准用户(以下klelee是我的用户名)

```
# 添加用户
useradd --create-home klelee
# 设置密码
passwd klelee
```

#### 设置用户组

```
usermod -aG wheel,users,storage,power,lp,adm,optical klelee
```

#### 修改当前用户权限

```
visudo
---------------------------------
# 取消注释以下行
%wheel ALL=(ALL) ALL
```

### 添加ArchLinuxCN 存储库

该仓库是由archlinux中文社区驱动的一个非官方的软件仓库。我们使用的很多软件都需要使用这个库去下载，比如typora。

```
# 编辑/etc/pacman.conf
vim /etc/pacman.conf
--------------------------------------
# 在最后添加
[archlinuxcn]
.Server = c.edu.cn/archlinuxcn/$arch   
# 这是<img src="">中科大的源，你也可以选择清华、阿里等，当我推荐中科大，因为我喜欢
```

然后更新GPG密钥

```
pacman -S archlinuxcn-keyring
```

**注** ： 如果以上更新密钥步骤出现错误，就是那种连着一串ERROR的情况，请执行以下步骤

```
# rm -rf /etc/pacman.c/gnupg
# pacman-key --init
# pacman-key --populate archlinux archlinuxcn
# pacman -Syy
```

### 显卡驱动
```
pacman -S xf86-video-intel vulkan-intel mesa
```

### 声卡配置
```
# pacman -S alsa-utils pulseaudio pulseaudio-bluetooth cups
```

## 图形界面

### 显示服务
```
pacman -S xorg
```

### 安装字体
#### 英文字体
```
pacman -S ttf-dejavu ttf-droid ttf-hack ttf-font-awesome otf-font-awesome ttf-lato ttf-liberation ttf-linux-libertine ttf-opensans ttf-roboto ttf-ubuntu-font-family
```
#### 中文字体
```
pacman -S ttf-hannom noto-fonts noto-fonts-extra noto-fonts-emoji noto-fonts-cjk adobe-source-code-pro-fonts adobe-source-sans-fonts adobe-source-serif-fonts adobe-source-han-sans-cn-fonts adobe-source-han-sans-hk-fonts adobe-source-han-sans-tw-fonts adobe-source-han-serif-cn-fonts wqy-zenhei wqy-microhei
```

#### 打开字体引擎
```
vim /etc/profile.d/freetype2.sh
--------------------------------------------
# 取消注释最后一句
export FREETYPE_PROPERTIES="truetype:interpreter-version=40"
```

### 安装桌面环境（KDE）

#### KDE

```
pacman -S plasma sddm konsole dolphin kate ark okular spectacle yay
```

plasma：就是桌面环境

sddm：登录管理器

konsole：kde下的终端

kate：文本编辑器

ark：解压与压缩

okular：PDF查看器

spectacle：截图工具

AUR：管理工具

#### 设置sddm登录

```
systemctl enable sddm
```

## 常用软件

### 中文输入法

```
# sudo pacman -S fcitx fcitx-im fcitx-configtool
```

```
yay -S fcitx-sogoupinyin
```

```
vim ~/.xprofile
-------------------------------
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

重启，这时候会看到系统托盘会有一个键盘的图标，我已经配置过了，这里显示的是sogou的图标

![fcitx](https://klpages-imgs.oss-cn-qingdao.aliyuncs.com/img/fcitx.jpg)

右击那个图标，点击configure,在配置界面点加号

![fcitx_configure](https://klpages-imgs.oss-cn-qingdao.aliyuncs.com/img/fcitx_configure.jpg)
去掉“只显示当前语言”的选项，拉倒最下面选择sogoupinyin，之后回到上面的页面，选择美式键盘，删掉即可

![fcitx](https://klpages-imgs.oss-cn-qingdao.aliyuncs.com/img/fcitx.jpg)
### 其他软件

```
sudo pacman -S typora visual-studio-code-bin netease-cloud-music
yay -S baidunetdisk-electron google-chrome qv2ray 
```

更多软件可以去wiki寻找。

[arc.hlinux wiki](%E4%BD%93%E4%B8%AD%E6%96%87)
[List of a.pplications](%E4%BD%93%E4%B8%AD%E6%96%87)

### 清理缓存

```
pacman -Scc
```

