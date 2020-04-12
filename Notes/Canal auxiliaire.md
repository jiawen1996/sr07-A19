## Canal auxiliaire:

在密码学中，旁路攻击是指绕过对加密算法的繁琐分析，利用密码算法的硬件实现的运算中泄露的信息，如执行时间、功耗、电磁辐射等，结合统计理论快速的破解密码系统。

我们要观察其他的东西，其他的sortie

攻击针对硬件本身，而不是依赖于输入和输出

这些Canal auxiliaire和正在进行的计算有关

我们不能侦测我们正在观察的东西

## Modèles d’adversaires

经典的模型是完美的

但是现实生活中：

### 经典模型中的攻击

观察电脑中正在运行什么

我们不能查看加密算法中的漏洞

### 不是经典模型下的攻击



可以通过écouter机器发出的声音破译密码

一个放在电脑旁边的手机可以攻击电脑

SSH使用的是DH算法加密，用来echange de session

各自都有一个不公开的私钥

基于l’exponentiation de grandes valeurs par de grands exposants，DH和RSA被认为是不会泄漏的

## Square and multiply

如何快速地计算![image-20191218201316965](/Users/haida/Library/Application Support/typora-user-images/image-20191218201316965.png)

将e拆成二进制，然后从左向右

如果是1就先square再乘x

## Analyse par canal auxiliaire

* 完全绕过加密系统数学性的安全，而是通过观察边缘效应
* 这类攻击适用于攻击者对这个系统有一个物理访问

## Attaques temporelles

```c
int password_ok(const char* user, const char* pwd) {
	const char* real_pwd = get_pwd_from_database(user);
	return strcmp(real_pwd, pwd) == 0; 
}
```

根据不同的加密数据，计算机的计算时间不同

取决于密码的长度

如果密码不对，程序就会停下来

### 如果不对输入的长度进行检测

#### Procédure d'attaque

* 对于第一个字母我们一个一个试过来，如果成功了，系统计算他的时间会和其他的不一样

* 成功了就继续试第二个

### Vérifier au préalable la longueur du mot de passe

#### Analyse de temps d’exécution

* 如果密码的长度是N，我们需要L次尝试来测量运行时间的不同
* 猜测每个字母都需要L次尝试
* 获得一个密码就是L*N次
* 运行时间不同会有漏洞

### 该怎么做

* 不要使用`strcmp`来验证密码，换一个函数来比较字符串 - 需要比较每一个字符，而不是停在第一个错误上

* 不要比较密码，应该比较hash

### Sur l'exponentiation modulaire

* 通过测量指数计算的运行时间来获得指数的值
* 请注意：与总时间无关，而是选择不同数据的时间差异。

#### Définissons l'attaque récursivement

* 我们假设攻击者获得了指数高位上的G个bit
* 攻击者选择一个新的N个bits的指数，其中高位上是之前的G个bits，其他的是随机的。用这个指数在本地运行指数计算
* 对于相同的x（密文）
  * 计算这次运行的时间
  * 然后在目标系统上计算运行时间（这时候指数变成了那个正确的指数）
  * 对于前G次操作，两个系统的运行时间相同，但是在第G+1次时？
  * 因为第G+1位只有两种可能，要么是0，要么是1，我们可以通过比较**Var(Tr - Tl)**来确定是0还是1，正确的那个var会更小一点

如何解决

* 在末端加一个délai使执行时间相同
* 另一个想法是从攻击者的手中删除对部分结果的控制。
* aveuglement cryptographique 

我们将运行时间不和数据联系，和密钥联系起来

在message加入一个随机数r - **blinding**

## Attaques par analyse de la consommation

### SPA

pic de consommation

仅使用一条trace就可以提取加密数据

### DPA

需要多条trace和统计处理来鉴别微小的variation

### SPA - Square and Multiply

* 在观察CPU的波形的时候0和1的波形时不一样的

### SPA - défense

blinding不在可行

#### square-and-always-multiply

TMP数组有0和1两个index，每次循环都会做，所以不存在运行时间的差异

因此存在moins performant的缺点，因为会有多余的计算

## Attaque par induction de faute

* 主要原理是，由材料或者软件导致的défaillance可能会造成信息的泄漏
* 在乘法运行的时候，攻击者引入了一个faute，结果导致乘法不正确
* 如果指数位为0，则没有关系，结果将被丢弃，因为0不用执行乘法，结果只会被分开
* 攻击者可以知道结果已被分开（通过加密结果并观察结果是否与初始消息相对应）
* 如果攻击者可以确定结果是否已被丢弃，则他知道指数的这一位是0还是1（只需对指数的每一位重复此过程）。

我们可以通过logiciel来voler cpu

## Attaques de caches

* 必要的时间来进入一个内存地址
* 在不到65毫秒的时间内从加密的Linux分区中提取完整的AES密钥
* 为了加速访问CPU
* ligne - une copie de bloc de 64 octets
* 为什么要很多的voie - 用来同时分享

* 主存储器被划分以在高速缓存中平均分配其64字节的块
* 缓存是共享资源
* 缓存的状态会影响所有进程并受其影响
* 缓存中的数据受内存保护机制的约束，不能受到攻击
* 另一方面，“元数据”提供有关内存访问配置文件的信息：读取和写入的数据的地址。

1. 保证table在cache

2. provoquer l'éviction d'un ensemble

3. chronométrer

   



如果一个bloc被修改了，那么我们可以déduire l'index

### 加密对缓存的影响

我把内存都填满了，之后沙盒进来的时候，我们观察有哪些bloc被改变了，因为这个时候内存会有微小的变化，因此我可以观察出所有的index

* 一次攻击可以获取更多的信息
* 攻击过程可以触发已知纯文本上的密码。

### AES同步攻击

在大量已知的明文加密操作期间获得（嘈杂的）高速缓存使用情况的度量

* 猜测密钥的第一个字节
* 对于每个加密的纯文本和每个假设，预测哪个缓存行将受到
  T0 [p [0]异或k [0]]
* 确定在测量值和预测值之间具有最强相关性的假设。
* 重复接下来的15个字节k [1]，k [2]，... k [15]。
* 实际上，仅几百个样本就足够了

#### Contre-mesure

* 计算最坏情况下需要的执行时间，并且加入一个补偿延迟，是的所有的执行时间和这个最坏的时间相等



想法：计算不是泄漏信息的唯一方法。

* 一旦切断电源，动态存储器将保留其内容
* 动态存储器使用电容器的电荷来存储位的值
* 负载电容器= 1，放电= 0
* 在正常操作中，电容器电荷会定期刷新
* 即使没有冷却，电容器也会花费时间放电，并且温度会降低。

每次重启是内存会被初始化



## Attaques par exécution spéculative

如果éxécution spéculative是错误的，会被扔掉

Meltdown

会产生一个很大的tableau







# TD

放空cache

victim想要获取文件的权限

好的那个他会返回0

