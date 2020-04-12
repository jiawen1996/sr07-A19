

# 外包存在的问题

* 需要加密
* 带宽问题
* 可用应用的限制

# VPN

## IPSec互联网安全协议

* 加密或者认证IP包

### 加密方式

* 只用对称加密
* DES，3DES，AES

## VPN SSL



# Chiffrement

## 非对称加密

私钥签字并解密，公钥认证并加密

![image-20191105080722530](/Users/haida/Library/Application Support/typora-user-images/image-20191105080722530.png)

1. 明文加密变密文
2. 密文签名
3. 用私钥对签过名的密文解密变为签名的信息
4. 再用公钥认证

## Hachage

哈希散列函数会有冲突问题，由于哈希函数的问题

## SA

为了提供安全的通信环境，在两个网络实体之间，所创建起的共享网络安全属性。

* Source
* Destination
* symétrique entre les pairs (clé)
* Algorithme de chiffrement

## ISAKMP

建立对应的身份认证和密钥的交换及维护

提供了SA的基础框架

IKE提供了密钥交换的机制

# SSL

## 会话与连接

![image-20191101204844835](/Users/haida/Library/Application Support/typora-user-images/image-20191101204844835.png)



![image-20191101205023750](/Users/haida/Library/Application Support/typora-user-images/image-20191101205023750.png)

会话相当于是公路，连接相当于是来往的车辆 

## 握手协议

认证和密钥交换协议

为什么需要服务器密钥交换和客户端密钥交换两个过程？ —— 因为连接是单向的，在同一个连接的两个方向采不同的密钥

![image-20191101211429593](/Users/haida/Library/Application Support/typora-user-images/image-20191101211429593.png)

![image-20191101211710577](/Users/haida/Library/Application Support/typora-user-images/image-20191101211710577.png)

第二阶段，服务器向客户发送证书和密钥，同时问客户端是不是可信任的人