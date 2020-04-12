# Comprendre les vulnérabilités des réseaux

### Importance des réseaux

1. 完全隔离一个SI

   如何安置code

2. 双极网络 Diode réseau

   可以收到信息但是没办法发送

   https://en.wikipedia.org/wiki/Unidirectional_network

   * 可变光纤 + 代理
   * Modèle de Bell et LaPadula 

3. systèmes connectés

   TCP/IP

### Vulnérabilité des réseaux

攻击的目的和方式

### Définition d'Internet

* 不同网络之间的互联

* 包寻址

  如果有一个死了，就会有另一个代替

* Contrôle de bout en bout

* TCP/IP协议

### 互联网安全

可以有弹性安全

### Choix de conception

* 包中有丰富的信息

* 没有加密的信息

* 缺少planification

  我们不知道他经过哪个trace

* 在连接互联网之前，自治网络缺少安全控制

### Absence de sécurité intrinsèque 内部的安全缺陷

* 大部分的IP协议sont vieux
* 没有认证
* 没有加密
* 没有签名

### Propagation

源是可以携带的 - bug就是可以携带的

实现现在是不同的，所以reconnect

有不同版本的TCP现在正在运行

### Rappels sur Internet

### Architecture inconnue

物理层 - 没有



#### Routage de paquets IP

我们不知道它经过哪个节点

### TCP

* 在IP上建立MC

* TCP建立连接的三次握手

  ![image-20191122103638548](/Users/haida/Library/Application Support/typora-user-images/image-20191122103638548.png)

* gestion des retransmissions

  ![image-20191122103700714](/Users/haida/Library/Application Support/typora-user-images/image-20191122103700714.png)

* 适应récepteur的接受能力 - 滑窗

  通过随时定义滑窗的大小

  ![image-20191122103824901](/Users/haida/Library/Application Support/typora-user-images/image-20191122103824901.png)

* 断开连接的四次握手

#### 管理拥塞

### DNS

等级化 + 分发

没有安全所以用DNSSEC 但是依然有问题

类似于信任链，层层询问上级DNS

### TCP包结构

![image-20191122104136998](/Users/haida/Library/Application Support/typora-user-images/image-20191122104136998.png)

équipement de sécurité - checksum

### IP结构

![image-20191122104406437](/Users/haida/Library/Application Support/typora-user-images/image-20191122104406437.png)

### 封装

### 包中的易损点

#### 源地址

* 可能填入一个假地址

#### TTL

* 决定源操作系统根据初始值

#### Checksum

占用了很多时间

### Fragmentation IP

问题是顺序混乱，重复

#### Exploitation



* 如果收到防火墙消极的恢复，可以重新写包头部
* 通过限制MTU的大小可以让攻击者发送的包被过滤掉

### IP选项

#### 和路由有关的选项

#####  RR 

可以储存之后的路由

##### Timestamp

可以记录包经过路由器的时间

要找到正确的包序号

à la plage

用来初始化算法，序列序号的问题

#### 用来控制流的选项

##### ECN

如果一个包超负荷了，会把这个拥塞

所有TCP协议的兼容性

#### Conséquence

* 可以放慢路由的速度
* 创建拥塞

# 在一个目标上搜索信息

一个黑客是如何搜寻信息的

是针对information publique

whois - zone exploitée

telnet

### 私人信息

### 搜索信息的工具

这些工具是为了攻击者还是为了管理者？

越是简单，快速强大的工具越能留下踪迹

#### Sniffer - 探测器

提供交换包的内容

需要访问网络，系统在二层获取流量

* tcpdump

* wireshark

* kismet

  用来无线802.11网络

  检测入侵

#### Identification des services - 识别服务

扫描端口

#### 服务识别的工具

扫描端口

* nmap

  开放的端口

* POF

  确定操作系统在没有发送包的情况下

* queso

#### methode

暴露公共信息

* traceroute

  avec一个icmp报文

* firewalk

### Outils intégrés

openvas 保证没有faille

* 易损性测试的数据库
* 有规律性地更新

# Types d'attaque

## 互联网下的攻击

### Spoofing 欺骗

switch查看mac和哪个ip配对

除了当switch溢出的时候

#### 端口spoofing

通过了防火墙

使用可以通过的端口

#### ip盲区欺骗

#### 风暴 flooding 泛滥？

仅仅给一个目标发送很多的包配了bloquer un service

#### OOB out of band



### Fragment

#### tiny fragment

### 改变IP流的方向

#### re-routage

将流转向黑客 （RIP，ICMP redirect)

#### DNS poisonning

直接在DNS服务器中改变ip地址



## LAN下的攻击

### sniffing

* 获得一个网络的流量
* 明文
* rejeu de paquets❓

### ARP poisonning



web服务器的地址 102

黑客的地址103

本机网桥地址：56.104

```bash
arp -i vboxnet0 
sudo arp -d 192.168.56.102
#如果删除了端口对应的mac，重启之后web服务器会重构
sudo arping -i vboxnet0 1
#arping 在3层 reremplir 
#如果我们改变了table ARP
#先把102的端口down掉
#然后重新up
#arping 0i eth0

nc -k -l 3000 | sudo wireshark -k -l -i -


sudo tcpdump -s0 -n -U -w - -i <interface> 'not port 3000' | nc @IP_hôte 3000
telnet两台电脑
所以我知道包的序号和端口号
在ssh中的paquet会被加密

```

右键 - suire TCP

## Attaques depuis la machine

### Débornement

#### Format string

**格式化字符串** - 用来告诉程序以什么格式进行输出

printf()接受一串无论是什么的参数，并且让数据出栈as 第一个收到的参数

printf()无法知道栈上是否放置了正确数量的变量供它操作，如果没有足够的变量可供操作，而指针按正常情况下递增，就会产生越界访问。甚至由于%n的问题，可导致任意地址读写。

通过组合变换格式化字符串参数，我们可以读取任意偏移处的数据或向任意偏移处写数据，从而达到利用格式化字符串漏洞的作用

https://www.anquanke.com/post/id/85785

#### Buffer overflow

**缓冲区溢出**是指当计算机向缓冲区内填充数据位数时超过了缓冲区本身的容量溢出的数据覆盖在合法数据上

要返回调用函数，必须有一个调用函数的记录：执行应该从函数调用指令后的指令恢复。该指令的地址称为返回地址。堆栈用于维护这些返回地址：每当调用一个函数时，返回地址都会被压入堆栈。每当函数返回时，返回地址都会从堆栈中弹出，处理器开始执行该地址的指令。

通过修改保存在堆栈帧中的函数的返回地址，我们改变了程序正常的控制流

bof攻击就是将函数的返回地址被修改为指向一段精心安排好的恶意代码

shellcode - 生成一个shell

#### off by one

在复制一个字符串的时候应该给内存空间

#### Buffer overflow

```bash
welcome 函数接受一个字符串的字符个数和一个字符串
把它复制到缓存中，没有验证
显示缓存中的东西
一个

在某个时刻一个字符串调用了一个函数
返回的指针倒序进栈
不断接近返回结果的那个指针
我们避免在栈中运行
会在函数地址之前加一个
canarie
```

## 攻击web



#### rootkit

通过隐藏攻击者的行为来维持对计算机的未经授权访问的软件组件。





# 检测攻击

当检测到奇怪行为的时候我们猜测这可能是一个攻击

### Approche scénaristique

在知道这个攻击的scénario的情况下，AS

* 如何修复一个攻击

  * 通过攻击分析签名
  * 根据预期的sénario来

  #### 寻找动机

  好处：简单

  #### 分析协议

  根据RFC检查流的一致性

  好处： 可以检测位置的攻击

  坏处：性能不够好 

### Aapproche comportementale 

了解资产的标准行为，行为方法是尝试修复所有异常行为

* 

### Détection de sniffers

#### AS

混杂模式是电脑网络中的术语。是指一台机器的网卡能够接收所有经过它的数据流，而不论其目的地址是否是它。 一般计算机网卡都工作在非混杂模式下，此时网卡只接受来自网络端口的目的地址指向自己的数据。当网卡工作在混杂模式下时，网卡将来自接口的所有数据都捕获并交给相应的驱动程序。

* 检测混杂模式
  * 发送一个ICMP包带有不存在的以太网地址"echo request"
  * 二层会放他过去
  * 三层不会检测 - 因为这不是三层做的事情
* 定位一个异常的DNS请求
  * 发送一个带有mauvais地址的IP包
  * 如果DNS请求带有这个ip地址，这就是一个sniffer

### 入侵检测

有两个工具

HIDS 主机检测入侵系统

KIDS 内核检测入侵系统

#### 在machine上

##### 分析code

##### 维护fichers

### HIDS

### 入侵检测的方法

* Analyse de code 

  * Surveillance des appels systèmes

    **系统调用**，指运行在用户空间的程序向操作系统内核请求需要更高权限运行的服务。

    首先要知道machine的appels systèmes 是哪一个

* Analyse du trafic réseau

  * 分析是le type de communication

* Surveillance des fichers

  * 使用checksum检验完整性

* Surveillance des configurations réseaux

  * Mode promiscuous
  * 检测有没有多余的端口正在运行或者奇怪的协议在

### NIDS

#### 结构

* 包的译码器 découdeur
* Moteur de détection avec des règles
* Module de sortie pour afficher le résultat

##### port mirroring

* 路由器和switch不使用的端口

如果被防火墙禁止了，就会更改règles

我们需要découper我们的网络用来filtage

我们会配置regle，如果有一个paquet

我们有过滤器在进口和出口

