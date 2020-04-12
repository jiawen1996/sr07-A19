```bash
# Installing SNMP Agent in ubuntu
sudo apt install snmpd 
# Install snmp client
sudo apt install snmp
cp /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf/backup

sudo netstat -antup | grep 161
sudo vim /etc/snmp/snmpd.conf
#set snmp Agent modify following lines
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
 #watch all interfaces in localhost and their corresponding numbers
 snmpwalk -v 2c -c sr07a006 localhost ifDescr
 #interroger le nombre de octets entrants de l'interface ens18 
 snmpwalk -v 2c -c sr07a006 localhost ifInOctets.2
 #after 5 seconds
 snmpwalk -v 2c -c sr07a006 localhost ifInOctets.2
 
 snmpwalk -v 2c -c sr07a006 localhost  .1.3.6.1.4.1.2021.11
 snmpwalk -v 2c -c sr07a006 localhost .1.3.6.1.4.1.2021.10.1.3.1
 snmpwalk -v 2c -c sr07a006 localhost .1.3.6.1.2.1.25.3.3.1.1
 
 #On peut aussi interroger depuis VM logstash
 sudo apt install snmp
 sudo apt-get install snmp-mibs-downloader
 sudo sed -i 's/mibs :/# mibs :/g' /etc/snmp/snmp.conf
 snmpwalk -v 2c -c sr07a006 172.23.3.93 ifInOctets.2
```

