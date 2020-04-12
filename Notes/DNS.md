# DNS

dns 是一本annuaire

他的根是，

根repartie 许多的domaine

年鉴的每一个记录称为RR

## DNS服务器的作用

* 认证一个domaine
* 解析器

## 解析的路径

因为根是已知的，所以解析要求

## DNSSEC

https://www.icann.org/resources/pages/dnssec-what-is-it-why-important-2019-03-20-zh

### DNS的问题

* 使用UDP
* 我们可以很简单地杀死一个解析缓存
* 我们不能给每个客户端配一个résolveur

* 信任问题，客户端不信任解析服务器，解析服务器不信任网络

## DANE

DNS-BASED AUTHENTICATION of named entities

* 在DNS中发布证书或者签名
* 创建新的RR TLSA
* 会有其他服务

## DNS CAA

为了证明自己被一个CA认证过了，加了一个新的RR——CAA