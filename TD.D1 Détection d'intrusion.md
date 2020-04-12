## TD.D1 Détection d'intrusion

**But du TD**

Le but de ce TD est d'analyser une (vieille) attaque survenue sur un pot de miel. Vous devez déterminer l'ampleur de l'attaque et la faille exploitée. Attention, il y a urgence, le pirate a peut-être pénétré vos systèmes...

1. Obtention des traces

   * [x] Télécharger l'archive `newdat3.log.gz`

   * [x] Décompresser le fichier

     ```bash
     unzip newdat3.log.gz -d /Users/haida/Projects/sr07-a19/TD_defense 
     #unzip doesn't work
     gunzip file.txt.gz
     ```

   * [x] Vérifier son empreinte md5 : `38e85119953076c904fd2105dfcb6cdb`

     ```bash
     md5 newdat3.log 
     ```

     ![image-20191122135853031](/Users/haida/Library/Application Support/typora-user-images/image-20191122135853031.png)

   * [x] Quelle est le type de ce fichier ? Quelle commande permet de le déterminer ?

     ```bash
     file newdat3.log 
     #le début de fichier 在之后的td中，file会犯错
     tcpdump
     ```

     ![image-20191122140056517](/Users/haida/Library/Application Support/typora-user-images/image-20191122140056517.png)

2. Tcpdump

   * [x]  Installer tcpdump
   * [x]  Utiliser tcpdump pour faire une capture des paquets sur votre interface réseau et la sauvegarder dans un fichier.

   ```bash
   tcpdump -i en0 -s 0 -w tcpdump.cap
   ```

   * [x] Comment relire ce fichier avec tcpdump ?

     ```bash
     tcpdump -r tcpdump.cap -X #détaile paquests -n journal numériqie
     'not port 22' - 如果使用ssh从主机登陆vm，因为ssh监听22端口，所以在主机运行tcpdump的时候会造成bouclage
     'port 23'
     ```

     ![image-20191122143126941](/Users/haida/Library/Application Support/typora-user-images/image-20191122143126941.png)

   * [x] Utiliser tcpdump pour lire le fichier `newdat3.log`. Est-ce que cette approche permet de déterminer s'il y a eu compromission du système ?

     ```bash
     tcpdump -r newdat3.log
     ```

     Non,东西太多了找不到我们想要的东西

3. Wireshark

   * [x] Installer wireshark
   * [x] Utiliser wireshark pour faire une capture de paquets sur votre interface réseau et la sauvegarder dans un fichier.

   ```bash
   wireshark -i <interface> -w wireshark.cap
   ```

   * [ ] Ouvrir le fichier `newdat3.log` avec `wireshark`.

     ```bash
     wireshark newdat3.log
     ```

   * [ ] Utiliser quelques filtres pour avoir une idée du type de flux.

     

   * [ ] Est-ce que cette approche permet facilement de détecter une compromission du système ?

4. Snort

   * [x]  Installer snort

   可以capture

   * [x] Utiliser snort pour faire une capture de paquets sur votre interface réseau.

     ```bash
     sudo snort -T -i ens33 -c /etc/snort/snort.conf
     ```

   * [x] Explorer les options de snort pour obtenir plus ou moins de détails sur les paquets capturés.

     ```bash
     snort -v/d/o #detail des paquets
     -v be verbose more information
     -d        Dump the Application Layer 扔掉应用层
     -e         Display the second layer header info
     ```

     

   * [ ] Comment sauvegarder les paquets dans un répertoire dédié ? Comment les classer selon les adresses distantes ?

     ```bash
     snort -dev -l ./log -h 192.168.1.0/24
     ```

5. Comprendre le mode NIDS de snort

   * [ ]  Qu'est-ce qu'un NIDS ? A quelle famille de NIDS appartient snort ? (cf. cours)

   * [ ]  Comprendre l'écriture d'une règle snort

      1. ```
         action protocole adresse_source port_source opérateur_de_direction adresse_destination port_destination options
         ```

      2. "Action" désigne l'action que snort devra réaliser lorsqu'un paquet correspondant à la règle aura été détecté :

         1. alert : génère une alerte et logue le paquet
         2. log : logue le paquet
         3. pass : ignore le paquet
         4. activate : alerte et active une règle dynamique
         5. drop : indique à iptables de jeter le paquet, logue le paquet
         6. reject : indique à iptables de jeter le paquet, logue le paquet, envoi TCP reset ou ICMP port unreachable selon le protocole de niveau 4 utilisé
         7. sdrop : indique à iptables de jeter le paquet mais ne le logue pas

         ```
      alert icmp any any -> any any (msg:"TRAFFIC ICMP"; sid:1000006; rev:1;)
         #D'après https://tools.letf.org/html/rfc792
      alert icmp any any -> any any (msg:"ICMP ECHO REQUERT"; itype:8; sid:1000007; rev:1;)
         #
      alert icmp any any -> any any (msg:"ICMP REPLY"; itype:0; priority:4; gid:22222; rev:3)
         ```

      3. "Protocole" désigne : TCP, UDP, IP, ICMP

      4. Les adresses sont des adresses IPv4 au format CIDR. Par exemple, `192.168.1.0/24 signifie` : toute adresse depuis `192.168.1.1` jusqu'à `192.168.1.255`. Il est possible de spécifier des ensemble d'adresses en utilisant des crochets et la virgule en séparateur (ex : [192.168.1.0/24,10.1.1.0/24]). Il est possible d'utiliser un opérateur de négation : en ajoutant "!" devant l'adresse ou l'ensemble d'adresse, snort sélectionnera le paquet uniquement s'il vient d'une autre adresse que celles spécifiées.

      5. Les ports sont donnés sous la forme d'un entier ou d'un intervalle, avec la syntaxe "début:fin" (ex 1:1024).

      6. Les opérateurs de directions sont "->" (source vers destination) et "<>" (source vers destination et destination vers source).

      7. Les options permettent de préciser les paquets à sélectionner. Elles sont listées entre parenthèses, séparées par des points-virgules. Elles utilisent des mots-clés. Elles se répartissent en 4 catégories :

         general (informations à propos de la règle elle-même),
      
         payload (recherche d'information dans le payload du paquet),
      
         non-payload (recherche d'information ailleurs que dans le payload) et
      
         post-detection (comportement une fois que la règle est vérifiée). Voici quelques mots-clés :
      
         1. msg : affiche un message
         2. reference : affiche une URL pour obtenir de l'information sur l'alerte.
         3. gid : indique quelle partie de snort est concerné par cette règle.
         4. sid : identifiant de la règle. Pour une règle 'maison', utiliser un sid supérieur à 1000000.
         5. rev : numéro de révision de la règle.
         6. classtype : permet de classer la règle par catégorie (eg. attempted-admin, shellcode-detect, trojan-activity...)
         7. priority : priorité de l'alerte
         8. content : recherche un texte, ou du binaire si la chaine est entre des caractères "|".

   3. Déterminer le rôle des règles suivantes :

      显示所有从外部到192.168.1.0/24的流量

      1. ```
         log tcp !192.168.1.0/24 any <> 192.168.1.0/24 23 #(telnet)
         ```

      2. ```
          alert tcp !192.168.1.0/24 any -> 192.168.1.0/24 111 (content: "|00 01 86 a5|"; msg: "external mountd access";)
          ```

      会警告从外部到192.168.1.0/24的tcp流量  
         111:Ce sont des servieces à distance. C-a-d que l'on arrive et envoie un paquet vers le port 111 et ensuite avec un numéro de service. Le port 111 va dispatch la service
         这个服务做了什么？这个alert含有一个二进制chaine（在content中）
         这个来自外部的服务要mouter 一个disque或者其他的东西
         ```
      alert tcp any any -> any any (flags:S; msg:"SYN packet"; sid:1000006; rev:1;)
         #il detecte ping 
         ```

   4. Ecriture de règles snort

      * [x] Ecrire dans un fichier une règle snort qui affiche "PING !" pour détecter les ping sur votre machine.

        ```bash
        alert icmp any any -> any any (msg:"TRAFFIC ICMP"; sid:1000006; rev:1;)
        ```

      * [x] Lancer snort en mode NIDS sur une interface réseau en lui indiquant ce fichier de règle et en lui demandant de loguer dans un répertoire les alertes et les paquets.

        ```bash
        sudo snort -A console -q -c ./test_alert.txt -i ens33
        ```

        在遇到proxy时，反向的时候端口会改变

      * [x] Créer un trafic ping sur l'interface réseau surveillée par snort. Vérifier que les alertes sont bien générées.

      * [ ] Même question pour le trafic web.

6. Exploiter le mode NIDS de snort pour analyser newdate3.log

   * [x] Lancer snort en mode NIDS avec les règles par défaut (cf. /etc/snort/snort.conf) sur le fichier newdate3.log

     在使用snort扫描web流量的时候要注意proxy

     ```bash
     cat /usr/local/etc/snort/snort.conf
     mv newdat3.log newdat3.pcap
     #我们保存了一段web流量并使用snort查看
     snort -dev -A full -l log -r newdat3.pcap
     snort -dev -A full -l log -r newdat3.pcap -c /etc/snort/snort.conf
     #之后会在log文件夹里自动生成一个alert文件
     cat alert | fgrep '[**]' | cut -d ' ' -f 3- | cut -d '[' -f 1 | sort | uniq  
     #以空格为间隔，并删除前两个字段 -> cut -d ' ' -f 3-
     ```

     ![image-20191219140627175](/Users/haida/Library/Application Support/typora-user-images/image-20191219140627175.png)

   * [ ] Analyser les alertes obtenues. Constater leur nombre. Peut-il y avoir de faux positifs ?

   * [x] En utilisant des commandes shell, classer les noms des alertes (messages fournis par le mot clé "msg") par ordre alphabétique, en enlevant les doubles.

     ```bash
     cat alert | fgrep '[**]' | cut -d ' ' -f 3- | cut -d '[' -f 1 | sort | uniq  
     ```

   * [x] Quelle alerte semble la plus sérieuse ? (indice : il semble que l'attaquant soit devenu administrateur)

     ATTACK-RESPONSES id check returned root

   * [ ] En utilisant des commandes shell, retrouver la règle correspondante dans le répertoire des règles snort.

   * [ ] Quel type de trafic a levé cette alerte ? (indice : utiliser les numéros de port)

7. Analyse de l'attaque avec wireshark

   * [x] Dans wireshark, appliquer un filtre sur le protocole correspondant au flux déterminé précédemment.

   * [x] En recherchant le mot-clé "string" avec `Ctrl F`, vous devriez retrouver le paquet 418.

     filter:tcp.stream eq 9 -> select une des paquets -> clic droit -> follow -> tcp stream

   * [x] En utilisant "Suivre le flot" dans le menu apparaissant avec le clic droit, analyser les échanges qui se sont produits durant la session incriminée.

   * [ ] Expliquer les tentatives de l'attaquant. Quelle faille a-t-il exploité ?

   * [ ] Est-il entré dans le système ? Est-il devenu administrateur comme l'indiquait l'alerte snort ?

   * [x] Expliquer les différentes commandes, dont 'passwd nobody -d', 'touch -acmr' etc. Que fait le pirate ?

     ```bash
     passwd nobody -d #该用户的密码将被注销
     touch -acmr
     
     touch -acmr /etc/passwd /etc/X11/applnk/Internet/.etcpasswd
     touch -acmr /etc /etc/X11/applnk/Internet/.etc
     #/etc/X11/applnk/Internet/.etc的更改时间将会和 /etc一致
     ```

     ![image-20191219183959177](/Users/haida/Library/Application Support/typora-user-images/image-20191219183959177.png)

   * [ ]  Quelle erreur fait-il ?

8. Utilisation de tcpflow

   couper des flows
   
   可以看见里面的路由
   
   * [x] Installer tcpflow
   
   * [x] Créer un répertoire spécifique, eg. FLOWS.
   
   * [x] Appliquer tcpflow au fichier newdate3.log de sorte que les flux TCP soient stockés dans le répertoire FLOWS.
   
     ```bash
     sudo tcpflow -i ens33 -o ./FLOWS -r newdat3.pcap 
     ```
   
     
   
   * [x] Examiner les flux obtenus. Utiliser la commande "file" pour déterminer leurs types. Cette commande peut-elle se tromper quant à la nature de certains fichiers ?
   
     ```bash
      ls | sed "s:^:`pwd`/:" | xargs file
      #report.xml
     ```
   
   * [x] En utilisant le shell, afficher le contenu des fichiers de flux qui sont de type texte. Indice : utiliser for, file, grep, cut, xargs
   
     ```bash
     ls | sed "s:^:`pwd`/:" | xargs file | grep -v xml | grep text | cut -d ':' -f 1 | xargs cat
     ```
   
   * [ ] Analyser ce contenu.
   
   * [ ] A votre avis, que contiennent les flux de type fichiers compressés ? (indice : se référer aux sessions ftp découvertes avec la commande précédente).
   
   * [ ]  Explorer ces fichiers compressés. Attention, il s'agit de (vieux) outils d'attaques et vous ne devriez pas les conserver après ce TD. Comment appelle-t-on ce type d'outils ?

* [ ]  **Finalement, peut-on encore faire confiance au système attaqué ?**