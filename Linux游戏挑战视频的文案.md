# linux玩游戏挑战/指南

## 文稿

linux系统能不能玩游戏？

今天这期视频我就抛开固有认知，用实践说话，挑战在linux上玩游戏。

我制定了难度逐级提高的挑战任务，从steam单机到热门网游再到反作弊严格的游戏，如果这些任务都能完成，那我就可以自信地说出：“linux可以玩游戏“ 这句话了。

硬件平台是华硕天选4锐龙版，核显780m，独显4060，具体配置如图。系统是上期视频配置出的archlinux。为了控制变量，我把4060显卡的频率限制在了2505Mhz，测试全程开启混合模式，使用核显进行录制。另外跟大家说明一下，我在linux上使用mangohud来显示帧数，但是这个软件记录的1%low帧和我肉眼实际看到的卡顿是不符的，甚至在帧生成时间没有波动的情况下依旧会记录到很低的low帧，所以大家看的时候主要当前帧和平均帧就好啦。

好的话不多说。挑战，现在开始。

steam是全球最大的游戏平台，steam的创造者v社发行过名为steamdeck的游戏掌机，其上搭载的就是基于archlinux的系统。由此可以推测，linux玩steam上的游戏应该相当轻松。安装steam只需要在终端输入sudo pacman -S steam，pacman包管理器就会帮我把steam安装好。开启后点击左上角进入设置，兼容性里设置默认兼容性工具为proton测试版。这样就可以运行steam上的游戏了。至于proton是什么，暂时先按下不表，我们直接来看游戏表现。

先试个压力小的，今年的开年大作，天国拯救2。2k分辨率默认中画质，开dlss transformer 平衡档。能有90到100帧，这个帧数相当可以啊，但是和windows比起来如何呢？诶！居然比windows高了足足10帧，这，有点不可思议啊。vulkan的效率已经可以媲美甚至超过dx12了吗？别急，我们换个压力更大的游戏试试，游戏评测自然少不赛博朋克2077！默认高画质，开dlss transformer 平衡档。linux上的2077帧数就明显比windows低很多。不过别急着下定论，n卡的黑科技，帧 生 成！也可以在linux上正常运行。开启帧生成之后linux的帧数就和windows相差无几了。实测下来，linux上开帧生成对帧数的提升是要大过winsows的。还可以悄悄告诉大家一消息，小黄鸭已经可以在linux上用啦。接着试试黑神话悟空，黑神话悟空在linux上有一些小问题，开启dlss的话默认就会开启帧生成，没有办法关闭。所以我只能开tsr测试关闭帧生成的帧数。tsr平衡档，默认中画质。两个系统的帧数非常接近，linux略低三四帧，大家觉得这个帧数表现怎么样呢？。差距这么小的话，根据2077的经验，开启帧生成之后linux的帧数应该会反超。果不其然，开启帧生成之后linux的帧数更高。接下来也是一位老朋友，巫师三。据说dx11的游戏在dxvk下运行帧数更高，所以我测试了dx11版本的巫师三，linux的帧数确实明显比windows要高。换成dx12版本之后情况反转，linux略低一些，开启帧生成之后情况又反转啦，linux略胜一筹。这，属实是打得有来有回啊。那在linux上能玩新出的游戏吗？作为游戏玩家，自己期待的游戏肯定是想第一时间玩到的。诶，明末：渊虚之羽，这个游戏节奏很大，不过我是来测试的，节奏什么的先放在一边。帧数情况跟前面的测试结果差不多，不开帧生成的话linux的帧数低一些。开启帧生成之后情况反转，linux的帧数反超。不得不说，这个游戏的优化是真的烂啊，两个系统掉帧都特别严重。剑星也是新登录steam平台的游戏，也可以正常运行，所以大家不用担心新出的游戏linux玩不了。测试结果依旧适用，不开帧生成的情况下linux的帧数略低一些，开启帧生成之后linux帧数反超。我还试了博德之门3、只狼，也都可以正常运行。其他的像星露谷物语、杀戮尖塔、饥荒这样的独立游戏，都可以做到满帧运行。时间有限就不继续测了，这个叫做protonDB的网站上有大量来自其他玩家的真实反馈，可以直接在上面搜索自己想玩的游戏，通过其他玩家的回复就知道能不能玩了。再linux上玩steam单机游戏，挑战成功。



接下来挑战升级，linux能不能玩热门端游呢？当然可以！燕云十六声，这个游戏和之前单机游戏的测试结果类似，linux的帧数比windows要低一些，开启帧生成之后情况反转，这种偏单机的游戏没什么压力，那我再上点强度，大家觉得linux能不能玩大型mmorpg呢？逆水寒可以在linux上玩，帧数完全没问题，不过只有逆水寒极速版能运行，标准版开启之后会报错。下一个，魔兽世界，linux下可以完美运行。既然测了魔兽世界，那就得试试那款竞技游戏啦。守望先锋2，完美运行，帧数甚至要比windows上高很多。永劫无间，可以正常运行，linux上的帧数要比windows高一些，不过有一个小问题，永劫无间在linux上只能运行dx11版本，切到dx12的话就打不开游戏了。

看到这里，你可能有一些手痒啦，想上手试一试自己想玩的游戏能不能在linux上运行，那么具体要怎么做呢？这里稍微科普一下wine和proton相关的知识。wine，全称**W**ine **I**s **N**ot an **E**mulator，它提供了一个兼容层，用来把windows的系统调用转换成linux能够理解的系统调用，同时还提供了windows的运行库和组件的替代品。通俗来说，wine充当了一个翻译，linux有wine帮忙翻译之后就能听懂windows软件说的话了。而proton是wine的一个分支，着重改善3d性能，适合用来玩游戏。proton通常会落后wine好几个版本，于是就有了GE-proton，这是GloriousEggroll大佬结合proton和最新版本的wine制作的，比如现在最新的GE-proton版本号是10.10，和最新的wine稳定版相同。可以用yay -S protonup-qt这个软件管理proton，优先使用GE-proton的最新版本，如果无法运行的话就换其他的试试。值得一提是GE大佬的Wine-GE已经在今年年初宣布弃用了[关于使用 Proton-GE 和 Wine-GE 构建的线程：r/linux_gaming --- A thread about using Proton-GE and Wine-GE builds : r/linux_gaming](https://www.reddit.com/r/linux_gaming/comments/uzrz2k/a_thread_about_using_protonge_and_winege_builds/)，要注意别再使用wine-ge了。

做一个小实验。尝试在steam里添加非steam游戏，右键属性强制使用proton，点击开始游戏之后可以注意到出现了一个新的文件夹。这是wine的一个特点：prefix，前缀。相当于一个虚拟c盘，里面存放着运行windows软件需要的运行库和组件，你的程序会在这个目录下运行。这就是steam用proton创建的prefix，可以看到刚刚安装的软件就存放在这。卸载软件只需要删除这个文件夹，这相当于用删除电脑c盘的方式卸载软件，相当干净。利用这个特性，你甚至可以为每一个windows软件创建单独的prefix。因为用命令行做这件事情比较繁琐，所以就有了prefix管理工具。

我前面运行热门端游用到的就是叫lutris的管理工具。使用这条命令安装lutris，第一次运行会下载各种组件，下载完成后点击这里设置wine的版本。然后点击左上角的加号，选择第二项就可以使用windows的安装包了。这一步指定安装路径，同时也是创建prefix的路径，选择要使用的安装包，确认安装路径在c盘下。安装完成后关闭窗口就会提示安装完成，有时候可能需要手动指定一下主程序的位置。如果有这样的报错或者没有提示安装完成，就点击左上角中止，不要删除文件。点击左上角的加号选择第五项手动添加，找到刚才安装软件时创建的prefix，手动添加主程序，设置prefix目录。如果你要玩epic或者gog上面的游戏的话可以使用这个软件，使用方法和lurtis类似，就不演示了。linux上玩游戏的主要口诀就是，steam有的用steam玩，steam没有的就用lutris这样的管理工具玩。

话说，怎么从刚才开始就一直有音乐啊。好耳熟啊……好像是，米！

原神！启动！

没错，linux也能玩原神！不过，要说明一下，这里运行的是国际服，国服的话游戏是可以正常运行的，但是不让进。好你个米哈游，区别对待是吧。其实可以通过一些特殊手段解决国服的反作弊问题的，但因为有封号的风险，我就不做介绍了。不仅是原神，绝区零也可以玩，这个游戏linux上的帧数甚至大幅领先windows，刚进游戏的时候会因为着色器渲染有一些卡顿，稍微玩一会就相当流畅啦。星穹铁道的用lutris安装的米哈游启动器没法运行，好在有大佬做了这个启动器项目，直接使用lutris提供的安装脚本下载刚才项目里的启动器，就可以正常运行啦。和绝区零一样，刚开游戏会有一些小掉帧，玩一会就好啦，可以满帧运行。好啦，时间有限，就玩到这里。在linux上玩热门端游。挑战成功。



既然谈到米哈游了，那就不得不聊一下在linux上玩手游的问题了。接下来我要挑战在linux上玩安卓手游。安卓系统其实用的也是linux内核，所以在archlinux上运行安卓程序应该也不是什么难事。这里用到的软件叫做waydroid。它会创建一个安卓容器，直接共享本机的linux内核，这让它有了直接调用电脑硬件的能力，所以性能上是完全原生的。使用这条命令安装waydroid，用这条命令从archlinuxcn仓库上下载系统镜像，再使用这条命令初始化，然后开启服务，这样就可以打开waydroid啦。不过由于我们的cpu是x86架构，想运行为arm平台制作的游戏需要使用github上的这个脚本安装arm转译。archwiki推荐amd用这个，intel用这个。然后就可以打开会话，像一台真正的安卓手机那样安装和运行软件啦。从b站上下几个游戏试试。像战双这样自带键盘和手柄支持的游戏是可以在waydriod上完美运行的。明日方舟也可以完美运行。碧蓝航线的摇杆有点问题，没法自己移动小人，其他功能都是正常的，勉强算运行成功吧。想玩谷歌play上的游戏的话可以使用之前那个脚本安装谷歌框架，再去waydroid的官方wiki上复制这一段命令获取id，点进这个链接注册设备，最后重启会话。这样就可以可以玩谷歌play上的游戏啦。玩个国际服影之诗试试，可以完美运行，话说这是影之诗的新作吗，特效还是一如既往的精致啊。就玩到这里吧，总体算是满足了在电脑上玩手游的需求，但是也有部分游戏无法运行，比如国服原神会在开场动画闪退，fgo会卡死在这个页面。waydroid上的按键映射也暂时没有一个开箱即用的解决方案，我使用的是xtmapper这个工具，配置起来比较复杂，有一点超纲了，就不在这期视频聊了，如果大家有什么更简单的方案可以在评论或者弹幕分享。另一个比较大的问题是无法用n卡进行图形渲染，如果你的电脑只有n卡的话就只能运行这条命令编辑配置文件，添加这两行内容，本地更新系统，重启服务，这样就开启了软件渲染。运行是可以正常运行，但是用cpu渲染图形的话就只能玩2d游戏了，3d游戏会很卡。由于n卡的驱动是闭源的，所以就只能看后续nvidia能不能主动做些什么了。如果你有n卡以外的显卡，并且玩的游戏不需要复杂的按键映射，那么waydroid玩安卓手游的体验还是非常不错的。在linux上玩安卓手游，算是挑战成功。



至此，我们已经基本满足了一个游戏玩家的需求，面对任何对linux玩游戏的质疑，我想我已经可以自信地说出：“是的，Linux可以玩游戏”。那么接下来请随我一起愚公移山，移走阻碍linux玩游戏的最后一座大山——反作弊。

我挑选了最具代表性的反作弊程序，分别是小蓝熊EAC（Easy anti cheat），战眼BattleEye，以及最后的终极大魔王，噔噔咚，腾讯ACE（Tencent Anti-Cheat Expert）。Apex用的是EAC，尝试运行，报错啦。绝地求生用的BattleEye，也报错啦。鸣潮使用的是ACE，能运行，但是不让进游戏。wegame的情况有点特殊，用lutris安装wegame会卡在联网这一步，即使复制windows上安装好的wegame运行，抛开ui bug不谈，登进去也没法下载游戏，所以wegame上的所有游戏都玩不了。难道只要有反作弊就无法在linux上运行吗？其实不是的，再看几个例子。艾尔登法环使用的是EAC，可以进线上模式，箴言系统在正常运作。有观众可能会说：”欸！老头环是单机游戏，怎么能算数呢！“ 没关系，糖豆人也采用了EAC，完全可以正常游玩。战争雷霆使用的是battleeye，也可以正常游玩。根据protonDB的反馈，在鸣潮的启动参数里加上steamdeck=1之后就可以完美运行了，所以甚至连腾讯的ACE都是可以玩的。事实证明带反作弊的游戏在linux上是可以玩的，battleEye和EAC早在2021年就已经宣布支持linux，从这个网站上可以看到有大量带反作弊的游戏支持linux，也有大量游戏不支持。所以问题的关键根本不是反作弊，而是厂商的态度。Apex就是最好的例子，原本apex也可以在linux上正常游玩，但是官方以外挂为由完全禁止了proton下的登入，我不太了解apex，有了解的观众可以说明一下，是不是apex在windows上就没有外挂了。至于wegame，大家觉得腾讯会给linux做适配吗？ 

不过，难道厂商不做适配linux就不能玩了吗，我们对wegame就一点办法都没有吗？当然不是！时候掏出上期视频打造出的秘密武器了，在这台直通了4060显卡的kvm虚拟机上，英雄联盟、dnf、瓦洛兰特、三角洲行动，甚至之前运行失败的原神国服、崩坏3、绝地求生、apex，全部都可以正常运行。可能会有观众担心虚拟机导致性能大幅降低，为此我特地进行了基准测试。cinebanch R23 测试cpu性能，显卡直通虚拟机比原生损失了3%左右。3Dmarck time spy测试显卡性能，比起原生损失了6%左右。大家觉得这个损失可以接受吗？看完理论测试，我们来看一下实际游戏的表现。英雄联盟这种对cpu延迟比较敏感的游戏会有较大的损失，帧数下降了有二三十帧。瓦洛兰特的情况和英雄联盟差不多，帧数也下降了几十帧，但是以电竞帧率畅玩肯定是没问题的。像三角洲行动这样对显卡有一定要求的游戏就和原生没什么差别了。apex和三角洲的情况类似，显卡有一定压力之后虚拟机的帧数就跟原生相差无几了。linux玩反作弊游戏，挑战成功！了吗？就在我以为反作弊挑战就此成功的时候，绝地求生给我浇了一盆冷水。游戏可以正常运行，但是进对局没一会儿就会被battleEye检测到虚拟机而踢出。经过一点点学习，我通过编辑xml更好地伪装了虚拟机，绝地求生得以成功运行，这个游戏比较吃内存和显卡，虚拟机的帧数和原生的差不多。顺便一提，我还尝试安装了安卓模拟器，之前没能运行的FGO也能正常玩了。linux玩反作弊游戏，挑战成功！



现在，我真的可以在linux上玩所有自己想玩的游戏了。可是关闭虚拟机之后，我感到的却是一种深深的失落。同时运行两个系统不可避免地会有性能损失。而且提供反作弊服务的公司都明确说过不支持虚拟机，进入这场虚拟机和反作弊之间的猫鼠游戏意味着随时有可能被检测出来封号。我为什么要舍近求远呢？什么不直接重启进入真正的windows呢？或者干脆别用linux了，直接wsl2，这样不是鱼和熊掌兼得吗？我绕这么一大圈，真的有意义吗？

（转场，以2077作为背景）

制作这期视频的时候我一直有上述的自我怀疑，是一次偶然让我找到了答案。eurobattle.net，这是一个欧洲的魔兽对战平台，它是由魔兽争霸的爱好者自发组织和维护的社区项目。它名不见经传，不只是我，我想大部分观众朋友也从来没有听说过这个平台。但是在它的官方wiki上我看到了这样一句话：“Wine on Linux is currently not working, we are in the process of trying to fix this” “暂时无法在linux上用wine运行，我们正在尝试解决这个问题。” 看到这句话那一刻，仿佛阳光驱散清晨的迷雾，让我豁然开朗。

一个非盈利的、不为人知的社区项目在努力适配wine。而那些拥有顶尖技术力人才、手握重金的商业巨头，却选择砌起高墙，把我们拒之门外。比起70%市场占有率的windows，linux4%的占比显得多么微不足道。还记得我之前说的话吗：“问题的关键根本不是反作弊，而是厂商的态度。”  现在我重新说一次：“阻碍我们的从来不是反作弊，而是冰冷的、唯利是图的商业逻辑。” 思考到这，脑海中浮现出之前在reddit上看到的一段话：“ 我曾经也因为自己想玩的游戏在linux上无法运行而烦恼，后来我意识到，如果一款游戏无法在linux上运行，那么就不值得玩它。” 我忽然明白，这不是傲慢，而是一种——斗争。

（转赛博朋克混剪）

（最后黑屏出现stallman的引用）

“如果用户不能控制程序，则会被程序控制。专有软件始终存在所有者。所有者控制程序并通过程序对用户行使权力。非自由软件是一种枷锁，是不公正力量的工具。” —— Richard Stallman



### 为看到这里的观众献上彩蛋。

#### 小黄鸭的使用方法

小黄鸭linux版的1.0正式版刚刚在7月31日发布。现在可以在linux上愉快地使用小黄鸭补帧啦。

养成好习惯，做自己不了解的事情之前先创建快照

由于需要用到小黄鸭的dll文件，所以先从steam下载小黄鸭，然后从aur下载lsfg-vk。

使用方法比较多，我这里只介绍其中一种，有兴趣的可以自己看官方文档。

打开lsfg-vk-ui，左下角新建一个profile，任意取一个英文的profile name，比如我取一个miyu。这里设置补帧倍率，官方文档说推荐启动性能模式。配置完成，在启动程序的时候加上这个环境变量LSFG_PROCESS="miyu"就可以激活小黄鸭读取刚刚的配置文件啦。

看视频的话推荐使用mpv，设置环境变量，空格mpv，指定视频的路径，回车启动。

steam游戏的话在启动参数里添加环境变量，lutris之类的软件和steam同理。配置的内容是可以实时更改实时应用的。

1.0版本难免会出现一些bug，如果遇到的问题的话可以去github上提交。

#### mangohud使用方法

测试过程中使用到的帧数显示工具叫mangohud。

从aur安装mangohud，再安装mangojuice用来配置具体的显示内容。

使用的时候只需要在启动参数里加上mangohud就可以啦。

lutris之类的工具提供了一个mangohud的开关，激活开关就可以使用啦。

#### looking glass使用方法

上一期视频我配置完显卡直通之后采用了sunshine+moonlight的串流方案，优点是全平台串流，但毕竟涉及到网络传输和视频编解码，性能上是有损失的。所以这期视频我换成了性能更强，延迟更低的looking glass，这是一个通过共享内存获取显卡直通虚拟机画面的程序。

用这条命令添加自己到kvm组里面，记得重启电脑。然后用groups确认添加成功。

然后在virt-manager的首选项里开启xml编辑。配置了显卡直通的虚拟机的设置里面来到xml的底部，找到device，在device的上面粘贴这一段内容添加一个共享内存设备。设备的大小需要根据分辨率计算，我直接往大了填啦，不在乎这点内存，具体计算过程可以看屏幕上这张图。

然后需要编辑一下配置文件。粘贴这段内容，f 代表定义文件规则，这个是刚刚那个共享内存设备对应的文件，0660意思是指定的所有者和组拥有读写权限，这里是所有者的名字填你的用户名，kvm是组， - 代表文件的保留时间无限。然后用这段命令创建文件。

回到虚拟机设置，设置虚拟机键鼠、剪贴板共享和音频。确认有spice显示协议，显卡设置为none。添加virtio的键盘，再添加virtio的鼠标，需要编辑xml把这里改成virtio，这一步是键鼠传输。确认有spice信道设备，设备类型为spice，这一步是开启剪贴板同步。确认有ich9声卡，点击概况来到xml底部，找到这一段，把类型改为spice，这一步是声音传输。

接着把memballoon这一行改为none，这一步是禁用memballon。memballon的作用是更有效率地利用虚拟机的空闲内存，但是memballon对内存的挪用反而导致显卡直通内存性能大幅下降，所以要关闭。

开启虚拟机，来到looking glass的官网下载服务端，既然是arch，那就要使用bleeding-edge版本。looking glass是需要virtual display driver安装一个虚拟显示器的，具体安装方法可以看上期视频。

来到主机，从aur安装客户端，客户端和服务端的版本要对应才能能连接上，bleeding-edge对应的是git包。

乘着他在build，编辑looking glass的配置文件，默认功能键是scroll lock，写入这段内容自定义功能键，可用的功能键可以运行这条命令查看。

开启looking glass后单击功能键可以把键鼠锁定在looking glass里，长按功能键可以看到很多功能，留给你们探索吧。最后一部，克隆虚拟机之后使用克隆机而不是原型机。

感谢观看，我是林长枫，我们下期视频，再见。





# 参考链接：

steam单机部分

[ilya-zlobintsev/LACT: Linux GPU Configuration And Monitoring Tool](https://github.com/ilya-zlobintsev/LACT)

[seerge/g-helper: Lightweight Armoury Crate alternative for Asus laptops and ROG Ally. Control tool for ROG Zephyrus G14, G15, G16, M16, Flow X13, Flow X16, TUF, Strix, Scar and other models](https://github.com/seerge/g-helper)

[Steam — 高质量游戏平台](https://store.steampowered.com/about/)

[Arch Linux 与 Valve 合作，让 Steam 游戏体验更上一层楼 --- Arch Linux and Valve team up to make Steam gaming even better](https://www.xda-developers.com/arch-linux-valve-team-up/)

[PancakeTAS/lsfg-vk: Lossless Scaling Frame Generation on Linux](https://github.com/PancakeTAS/lsfg-vk)

[Using DXVK on Windows to improve Final Fantasy XIV performance](https://rkblog.dev/posts/ffxiv/using-dxvk-on-windows-to-improve-final-fantasy-xiv-performance/)

[ProtonDB | Gaming know-how from the Linux and Steam Deck community](https://www.protondb.com/)



热门端游部分

[WineHQ - 在 Linux，BSD，Solaris 和 macOS 上运行 Windows 应用](https://www.winehq.org/)

[Wine - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/Wine)

[Proton (软件) - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/Proton_(%E8%BB%9F%E9%AB%94))

[如何管理 Wine/Proton 版本：r/linux_gaming --- How to manage Wine/Proton Version : r/linux_gaming](https://www.reddit.com/r/linux_gaming/comments/138k49m/how_to_manage_wineproton_version/)

[GloriousEggroll/proton-ge-custom: Compatibility tool for Steam Play based on Wine and additional components](https://github.com/GloriousEggroll/proton-ge-custom)

[GloriousEggroll/wine-ge-custom: My custom build of wine, made to use with lutris. Built with lutris's buildbot.](https://github.com/GloriousEggroll/wine-ge-custom)

[A thread about using Proton-GE and Wine-GE builds : r/linux_gaming](https://www.reddit.com/r/linux_gaming/comments/uzrz2k/a_thread_about_using_protonge_and_winege_builds/)

[Lutris - Open Gaming Platform](https://lutris.net/)

[Heroic Games Launcher](https://heroicgameslauncher.com/)

[An Anime Team](https://github.com/an-anime-team)



手游部分

[内核概览  |  Android Open Source Project](https://source.android.com/docs/core/architecture/kernel?hl=zh-cn)

[Waydroid | Android in a Linux container](https://waydro.id/)

[Waydroid - ArchWiki](https://wiki.archlinux.org/title/Waydroid)

[Xtr126/XtMapper: Keyboard and mouse input mapper](https://github.com/Xtr126/XtMapper)

[Linus Torvalds To Nvidia - "Fuck You" - YouTube](https://www.youtube.com/watch?v=_36yNWw_07g)

[anbox/anbox: Anbox is a container-based approach to boot a full Android system on a regular GNU/Linux system](https://github.com/anbox/anbox)

反作弊部分

[Easy Anti-Cheat](https://www.easy.ac/en-US)

[BattlEye – The Anti-Cheat Gold Standard](https://www.battleye.com/)

[游戏安全 | 腾讯云](https://www.tencentcloud.com/zh/products/ace?lang=zh&pg=)

[Are We Anti-Cheat Yet?](https://areweanticheatyet.com/)

[X 上的 Apex Legends：“Dev Team Update: Linux & Anti-Cheat” / X](https://x.com/PlayApex/status/1852019667315102151)

[开发团队更新：Linux & 反作弊 | EA 论坛 - 9468053 --- Dev Team Update: Linux & Anti-Cheat | EA Forums - 9468053](https://forums.ea.com/blog/apex-legends-game-info-hub-en/dev-team-update-linux--anti-cheat/9468053)

[《Epic Online Services 推出对 Linux、Mac 和 Steam Deck 的反作弊支持 - Epic Online Services》 --- Epic Online Services launches Anti-Cheat support for Linux, Mac, and Steam Deck - Epic Online Services](https://onlineservices.epicgames.com/en-US/news/epic-online-services-launches-anti-cheat-support-for-linux-mac-and-steam-deck)

[Steamworks Development - 关于 BattlEye + Proton 支持的更新 - Steam 新闻](https://store.steampowered.com/news/group/4145017/view/3104663180636096966)





虚拟机部分

[PCI passthrough via OVMF - ArchWiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF)

[「Archlinux究极指南」从手动安装到显卡直通，最后删除Linux_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1L2gxzVEgs/?spm_id_from=333.1387.homepage.video_card.click)

[How to play PUBG (with BattleEye) on a Windows VM : r/VFIO](https://www.reddit.com/r/VFIO/comments/18p8hkf/how_to_play_pubg_with_battleeye_on_a_windows_vm/)

[虚拟化调整和优化指南 | Red Hat Enterprise Linux | 7 | Red Hat Documentation](https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/7/html-single/virtualization_tuning_and_optimization_guide/index)




结尾部分

[BattlEye on X: "We want to emphasize that we do not ban anyone for simply running the game in a VM, but as always we will ban any users who actively try to bypass our measures. Normal users will only receive a kick (like @OrdinaryGamers). (3/3)" / X](https://x.com/TheBattlEye/status/1289027890227621889)

[X 上的 BattlEye：“We are currently seeing a lot of misinformation regarding our VM-related countermeasures in several games. We understand that some players might want to play the game in a VM, e.g. if they are using Linux, but we hope for your understanding that we can't support such (1/3)” / X](https://x.com/TheBattlEye/status/1289027672186720263)

[使用反作弊接口 |Epic 在线服务开发人员 --- Using the Anti-Cheat Interfaces | Epic Online Services Developer](https://dev.epicgames.com/docs/game-services/anti-cheat/using-anti-cheat?sessionInvalidated=true)

[Eurobattle.net | Community](https://eurobattle.net/)

[Desktop Operating System Market Share Worldwide | Statcounter Global Stats](https://gs.statcounter.com/os-market-share/desktop/worldwide/)

[为什么软件应该是自由的 - GNU工程 - 自由软件基金会](https://www.gnu.org/philosophy/shouldbefree.zh-cn.html)



[flightlessmango/MangoHud: A Vulkan and OpenGL overlay for monitoring FPS, temperatures, CPU/GPU load and more. Discord: https://discordapp.com/invite/Gj5YmBb](https://github.com/flightlessmango/MangoHud)

[LizardByte/Sunshine: Self-hosted game stream host for Moonlight.](https://github.com/LizardByte/Sunshine)

[Looking Glass - Home](https://looking-glass.io/)