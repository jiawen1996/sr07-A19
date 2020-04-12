# Virtualisation

* Machine hôte 





# Cloud

## 如何建造一个私有云

* 有地点 —— datacenter
* 有设备
* 问RIPE买IP地址
* 选一个运营商

## Mise en place du cloud privé

### 自动部署私有云的软件

* openstack
* proxmox
* oVirt
* OpenNebula

### Les services vitaux

* 数据中心架子的清单
* 网络地址的清单
* 备份
* 监听工具
* 度量工具(de la métrologie)
* 安全访问平台

### 私有云的自动化

* 部署配置
* 在想要的状态下维护平台
* 安装应用
* 允许应用化的部署

### Ceph

## 公有云

### 公司

* Alibaba Cloud
* Amazon
* OVH.com

### 公有云也是加密的！

* HTTPS
* 内部交流加密TLS/SSL

### 公有云的好处

* 从此我们使用提供商的服务
* 我们不再负责Infrastructure
* 我们要为使用付钱
* 只为了需求而设计结构

### AWS

* ALB(Application load balancer)
* Web
* Database

### 如何创建一个云结构

#### Terraform

##### Infrastructure as Code  ![image-20191105085457664](/Users/haida/Library/Application Support/typora-user-images/image-20191105085457664.png)

### 蓝绿部署（冗余）