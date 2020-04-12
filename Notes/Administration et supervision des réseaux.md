# Administration et supervision des réseaux

## SNMP

échange d'info de réseau

获得设备的信息，尤其是管理层的信息

带宽是否能够运作，管理路由

* 我们定义了一个树型结构，所有的元素都会根据这个回应
* MIB 信息的数据库

所有协议的信息都会

infor générique

一个连接在UTC的AP，我可以看到bonde passant 在AP的interface 所有évolution de traffic



可以询问比如说有多少个paquet从这个interface出去

list de ssid 

MIB privée - IBM, CISCO

有多少客户端连接在这个AP上

valeur spécifique 

有一个id用来识别是CISCO

如果我没有加密这些信息，所有

如果我获得了version，就相当于暴露所有的défaillance

### Niveaux de sécurité

* 至少要sécuriser 交换机

### Méthodes d'échange

#### SNMPGET

可以获得一个唯一的indentifiant

#### 

改变一个interface的速度或者在一个特殊的元素上，如果看到一个流量百分之百的接口，我们可以把它couper

#### SNMPTRAP

trame 可以收到

我们激活交换机上的端口使用SNMPTRAP，如果检测到什么东西的话系统会发送一个script过去

#### SNMPWALK

以树型结构来表示一个interface

```bash
snmpwalk -v2c –c mycommunity 172.32.254.33 ...
iso.3.6.1.2.1.2.2.1.1.10101 = INTEGER: 10101 iso.3.6.1.2.1.2.2.1.1.10102 = INTEGER: 10102
iso.3.6.1.2.1.2.2.1.1.10103 = INTEGER: 10103
#tel feuille de l'arbre
....
iso.3.6.1.2.1.2.2.1.2.10101 = STRING: "GigabitEthernet1/0/1" iso.3.6.1.2.1.2.2.1.2.10102 = STRING: "GigabitEthernet1/0/2"

ifType.10101 = INTEGER: ethernetCsmacd(6)
ifMtu.10101 = INTEGER: 1500
ifSpeed.10101 = Gauge32: 1000000000
ifPhysAddress.10101 = STRING: 0:9e:1e:6b:89:1 ifAdminStatus.10101 = INTEGER: up(1)
ifOperStatus.10101 = INTEGER: down(2)
#是否有什么东西和这个接口连接起来了
ifLastChange.10101 = Timeticks: (173095360) 20 days, 0:49:13.60
ifInOctets.10101 = Counter32: 10096 ifInUcastPkts.10101 = Counter32: 2 ifInDiscards.10101 = Counter32: 0 ifInErrors.10101 = Counter32: 0


```

#### Ansible

base de inventaire - list de machine de @ip

list de config on peut envoyer

### Puppet

avoir un config homogène de l'ensemble de 

我们可以看到一个有config的文件

我们可以把radios的config放在puppet

puppet可以把config配置到每一个服务器上

### API-REST

sur l'équipement

我们现在

所有的数据有JSON呈现

interrogeables facilement web

### Stockage d'informations

facilitie - 一个catégorie

对于ssh - auth

priorité

### Rsyslog

centraliser des logs



syslog的问题是，东西太多，不interrogeables，所以我们用数据库来存储info supervision

### NoSQL

### Time-Series

有一个主键 - 来表明时间，用来筛选信息



influxDB

Volumn不一样

每一个client-clé是一个id，会指向一个文件

所以一个client-clé可以被当作一个@mac，

### Elasticsearch

文件里存的是serveur de web

哪一个数据库经常使用

RRDT tool

outbound:输出的traffic

Nagios可以送

### Check_MK

### Grafana

## ELK

exporter

### Netflow 

从

收集并且处理数据 - 可以存在一个文件里，许多的index 

使用logstash来筛选log



E的好处

使用RESTful模式

我们可以快速地加一个结点

两个结点可以保证正确地运行



# Machine learning supervision

有许多的connexion但是debit不是很高

使用分析的方法来检测comportement normal ou pas - réguliere ou pas



### Apprentissage

使用所有的métique 给出正常点的坐标集

#### Class

classifier des point par rapport à normale

Y = {0,1}

0 - 有攻击

1 - 没有攻击

#### Données d'学习

étiquer 用来获得点的成本有点高

classifieur 是一个函数，

vrai positif - 没有攻击

在有攻击的情况下我们认定是没有攻击 - 这个的成本是1 - 这个成本会随着时间的增长而增长

判别分析

* 线性的
* 二次的

SVM - 寻找最大的marche





判断一个点的statu normal ou pas

看他的邻居是不是都是rouge

如果都是rouge，例子中询问了3个点

​	然后会去询问邻居，同不同意他rouge

3是不是一个好的数字

如果不好判断，我们会进行sur-apprentissage

需要寻找一个décision        



## 在没有任何étiquette的情况下

### estimation de densité

在稀疏的区域，我们可以认为这些点是问题点

### Clustering

### Recherche de régularité

比较régularité，这些点

找到一根直线和

接近这根直线的点是régulié的

#### one class SVM

用来分离密集和稀疏的zone





### autoencodeur





## Quelques oublis

* 需要simplifier temporalité

### 在线学习

可以持续地有数据进入

### par renforcement 

考虑用户返回的结果

返回决定了边界