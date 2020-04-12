# Réseaux sécurisés

## Introduction
### Faiblesses inhérente aux réseaux IP
- Protocole non-connecté.	->UDP
- Les protcoles IP n'ont pas été conçus pour être sûr : pas de protection de bout en bout, pas d'anthentification du destinataire/expéditeur. On ne peut même pas savoir qui est l'expéditeur (car protocole non-connecté).
  - 端对端之间的流量没有保护
  - 对目的地没有认证
- La sécurité est donc au départ un problème de la couche 7, pas de la couche 6.

### Structure des réseaux d'entreprises

公司网络的需求：

* 保证人员的移动
* 保证整体应用的供应
* 

- Assurer la mobilité du personnel : toutes les personnes peuvent travailler de n'importe où
- Toutes les applis mises à disposition doivent être disponibles pour tout le monde
- Assurer la supervision et l'administration du matos : on ne peut pas avoir un ingénieur réseau dans chaque usine, il faut pouvoir administrer le réseau à distance.
- Assurer la confidentialité des données et des échanges

### Solutions
- **Externalisation** (IaaS - Infrastructure as a Service, Paas - Plateforme as a Service, SaaS - Software as a Service)
  
  ![image-20191105074829438](/Users/haida/Library/Application Support/typora-user-images/image-20191105074829438.png)
  
  - Limité par les protocoles disponibles : comme c'est sur internet, le chiffrement est obligatoire
  - Problématique de la bande passante
  - Limité par les applications disponibles
  
- **Décloisonnement** (on garde les services chez nous mais on les met sur internet) : même genre de problématiques que pour l'externalisation. En revanche, l'entreprise assure elle-même la sécurité, ce qui permet d'avoir des vrais audits et donc de savoir ce qui marche ou ce qui ne marche pas.

- **Réseaux délégués** (en passant par des opérateurs tiers)
  - Problématiques de disponibilité géographique (l'opérateur peut refuser de relier un site si ça coût trop cher par exemple) et disponibilité technique limitée (pas forcément la fibre partout), on est "à la merci" de l'opérateur.
  - Grandit très mal, au delà d'une certaine échelle ou dans un contexte international ça devient compliqué
  - Pas de "vraie" sécurité : la plupart des opérateurs ne se préoccupe pas d'aller chiffrer les données
  
- **VPN (Virtual Private Network)** : système permettant de créer un lien direct entre les ordinateurs distants, en isolant leurs échanges du reste du trafic. Cela permet d'accéder à des ordinateurs distants comme si l'on était connecté au réseau local, donc d'avoir un accès au réseau interne d'une entreprise par exemple.
  - Protocoles : SSL/TLS, SSH...
  - Les connexions ne sont pas nécessairement chiffrées. Des techniques de DPI (analyse de flux au delà de l'en-tête) permettent à des pare-feux de filtrer le trafic du VPN s'il n'est pas chiffré.

## Cryptographie
- Clef partagée
- Chiffrement & déchiffrement avec la même Clef
- Taille des messages théoriquement illimitée : on peut donc chiffrer des flux.

### Algorithme de chiffrementutilisés par l'IPSEC
- Uniquement des algos à chiffrement symétriques (cipher)
- DES (Data Encryption Standard): chiffrement 56 bits, donc vulnerable
- 3DES : 3x DES donc vulnérable
- AES : chiffrement sur 128, 192, 256 bits

### Fonctions de hachage
- Empreinte cryptographique, sorte de signature numérique.
- Fonction injective entre l'ensemble à chiffrer et l'ensemble chiffré, très difficilement réversible


## ISAKMP (Internet Security Association and Key Management Protocol)
- Protocole défini pour établir des clefs cryptographique dans un environnement internet.

### Etablissement d'un tunnel VPN
- Permet de s'assurer de l'identité du correspondant
- S'assurer des capacités du correspondant
- Etablir des canaux de communication.

### ISAKMP
- Cadre général/framework pour l'établissement de l'identité du correspondant, de l'échange et du maintien des clefs de chiffrement
- Doit être complètement indépendant de la partie échange de clef qui évolue très vite, il ne faut pas être dépendant d'un protocole en particulier.

### Echange de clefs Diffie-Hellman
- Comment créer un secret partagé sur un réseau qui n'est pas secure. Cf MT10.

## VPN (LAN to LAN)
### Internet Key Exchange (IKE)
- Assure la sécurité des VPN.
- Protocole chargé de négocier la connexion pour mettre en place les information de sécurité partagées dans le protocole IPsec. IKE authentifie les deux extrémités d'un tunnel sécurisé en échangeant des clefs partagées.
- Certificat signé par l'autorité de certification, qui assure la non répudiation.

### Security Association (SA)
- Etablissement d'attributs de sécurité similaires entre deux éléments du réseau, qui permet d'assurer la sécurité des communications. Cela peut inclure de la cryptographie, des clefs d'encryption, etc.
- Source, Destination : une plage d'adresse, etc
- Algorithme de chiffrement associé à chaque SA
- Algo symétrique entre les paires (on switche la source et la destination)

## VPN IPSEC (Internet Protocol Security)
- Nécessute un logiciel "agent" pour établir un tunnel IPSEC vers un serveur VPN
- Cadre de standards ouverts pour assurer des communication privées sur un réseau IP, avec l'utilisation de cryptographie. Ensemble de protocoles utilisant des algorithmes permettant le transport de données sécurisées sur un réseau IP.
- Opère à la couche réseau (couche 3) -> indépendant des applications, donc les applis n'ont pas besoin d'être configurées aux standards IPSec.
  - **Echange de clefs** ; Un canal d'échange de clefs sur une connexion UDP (port 5000). Le protocole IKE négocie la connexion et authentifie les deux extrémités en échangeant des clefs partagées. On peut utiliser un secret partagé (PSK) pour la génération de clefs de session RSA (une partie peut nier être à l'origine des envois) ou bien des certificats (non-répudiation). IPSec utilise une SA pour dicter comment les parties font usage de l'AH.
  - **Transfert de données** : deux protocoles sont possibles
    - Protocole pour l'intégrité et la confidentialité (cryptographie) : ESP (Encapsulating security payload), protocole 50
    - Protocole pour l'authentification et l'intégrité (Protocole 51) : AH (Authentification Header, format d'entête avec des infos d'authentification + encapsulation de la charge utile d'un paquet). Authentifie les paquets en les signant (signature unique pour chaque paquet envoyé, l'info ne peut donc pas être modifiée)
- **2 modes de fonctionnement** :
  - Transport : on sécurise seulement les données transférées entre deux points d'un même réseau (utilisé quand la couche 7 ne peut pas chiffrer, par ex. avec Maria DB) qui sont chiffrées et/ou authentifiées. On ne fait qu'authentifier le paquet avec une en-tête supplémentaire qui permet de s'assurer que les données n'ont pas été modifiées ; en revanche le orutage n'est pas modifié.
  - Tunnel : On chiffre/authentifie le paquet IP en entier, puis on l'encapsule dans un nouveau paquet IP. Cela permet de communiquer en passant par un réseau public ou de traverser des NAT.
- Pas un protocole de routage
- Mode routé ou non-routé.

### Ecapsulating Security Payload (ESP)
- Protocole appartenant à la suite IPSec, permettant de coùbiner la confidentialité, l'authentification & l'intégrité.
- Chiffre les données puis les encapsule (-> donc différent de l'AH qui encapsule seulement)
- **Mode transport** : on peut authentifier et chiffrer toutes les données. On est obligé de morceler les paquets IP pour les faire entrer dans les paquets ESP
- **Mode tunnel** : on encapsule dans un autre paquet IP et on chiffre tout.

### OpenBSD
- OS libre type Unix, grande importance accordée à la sécurité et à la crypto.

### Topologie & routage
- Topologie en étoile, en général autour du siège.
- On peut faire du réseau maillé en VPN, mais très complexe à mettre en place.

### Intégration dans les réseaux d'entreprise
- En général on a une passerelle VPN redondée : une seule est active mais elle synchronise toutes les SA avec l'autre passerelle. Ainsi, si la première tombe l'autre peut assurer une continuité.

### VPN sur le poste client
- Permet de favoriser la mobilité et le nomadisme.
- On veut simplifier les plans de reprise d'activité (les services pour lesquels on veut une continuité, par ex redondance, machine virtuelles)
- Contraintes :
  - Authentification du poste
  - Révocation des accès
  - Sécurité du poste client
  - IPSEC supporte mal les NAT
  - IPSEC est souvent filtré.

## VPN SSL
- SSL = Secure Sockets Layer
- VPN qui fonctionne au dessus du TLS, accessible avec un navigateur web et passe par HTTP/HTTPS -> basé sur un protocole de niveau 7
- Permet d'établire une connexion sécurisée au réseau intranet depuis n'importe quelle machine possédant un navigateur adéquat.
- Passe pour du HTTPS auprès des proxy et des NAT : les supporte très bien
- Utilisable en TCP ou en UDP
- La plupart du temps on utilise du TCP:443 qui permet d'être associé à du HTTPS.

## Certificats

### Certificat X.509
- Norme spécifiant les formats pour les certificats à clef publicque, les listes de révocation, les attributs et un algorithme de validation du chemin de certification. Etablit un format standard de certificat électronique.
- une CA attribue un certificat liant une clef publique à un nom. "Ce certificat place la signature d'une autorité de certification dans le dernier champ. Concrètement cette signature est réalisée par un condensat de tous les champs précédents du certificat et un chiffrement de ce condensat par la clé privée de l'autorité de certification. N'importe qui possédant la clé publique de cette autorité de certification peut déchiffrer le condensat et le comparer au calcul de son propre condensat du certificat. **Si les deux condensats sont identiques cela garantit que le certificat est intègre**, il n'a pas été modifié. Le certificat de l'autorité de certification qui contient sa clé publique peut à son tour être signé par un autre certificat de plus haut niveau, formant ainsi une chaîne. Tout en haut de la chaîne on trouve les certificats les plus importants : les certificats racines." (Wikipédia)
- Chiffrement assymétrique : Clef privée + clef publique corrrespondante
- Info d'identité & de validité
- Le tout est signé par une 2ème clef privée (autorité de certification)

### Gestion des certificats
1. On crée une autorité de certification : elle a une clef privée + une clef publique + des infos de validation. Elle se signe toute seule (clef publique & reste du certificat). Elle est donc le départ de l'autorité (= **certificat racine**).
2. Cette clef privée va tamponner les certificats côté serveur.
3. On tamponne aussi les clients.

### Transaction TLS/SSL
- SSL = précecesseur de TLS. Protocoles de sécurisation des échanges sur internet.
- Mode client/serveur
- Poste client qui cherche à se connecter au VPN. Le poste a sa clef privée + son certificat + le certificat (la clef publique) de l'autorité de certification
- Le serveur a son certificat, sa clef privée, et le certif de l'autorité.
1. Le client envoie un grand nombre aléatoire au serveur ("client Hello").
2. Le serveur envoie un très gd nombre aléatoire au client ("Server Hello")
3. Le client demande le certificat au serveur. Il peut vérifier la signature du certificat qui a été envoyé par le serveur, car il a la clef de l'autorité de certification.
4. Le serveur envoie au client son certificat, avec la clef publiques, ses informations et une signature numérique
5. Le client vérifie la signature numérique en utilisant les clefs publiques contenues dans les certificat du CA qu'il possède.
6. Le client génère une clef de chiffrement symétrique (clef de session), chiffrée avec la clef publique contenue dans le certificat de serveur. Il envoie la clef de session au serveur (= master secret).
7. Le serveur la déchiffre grâce à sa clef privée
8. Les deux vont créer le master secret avec le pre-master secret et les gd nombres du début.
9. Echange de données en les chiffrant avec la clef de session (master secret) qu'ils ont en commun.
10. Une fois l'échange terminé, le serveur révoque la clef de session

![](https://s1.qwant.com/thumbr/0x0/9/9/f4d93faf7a1a54b82a0727f40702d522af60a79ba5ab01617542a50456a913/1229px-Full_TLS_1.3_Handshake_with_Mismatched_Parameters.svg.png?u=https%3A%2F%2Fupload.wikimedia.org%2Fwikipedia%2Fcommons%2Fthumb%2F1%2F1a%2FFull_TLS_1.3_Handshake_with_Mismatched_Parameters.svg%2F1229px-Full_TLS_1.3_Handshake_with_Mismatched_Parameters.svg.png&q=0&b=1&p=0&a=1)

### Transactions TLS et magasin de certificat
- Les OS, navigateurs, etc décident en qui on doit avoir confiance, alors que ces autorités n'ont pas toujours été très fiable par le passé.
- On fait confiance au magasin dans son ensemble, alors qu'il est facile d'y glisser un rogue CA. Il suffit d'une seule CA pour que ça merde. Pour éviter ça, on a plusieurs outils.

> Quand on parle des trous, il faut aussi parler des gens qui essaient de les combler.

#### Authentification des certificats X509.
- (cf. plus haut)
- DANE : On va stocker dans le DNS une signature du certificat, permet aux gens de piblier leur signature du certificat
- DNS CAA : Permet aux autorités de s'auto-réguler.
- Problème : tout cela est basé sur du DNS :
  - Protocole basé sur UDP (non-connecté)
  - non-authentifié
  - non-sécurité (tout passe en clair)
  - s'appuie beaucoup sur des relais.

### DNS (Domain Name System)
- Ce qui traduit les noms de domaine internet en IP.
- Les serveurs DNS "racine" faiit autorité

#### Un système hiérarchique
- Racine (.) qui délègue des TLD (Top Level Domain : .com, .fr...), qui délèguent à leur tour d'autres domaines
- Tous les sous-domaines ne sont pas forcément délégués.
- Les délégations peuvent créer des zones (ensembles de domaines et leurs sous-domaines non délégués)
- La racine est connue de tous, c'est donc seul point de fragilité du système.

#### Zones et enregistrements
- Chaque entrée dans l'annuaire est une RR (ressource  record), par ex : SOA (start of authority record, le chef de la zone), NS (name server, qui crée une délégation d'un sous domaine vers une liste de serveurs...)
- Toutes les entrées d'un domaine sont regroupées dans une zone.
- Les zones sont facilement réplicables

### DNSSEC
- Domain Name System Extensions : protocole permettant de résoudre certains problèmes de sécurité de DNS
- Sécurise les données envoyées par le DNS. Il ne sécurise pas que le canal , mais protège les données de bout en bout.
- Signature cryptographiques des enregistrements DNS. On délègue les sognatures : le registre d'un domaine de niveau n peut annoncer qu'un sous-domaine (niveau n-1) est signé. On bâtit une chaîne de confiance depuis la racine du DNS.

#### Principaux problèmes de DNS
- Utilise de l'UDP, qui n'est pas conçu pour être fiable
- Les résolveurs peuvent très facilement mentir ou manipuler les requêtes.
- On peut empoisonner le cache d'un résolveur

## Plusieurs degrés de signature
- Création d'un enregistrement RRSIG (types d'enregistrement DNS, RR = ressource records)
- Chaque zone d'enregistrement  est signé par une ZSK (Zone Signing Key)
- Cette ZSK est signée par une KSK (Key Signing Key).
- L'empreinte de la KSK est stockée dans un enregistrement DS (Delegation Signer) au niveau supérieur de l'arborescence.
- Cette empreinte est elle-même signée par la ZSK.
- Racine signée par elle-même et connue du tous.
