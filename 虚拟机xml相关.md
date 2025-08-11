## 虚拟机性能优化

我在使用的过程中，最大的感受就是cpu和gpu以近乎原生的性能在运行，但是内存性能很差、非常差、难以置信的差。在研究looking glass的时候偶然解决了这个问题。元凶是memballoon

### 禁用memballoon

[libvirt/QEMU Installation — Looking Glass B7 documentation](https://looking-glass.io/docs/B7/install_libvirt/#memballoon)

memlbaloon的目的是提高内存的利用率，但是由于它会不停地“取走”“归还”虚拟机内存，导致显卡 直通时虚拟机内存性能极差。

将虚拟机xml里面的memballoon改为none，这将极大极大极大地！！！提高显卡直通虚拟机的性能。总体可以达到9成原生的性能。

```
<memballoon model="none"/>
```
### 可选：内存大页

[KVM - Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/KVM#%E5%BC%80%E5%90%AF%E5%86%85%E5%AD%98%E5%A4%A7%E9%A1%B5)

没感觉到有提升，wiki说有，那姑且设置一下

- 计算大页大小

内存（GB）* 1024 / 2 = 需要的大小

比如16GB内存就是16*1024/2=8192，wiki建议略大一些，那就8200。

我通常给虚拟机分24GB内存，24*1024/2=12288，略大一些就是12300。

- 编辑虚拟机xml

在virt-manager的g首选项里开启xml编辑，找到```<memoryBacking>```并添加```<hugepages/>```：
```
  <memoryBacking>
    <hugepages/>
  </memoryBacking>
```
- 永久生效

记得把数字改成自己需要的

```
sudo vim /etc/sysctl.d/40-hugepage.conf

vm.nr_hugepages = 8800
```

- reboot
- 虚拟机开启后查看大页使用情况

```
grep HugePages /proc/meminfo
```

- 取消大页

```
sudo rm /etc/sysctl.d/40-hugepage.conf

reboot
```

### cpupin

[PCI passthrough via OVMF - ArchWiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#CPU_pinning)

设置cpu拓扑和自己的cpu核心线程数对应。然后把虚拟机的cpu线程绑定到物理线程上，避免虚拟机cpu线程对应的物理cpu线程变化导致缓存性能下降。

1. 查看物理cpu拓扑

   ```
   lscpu -e
   ```

   输出结果里的cpu是线程，core是线程对应的物理核心。如果开启超线程的话会出现一个core对应多个cpu的情况。

2. 修改xml，在```  <vcpu placement="static">16</vcpu>```下方插入

   ```
     <cputune>
       <vcpupin vcpu="0" cpuset="0"/>
       <vcpupin vcpu="1" cpuset="8"/>
       <vcpupin vcpu="2" cpuset="1"/>
       <vcpupin vcpu="3" cpuset="9"/>
       <vcpupin vcpu="4" cpuset="2"/>
       <vcpupin vcpu="5" cpuset="10"/>
       <vcpupin vcpu="6" cpuset="3"/>
       <vcpupin vcpu="7" cpuset="11"/>
       <vcpupin vcpu="8" cpuset="4"/>
       <vcpupin vcpu="9" cpuset="12"/>
       <vcpupin vcpu="10" cpuset="5"/>
       <vcpupin vcpu="11" cpuset="13"/>
       <vcpupin vcpu="12" cpuset="6"/>
       <vcpupin vcpu="13" cpuset="14"/>
       <vcpupin vcpu="14" cpuset="7"/>
       <vcpupin vcpu="15" cpuset="15"/>
     </cputune>
   ```

   数字需要修改成自己对应的。虚拟机有几个线程就写几行vcpu，0算第一个。上述这个配置声明了16个虚拟机cpu线程（vcpu）。cpuset设置vcpu对应的本机cpu线程，也就是```lscpu -e```输出结果里的cpu那一列。

   我的cpu支持超线程，cpu0和cpu8对应的是核心0。于是我把vcpu的0和1，绑定到cpu0和8上，以此类推。

### 伪装虚拟机

[How to play PUBG (with BattleEye) on a Windows VM : r/VFIO](https://www.reddit.com/r/VFIO/comments/18p8hkf/how_to_play_pubg_with_battleeye_on_a_windows_vm/)

为了避免被反作弊程序检测到虚拟机，需要修改xml伪装虚拟机。

### 警告：每进行一步都要确认虚拟机能正常运行再进行下一步

1. 使用sata硬盘和e1000网卡

2. 在```</hyperv> ```下面一行插入：

```
<kvm>
<hidden state="on"/>
</kvm> 
```

3. 在``` <os firmware="efi">```上面一行插入，这是伪装bios。然后复制xml顶部的uuid，替换下面这段里的【这里要粘贴自己虚拟机的uuid】。里面的name信息可以按需修改。

```
<sysinfo type="smbios">
<bios>
<entry name="vendor">American Megatrends International, LLC.</entry>
<entry name="version">F21</entry>
<entry name="date">10/01/2024</entry>
</bios>
<system>
<entry name="manufacturer">Gigabyte Technology Co., Ltd.</entry>
<entry name="product">X670E AORUS MASTER</entry>
<entry name="version">1.0</entry>
<entry name="serial">12345678</entry>
<entry name="uuid">【这里要粘贴自己虚拟机的uuid】</entry>
<entry name="sku">GBX670EAM</entry>
<entry name="family">X670E MB</entry>
</system>
</sysinfo> 
```
4. 禁用migratable

```
  <cpu mode="host-passthrough" check="none" migratable="off">  
  migratable是为服务器集群准备的“搬家”功能，关闭。
```

5. 在```    <topology sockets="1" dies="1" clusters="1" cores="8" threads="2"/>```下面一行插入

主要是为了伪装成一个友好的hyper-v，调整cpu时钟，修复cpu安全漏洞、设置高级指令集、隐藏cpu虚拟化

```
    <cache mode="passthrough"/>
    <feature policy="require" name="hypervisor"/> 
    <feature policy="disable" name="aes"/>
    <feature policy="require" name="topoext"/>
    <feature policy="disable" name="svm"/> 
    <feature policy="require" name="amd-stibp"/>
    <feature policy="require" name="ibpb"/>
    <feature policy="require" name="stibp"/>
    <feature policy="require" name="virt-ssbd"/>
    <feature policy="require" name="amd-ssbd"/>
    <feature policy="require" name="pdpe1gb"/>
    <feature policy="require" name="tsc-deadline"/>
    <feature policy="require" name="tsc_adjust"/>
    <feature policy="require" name="rdctl-no"/>
    <feature policy="require" name="skip-l1dfl-vmentry"/>
    <feature policy="require" name="mds-no"/>
    <feature policy="require" name="pschange-mc-no"/>
    <feature policy="require" name="invtsc"/>
    <feature policy="require" name="cmp_legacy"/>
    <feature policy="require" name="xsaves"/>
    <feature policy="require" name="perfctr_core"/>
    <feature policy="require" name="clzero"/>
    <feature policy="require" name="xsaveerptr"/>
```

6. 时钟，找到clock offset那段修改，时区可以按需修改，不改也没事。

```
  <clock offset="timezone" timezone="Asia/Japan">
    <timer name="rtc" present="no" tickpolicy="catchup"/>
    <timer name="pit" tickpolicy="discard"/>
    <timer name="hpet" present="no"/>
    <timer name="kvmclock" present="no"/>
    <timer name="hypervclock" present="yes"/>
    <timer name="tsc" present="yes" mode="native"/>
  </clock>
```
