## 一、WiFi相关基础概念

### 1、什么是 `Wi-Fi`

我们看一下百度百科是如何定义的：

`Wi-Fi`是一种可以将个人电脑、手持设备(如pad、手机)等终端以无线方式互相连接的技术，事实上它是一个高频无线电信号。目的是改善基于IEEE 802.11标准的无线网路产品之间的互通性。

> `Wi-Fi` 英文全称是`WIreless-FIdelity`，翻译成中文就是无线保真，英文简称WiFi。无线保真是一个无线网络通信技术的品牌，由Wi-Fi联盟所持有。

有人把使用IEEE 802.11系列协议的局域网就称为无线保真。甚至把无线保真等同于无线网际网路。`实际上Wi-Fi是WLAN的重要组成部分`。

### 2、什么是`WLAN`

> 无线局域网络英文全名：Wireless Local Area Networks；简写为： WLAN。

它是相当便利的数据传输系统，它利用射频(`Radio Frequency - RF`)的技术，使用电磁波，取代旧式碍手碍脚的双绞铜线(`Coaxial`)所构成的局域网络，在空中进行数据通信。

> 该技术的出现绝不是用来取代有线局域网络，而是用来弥补有线局域网络之不足，以达到网络延伸之目的，使得无线局域网络能利用简单的存取架构让用户透过它，实现无网线、无距离限制的通畅网络。

其实很多时候，人们将`WiFi`和`WLAN`二者混用，其实`WiFi仅仅是实现WLAN的一种技术`。

> 其他比较常见的WLAN技术比如有 蓝牙(`BlueTooth`)、wimax等；

### 3、`无线网` VS `有线网`

无线网络相比有线网络，还是有许多的缺点的：

* 通信双方因为是无线通信，所以`通信之前需要建立连接`；而有线网络就直接用线缆连接，不用这个过程了。

* 通信双方通信方式是`半双工的通信方式`；而有线网络可以是全双工。

* 通信时在`网络层以下出错的概率非常高，所以帧的重传概率很大，需要在网络层之下的协议添加重传的机制`(不能只依赖上面TCP/IP的延时等待重传等开销来保证)；而有线网络出错概率非常小，无需在网络层有如此复杂的机制。

* 数据是在无线环境下进行的，所以`抓包非常容易，存在安全隐患`。

* 因为收发无线信号，所以`功耗较大`，对电池来说是一个考验。

* 相对有线网络吞吐量低，这一点正在逐步改善，802.11n协议可以达到600Mbps的吞吐量。

## 二、IEEE 802.11协议

IEEE802协议有点大，下面只介绍其中一些概念：

Ethenet 和 WiFi 采用的协议都属于IEEE 802协议集。其中，

* Ethenet 以`802.3协议`做为其网络层以下的协议；
* 而 WiFi 以 `802.11协议`做为其网络层以下的协议。

无论是有线网络，还是无线网络，其网络层以上的部分，基本一样。

### 1、802.11简介

IEEE802家族是由一系列局域网络(Local Area Network,LAN)技术规格所组成。802.11属于其中一员。
虽然WI-FI使用了802.11的`媒体访问控制层(MAC)` 和 `物理层(PHY)`，但是两者并不完全一致。

IEEE802.11协议族成员如下：

 ![img](assets/20160409104536286.png)

802.11基本规格涵盖了802.11 MAC 以及两种物理层(physical layer)：

* 一是`跳频展频(frequency-hopping spread-spectrum，简称FHSS)`物理层，
* 另一是`直接序列展频(direct-sequence spread-spectrum，简称DSSS)`物理层。

`802.11a` 所规范的物理层，主要是以`正交分频多工(orthogonal frequency division multiplexing，简称OFDM)`技术为基础.

802.11将PHY进一步划分为两个组成元件：

* 一是物理层`收敛程序(Physical Layer ConvergenceProcedure，简称PLCP)--` 负责将MAC帧对映到传输介质
* 另一是`实际搭配介质Physical Medium Dependent，简称PMD)--`负责传送这些帧。

### 2、802.11b

IEEE802.11b是无线局域网的一个标准。其`载波的频率为2.4GHz，传送速度为11Mbit/s`。

IEEE802.11b是所有无线局域网标准中最著名，也是普及最广的标准。

> 它有时也被错误地标为Wi-Fi。实际上Wi-Fi是无线局域网联盟(WLANA)的一个商标，该商标仅保障使用该商标的商品互相之间可以合作，与标准本身实际上没有关系。

在2.4-GHz-ISM频段共有14个频宽为22MHz的频道可供使用。IEEE802.11b的后继标准是`IEEE802.11g，其传送速度为54Mbit/s。`

### 3、802.11网络包含四种主要实体原件

 ![img](assets/20160409105245468.png)

* `工作站(Station) --`  具有无线网络接入功能的电子设备(笔记本，手持设备等).
* `基站(Access Point) --` 802.11网络所使用的帧必须经过转换才能被传到其它不同类型的网络，具有无线至有线桥接功能的设备称为基站(Access Point,AP).此外基站还有其它功能.
* `无线介质(Wireless Medium) --`802.11标准以无线介质(Wireless medium)在工作站之间传递帧.其所定义的物理层不只一种.
* `传输系统(Distribution System) --`传输系统是基站间转送帧的骨干网络(`backbone network`)。当几部基站串连以覆盖较大区域时，彼此之间必须相互通信，才能够掌握移动式工作站的行踪。而传输系统(distribution system )属于802.11的逻辑元件，负责将帧(frame)转送至目的地。

  > 大多数商用产品，是以`桥接引擎(bridging engine)`和`传输系统介质(distribution system medium)`共同组成传输系统.

### 4、802.11 工作方式

802.11定义了两种类型的设备：

* 一种是`无线站 --` 通常是通过一台PC机器加上一块无线网络接口卡构成的，
* 另一个称为`无线接入点(Access Point, AP) --` 它的作用是提供无线和有线网络之间的桥接。

一个无线接入点通常由`一个无线输出口`和`一个有线的网络接口(802.3接口)`构成，`桥接软件符合802.1d桥接协议`。

接入点就像是无线网络的一个无线基站，将多个无线的接入站聚合到有线的网络上。

> 无线的终端可以是802.11PCMCIA卡、PCI接口、ISA接口的，或者是在非计算机终端上的嵌入式设备(例如802.11手机)。

 ![img](assets/20160409110100636.png)

802.11的数据链路层由两个之层构成，`逻辑链路层LLC(Logic Link Control)` 和 `媒体控制层MAC(Media Access Control)`。

802.11使用和802.2完全相同的LLC层和802协议中的48位MAC地址，这使得无线和有线之间的桥接非常方便。但是`802.11的MAC地址只对无线局域网唯一`。  

802.11的MAC和802.3协议的MAC非常相似，都是在一个共享媒体之上支持多个用户共享资源，由发送者在发送数据前先进行探测网络的可用性。

> 在802.3协议中，是由`CSMA/CD(Carrier Sense Multiple Access with Collision Detection)协议`来完成调节，这个协议解决了在Ethernet上的各个工作站如何在线缆上进行传输的问题，利用它检测和避免当两个或两个以上的网络设备需要进行数据传送时网络上的冲突。在802.11无线局域网协议中，冲突的检测存在一定的问题，这个问题称为"Near/Far"现象，这是由于要检测冲突，设备必须能够一边接受数据信号一边传送数据信号，而这在无线系统中是无法办到的。

## 三、WiFi相关知识进阶

### 1、频谱划分

WiFi总共有14个信道，如下图所示：

 ![img](assets/20160409111045226.png)

1. IEEE 802.11b/g标准工作在2.4G频段，频率范围为2.400—2.4835GHz，共83.5M带宽
2. 划分为14个子信道(CN使用前13个,14信道仅在JP使用)
3. 每个子信道宽度为22MHz
4. 相邻信道的中心频点间隔5MHz
5. 相邻的多个信道存在频率重叠(如1信道与2、3、4、5信道有频率重叠)
6. 整个频段内只有3个(`1`、`6`、`11`)互不干扰信道

### 2、SSID和BSSID

* `基本服务集(BSS -- Basic Service Set)`

  基本服务集是802.11 LAN的基本组成模块。能互相进行无线通信的STA可以组成一个BSS(Basic Service Set) 。如果一个站移出BSS的覆盖范围，它将不能再与BSS的其它成员通信。

  > BSSID是一个BSS的标识，BSSID实际上就是`AP的MAC地址`。
  >
  > 在同一个AP内BSSID和SSID`一一映射`。(同一个AP的内部有多个BSSID则对应着多个SSID)

   ![img](assets/53fdba9b9467c.png)

* `扩展服务集(ESS -- Extend Service Set)`

  多个BSS可以构成一个扩展网络，称为扩展服务集(ESS)网络，一个ESS网络内部的STA可以互相通信，是采用相同的SSID的多个BSS形成的更大规模的虚拟BSS。连接BSS的组件称为分布式系统(Distribution System，DS)。

* `服务集的标识SSID -- Service Set Identifier`

  > 在同一ESS内的所有STA和AP必须具有相同的SSID，否则无法进行通信

在一个ESS内SSID是相同的(也可以认为`SSID是一个ESS的网络标识(如:TP_Link_1201)`)，但对于ESS内的每个AP与`SSID`对应的BSSID是不相同的。`如果一个AP可以同时支持多个SSID的话，则AP会分配不同的BSSID来对应这些SSID。`

BSSID(MAC)<---->SSID 映射关系如下图:

 ![img](assets/20160409111309937.png)

### 3、无线接入过程三个阶段

STA(工作站)启动初始化、开始正式使用AP传送数据帧前，要经过三个阶段才能够接入:

> 802.11MAC层负责客户端与AP之间的通讯，功能包括扫描、接入、认证、加密、漫游和同步等功能

1. 扫描阶段(`SCAN`)
2. 认证阶段 (`Authentication`)
3. 关联(`Association`)

 ![img](assets/20160409111404453.png)

### 4、WiFi - 组成结构

一般架设无线网络的基本配备就是`无线网卡`及`一台AP`，如此便能以无线的模式，配合既有的有线架构来分享网络资源，架设费用和复杂程度远远低于传统的有线网络。

如果只是几台电脑的对等网，也可不要AP，只需要每台电脑配备无线网卡。

AP为`Access Point`简称，一般翻译为“`无线访问接入点`”，或“`桥接器`”。它主要在`媒体存取控制层MAC`中扮演无线工作站与有线局域网络的桥梁。

有了AP，就像一般有线网络的Hub一般，无线工作站可以快速且轻易地与网络相连。

特别是对于宽带的使用，无线保真更显优势，有线宽带网络(ADSL、小区LAN等)到户后，连接到一个AP，然后在电脑中安装一块无线网卡即可。

普通的家庭有一个AP已经足够，甚至用户的邻里得到授权后，则无需增加端口，也能以共享的方式上网。
