本文档是主要是btrfs文件系统的archlinux+Gnome环境的搭建

1. [手动安装](#手动安装)
2. [脚本安装](#脚本安装)
3. [配置系统](#配置系统)
4. [美化](#美化)
5. [笔记本显卡切换和电源管理](#显卡切换)
6. [KVM虚拟机](#KVM虚拟机)
7. [显卡直通](#显卡直通)
8. [在linux上玩游戏](#在linux上玩游戏)
9. [性能优化](#性能优化)
10. [删除linux](#删除linux)
11. [issues](#issues)
12. [附录](#附录)
13. [参考资料](#参考资料)
## vim基础操作
i 键进入编辑模式
esc 退出编辑模式
:q 冒号小写q，退出
:w 冒号小写w，写入
:wq 冒号小写wq保存并退出

## 双系统安装后时间错乱，windwos开机磁盘检查

参考链接：
[双系统时间同步-CSDN博客](https://blog.csdn.net/zhouchen1998/article/details/108893660)

管理员打开powershell 运行
```
Reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
```

# 安装系统
## 手动安装
参考链接：
[archlinux 简明指南](https://arch.icekylin.online/)

[安装指南 - Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97)



### 确认网络
```
ip a 查看网络连接信息
ping bilibili.com 确认网络正常
```
#### iwctl连接wifi
```
iwctl
```
```
station wlan0 connect <wifiname>
```
```
exit
```

### 同步时间
```
timedatectl set-ntp true 
```

### 硬盘分区
```
lsblk -pf  查看当前分区情况
fdisk -l 小写字母l，查看详细分区信息
```
```
cfdisk /dev/nvme0n1 选择自己要使用的硬盘进行分区
```
创建512MB或者1g efi system
其余全部分到一个分区里，类型linux filesystem 


#### 格式化分区
```
lsblk -pf 查看分区情况
fdisk -l 小写字母l，查看详细分区信息
```

- 格式化efi启动分区
```
mkfs.fat -F 32 /dev/efi_system_partition
```
- 格式化btrfs根分区
```
mkfs.btrfs /dev/root_partition
```

#### btrfs子卷

- 挂载
```
mount -t btrfs -o compress=zstd /dev/root_partition /mnt
```

- 创建子卷
```
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@swap
```

- 可选：确认
```
btrfs subvolume list -p /mnt
```

- 取消挂载
```
umount /mnt
```

### 挂载

```
mount -t btrfs -o subvol=/@,compress=zstd /dev/root_partition /mnt #根目录
mount --mkdir -t btrfs -o subvol=/@home,compress=zstd /dev/root_partition /mnt/home #/home目录
mount --mkdir -t btrfs -o subvol=/@swap,compress=zstd /dev/root_partition /mnt/swap #/swap目录
mount --mkdir /dev/efi_partition /mnt/boot #/boot目录
mount --mkdir /dev/winefi_partition /mnt/winboot #windows的启动分区，为双系统引导做准备
```
```
df -h 复查挂载情况
```

### 安装系统
#### 设置镜像源
##### reflector自动设置
```
reflector -a 48 -c cn -f 10 --sort rate --save /etc/pacman.d/mirrorlist --v

-a（age） 48 指定最近48小时更新过的源
-c（country） cn 指定国家为中国
-f（fastest） 10 筛选出最快的10个
--sort rate 按照下载速度排顺序
--save /etc/pacman.d/mirrorlist 将结果保存到/etc/pacman.d/mirrorlist
--v（verbose） 过程可视化
```
##### 手动设置
```

vim /etc/pacman.d/mirrorlist

拿出手机，浏览器搜索 archlinux中国镜像源，找一个镜像源添加
```

- 更新密钥
```
pacman -Sy archlinux-keyring
```

- 安装系统
```
pacstrap -K /mnt base base-devel linux linux-firmware btrfs-progs

-K 复制密钥
base-devel是编译其他软件的时候用的
linux是内核，可以更换
linux-firmware是固件
btrfs-progs是btrfs文件系统的管理工具
```

- 安装必要的功能性软件
```
pacstrap /mnt networkmanager vim sudo amd-ucode

networkmanager 是联网用的
vim 是文本编辑器
sudo 和权限管理有关
amd-ucode 是微码，用来修复和优化cpu
```

## 设置swap

参考链接：[Swap - ArchWiki](https://wiki.archlinux.org/title/Swap)



创建swap文件

```
btrfs filesystem mkswapfile --size 64g --uuid clear /mnt/swap/swapfile
```

启动swap
```
swapon /mnt/swap/swapfile
```

## 生成fstab文件
```
genfstab -U /mnt > /mnt/etc/fstab
```

## change root

```
arch-chroot /mnt
```

### 主机名

```
vim /etc/hostname
```

### 设置时间和时区
```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
```
hwclock --systohc
```

### 本地化设置
```
vim /etc/locale.gen

取消en_US.UTF-8 UTF-8和zh_CN.UTF-8的注释
```
```
locale-gen
```
```
vim /etc/locale.conf
写入 LANG=en_US.UTF-8
```

### 设置root密码
```
passwd 
```

### 安装引导程序
```
pacman -S grub efibootmgr os-prober

efibootmgr 管理uefi启动项
os-prober 用来搜索win11
```
```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH #此处的id可以自取
```

- 编辑grub的源文件
```
vim /etc/default/grub
```

去掉quiet， loglevel改成5，添加nowatchdog modprobe.blacklist=sp5100_tco，intelcpu用户把sp5100_tco换成iTCO_wdt
手动写入或者取消GRUB_DISABLE_OS_PROBER=false的注释让grub生成其他系统的启动项

- 生成配置文件
```
grub-mkconfig -o /boot/grub/grub.cfg
```

## 完成安装
```
exit 退出changeroot
reboot 重启，会自动取消所有的挂载
```
## 启动网络
```
systemctl enable --now NetworkManager
```

连接wifi
```
nmcli dev wifi connect <wifiname> password <password>

或者使用nmtui这个工具进行连接
```

### 放松一下吧

```
pacman -S fastfetch lolcat cmatrix
```

使用示例： fastfetch | lolcat

## 脚本安装

### 确认网络连接
- ip a 查看网络连接
- ping 一个网址确认网络正常
* 连接wifi
```
iwctl #开启wifi连接
station wlan0 connect wifi名 
在跳出的条目内输入密码
exit退出iwctl
```
* ping一个网址确认网络正常

### 更新archinstall
* 更新archlinux-keyring
```
pacman -Sy archlinux-keyring
```
- 更新archinstall
```
pacman -S archinstall
```
* archinstall开启安装脚本（脚本内上下左右移动光标，回车选中，tab给括号打勾，/ 左斜杠搜索）

### 设置镜像源
* mirror选自己所在地区或国家，也可以不手动选，在optional repositories里开启32位源（multilib）玩游戏或者运行windows程序需要32位支持

### 磁盘分区 disk configuration
#### 启动分区
wiki推荐是1GB，所以填入1024MB，小点也行，类型fat32,挂载点是/boot
#### swap交换空间
swap与虚拟内存和休眠有关，可以创建swap分区或者swap文件，二选一，前者配置更简单，后者配置稍复杂，但是更加灵活。
###### swap分区
  创建一个和内存大小相同的硬盘分区，类型选择swap
###### swap文件
将所有剩余空间分到一个分区，类型选择btrfs,设置compress（透明压缩,可以节省磁盘空间），添加sub volume。
- @ 对应 /

- @home 对应 / 

- @swap 对应 /swap
  交换文件的创建方法：[Swap - ArchWiki](https://wiki.archlinux.org/title/Swap)

  
```
btrfs filesystem mkswapfile --size 4g --uuid clear /swap/swapfile
```
```
swapon /swap/swapfile
```
```
vim /etc/fstab
写入
/swap/swapfile none swap defaults 0 0
```

### 其他
* bootloader 选择grub，因为主流发行版用的都是grub,配置简单，遇到问题时网上帖子多
* 设置root密码
* 设置普通用户，添加管理员权限
* profile可以预装桌面环境，有需要的自行选择，选择桌面之后还可以选择安装显卡驱动
* 内核（kernel）台式机选zen,笔记本用linux
* networ configuration选择第三项networkmanager，因为主流桌面环境默认与这个集成
* additional pakages 里面选vim和os-prober, zen内核的话这里再选个linux-zen-headers
* 设置时区

---

# 配置系统

## 安装桌面环境及必要组件
```
pacman -S gnome-desktop gdm ghostty gnome-control-center gnome-software flatpak
```
```
#gnome-desktop最小化安装gnome
#gdm是显示管理器(gnome display manager)
#ghostty是一个可高度自定义的终端模拟器（terminal emulator)
#gnome-control-center是设置中心
#software和flatpak是软件商城
```

## 创建用户
(archinstall安装的可以跳过)
```
useradd -m -g wheel <username> #不需要输入<>符号
```
* 设置密码
```
passwd <username>
```
* 编辑权限
```
EDITOR=vim visudo
```
* 搜索 wheel，取消注释
```
%wheel ALL=（ALL：ALL） ALL
```
## 安装N卡显卡驱动和硬件编解码
参考链接：[NVIDIA - ArchWiki](https://wiki.archlinux.org/title/NVIDIA)



### 检查头文件
```
sudo pacman -S linux-headers
#linux替换为自己的内核
```
### 安装显卡驱动 

N卡此时如果不安装显卡驱动，可能无法启动桌面环境，此处以4060为例
```
sudo pacman -S nvidia nvidia-utils
```
非stable内核要安装的驱动不一样，具体看wiki，zen内核装nvidia-dkms

#### AMD显卡建议检查是否安装vulkan驱动
```
sudo pacman -S vulkan-radeon 
```
- 混合模式软件还是跑在N卡上的话检查有没有安装vulkan-mesa-layers
参考链接：[gnome-shell uses dgpu instead of igpu : r/gnome](https://www.reddit.com/r/gnome/comments/1irvmki/gnomeshell_uses_dgpu_instead_of_igpu/)

```
sudo pacman -S vulkan-mesa-layers
```
### 硬件编解码
 - nvidia4060
```
sudo pacman -S libva-nvidia-driver
```
- intel xe核显
```
sudo pacman -S intel-media-driver libva
```
* amd 780M
确认安装了libva-mesa-driver
```
sudo pacman -Q libva-mesa-driver
```
* 使用vainfo确认是否安装完成
```
vainfo
```
* 环境变量名（不需要手动设置，只在指定某块gpu时使用）
```
LIBVA_DRIVER_NAME=nvidia #nvidia
LIBVA_DRIVER_NAME=radeonsi #amd
```

* 安装字体
```
sudo pacman -S wqy-zenhei noto-fonts noto-fonts-emoji
```
* 重启激活显卡驱动
```
reboot 
```
* 临时开启GDM
```
sudo systemctl start gdm #即使出了问题重启也能恢复，避免进不了tty的情况
```
* 设置gdm开机自启

桌面环境正常开启后设置开机自启

```
sudo systemctl enable gdm
```
* 可选：开启32位源 (archinstall可以跳过)
```
sudo vim /etc/pacman.conf #编辑pacman配置文件
去掉[multilib]两行的注释
sudo pacman -Syyu #刷新源
```
## 生成home下目录（如果没有的话）
```
xdg-user-dirs-update
```
## 删除或隐藏不必要的快捷方式
```
flatpak install flathub io.github.fabrialberio.pinapp
```
也可以用menulibre，使用pacman安装

想隐藏的图标激活invisible，然后保存

## 安装声音固件和声音服务

- 安装声音固件

```
sudo pacman -S sof-firmware alsa-firmware alsa-ucm-conf
```
- 安装声音服务
```
sudo pacman -S pipewire pipewire-pulse pipewire-alsa pipewire-jack wireplumber
```
* 启用服务
```
systemctl --user enable pipewire pipewire-pulse wireplumber
systemctl --user start pipewire pipewire-pulse wireplumber
```
* 可选：安装GUI
```
sudo pacman -S pavucontrol 
```
## 安装高级网络配置工具nm-connection-editor
```
sudo pacman -S network-manager-applet dnsmasq
```
* 设置跃点
```
启动安装的软件或输入nm-connection-editor
跃点需手动设置为100,默认的-999会导致网络速率异常
```
## 安装yay
- 编辑pacman配置文件
```
sudo vim /etc/pacman.conf
```
- 在文件底部写入以下内容
```
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch 
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch 
Server = https://mirrors.hit.edu.cn/archlinuxcn/$arch 
Server = https://repo.huaweicloud.com/archlinuxcn/$arch 
```
- 安装密钥
```
sudo pacman -Sy archlinuxcn-keyring 
```
- 安装yay
```
sudo pacman -S yay 
```

### 或者从git安装
[GitHub - Jguer/yay: Yet another Yogurt - An AUR Helper written in Go](https://github.com/Jguer/yay)

```
sudo pacman -S git base-devel && git clone https://aur.archlinux.org/yay.git && cd yay && makepkg -si
```

## 自定义安装软件

### 我安装的软件

这是我会安装的，你可以按需求选择
- 安装后没显示图标的话登出一次
- pacman
```
sudo pacman -S mission-center gnome-text-editor gnome-disk-utility gnome-clocks gnome-calculator loupe snapshot baobab showtime fragments file-roller foliate zen-browser zen-browser-i18n-zh-cn gst-plugin-pipewire gst-plugins-good pacman-contrib amberol 
```
```
#mission-center 类似win11的任务管理器
#gnome-text-ditor记事本
#gnome-disk-utility磁盘管理
#gnome-clocks时钟
#gnome-calculator计算器
#loupe图像查看
#snapshot相机，摄像头
#baobab磁盘使用情况分析工具，
#showtime 极度简洁的视频播放器，要强大功能可以用MPV,不推荐使用VLC
#fragments是符合gnome设计理念的种子下载器
#file-roller解压
#foliate 电子书阅读器
#zen-browser zen-browser-i18n-cn 基于firefox的浏览器和cn语言包
#gst-plugin-pipewire gst-plugins-good gnome截图工具自带的录屏，需登出
#pacman-contrib 是pacman的一些小工具
#amberol 音乐播放器

zen浏览器一定要在设置>zen模组里面安装transparent zen模组，可以获得特别流畅的动画效果
```
- qq、微信、wps
```
yay -S linuxqq-appimage wechat-appimage wps-office-cn  
```
- markdown编辑器

```
yay -S typora-free
```

- flathub

```
flatpak install flathub be.alexandervanhee.gradia io.github.Predidit.Kazumi io.gitlab.theevilskeleton.Upscaler com.github.unrud.VideoDownloader io.github.ilya_zlobintsev.LACT xyz.ketok.Speedtest com.geeks3d.furmark com.rafaelmardojai.Blanket
```
```
gradia编辑截图
kazumi追番
upscaler图片超分
video downloader下载youtube 144p～8k视频
LACT 显卡超频、限制功率、风扇控制等等
speedtest 测试网速
furmark 显卡烤鸡
Blanket 白噪音播放器
```
- gradia编辑截图使用方法
可以对截图进行一些简单的添加文字、马赛克、图表、背景之类的操作
设置自定义快捷键的时候命令写：
```
flatpak run be.alexandervanhee.gradia --screenshot=INTERACTIVE
```
我设置了两个截图快捷键，ctrl+alt+a普通系统截图（仿qq截图快捷键），super+shift+s截图并进入编辑界面（仿win截图快捷键）。

### appimage
appimage是一个下载即用、无需安装的文件。需要确认安装了fuse才能运行appimage。

安装appimagelauncher管理appimage软件
```
yay -S appimagelauncher
```
安装后启动appimage时会弹出appimagelauncher的窗口，第一次启动会让你设置安装路径，默认是home目录下的Applications目录。然后让你选择运行一次还是集成到系统。不过有时候会安装失败或者安装之后无法运行。
- 卸载appimage软件
右键快捷方式，点击remove appimage from system，或者手动删除~/.local/share/Applications下的destop文件和安装目录下的appimage文件。
## 设置系统语言
右键桌面选择setting，选择system，选择region&language

如果是archinstall安装，这里只有英文选项，解决办法：

* 本地化设置
```
sudo vim /etc/locale.gen 
```
```
取消zh_CN.UTF-8的注释
```
```
sudo locale-gen
```

## 安装输入法

### fcitx5-rime 雾凇拼音

```
sudo pacman -S fcitx5-im fcitx5-mozc fcitx5-rime rime-ice-pinyin-git
```
```
fcitx5-im 包含了fcitx5的基本包
fcitx5-mozc是开源谷歌日语输入法
fcitx5-rime是输入法引擎
rime-ice-pinyin-git是雾凇拼音输入法
```

- 打开fcitx 5 configuration添加rime和mozc输入法，没有的话登出一次

- 编辑rime的配置文件设置输入法方案为雾凇拼音，如果没有文件夹和文件的话自己创建文件夹，然后运行如下命令

```
vim ~/.local/share/fcitx5/rime/default.custom.yaml 
```

```
写入：

patch:
  # 这里的 rime_ice_suggestion 为雾凇方案的默认预设
  __include: rime_ice_suggestion:/
```

- 商店搜索extension，安装蓝色的extensionmanager
- 安装扩展：input method panel
  https://extensions.gnome.org/extension/261/kimpanel/
- 编辑环境变量

```
sudo vim /etc/environment
```
```
GTK_IM_MODULE=fcitx 
QT_IM_MODULE=fcitx 
XMODIFIERS=@im=fcitx
```
#### wps用不了fcitx5
由于wps自身的问题，我们需要手动设置变量：
- 文字 (Writer): `/usr/bin/wps`
- 表格 (Spreadsheets): `/usr/bin/et`
- 演示 (Presentation): `/usr/bin/wpp`
```
export XMODIFIERS=@im=fcitx 
export QT_IM_MODULE=fcitx 
export GTK_IM_MODULE=fcitx
```


### ibus-rime

[Rime - Arch Linux 中文维基](https://wiki.archlinuxcn.org/zh-hant/Rime)

[可选配置（基础篇） | archlinux 简明指南](https://arch.icekylin.online/guide/advanced/optional-cfg-1#%F0%9F%8D%80%EF%B8%8F-%E8%BE%93%E5%85%A5%E6%B3%95)

[RIME · GitHub](https://github.com/rime)


ibus输入法在gnome的兼容性极佳，无须配置环境变量即可使用，rime可以解决ibus-libpinyin词库垃圾的问题

- 删除fcitx5输入法（记得关闭fcitx5的扩展)

```
sudo pacman -Rns fcitx5-im fcitx5-mozc fcitx5-rime rime-ice-pinyin-git
```

- 安装ibus-rime

```
sudo pacman -S ibus ibus-rime ice-rime-pinyin-git
yay -S ibus-mozc #日语输入法
```

- 在gnome的设置中心 > 键盘 里面搜索rime添加输入法
- 如果之前禁用过系统设置里的打字快捷键的记得恢复

- 编辑配置文件设置输入法为ice

```
vim ~/.config/ibus/rime/default.custom.yaml
```

如果没有的自己创建，mkdir命令创建文件夹，touch命令创建文件

```
patch:
  # 这里的 rime_ice_suggestion 为雾凇方案的默认预设
  __include: rime_ice_suggestion:/
```
- 编辑环境变量
```
sudo vim /etc/environment
```
删除或者注释fcitx5相关的环境变量，gnome使用ibus输入法在正常情况下不需要设置环境变量，如果出了问题可以把fcitx5替换为ibus

- 第一次切换至rime输入法需要等待部署完成
- 出现异常可以登出一次
- 可选：添加萌娘百科词库

```
yay -S rime-pinyin-moegirl

vim /usr/share/rime-data/rime_ice.dict.yaml 
按照指引在合适的位置添加
- moegirl
```

## 快照

**快照相当于存档，每次试验什么之前最好都存个档**

**！！！警告！！！**

**删除已创建快照必须一个一个删除，否则大概率崩盘。**

- 安装timeshift
```
sudo pacman -S timeshift 
```
- 开启自动备份服务
```
sudo systemctl enable --now cronie.service 
```
### 自动生成快照启动项
- 安装必要组件
```
sudo pacman -S grub-btrfs 
```
- 开启服务
```
sudo systemctl enable --now grub-btrfsd.service 
```
- 修改配置文件
```
sudo systemctl edit grub-btrfsd.service 
```
- 在默认位置添加
```
[Service]
ExecStart=
ExecStart=/usr/bin/grub-btrfsd --syslog --timeshift-auto
```
- 重启服务
```
sudo systemctl daemon-reload
sudo systemctl restart grub-btrfsd.service
```
- 避免id变更导致挂载失败
```
sudo sed -i -E 's/(subvolid=[0-9]+,)|(,subvolid=[0-9]+)//g' /etc/fstab
```

## open in any terminal

[GitHub - Stunkymonkey/nautilus-open-any-terminal](https://github.com/Stunkymonkey/nautilus-open-any-terminal)

这是一个在文件管理器“右键在此处打开终端”的功能

- 如果用的是ghostty
```
sudo pacman -S nautilus-python
```
- 其他终端仿真器
```
yay -S nautilus-open-any-terminal 
```
```
sudo glib-compile-schemas /usr/share/glib-2.0/schemas 
```
```
sudo pacman -S dconf-editor
```
```
修改配置，路径为/com/github/stunkymonkey/nautilus-open-any-terminal
```
- 重载nautilus
```
nautilus -q 
```

## 可变刷新率和分数缩放
商店安装refine修改

## 配置系统快捷键
### 交换大写锁定键和esc键
- 安装gnome-tweaks
```
sudo pacman -S gnome-tweaks
```
在键盘→其他布局里面交换CAPSLOCK和ESC键

右键桌面打开设置，选择键盘>查看及自定义快捷键
我的配置：

* 导航
```
super+shift+数字键 #将窗口移到工作区
super+shift+A/D #将窗口左右移动工作区
Super+数字键 #切换工作区
gnome默认super+滚轮上下可以左右切换工作区
alt+tab #切换应用程序
super+M #隐藏所有正常窗口
alt+` #在应用程序的窗口之间切换窗口
```
* 截图
```
ctrl+alt+A #交互式截图
```
- 打字
```
禁用快捷键
在fcitx5的configtool里面设置super+space切换输入法
```
 * 无障碍
```
全部backspace退格键禁用
```
* 窗口
```
super+Q #关闭窗口
super+F #切换最大化
super+alt+F #切换全屏
```
* 系统
```
ctrl+super+S #打开快速设置菜单
super+G #显示全部应用
```
* 自定义快捷键<快捷键>   <命令>
```
super+B   zen
super+T   ghostty
ctrl+alt+S    missioncenter
super+E   nautilus
super+shift+S   flatpak run be.alexandervanhee.gradia --screenshot=INTERACTIVE
```

## 功能性扩展
[arch + gnome美化教程_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1ym4y1G76s/?share_source=copy_web&vd_source=1c6a132d86487c8c4a29c7ff5cd8ac50)

[the best tiling extension](https://www.reddit.com/r/gnome/comments/1ei9bj0/the_best_tiling_extension/)

```
flatpak install flathub com.mattjakeman.ExtensionManager
```
```
#安装扩展
input method panel #fcitx需要的扩展

AppIndicator and KStatusNotifierItem Support #右上角显示后台应用

workspace indicator #显示工作区

caffeine #防止熄屏

lock keys #osd显示大写锁定和小键盘锁定

clipboard indicator #剪贴板历史

GNOME Fuzzy App Search #模糊搜索

steal my focus window #如果打开窗口时窗口已经被打开则置顶

tiling shell #窗口平铺，tilingshell是用布局平铺,另一个叫forge是hyprland那种自动平铺但是很卡。推荐用tilingshell，记得自定义快捷键，我快捷键是super+w/a/s/d对应上下左右移动窗口，Super+Alt+w/a/s/d对应上下左右扩展窗口，super+c取消平铺。

color picker #对自定义非常有用
vitals #右上角显示当前资源使用情况
emoji copy #快捷输入emoji,很有趣
```


# 笔记本

## 显卡切换

### 切换为集显模式

#### asus华硕用户可以用supergfxctl
[Linux for ROG Notebooks](https://asus-linux.org/)

```
yay -S supergfxctl
```

```
sudo systemctl enable --now supergfxd
```

```
扩展下载GPU supergfxctl switch
```

```
使用方法：
Integrated supergfxctl --mode Integrated 
Hybrid supergfxctl --mode Hybrid 
VFIO supergfxctl --mode Vfio 
AsusEgpu supergfxctl --mode AsusEgpu 
AsusMuxDgpu supergfxctl --mode AsusMuxDgpu
```

#### envycontrol
[GitHub - bayasdev/envycontrol: Easy GPU switching for Nvidia Optimus laptops under Linux](https://github.com/bayasdev/envycontrol)

* 笔记本BIOS内切换为混合模式

```
yay -S envycontrol 
```

* 安装gnome插件,GPU Profile Selector

```
https://extensions.gnome.org/extension/5009/gpu-profile-selector/
```

* 在右上角切换显卡至integrated

### 混合模式下用独显运行程序

####  PRIME

```
sudo pacman -S nvidia-prime
```

- 命令行内使用 prime-run命令使用独显运行软件

```
prime-run firefox 
```

- 使用pinapp修改.desktop文件，在command的最前面加上 prime-run 

#### 在gnome桌面环境下右键快捷方式选择使用独显运行

```
sudo pacman -S switcheroo-control 
```

```
sudo systemctl enable --now switcheroo-control 
```

## 电源管理
[Power management/Suspend and hibernate - ArchWiki](https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate)

### 休眠到硬盘
硬盘上必须有交换空间才能休眠到硬盘
- 添加hook
```
sudo vim /etc/mkinitcpio.conf
```
```
在HOOKS()内添加resume,注意需要添加在udev的后面,建议加在末尾
```
- 重新生成initramfs
```
sudo mkinitcpio -P
```
- reboot
```
reboot
```
- 使用命令进行休眠
```
systemctl hibernate
```

### 内核参数

[[HowTo] Disable watchdogs (and silence "watchdog did not stop!") - Contributions / Tutorials - Manjaro Linux Forum](https://forum.manjaro.org/t/howto-disable-watchdogs-and-silence-watchdog-did-not-stop/148561/4)

```
sudo vim /etc/default/grub
```

在GRUB_CMDLNE_LINUX_DEFAULT=""里面添加参数
nowatchdog modprobe.blacklist=sp5100_tco
禁用watchdog，intelcpu用户把sp5100_tco换成iTCO_wdt
可选参数： pcie_aspm=force 强制pcie活动电源管理，可以略微降低功耗。

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### power-profiles-daemon

性能模式切换，有三个档位，performance性能、balance平衡、powersave节电
不建议使用tlp或者auto-cpufreq，意义不大。这个易用而且足够，如果想折腾的话可以自己试试tlp。

```
sudo pacman -S power-profiles-daemon
```

```
sudo systemctl enable --now power-profiles-daemon 
```

### 实用插件扩展

```
power tracker #显示电池充放电
auto power profile #配合powerProfilesDaemon使用，可以自动切换模式
power profile indicator # 配合powerProfilesDaemon使用，顶栏显示当前模式
```

---

# 美化
## 更换壁纸
```
右键桌面选择更换背景
```
## 扩展美化

[arch + gnome美化教程_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1ym4y1G76s/?share_source=copy_web&vd_source=1c6a132d86487c8c4a29c7ff5cd8ac50)

```
#安装扩展
lock screen background #更换锁屏背景
blur my shell #透明度美化
hide top bar #隐藏顶栏
burn my windows #应用开启和打开的动画
user themes #主题，浏览器搜索gnome shell theme下载主题
logo menu # top bar的左上角显示一个logo,好玩
```

## 主题美化
- 去掉标题栏用来关闭窗口的x
```
gsettings set org.gnome.desktop.wm.preferences button-layout 'appmenu:'
```

### 光标主题
主题下载网站 https://www.gnome-look.org/browse?cat=107&ord=latest

将下载的.tar.gz文件里面的文件夹放到～/.local/share/icons/目录下，没有icons文件夹的话自己创建一个

### gnome主题
https://www.gnome-look.org/browse?cat=134&ord=latest

通常下载页面都有指引，文件路径是~/.themes/，放进去之后在user themes扩展的设置里面改可以改

## 终端美化

- 安装字体
```
sudo pacman -S ttf-jetbrains-mono-nerd
```
 - 安装zsh
```
sudo pacman -S zsh
```
- 修改shell为zsh
```
chsh -s /usr/bin/zsh
```
```
#登出
```
```
#启动终端按0生成默认的配置文件
```

### starship
[Starship](https://starship.rs/)

```
sudo pacman -S starship
```
```
vim ~/.zshrc
```
```
eval "$(starship init zsh)"
```

#### preset主题
https://starship.rs/presets/
挑一个自己喜欢的，下载后改名为starship.toml，移动到~/.config目录
### 语法高亮和自动补全
- 语法检查、补全、tab
```
sudo pacman -S zsh-syntax-highlighting zsh-autosuggestions zsh-completions
```

```
vim ~/.zshrc
```
写入
```
#语法检查和高亮
source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
#开启tab上下左右选择补全
zstyle ':completion:*' menu select
autoload -Uz compinit
compinit
```
```
source ~/.zshrc
```

### ghostty美化
- 下载catppuccin颜色配置，粘贴到~/.config/ghostty/themes/
```
https://github.com/catppuccin/ghostty?tab=readme-ov-file
```
- 修改~/.config/ghostty/conf 配置文件，例如下载的是frappe的话：
```
theme = /home/shorin/.config/ghostty/catppuccin-frappe.conf
```
- 隐藏标题栏
```
window-decoration = none
```
- 设置透明度
```
background-opacity=0.8
```
- 设置字体和字体大小
```
font-family = "Adwaita Mono" 
font-size = 15
```

# KVM虚拟机
[[已解决] KVM Libvirt 中无法访问存储文件，权限被拒绝错误](https://cn.linux-terminal.com/?p=4593)

[How to Install KVM on Ubuntu | phoenixNAP KB](https://phoenixnap.com/kb/ubuntu-install-kvm)

[如何在 Linux 主机和 KVM 中的 Windows 客户机之间共享文件夹 | Linux 中国 - 知乎](https://zhuanlan.zhihu.com/p/645234144)


* 安装qemu，图形界面， TPM
```
sudo pacman -S qemu-full virt-manager swtpm 
```
* 开启libvirtd系统服务
```
sudo systemctl enable --now libvirtd
```
* 开启NAT default网络
```
sudo virsh net-start default
sudo virsh net-autostart default
```
* 添加组权限 需要登出
```
sudo usermod -a -G libvirt $(whoami)
```
* 编辑配置文件提高权限
```
sudo vim /etc/libvirt/qemu.conf
```
```
#把user = "libvirt-qemu"改为user = "用户名"
#把group = "libvirt-qemu"改为group = "libvirt"
#取消这两行的注释
```
* 重启服务
```
sudo systemctl restart libvirtd
```
### 嵌套虚拟化
intel的话用 kvm_intel

- 临时生效

```
modprobe kvm_amd nested=1
```
- 永久生效

```
sudo vim /etc/modprobe.d/kvm_amd.conf
```
- 写入

```
options kvm_amd nested=1
```
- 重新生成

```
sudo mkinitcpio -P
```

### 配置桥接网络

* 启动高级网络配置工具
```
nm-connection-editor
```
```
#添加虚拟网桥，接口填bridge0或者别的
```
```
#添加网桥连接，选择以太网，选择网络设备
```
```
#保存后将网络连接改为刚才创建的以太网网桥连接
```
### 安装win11 LTS虚拟机
[手把手教你给笔记本重装系统（Windows篇）_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV16h4y1B7md/?spm_id_from=333.337.search-card.all.click)

[太突然！Win11 LTSC 官方精简版，终于来了 - 知乎](https://zhuanlan.zhihu.com/p/1000648759)

* 下载win11 iot LTS iso 镜像
```
https://go.microsoft.com/fwlink/?linkid=2270353&clcid=0x409&culture=en-us&country=us
```
* 下载virtiowin11 iso
```
https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.271-1/virtio-win-0.1.271.iso
```
```
根据视频指引安装
```
- 显示协议里监听类型选无，OpenGL，选择AMD显卡（N卡暂时不支持3d加速，可以用vmware），显卡里选virtio，勾选3d加速
- 跳过联网
确保机器没有连接到网络，按下shift+f10 ，鼠标点选窗口，输入
```
oobe\bypassnro
```
* 和本机进行文件分享
[如何在 Linux 主机和 KVM 中的 Windows 客户机之间共享文件夹 | Linux 中国 - 知乎](https://zhuanlan.zhihu.com/p/645234144)

```
确认开启共享内存
```
```
打开文件管理器，复制要共享的文件夹的路径
```
```
在虚拟机管理器内添加共享文件夹,粘贴刚才复制的路径，取个名字
```
```
虚拟机内win11安装winFSP
https://winfsp.dev/rel/
```
```
搜索service（服务），启用VirtIO-FS Service，设置为自动
```
### 显卡直通
[PCI passthrough via OVMF - ArchWiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF)

- 确认iommu是否开启，有输出说明开启
```
sudo dmesg | grep -e DMAR -e IOMMU
```
- 获取显卡的硬件id，如果是显卡所在group的所有设备的id都记下。如果group里有cpu的话去看wiki。
```
for d in /sys/kernel/iommu_groups/*/devices/*; do 
    n=${d#*/iommu_groups/*}; n=${n%%/*}
    printf 'IOMMU Group %s ' "$n"
    lspci -nns "${d##*/}"
done
```
- 隔离GPU
```
sudo vim /etc/modprobe.d/vfio.conf
```
写入如下内容
```
options vfio-pci ids=硬件id,硬件id
```
- 让vfio-pci抢先加载
```
sudo vim /etc/mkinitcpio.conf
```
MODULES=（）里面写入vfio_pci vfio vfio_iommu_type1 
```
MODULES=(... vfio_pci vfio vfio_iommu_type1  ...)
```
HOOKS=()里面写入 modconf
```
HOOKS=(... modconf ...)
```
- 重新生成
```
sudo mkinitcpio -P
```
- 重启电脑

- 安装ovmf
```
sudo pacman -S edk2-ovmf
```
编辑配置文件
```
sudo vim /etc/libvirt/qemu.conf
```
```
nvram = [
	"/usr/share/ovmf/x64/OVMF_CODE.fd:/usr/share/ovmf/x64/OVMF_VARS.fd"
]
```
重启服务
```
sudo systemctl restart libvirtd
```
virt-manager的虚拟机页面内添加设备，PCI Host Device里找到要直通的显卡。 然后USB hostDevice里面把鼠标键盘也直通进去。
- 取消显卡直通
```
sudo vim /etc/modprobe.d/vfio.conf
```
注释掉里面的内容
```
sudo mkinitcpio -P
```
```
重启
```
#### 虚拟机性能优化
我在使用的过程中，最大的感受就是cpu和gpu反倒以近乎原生的性能在运行，但是内存性能很差、非常差、难以置信的差。在研究looking glass的时候偶然解决了这个问题。元凶是memballoon

#####  memballoon

[libvirt/QEMU Installation — Looking Glass B7 documentation](https://looking-glass.io/docs/B7/install_libvirt/#memballoon)

memlbaloon的目的是提高内存的利用率，但是由于它会不停地“取走”“归还”虚拟机内存，导致显卡 直通时虚拟机内存性能极差。

将虚拟机xml里面的memballoon改为none，这将极大极大极大地！！！提高显卡直通虚拟机的性能。总体可以达到9成原生的性能。

```
<memballoon model="none"/>
```
##### 可选：内存大页
[KVM - Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/KVM#%E5%BC%80%E5%90%AF%E5%86%85%E5%AD%98%E5%A4%A7%E9%A1%B5)

没感觉到有提升，wiki说有，那姑且设置一下

- 计算大页大小

1g 虚拟机内存对应550，以16GB为例，大页需要8800（粗略计算，具体可以看wiki）

- 临时（重启失效）
```
echo 8800 > /proc/sys/vm/nr_hugepages
```
- 编辑虚拟机xml

在virt-manager的g首选项里开启xml编辑，找到```<memoryBacking>```并添加```<hugepages/>```：
```
  <memoryBacking>
    <hugepages/>
  </memoryBacking>
```
- 永久生效

```
sudo vim /etc/sysctl.d/40-hugepage.conf

vm.nr_hugepages = 8800
```

- 查看大页使用情况

```
grep HugePages /proc/meminfo
```



## 远程桌面
三种方案，parsec、sunshineo+moonlight、looking glass，配置难度和最终效果逐级上升

### parsec
- windows上浏览器搜索安装
- linux上安装
```
yay -S parsec-bin
```
- 登录相同账号


### sunshine+moonlight
[GitHub - LizardByte/Sunshine: Self-hosted game stream host for Moonlight.](https://github.com/LizardByte/Sunshine)

虚拟机win11内安装sunshine

```
https://github.com/LizardByte/Sunshine
```
安装虚拟显示器（parsec-vdd很好用但是有bug,重启会重置）
```
https://github.com/VirtualDrivers/Virtual-Display-Driver
```
客机安装moonlight
https://moonlight-stream.org/

```
sudo pacman -S moonlight-qt
```
sunshine在web设置pin码添加设备之后就可以连接了。

### looking glass

[Installation — Looking Glass B7 documentation](https://looking-glass.io/docs/B7/install/)

[PCI passthrough via OVMF - ArchWiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF)

looking glass通过共享内存实现屏幕分享，也需要安装[Virtual-Display-Driver](https://github.com/VirtualDrivers/Virtual-Display-Driver)

- 计算需要的共享内存大小
具体可以看官方档案，我是2560x1440@180hz 非HDR，需要大小是64M
- 设置共享内存设备
打开virt-manager，点击编辑 > 首选项，勾选启用xml编辑。
打开虚拟机配置，找到xml底部的  ```</devices>```，在  ```</devices>```的上面添加设备，就是这种感觉：
```
<devices>
    ...
  <shmem name='looking-glass'>
    <model type='ivshmem-plain'/>
    <size unit='M'>64</size>
  </shmem>
</devices>
```
64改为自己需要的大小

- 开启终端，添加kvm组
```
sudo gpasswd -a $USER kvm 

#添加自己到kvm组里面， 需要重启，重启后用groups确认自己的组
```
- 每次开机检查文件，不存在的话创建并编辑权限

```
sudo vim /etc/tmpfiles.d/10-looking-glass.conf

写入（user改为自己的用户名）：

f	/dev/shm/looking-glass	0660	user	kvm	-

f是创建文件
/dev/shm/looking-glass是共享内存文件的路径
0660设置拥有者和组拥有读写权限
user设置拥有者
kvm设置组
- 代表不进行清理
```
- 无须重启，现在手动创建文件

```
sudo systemd-tmpfiles --create /etc/tmpfiles.d/10-looking-glass.conf
```

- 回到虚拟机设置
  确认有spicei显示协议
  显卡设置为none
  添加virtio键盘和virtio鼠标（要在xml里面更改bus=“ps2”为bus=“virtio”）
  添加通道，设备类型选择spice
  里面找到下面这段，把type从none 改成spice

```
<sound model='ich9'>
  <audio id='1'/>
</sound>
<audio id='1' type='spice'/>
```

- 开启虚拟机，安装looking glass 服务端

 [Looking Glass - Download Looking Glass](https://looking-glass.io/downloads)

浏览器搜索 looking glass，点击download，下载最新的stable版本，解压后安装

- linux安装客户端

 服务端和客户端的版本要匹配，looking-glass包对应最新的stable版本，looking-glass-git包对应bleeding-edge版本

```
yay -S looking-glass
```
- 桌面打开即可连接
- 可选:

由于这样连接不会捕获快捷方式，无法在win里面 使用win键，可以微软商店下载powertoys重新映射快捷键解决这个问题。或者买3模键鼠。

关于虚拟机性能优化，见[虚拟机性能优化](#虚拟机性能优化)

# 在linux上玩游戏

这一节不仅适用于windows的游戏程序，还适用于windows的软件。
首选用steam玩游戏，steam没有的游戏通过lutris管理，使用proton或者wine运行。安卓手游用waydroid运行。如果都不行，用配置了显卡直通的win11虚拟机。

## 玩steam游戏

[Steam - ArchWiki](https://wiki.archlinux.org/title/Steam)

```
sudo pacman -S steam
```
在设置→兼容性里面选择默认兼容性工具即可运行大部分无反作弊的游戏

## 玩minecraft
- 从aur安装
```
yay -S minecraft-launcher #官方启动器
yay -S hmcl-bin
```
安装时选择最新的jdk

## 玩安卓手游
### waydroid
[Install Instructions | Waydroid](https://docs.waydro.id/usage/install-on-desktops)

[Waydroid - Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/Waydroid)

安卓系统也是linux内核，那linux发行版自然也能运行安卓，并且性能还是接近原生的。waydroid是linux上的安卓容器，相当于一个完整的安卓系统。

- 安装
```
yay -S waydroid
```
可选：从archlinuxcn安装waydroid-image（要求添加cn仓库，按照流程，在本文档的yay安装部分已经添加）
```
sudo pacman -S waydroid-image
#带谷歌play的
sudo pacman -S waydroid-image-gapps
```
- 初始化
```
sudo waydroid init
#带谷歌的
sudo waydroid init -s GAPPS
```
- 启动服务
```
sudo systemctl enable --now waydroid-container
```
- 安装arm转译
[GitHub - casualsnek/waydroid_script: Python Script to add OpenGapps, Magisk, libhoudini translation library and libndk translation library to waydroid !](https://github.com/casualsnek/waydroid_script)

我们的cpu架构是x86_64,要运行arm应用需要安装arm转译, amd装libndk, intel装libhoudini
```
sudo pacman -S lzip
git clone https://github.com/casualsnek/waydroid_script
cd waydroid_script
python3 -m venv venv
venv/bin/pip install -r requirements.txt
sudo venv/bin/python3 main.py
按照窗口的指引进行安装

```
- 开启会话
```
waydroid session start
```
然后应该就能在桌面看到一大堆图标了
- 软件默认是全屏打开，可以设置窗口化打开软件，f11切换全屏和窗口化
```
waydroid prop set persist.waydroid.multi_windows true
```
然后用命令重启会话，这一步会隐藏桌面的waydroid图标，可以设置显示。如果开启不了的话可以stop之后在尝试用桌面快捷方式开启
```
waydroid session stop
waydroid session start 
```
- 安装软件
```
waydroid app install /apk/的/路径
```

#### 软件渲染
n卡用户用不了waydroid，可以用软件渲染，但是性能很差，勉强玩2d游戏。
- 编辑配置文件
```
/var/lib/waydroid/waydroid.cfg
```
- 本地更新应用一下更改后的配置
```
sudo waydroid upgrade --offline
```
- 重启服务
```
systemctl restart waydroid-container
```

#### 卸载waydroid

```
waydroid session stop
sudo systemctl disable --now waydroid-container.service
yay -Rns waydroid #如果下载了waydroid-image的话需要一并删除
sudo rm -rf /var/lib/waydroid ~/.local/share/waydroid ~/.local/share/applications/waydroid*

```

## wine/proton 兼容层运行
wine是在linux下运行windows程序的兼容层，proton是steam的母公司v社基于wine开发的专门用来玩游戏的兼容层。原理是把window程序发出的请求翻译成linux系统下的等效请求。通常使用最新的wine或者proton版本即可。steam添加非steam游戏的proton路径。r

### lutris
[Download Lutris](https://lutris.net/downloads)

lutris是一个专为玩游戏设计的工具，可以很方便地管理需要用wine或者proton兼容层运行的软件。lutris可以完全取代steam的“添加非steam游戏”功能。

- 安装
```
sudo pacman -S lutris
```
- 第一次打开会自动下载各种需要的组件，点击左上角的加号可以看到主要功能
- 卸载lutris
```
sudo pacman -Rns lutris
```
```
sudo rm -rfv ~/.config/lutris ~/.cache/lutris ~/.local/share/lutris ~/.local/share/applications/net.lutris.Lutris.desktop ~/.local/share/applications/lutris-game-*.desktop
```
steam下载proton之后可以在lutris里面设置wine版本为proton


## 用显卡直通玩游戏
经过前面显卡直通的操作，我已经有了一台4060显卡的win11，并且配置了looking glass，理论上所有win11能干的事情我都能在这台虚拟机上干。具体的就不用再往下说了吧🤓☝️
至于为什么显卡直通虚拟机win11而不是重启到真的win11里面。
因为just for fun，想双系统就重启，想直通就直通，想wsl就wsl，还要追问为什么的话我祝你万事如意身体健康。

# 性能优化

## N卡动态功耗调节 

```
sudo systemctl enable --now nvidia-powerd.service
```
## LACT进行显卡offset

## 交换空间
关于交换空间大小：
[Swap - Manjaro --- Swap - Manjaro](https://wiki.manjaro.org/index.php?title=Swap)

[电源管理/挂起与休眠 - Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/%E7%94%B5%E6%BA%90%E7%AE%A1%E7%90%86/%E6%8C%82%E8%B5%B7%E4%B8%8E%E4%BC%91%E7%9C%A0#%E7%A6%81%E7%94%A8_zswap_%E5%86%99%E5%9B%9E%E4%BB%A5%E4%BB%85%E5%B0%86%E4%BA%A4%E6%8D%A2%E7%A9%BA%E9%97%B4%E7%94%A8%E4%BA%8E%E4%BC%91%E7%9C%A0)

[zswap - ArchWiki](https://wiki.archlinux.org/title/Zswap)

[zram - ArchWiki](https://wiki.archlinux.org/title/Zram)

[Swap - ArchWiki](https://wiki.archlinux.org/title/Swap)

[Zram vs zswap vs swap? : r/archlinux](https://www.reddit.com/r/archlinux/comments/1ivwv1l/zram_vs_zswap_vs_swap/)

[Zswap vs zram in 2023, what's the actual practical difference? : r/linux](https://www.reddit.com/r/linux/comments/11dkhz7/zswap_vs_zram_in_2023)

[linux - ZRAM vs ZSWAP for lower end hardware? - Super User](https://superuser.com/questions/1727160/zram-vs-zswap-for-lower-end-hardware)

[Zswap or Zram: at this time, which one is more efficient? : r/archlinux](https://www.reddit.com/r/archlinux/comments/13ujemv/zswap_or_zram_at_this_time_which_one_is_more/)

[Zram, zswap and hibernation - Support - Manjaro Linux Forum](https://forum.manjaro.org/t/zram-zswap-and-hibernation/82348)

[kernel - zram vs zswap vs zcache Ultimate guide: when to use which one - Ask Ubuntu](https://askubuntu.com/questions/471912/zram-vs-zswap-vs-zcache-ultimate-guide-when-to-use-which-one/472227#472227)

[zswap — The Linux Kernel documentation](https://www.kernel.org/doc/html/v4.18/vm/zswap.html)

[zram: Compressed RAM-based block devices — The Linux Kernel documentation](https://docs.kernel.org/admin-guide/blockdev/zram.html)


### 不需要休眠的话
如果不需要休眠功能可以禁用swap，然后开启zram
```
sudo swapoff /swap/swapfile
```
```
sudo rm /swap/swapfile
```
- 编辑fstab
```
sudo vim /etc/fstab
```
```
删除与swap相关的挂载
```
### zram内存压缩

```
sudo pacman -S zram-generator
```
```
sudo vim  /etc/systemd/zram-generator.conf
```
```
[zram0]
zram-size = "ram*0.5"
compression-algorithm = zstd #重视cpu开销和速度选择lz4
```
- 禁用zswap
```
sudo vim /etc/default/grub
```
```
编辑GRUB_CMDLINE_LINUX_DEFAULT=""
写入zswap.enabled=0
```
- 重新生成grub的配置文件
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
- reboot
- 验证zswap是否关闭
```
sudo grep -R . /sys/kernel/debug/zswap/
```
- 验证zram是否开启
```
sudo zramctl
或者
swapon
```
## 安装zen内核
ps：会导致功耗略微增加
* 安装内核
```
sudo pacman -S linux-zen linux-zen-headers
```
* 安装显卡驱动，用nvidia-dkms替换nvidia驱动
```
sudo pacman -S nvidia-dkms
```
* 重新生成grub
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
* 重启
```
reboot #重启时在grub的arch advance启动项里选择zen
```
* 确认正常运行后删除stable内核
```
sudo pacman -R linux linux-headers
```
* 重新生成grub
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

# 删除linux
## 和windows共用efi分区时
[(重制)彻底删除Linux卸载后的无用引导项_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV14p4y1n7rJ/?spm_id_from=333.1387.favlist.content.click)

win+x 选择磁盘管理，找到efi在第几个磁盘的第几个分区

win+R 输入 diskpart 回车
select disk 命令选择efi分区所在磁盘，从0开始，第一个磁盘是数字0

```
select disk 0
```
select partition 选择efi分区，从1开始，第一个分区是数字1
```
select partition 1
```
以上两条命令代表选中了第一个磁盘的第一个分区
分配盘符
```
assign letter p
```
管理员运行记事本
ctrl+s 打开保存窗口
选择p盘,删除里面的linux 启动相关文件
移除盘符
```
remove letter p
```

## 单独efi分区时
[windows10删除EFI分区(绝对安全)-CSDN博客](https://blog.csdn.net/sinat_29957455/article/details/88726797)

diskpart选中efi分区后输入：

```
 SET ID=ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
```
即可在磁盘管理工具里面删除分区
或者使用diskgeniux，图吧工具箱里面有

---
# issues

## 磁盘占用异常

明明没有多少文件，磁盘占用却很高。可以试试删除btrfs快照。



## 时间错乱，windwos开机磁盘检查
[双系统时间同步-CSDN博客](https://blog.csdn.net/zhouchen1998/article/details/108893660)

管理员打开powershell 运行

```
Reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
```

## nautilus创建文件的symlinks符号链接
[Creating Symlinks in Files under Wayland : r/gnome](https://www.reddit.com/r/gnome/comments/10qayrs/creating_symlinks_in_files_under_wayland/)



ctrl+c复制文件后ctrl+m

## NAUTILUS无法访问smb共享
如果你的路由器或者别的设备开启了smb文件共享，安装gvfs-smb可以使你在nautilus访问那些文件
```
sudo pacman -S gvfs-smb
```

## 域名解析出现暂时性错误
[解决 Ubuntu 系统中 “Temporary Failure in Name Resolution“ 错误-CSDN博客](https://blog.csdn.net/qq_15603633/article/details/141032652)



```
sudo vim /etc/resolv.conf
```
内容修改为
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

## 自定义安装字体
* 复制字体到该目录下（可以自由创建子目录）
```
～/local/share/fonts/
```
* 刷新字体缓存
```
fc-cache -fv
```

## 扩展windwos的efi分区空间
```
NIUBI partition Editor free edition
```

## grub卡顿
n卡的锅，没辙

# 附录

## pacman常用指令

- 下载包但不安装

```
sudo pacman -Sw
```

* 删除包，同时删除不再被其他包需要的依赖和配置文件,-R删除包，s删除依赖，n删除配置文件
```
sudo pacman -Rns
```
* 查询包
```
sudo pacman -Ss
```
* 列出所有已安装的包
```
sudo pacman -Qe
```
* 列出所有已安装的依赖
```
sudo pacman -Qd
```
* 清理包缓存
```
sudo pacman -Sc
```
* 列出孤立依赖包
```
sudo pacman -Qdt
```
* 清理孤立依赖包
```
sudo pacman -Rns $(pacman -Qdt)
```
## cpu资源优先级
因为影响steam下载速度已弃用（这是已知问题，估计还有其它问题）
```
sudo pacman -S ananicy-cpp
```
```
sudo systemctl enable --now ananicy-cpp.service
```
## TLP相关
```
sudo pacman -S tlp tlp-rdw 
```
```
yay -S tlpui
```
设置方法参考官方文档[Settings — TLP 1.8.0 documentation](https://linrunner.de/tlp/settings/index.html)

这里给一个现代电脑的通用设置：

```
processor选项卡中

CPU DRIVER OPMODE 
AC active
BAT active

CPU SCALING GOVERNOR 
AC schedutil
BAT powersave

CPU ENERGY PERF POLICY
AC balance_performance
BAT power

CPU BOOST
AC on
BAT off

PLATFORM PROFILE
AC balanced
BAT low-power

MEM SLEEP
BAT deep
```
- 开启服务
```
sudo systemctl enable --now tlp
```

## 安装alhp 
（下载太慢，容易下载失败，我不建议使用，遂删除）
*参考链接: [ALHP：优化你的archlinux性能 - 哔哩哔哩](https://www.bilibili.com/opus/745324585822453908?from=search&spm_id_from=333.337.0.0%2a)



* 检查芯片支持,记住结果里是x86-64-v几
```
/lib/ld-linux-x86-64.so.2 --help
```
* 安装密钥和镜像列表
```
yay -S alhp-keyring alhp-mirrorlist
```
* 编辑配置文件
```
sudo vim /etc/pacman.conf
```
- 搜索core，在core上方加入
```
   [core-x86-64-v4]
   Include = /etc/pacman.d/alhp-mirrorlist
   [extra-x86-64-v4]
   Include = /etc/pacman.d/alhp-mirrorlist
   [multilib-x86-64-v4]
   Include = /etc/pacman.d/alhp-mirrorlist
```
* 刷新源
```
sudo pacman -Syyu
```

## ibus输入法
（由于词库和联想实在远不如fctix5,换掉了）
ibpinyin是中文拼音输入法，anthy是日文输入法登出一次，设置里找到键盘，添加输入源
```
sudo pacman -S ibus ibus-pinyin
更好用的中文输入法：
sudo pacman -S ibus-rime 
日语输入法：
yay -S ibus-mozc
```
- 配置输入法
常规里勾选候选词，设置候选词排序为词频
拼音模式里启用云输入
辞典里勾选辞典
用户数据里取消所有勾选
*登出，测试输入是否正常

## ranger预览图片
```
sudo pacman -S python-pillow ranger kitty 
```
```
vim ~/.config/ranger/rc.conf
```
```
set preview_images true
set preview_images_method kitty
```

## 美化kitty
（多个显示器的情况下，kitty用tiling shell扩展的自动平铺有bug，无法在当前显示器开启第一个窗口，所以换掉了）
```
sudo pacman -S kitty
```
```
#下载配置文件 https://github.com/catppuccin/kitty
以frappe为例，下载frappe.conf，复制到~/.config/kitty/目录下，重命名为kitty.conf
```
```
#编辑配置文件
写入：
linux_display_server x11 #修复kitty奇怪的刘海
hide_window_decorations yes #隐藏顶栏，隐藏后无法调整窗口大小，建议配合tiling shell扩展使用
background_opacity 0.8 #设置背景透明度
font_family 字体名
font_size 字体大小数字
```
```
#如果波浪号在左上角，配置文件写入：
symbol_map U+007E Adwaita Mono
#强制指定notosansmono字体，也可以选择别的
```
```
#我的示例配置
hide_window_decorations yes
background_opacity 0.8
font_family Adwaita Mono
font_size 14
```
```
#重启终端
```

# 特别鸣谢
[Google Gemini](https://gemini.google.com/app)


### 参考资料：
[archlinux 简明指南](https://arch.icekylin.online/)

[安装指南 - Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97)

[双系统时间同步-CSDN博客](https://blog.csdn.net/zhouchen1998/article/details/108893660)

[Swap - ArchWiki](https://wiki.archlinux.org/title/Swap)

[gnome-shell uses dgpu instead of igpu : r/gnome](https://www.reddit.com/r/gnome/comments/1irvmki/gnomeshell_uses_dgpu_instead_of_igpu/)

[NVIDIA - ArchWiki](https://wiki.archlinux.org/title/NVIDIA)

[GitHub - Jguer/yay: Yet another Yogurt - An AUR Helper written in Go](https://github.com/Jguer/yay)

[Steam - ArchWiki](https://wiki.archlinux.org/title/Steam)

[Download · Wiki · wine / wine · GitLab](https://gitlab.winehq.org/wine/wine/-/wikis/zh_CN/Download)

[Lutris - Open Gaming Platform](https://lutris.net/)

[GitHub - Stunkymonkey/nautilus-open-any-terminal](https://github.com/Stunkymonkey/nautilus-open-any-terminal)

[arch + gnome美化教程_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1ym4y1G76s/?share_source=copy_web&vd_source=1c6a132d86487c8c4a29c7ff5cd8ac50)

[the best tiling extension](https://www.reddit.com/r/gnome/comments/1ei9bj0/the_best_tiling_extension/)

[Linux for ROG Notebooks](https://asus-linux.org/)

[GitHub - bayasdev/envycontrol: Easy GPU switching for Nvidia Optimus laptops under Linux](https://github.com/bayasdev/envycontrol)

[[已解决] KVM Libvirt 中无法访问存储文件，权限被拒绝错误](https://cn.linux-terminal.com/?p=4593)

[How to Install KVM on Ubuntu | phoenixNAP KB](https://phoenixnap.com/kb/ubuntu-install-kvm)

[如何在 Linux 主机和 KVM 中的 Windows 客户机之间共享文件夹 | Linux 中国 - 知乎](https://zhuanlan.zhihu.com/p/645234144)

[手把手教你给笔记本重装系统（Windows篇）_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV16h4y1B7md/?spm_id_from=333.337.search-card.all.click)

[太突然！Win11 LTSC 官方精简版，终于来了 - 知乎](https://zhuanlan.zhihu.com/p/1000648759)

[PCI passthrough via OVMF - ArchWiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF)

[GitHub - LizardByte/Sunshine: Self-hosted game stream host for Moonlight.](https://github.com/LizardByte/Sunshine)

[Swap - Manjaro --- Swap - Manjaro](https://wiki.manjaro.org/index.php?title=Swap)

[电源管理/挂起与休眠 - Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/%E7%94%B5%E6%BA%90%E7%AE%A1%E7%90%86/%E6%8C%82%E8%B5%B7%E4%B8%8E%E4%BC%91%E7%9C%A0#%E7%A6%81%E7%94%A8_zswap_%E5%86%99%E5%9B%9E%E4%BB%A5%E4%BB%85%E5%B0%86%E4%BA%A4%E6%8D%A2%E7%A9%BA%E9%97%B4%E7%94%A8%E4%BA%8E%E4%BC%91%E7%9C%A0)

[zswap - ArchWiki](https://wiki.archlinux.org/title/Zswap)

[Zram vs zswap vs swap? : r/archlinux](https://www.reddit.com/r/archlinux/comments/1ivwv1l/zram_vs_zswap_vs_swap/)

[Zswap vs zram in 2023, what's the actual practical difference? : r/linux](https://www.reddit.com/r/linux/comments/11dkhz7/zswap_vs_zram_in_2023)

[linux - ZRAM vs ZSWAP for lower end hardware? - Super User](https://superuser.com/questions/1727160/zram-vs-zswap-for-lower-end-hardware)

[Zswap or Zram: at this time, which one is more efficient? : r/archlinux](https://www.reddit.com/r/archlinux/comments/13ujemv/zswap_or_zram_at_this_time_which_one_is_more/)

[Zram, zswap and hibernation - Support - Manjaro Linux Forum](https://forum.manjaro.org/t/zram-zswap-and-hibernation/82348)

[kernel - zram vs zswap vs zcache Ultimate guide: when to use which one - Ask Ubuntu](https://askubuntu.com/questions/471912/zram-vs-zswap-vs-zcache-ultimate-guide-when-to-use-which-one/472227#472227)

[zswap — The Linux Kernel documentation](https://www.kernel.org/doc/html/v4.18/vm/zswap.html)

[zram: Compressed RAM-based block devices — The Linux Kernel documentation](https://docs.kernel.org/admin-guide/blockdev/zram.html)

[windows10删除EFI分区(绝对安全)-CSDN博客](https://blog.csdn.net/sinat_29957455/article/details/88726797)

[Creating Symlinks in Files under Wayland : r/gnome](https://www.reddit.com/r/gnome/comments/10qayrs/creating_symlinks_in_files_under_wayland/)

[解决 Ubuntu 系统中 “Temporary Failure in Name Resolution“ 错误-CSDN博客](https://blog.csdn.net/qq_15603633/article/details/141032652)

[Settings — TLP 1.8.0 documentation](https://linrunner.de/tlp/settings/index.html)
[ALHP：优化你的archlinux性能 - 哔哩哔哩](https://www.bilibili.com/opus/745324585822453908?from=search&spm_id_from=333.337.0.0%2a)

[kitty.conf - kitty](https://sw.kovidgoyal.net/kitty/conf/)

[Steam - ArchWiki](https://wiki.archlinux.org/title/Steam)

[Install Instructions | Waydroid](https://docs.waydro.id/usage/install-on-desktops)

[Waydroid - Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/Waydroid)

[GitHub - casualsnek/waydroid_script: Python Script to add OpenGapps, Magisk, libhoudini translation library and libndk translation library to waydroid !](https://github.com/casualsnek/waydroid_script)

[Download Lutris](https://lutris.net/downloads)

 [Looking Glass - Download Looking Glass](https://looking-glass.io/downloads)

[Installation — Looking Glass B7 documentation](https://looking-glass.io/docs/B7/install/)

[libvirt/QEMU Installation — Looking Glass B7 documentation](https://looking-glass.io/docs/B7/install_libvirt/#memballoon)

