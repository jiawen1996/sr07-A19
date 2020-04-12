如果是在visualbox上

NAT 第一个接口允许我们和另一个



有一个interface上配置

vboxnet





client web 是我们的主机

filebeat - logstash

如果两个服务器之间的连接断了，filebeat会开始分析log，一旦

log会被存储在E中

在E中执行curl http://localhost:9200

另一种方法netstat

0.0.0.0

我们可以给一个东西打上特殊的标记

在客户端访问nginx的时候产生一个log

logstash不一定收到的就是ngix的log，会有延迟

第二个champ data - 查看是否是一个允许的

我们可以整理ngix中收到的log，可以扔掉 remove message

geoip - géographiquement 这个ip在哪里

我们删除所有的错误日志

output是elasticsearch

stdout - décomposer

file - 可以看到很多的附加info，可以帮助我们debug

logstash - 一行一行查看access.log

user

## Prérequis :

2 VM type Debian ou Ubuntu Server

### Sur Virtualbox :

 2 cartes réseau :
 \- 1 NAT
 \- 1 Host-Only adapter sur vboxnet0 (adresse IP en 192.168.56.xx automatiquement alloués par Virtualbox))

### Sur Proxmox :

* [ ] 1 carte réseau sur vmbr2 (adresse en 172.23.x.x au lieu de 192.168.56.xx)

![image-20191206150545867](/Users/haida/Library/Application Support/typora-user-images/image-20191206150545867.png)

## Préparation de 2 VMS :

* [ ] Cloner la VM modèle en Linked-Clone vers :

* 1 VM nommée Nginx, avec 1Go de RAM (pour Nginx et Filebeat) 

* 1 VM nommée Logstash avec 2Go de RAM (pour Logstash, Elasticsearch et Kibana)

* [ ] 1 fois les Vms démarées , vérifier les fichiers `/etc/network/interfaces` qui doivent contenir:

```
auto enp0s8
iface enp0s8 inet dhcp
```

`/etc/hostname` doit contenir le nom de la VM (respectivement nginx et logstash)

* [ ] Rédémarrer les Vms et vérifier

  * l’adresse IP obtenue grâce à la commande “ip addr” 

  * la communication entre les Vms
  * la communication depuis le poste hôte vers les Vms

## Installation de Nginx :

```bash
apt-get isntall nginx
systemctl status nginx.service
```

Le navigateur de la machine hôte doit être capable de se connecter en http sur la VM nginx
`http://<@IP_VM_nginx>`

## Installation d’Elastic Search sur la VM Logstash:

Pour plus d’infos , se referrer à :

https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html

```bash
wget http://www.utc.fr/~quetwilf/sr07/elk_repo.sh 
chmod 750 elk_repo.sh
./elk_repo.sh
```

Ce script contient les lignes suivantes :

```bash
# sudo bash
# wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
# apt-get install apt-transport-https # apt-get install curl
# apt-get install openjdk-11-jdk
# echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" |
sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list # apt-get update
# apt-get install elasticsearch
```

* [ ] Ensuite active le service elasticsearch :

```bash
# systemctl enable elasticsearch
```

* [ ] Editer le fichier `/etc/elasticsearch/elasticsearch.yml` ajouter :

  ```bash
  network.host: localhost 
  network.bind_host: 0.0.0.0
  # process écoute tous les @ip
  ```

* [ ] Editer le fichier `/etc/elasticsearch/jvm.options` et remplacer :

  ```
  -Xms1g 
  -Xmx1g
  #par
  -Xms512m 
  -Xmx512m
  # mémoire on peut rester 1G0
  ```

  ```bash
  systemctl start elasticsearch
  systemctl status elasticsearch => le status doit être active
  ```

  En cas d’échec, analyser le démarrage du service elasticsearch :

  ```bash
  journalctl --unit elasticsearch
  # Vérifier le fonctionnement d’elasticsearch :
  curl localhost:9200
  #notre façon pour écouter les processes
  netstate -laputn 
  ```

  du texte au format JSON doit être renvoyé à l’écran 

## Installation de Logstash sur la VM Logstash:

```bash
apt-get install logstash
```

* [ ] Créer le fichier /etc/logstash/conf.d/01-sr07.conf : Vous pouvez aussi le récupérer avec wget :

  ```bash
  # cd /etc/logstash/conf.d
  # wget http://www.utc.fr/~quetwilf/sr07/01-sr07.conf
  ```

  ```conf
  input { 
      beats {
        port => 5044
        host => "0.0.0.0" 
      }
     #c'est la réception de l'information
     #c'est filebeat qui écoute le prot 5044 sur @ip 0.0.0.0 qui peut venir partout dans le réseau
  }
  
  filter {
  	if [fileset][module] == "nginx" { 
  		if [fileset][name] == "access" {
  			grok {
  				match => { "message" =>
  ["%{IPORHOST:[nginx][access][remote_ip]}
  - %{DATA:[nginx][access][user_name]} \[%{HTTPDATE:[nginx][access][time]}\] \"%{WORD:[nginx][access][method]} %{DATA:[nginx][access][url]} HTTP/%{NUMBER:[nginx][access][http_version]}\" %{NUMBER:[nginx][a ccess][response_code]} %{NUMBER:[nginx][access][body_sent][bytes]} \"%{DATA:[nginx][access][referrer]}\" \"%{DATA:[nginx][access][agent]}\""] }
  				remove_field => "message" 
  				# remover des champs
  			}
      mutate {
      add_field => { "read_timestamp" => "%{@timestamp}" }
      #ajouter des champs 
      #on peut ajouter un tag spécifique pour faciliter la recherche dans Kibana
      } 
      date {
      match => [ "[nginx][access][time]", "dd/MMM/YYYY:H:m:s Z" ]
      remove_field => "[nginx][access][time]" 
      }
      useragent {
      source => "[nginx][access][agent]" target => "[nginx][access][user_agent]" remove_field => "[nginx][access][agent]"
      } geoip {
      source => "[nginx][access][remote_ip]"
      target => "[nginx][access][geoip]" }
      }
    else if [fileset][name] == "error" { grok {
    match => { "message" => ["%{DATA:[nginx][error][time]} \[%{DATA:[nginx][error][level]}\] %{NUMBER:[nginx][error][pid]}#%{NUMB ER:[nginx][error][tid]}:
    (\*%{NUMBER:[nginx][error][connection_id]} )?%{GREEDYDATA:[nginx][ error][message]}"] }
    remove_field => "message" 
   }
  mutate {
  rename => { "@timestamp" => "read_timestamp" }
  } 
  
  
  date {
  match => [ "[nginx][error][time]", "YYYY/MM/dd H:m:s" ]
  remove_field => "[nginx][error][time]" }
  } }
  }
  output {
    stdout {
    codec => rubydebug
    } 
    elasticsearch {
      hosts => localhost 
      manage_template => false 
      index =>
      "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}" 
    }
    file {
      path => "/var/log/logstash/sr07.log"
      codec => rubydebug 
    }
  }
  ```

  logstash il a 3 composants: filter, input, output

```bash
sudo /usr/share/logstash/bin/logstash --debug --path.settings /etc/logstash -f /etc/logstash/conf.d/01-sr07.conf -t
sudo curl "localhost:9200/_cat/indices?v"
sudo curl -XDELETE localhost:9200/filebeat-*
systemctl restart logstash
```



discovery.type: single-node



Un serveur web qui génère du log.

stocker des process d'intermédiaire

générer un ligne de log par `curl ...` dans le serveur nginx

filebeat:

nginx下所有的log

```bash


sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -F
```



## SNMP

https://www.debianhelp.co.uk/snmp.htm - for debian

https://blog.51cto.com/xpleaf/1757333 - for ubuntu

https://medium.com/@CameronSparr/downloading-installing-common-snmp-mibs-on-ubuntu-af5d02f85425

```bash
# Installing SNMP Server in ubuntu
sudo apt install snmpd
#sudo apt-get remove --auto-remove snmp
cp /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf/backup

sudo netstat -antup | grep 161

```

```bash
#set snmp Agent
 14 #  Listen for connections from the local system only
 15 #agentAddress  udp:127.0.0.1:161
 16 #  Listen for connections on all interfaces (both IPv4 *and    * IPv6)
 17 agentAddress udp:161,udp6:[::1]:161
 
 45 #view   systemonly  included   .1.3.6.1.2.1.1
 46 #view   systemonly  included   .1.3.6.1.2.1.25.1
 47 view systemonly included .1
 
 51  rocommunity sr07a006  default    -V systemonly
 52 #  rocommu    nity6 is for IPv6
 53  rocommunity6 sr07a006  default   -V systemonly
 
 sudo /etc/init.s/snmp restart
 sudo apt-get install snmp-mibs-downloader
 # tells Ubuntu to load SNMP MIB files:
sudo sed -i 's/mibs :/# mibs :/g' /etc/snmp/snmp.conf
 #watch all interfaces in localhost and their numbers
 snmpwalk -v 2c -c sr07a006 localhost ifDescr
 #read 
 snmpwalk -v 2c -c sr07a006 localhost ifInOctets.2
 #after 5 seconds
 snmpwalk -v 2c -c sr07a006 localhost ifInOctets.2
 
 snmpwalk -v 2c -c sr07a006 localhost  .1.3.6.1.4.1.2021.11
 snmpwalk -v 2c -c sr07a006 localhost .1.3.6.1.4.1.2021.10.1.3.1
 snmpwalk -v 2c -c sr07a006 localhost 1.3.6.1.2.1.25.3.3.1.1
 
 snmpwalk -v 2c -c sr07a006 localhost .1.3.6.1.4.1.2021.11.9.0
```

![image-20191218093452561](/Users/haida/Library/Application Support/typora-user-images/image-20191218093452561.png)

![image-20191218094648036](/Users/haida/Library/Application Support/typora-user-images/image-20191218094648036.png)





## netflow

```bash
sudo fprobe -i ens18 172.23.3.195:5044

sudo tcpdump -i ens18 host 172.23.3.93
```

