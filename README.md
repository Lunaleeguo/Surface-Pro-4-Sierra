# Surface-Pro-4-Sierra
微软 Surface Pro 4-macOS Sierra 10.12 黑苹果安装教程



## 机器配置


主板                微软 Surface Pro 4 ( 英特尔 PCI 标准主机 CPU 桥 - 100 Series 芯片组 )




## 准备工作

- 安全启动`Secure Boot`关掉
- `BitLocker`关掉
- 从U盘启动的方法：长按音量减键保持，然后轻按一下开机键，直到出现Clvoer界面时就可松开音量键了
- **安装Clvoer时不要安装CsmVideoDxe-64.efi，drivers64UEFI里检查下有的要删除，不删除会黑屏**



## 安装说明

**安装原版系统的原则是：无须`DSDT`和`SSDT`，配置和驱动文件要尽量精简，便于后期五国查错**

- `drivers64UEFI`文件包含：

	> DataHubDxe-64.efi
	> EmuVariableUefi-64.efi
	> Fat-64.efi
	> OsxAptioFix2Drv-64.efi
	> PartitionDxe-64.efi
	> VBoxHfs-64.efi


- kexts中的`Others`文件包含：

	> FakeSMC.kext
	> VoodooPS2Controller.kext
	> USBInjectAll.kext



## config配置需注意的几点


### NVMe直接Patch

自从10.12以后不需要安装单独的NVMe驱动了，直接利用Clvoer的Patch功能即可，**特别注意安装不同版本对应的Patch不完全相同**


### Iris 540在安装时ig-platform-id注入为0x12345678

目前HD 520/530/540显卡要想驱动一般要注入`ig-platform-id：0x19160000`，有的机型`DVMT`预读显存和苹果规定的大小不一致，就容易在安装过程中卡AppleIntelSKLGraphicsFramebuffer，这里远景论坛里面也有各种各样的解决办法。对于非Surface Pro 4的机器，大家可以借鉴一些解决办法，以下是在远景论坛上搜集的一些解决方法：       
                     
> - 法一：有直接在BIOS里将DVMT改为96M以上
> 这个办法可以但是前提是要Bios里有这个修改选项，Pro4里就没有这个选项
> 
> - 法二：有的是通过直接升级Bios解决的 
> 其他机器可能可行，但是Pro 4目前看来是没法升级Bios的
> 
> - 法三：有利用Clover的Patch直接对AppleIntelSKLGraphicsFramebuffer打二进制补丁解决的   
> 正常情况下，Clover里Patch过后就能解决问题，实际上Clover的Patch功能经常抽风，远景论坛上大把的人打了补丁还是卡这儿，Pro4同样不行
> 
> - 法四：有的干脆直接上懒人版，然后替换SLE下的自己修改过的AppleIntelSKLGraphicsFramebuffer.kext  
> 这个办法一般情况下是能够解决问题的，但是考虑到要用到懒人版，而且还得装HFS+这个软件，容易造成HFS分区不稳定，不是很建议大家使用



**综上所述，个人认为目前解决卡`AppleIntelSKLGraphicsFramebuffer`最好的办法就是直接仿冒一个无用的显卡ID如：`fakeID=0X12345678`，也可以不是这个，只要仿冒一个无用的显卡ID即可）就行，目的是保证在初次安装系统时不加载显卡驱动。等安装完毕进入系统后再替换修改的`AppleIntelSKLGraphicsFramebuffer.kext`，然后修复权限即可。具体操作流程分两步进行**

#### **Step 1：**

初次安装，仿冒无用显卡ID以进入系统，`config`注入`ig-platform-id：0x12345678`，代码如下： 
 
	<key>Graphics</key>
	        <dict>
	                <key>Inject</key>
	                <dict>
	                        <key>ATI</key>
	                        <false/>
	                        <key>Intel</key>
	                        <true/>
	                        <key>NVidia</key>
	                        <false/>
	                </dict>
	                <key>NvidiaSingle</key>
	                <false/>
	                <key>VRAM</key>
	                <integer>128</integer>
	                <key>ig-platform-id</key>
	                <string>0x12345678</string>
	        </dict> 

#### **Step 2：**

利用原版镜像安装完成后注意：安装完成后替换`S/L/E`下的`AppleIntelSKLGraphicsFramebuffer.kext`，然后把`ig-platform-id`修改为注入为`0x19160000`，修复权限重启后即可驱动`Iris HD 540`，`config`注入代码如下：

	<key>Graphics</key>
	        <dict>
	                <key>Inject</key>
	                <dict>
	                        <key>ATI</key>
	                        <false/>
	                        <key>Intel</key>
	                        <true/>
	                        <key>NVidia</key>
	                        <false/>
	                </dict>
	                <key>NvidiaSingle</key>
	                <false/>
	                <key>VRAM</key>
	                <integer>128</integer>
	                <key>ig-platform-id</key>
	                <string>0x19160000</string>
	        </dict> 

至此，Surface Pro 4通过仿冒无用显卡ID先安装原版镜像，进入系统后，替换修改版`AppleIntelSKLGraphicsFramebuffer.kext`并修复权限，再注入正确的显卡ID即可驱动显卡，并避免了在安装过程中卡`AppleIntelSKLGraphicsFramebuffer`的问题。


---- 
## 安装完成后对系统进行修正


### - ALC298声卡修正

- ALC298声卡的驱动

	通过从[vit9696]()[1]()的主页上下载`AppleALC`的源码，保留`ALC298`的相关文件，删除其他无用的文件，并利用`Xcode`编译得到ALC298的仿冒声卡驱动`AppleALC.kext`，然后注入声卡ID为3即可。其中，注入声卡ID有两种方法，任选其一即可：

1. 方法一：利用Clover直接注入：

		<key>Devices</key> 
		     <dict> 
		          <key>Audio</key> 
		          <dict> 
		               <key>Inject</key> 
		               <string>3</string> 
		          </dict> 
		     </dict> 

2. 方法二：利用Rehabman的`HotPatches`直接通过`SSDT`注入：

	在`SSDT-Config.dsl`修改`Name(AUDL, 你的id十进制)`，当然，对于Surfacre Pro 4则为`Name(AUDL, 3)`，然后编译成aml文件，放回`ACPI/patched`

	- ALC声卡唤醒无声的解决

		直接利用Rehabman的`CodecCommander.kext`驱动便可解决


---- 
### - 亮度修复

利用Rehabman的`HotPatches`加入`SSDT-PNLF.aml`放入`ACPI/patched`并配合`IntelBacklight.kext`，实现亮度可调


---- 
### - 电池电量修复

- 电池电量修复分两步进行：
	- Step 1 ：
	下载MaciASL，并添加Rehabman的补丁源网址：[http://raw.github.com/RehabMan/Laptop-DSDT-Patch/master][3]
	- Step 2 ：
	找到`bat  Surface Pro v4`,打上对应的补丁，并配合`ACPIBatteryManager.kext`实现电池电量显示


---- 
### - 网卡修正

- 点击右上角的WiFi图标，选择最后一项，在左边列表删除掉所有网络。
- 终端执行`sudo rm /Library/Preferences/SystemConfiguration/NetworkInterfaces.plist`
- 等系统重启完了，点击右上角的WiFi图标，选择最后一项，按顺序重新添加**以太网**，**Wi-Fi**，应用。蓝牙可以不添加，之后自动会加的。
- 利用Rehabman的`HotPatches`加入`ssdt-rmne.aml `放入`ACPI/patched`并配合`NullEthernet.kext`，实现App Store无障碍登录。


---- 
### - TF卡读卡器的修正

配合`GenericUSBXHCI.kext`，可以完美使用TF读卡器，并且USB 3.0以及拓展坞的正常使用


---- 
### - 电源管理及变频的修正

由于苹果在 Skylake 平台已经不再使用 `AppleLPC` 机制，所以不再需要加载 AppleLPC 。特别的，在新的 `Skylake` 平台下，也不用像以前一样利用脚本产生SSDT来变频了，新平台下无须SSDT，直接在`Clover中的CPU`选项中开启`HWPEnable`即可，或者直接利用`HWPEnabler.kext`实现变频。

经过实际测试，在Surface Pro 4 上，利用`HWPEnable`可实现12级变频调节。

## 完成情况

- NVMe SSD可用
- Iris 540显卡驱动，HIDPI模式开启
- 亮度可调节
- 电池电量显示正常
- 声卡ALC298可用，且唤醒有声
- 睡眠唤醒正常，合盖睡眠正常
- 电源管理可用，变频正常
- USB3.0正常，TF卡读卡器可用，包括扩展坞可以正常使用
- 有线网卡正常驱动
- Type Cover键盘可用



## 目前无解

- Marvell的无线蓝牙二合一卡均无解
- 触控无解，Surface Pen无法使用
- 前后摄像头+红外线无解   



## 特别鸣谢

- [RehabMan][4]
- [ Piker-Alpha ][5]
- [ vit9696 ][6]

[3]:	http://raw.github.com/RehabMan/Laptop-DSDT-Patch/master
[4]:	https://github.com/RehabMan
[5]:	https://github.com/Piker-Alpha
[6]:	https://github.com/vit9696