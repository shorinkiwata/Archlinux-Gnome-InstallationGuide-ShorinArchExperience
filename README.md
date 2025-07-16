[Archlinux+Gnome安装与配置.md](https://github.com/user-attachments/files/21246919/Archlinux%2BGnome.md)

本文档是主要是btrfs文件系统的archlinux+Gnome环境的搭建，内容涉及archlinux的手动安装、脚本安装、swap文件、gnome精简安装、桌面配置、美化、kvm虚拟机、显卡直通、笔记本显卡切换、性能优化、电源管理

# 安装系统
## 手动安装
参考链接：
https://arch.icekylin.online/
https://wiki.archlinuxcn.org/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97

### 确认网络
```
ip link 查看网络接口
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
lsblk -p  查看当前分区情况
fdisk -l 小写字母l，查看详细分区信息
```
```
cfdisk /dev/nvme0n1 选择自己要使用的硬盘进行分区
```
创建512MB efi system
其余全部分到一个分区里，类型linux filesystem 


#### 格式化分区
```
lsblk -p 查看分区情况
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

- 确认
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

```
可选：下载速度慢的话

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

##设置swap
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

取消en_US.UTF-8 UTF-8的注释
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
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH
```

- 编辑grub的源文件
```
vim /etc/default/grub
去掉quiet， loglevel改成5，添加nowatchdog modprobe.blacklist=sp5100_tco，intelcpu用户把sp5100_tco换成iTCO_wdt
手动写入或者取消GRUB_DISABLE_OS_PROBER=false的注释
```

- 生成配置文件
```
grub-mkconfig -o /boot/grub/grub.cfg
```

## 完成安装
```
exit 推出changeroot
reboot 重启，会自动取消所有的挂载
```
## 启动网络
```
systemctl enable --now NetworkManager
```

连接wifi
```
nmcli dev wifi connect <wifiname> password <password>
```
##放松一下吧
```
pacman -S fastfetch lolcat cmatrix
```

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
wiki推荐是1GB，所以填入1024MB，类型fat32,挂载点是/boot
#### swap交换空间
swap与虚拟内存和休眠有关，可以创建swap分区或者swap文件，二选一，前者配置更简单，后者配置稍复杂，但是更加灵活。
###### swap分区
  创建一个和内存大小相同的硬盘分区，类型选择swap
###### swap文件
将所有剩余空间分到一个分区，类型选择btrfs,设置compress（透明压缩,可以节省磁盘空间），添加sub volume。
- @ 对应 /
- @home 对应 / 
- @swap 对应 /swap
交换文件的创建方法：https://wiki.archlinuxcn.org/wiki/Swap#%E4%BA%A4%E6%8D%A2%E6%96%87%E4%BB%B6
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
*参考链接https://wiki.archlinux.org/title/NVIDIA*
### 检查头文件
```
sudo pacman -S linux-headers
#linux替换为自己的内核
```
### 安装显卡驱动 

N卡此时如果不安装显卡驱动，可能无法启动桌面环境，此处以4060为例
```
sudo pacman -S nvidia nvidia-utils nvidia-settings 
```
非stable内核要安装的驱动不一样，具体看wiki，zen内核装nvidia-dkms
命令行使用 sudo nvidia-settings可以对显卡进行超频
#### AMD核显建议检查是否安装vulkan驱动
```
sudo pacman -S vulkan-radeon 
```
```
sudo pacman -S vulkan-tools
#测试工具，vulkaninfo --summary查看vulkan是否启用，vkcube测试vulkan渲染
```
- 混合模式软件还是跑在N卡上的话检查有没有安装vulkan-mesa-layers
https://www.reddit.com/r/gnome/comments/1irvmki/gnomeshell_uses_dgpu_instead_of_igpu/

```
sudo pacman -S vulkan-mesa-layers
```
### 硬件编解码
 - nvidia4060
```
sudo pacman -S libva-nvidia-driver libva libva-utils
```
- intel xe核显
```
sudo pacman -S intel-media-driver libva libva-utils
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
```
- qq、微信、wps
```
yay -S linuxqq wechat wps-office-cn  
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

###  游戏和wine运行exe程序

```
sudo pacman -S steam wine lutris
```

```
winecfg
```

lutris官网有大量游戏安装脚本，steam要开启32位源后才能安装

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

### fcitx5
```
sudo pacman -S fcitx5-im fcitx5-chinese-addons fcitx5-mozc
```
mozc是g谷歌日语输入法的开源版本
- 安装gnome扩展：input method panel
https://extensions.gnome.org/extension/261/kimpanel/
- 在fcitx5配置里面添加输入法，没有的话登出重新载入一次
- 编辑环境变量
```
sudo vim /etc/environment
```
```
GTK_IM_MODULE=fcitx 
QT_IM_MODULE=fcitx 
XMODIFIERS=@im=fcitx
```
```
重启电脑
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
- 如果用的是ghostty
```
sudo pacman -S nautilus-python
```
- 其他终端仿真器
这是一个在文件管理器“右键在此处打开终端”的功能
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

## 休眠
*参考链接 https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate*
休眠分为四种，休眠到内存（deep suspend）、休眠到idle(s2idle suspend)，休眠到硬盘（hibernate）、同时休眠到内存和硬盘（hybrid sleep）。
suspend是暂时挂起。s2idle是微软提供的一种唤醒速度极快的挂起，缺点是耗电。deep比s2idle更省电，唤醒速度稍慢。hibernate完全不耗电，但是唤醒慢。

### 休眠到内存
最新的电脑通常只支持s2idle,所以不需要修改
- 查看支持的suspend方式
```
cat /sys/power/mem_sleep
```
通常输出结果是s2idle shallow deep
- 临时修改suspend方式
```
echo deep > /sys/power/mem_sleep
```
- 永久修改
```
sudo vim /etc/systemd/sleep.conf.d/mem-deep.conf
```
```
[Sleep]
MemorySleepMode=deep
```

### 休眠到硬盘
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
- 尝试hibernate
```
systemctl hibernate
```
 如果成功hibernate的话电脑在一小会儿的磁盘写入后会进入关机状态。开机后进到锁屏界面而不是登入界面，输入密码后休眠前的窗口仍然在原位。
 - 查看hibernate过程中的日志
```
 journalctl -b 0 | grep -i -e "hibernation" -e "resume"
```

#### 如果失败的话尝试指定交换空间
- 获取swap分区的UUID：ctrl+shift+c复制
 ```
lsblk -o name,mountpoint,size,uuid 
 ```
- 编辑grub
```
sudo nano /etc/default/grub 
```
- 在GRUB_CMDLINE_LINUX_DEFAULT里添加
```
resume=UUID=刚才复制的UUID
```
- 重新生成grub.cfg
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

## 可变刷新率和分数缩放
商店安装refine修改

## 配置系统快捷键
### 交换大写锁定键和esc键
- - 安装gnome-tweaks
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

tiling shell#窗口平铺，tilingshell是用布局平铺,另一个叫forge是hyprland那种自动平铺但是很卡。推荐用tilingshell，记得自定义快捷键，我快捷键是super+w/a/s/d对应上下左右移动窗口，Super+Alt+w/a/s/d对应上下左右扩展窗口，super+c取消平铺。
#其他建议开启的设置选项
#启用自动平铺
#聚焦2）

color picker #对自定义非常有用
vitals #右上角显示当前资源使用情况
emoji copy #快捷输入emoji,很有趣
```


# 笔记本

## 显卡切换

### 切换为集显模式

#### asus华硕用户可以用supergfxctl

https://gitlab.com/asus-linux/supergfxctl

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

- 使用menulibre修改.desktop文件，在command的最前面加上 prime-run 

#### 在gnome桌面环境下右键快捷方式选择使用独显运行

```
sudo pacman -S switcheroo-control 
```

```
sudo systemctl enable --now switcheroo-control 
```

## 电源管理

### 内核参数

https://forum.manjaro.org/t/howto-disable-watchdogs-and-silence-watchdog-did-not-stop/148561/4

```
sudo vim /etc/default/grub
```

在GRUB_CMDLNE_LINUX_DEFAULT=""里面添加参数
nowatchdog modprobe.blacklist=sp5100_tco
禁用watchdog，intelcpu用户把sp5100_tco换成iTCO_wdt
可选参数： pcie_aspm=force 强制pcie活动电源管理，可以略微降低功耗。

### power-profiles-daemon

性能模式切换，有三个档位，performance性能、balance平衡、powersave节电
*不建议使用tlp或者auto-cpufreq，这个易用而且足够，如果想折腾的话可以自己试试*

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

# 

---

# 美化
## 更换壁纸
```
右键桌面选择更换背景
```
## 扩展美化
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
临时生效
```
modprobe kvm_amd nested=1
```
永久生效
```
sudo vim /etc/modprobe.d/kvm_amd.conf
```
写入
```
options kvm_amd nested=1
```
重新生成
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
- 
显示协议里监听类型选无，OpenGL，选择AMD显卡（N卡暂时不支持3d加速，可以用vmware），显卡里选virtio，勾选3d加速

- 跳过联网
确保机器没有连接到网络，按下shift+f10 ，鼠标点选窗口，输入
```
oobe\bypassnro
```
* 和本机进行文件分享
*参考链接https://zhuanlan.zhihu.com/p/645234144*
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
https://wiki.archlinuxcn.org/wiki/%E4%BD%BF%E7%94%A8_OVMF_%E7%9B%B4%E9%80%9A_PCI#
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

#此处要是自己的内核，例如zen的话是linux-zen
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
## 远程桌面
两种方案，parsec和sunshine+moonlight，前者安装即用，后者需要一些配置但是上限最高，可以做到原生的效果。
### sunshine+moonlight
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

# 性能优化
## 内存压缩
zram
```
sudo pacman -S zram-generator
```
```
sudo vim  /etc/systemd/zram-generator.conf
```
```
[zram0]
zram-size = "ram*0.25"
compression-algorithm = zstd
```
```
reboot
```

## cpu资源优先级
```
sudo pacman -S ananicy-cpp
```
```
sudo systemctl enable --now ananicy-cpp.service
```
## N卡动态功耗调节 

```
sudo systemctl enable --now nvidia-powerd.service
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
sudo pacman -R linux 
```
* 重新生成grub
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

# 删除linux
## 和windows共用efi分区时
https://www.bilibili.com/video/BV14p4y1n7rJ/?spm_id_from=333.1387.favlist.content.click
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
https://blog.csdn.net/sinat_29957455/article/details/88726797
diskpart选中efi分区后输入：
```
 SET ID=ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
```
即可在磁盘管理工具里面删除分区

---
# issuses

## 时间错乱，windwos开机磁盘检查
https://blog.csdn.net/zhouchen1998/article/details/108893660
管理员打开powershell 运行
```
Reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1

```

## nautilus创建文件的symlinks符号链接
ctrl+c复制文件后ctrl+m

## NAUTILUS无法访问smb共享
如果你的路由器或者别的设备开启了smb文件共享，安装gvfs-smb可以使你在nautilus访问那些文件
```
sudo pacman -S gvfs-smb
```

## 域名解析出现暂时性错误
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

## TLP相关
```
sudo pacman -S tlp tlp-rdw 
```
```
yay -S tlpui
```
设置方法参考官方文档https://linrunner.de/tlp/settings/index.html
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
*参考链接https://www.bilibili.com/opus/745324585822453908?from=search&spm_id_from=333.337.0.0*

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
sudo pacman -S ibus ibus-libpinyin ibus-anthy
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

## 参考资料：
https://arch.icekylin.online/guide/rookie/basic-install
https://wiki.archlinuxcn.org/wiki/
https://www.bilibli.com
https://cn.linux-terminal.com/?p=4593
https://www.microsoft.com/zh-cn/privacy/privacystatement
https://github.com/Stunkymonkey/nautilus-open-any-terminal
https://aur.archlinux.org/packages/nautilus-open-any-terminal
https://phoenixnap.com/kb/ubuntu-install-kvm
https://missioncenter.io/
https://www.reddit.com/r/ChromeOSFlex/comments/ucno4b/qemukvm_virtmanager_windows_vm_very_slow/
https://cn.linux-terminal.com/?p=4593
https://zhuanlan.zhihu.com/p/645234144
https://github.com/bayasdev/envycontrol
https://github.com/LorenzoMorelli/GPU_profile_selector 
https://www.bilibili.com/video/BV1ym4y1G76s/?share_source=copy_web&vd_source=1c6a132d86487c8c4a29c7ff5cd8ac50
https://www.youtube.com/watch?v=8WkcLwXCFJQ&t=1399s
https://grok.com/
https://www.youtube.com/watch?v=AE1-W2bMVEs&t=316s
https://www.reddit.com/r/linux_gaming/comments/17h1i7n/linux_vs_windows_tested_in_10_games_linux_17/
https://linuxblog.io/boost-battery-life-on-linux-laptop-tlp/
https://linrunner.de/tlp/settings/index.html
https://blog.csdn.net/sinat_29957455/article/details/88726797
https://forum.manjaro.org/t/howto-disable-watchdogs-and-silence-watchdog-did-not-stop/148561/4
https://www.bilibili.com/video/BV14p4y1n7rJ/?spm_id_from=333.1387.favlist.content.click
https://forum.manjaro.org/t/howto-disable-watchdogs-and-silence-watchdog-did-not-stop/148561
https://wiki.archlinuxcn.org/wiki/%E4%BD%BF%E7%94%A8_OVMF_%E7%9B%B4%E9%80%9A_PCI#
