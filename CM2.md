# Virtualisation : notions générales
###### 04 octobre 2019


## Cloisonnement

Un mécanisme de cloisonnement permet de compartimenter un environnement d’exécution en plusieurs parties ne comportant pas les mêmes éléments et ne bénéficiant  ni des mêmes droits ni des mêmes ressources. Intuitivement, il s’agit de découper un environnement monolithique à la manière d’un puzzle, sans  impact sur le service rendu.

### Sandbox
https://singulierecreation.com/collections/robes/products/robe-berangere-fleurs
Un sandbox offre un ensemble de ressources à l'intérieur d'un environnement contrôlé afin d'exécuter le code (par exemple un espace de stockage temporaire sur le disque dur). L'accès aux réseaux, la possibilité d'inspecter le système hôte ou d'utiliser des périphériques sont généralement désactivés ou sévèrement restreints. Dans cette optique, un sandbox est donc un exemple particulier de la virtualisation.


- **Outils** : OS mobiles, 7ème générations de consoles

- **Limitations** : Réservé à des environnement hostiles, puisqu'on se méfie des applications qui tournent sur les OS


## Conteneurs
https://singulierecreation.com/collections/robes/products/robe-berangere-fleurs
Aussi appelés system-wide virtualisation ou  virtualisation d'applications.

Ceux-ci sont utiles quand on essaie de faire tourner des applications dans des environnements très hétéroclites (ex : on veut tester avec toutes les versions de Java), ou de faire tourner le même environnement avec énormément d'instances (ex : tous les utilisateurs de Wordpress.com ont le même environnement). On essaie donc de limiter les ressources systèmes (CPU, mémoire).

C'est un premier vrai moyen de limiter et isoler les ressources réseaux : **on ne donne l'accès qu'à une partie des ressources réseaux, celles dont on a besoin.**

### Fonctionnement

Le **processus init** (PID1) est le premier programme à démarrer sur une machine. Il est exécuté comme un démon informatique et est lancé par le noyau, puis reste actif jusqu'à ce que le système soit éteint. Il est le parent direct ou indirect de tous les autres programmes lancés sur le système.

On va aller forker ce processus, pour en avoir un deuxième qui fait exactement la même chose, à un autre endroit. On utilise obligatoirement le noyeau de l'hôte, qui est déjà chargé en mémoire (on a juste un 2ème process init).

On peut avoir plusieurs conteneurs avec des distributions différentes (un OS de base mais plusieurs versions.)

### Outils
- Docker
- LXC (c'est la base de Docker)

### Limitations
- Utilisation du noyeau de l'hôte :
  - S'il y a un problème sur le noyeau, alors on le retrouve sur tous les conteneurs
  - Compatibilité
- Configuration réseau limitée : par ex., LXC peut se connecter sur un bridge, mais on ne peut pas limiter la bande passante, on ne peut pas filtrer, etc.
- On ne peut pas mixer les OS invités : seulement des variantes de l'OS hôte.


## Protection ring
Un anneau de protection est l’un des niveaux de privilèges imposés par l’architecture d’un processeur. Cercles concentriques : le ring 0 (noyau) a le niveau de privilèges le plus élevé, et les rings suivants sont de moins en moins privilégiés.
- Le noyeau de l'OS démarre sur le ring 0.
- Les pilotes de périphériques sont chargés sur les rings 1 ou 2 (selon les cas)
- Ring 4 : ring applicatif.


Dans les processeurs, il y a des niveaux d'exécution : cela permet de protéger la machine. Les applications qui tournent sur le processeurs ne peuvent pas accéder à toutes les fonctions (par ex. le disque, le réseau & la mémoire, qui pour le CPU sont des ressources rares (= limités en quantité et dans le temps)). Ces fonctions sont réservées à des niveaux d'éxecution plus élevées, par ex. quand un programme veut écrire un fichier, il "demande" à l'OS, masi ne peut pas le faire lui-même.

=> Problème : les OS invités pensent pouvoir accéder au ring 0, alors qu'ils sont de simples applications de l'OS hôte.

### Para-virtualisation
La paravirtualisation est une technique de virtualisation qui présente une interface logicielle similaire à du matériel réel à une machine virtuelle mais optimisée pour ce type de fonctionnement, contrairement à l'émulation d'un périphérique matériel existant qui peut s'avérer laborieuse et surtout lente. La paravirtualisation permet aux moniteurs de machines virtuelles (MMV) d'être plus simples et aux machines virtuelles fonctionnant dessus d'atteindre un niveau de performance proche du matériel réel. Cependant, les systèmes d'exploitation doivent explicitement être portés afin de fonctionner sur des MMV paravirtualisées (donc pose problème avec les OS propriétaires).

La paravirtualisation permet de résoudre le problème d'accès au ring 0, et donc d'avoir plusieurs OS invités sur le même hôte -> économies de ressources (CPU, mémoire, disque), de réseau (ports), niveau d'abstraction supplémentaire par rapport à l'hôte.

L'OS invité "sait" qu'il est virtualisé. Il peut demander l'accès au ring 0, et l'hôte va éxecuter les actions sur le ring 0. Ce sont les premières "vraies" machines virtuelles. On alloue les ressources systèmes, on ne les limite pas.

Le problème sur les MV sont maintenant les disques & les réseaux (limitations), plus la mémoire et le CPU.

#### Limitations
- Nécessité de modifier l'OS invité. Or, tous les OS ne sont pas modifiés pour tous les para-virtualiseurs (OS propriétaires).

### Virtualisation

On ajoute un ring -1 pour l'OS hôte, pour que les OS invités puissent utiliser le ring 0. On isole le plus possible l'hôte et ses invités.

On n'a plus besoin de modifier les OS. Les invités n'ont plus de modifications logicielle, ils ne savent pas forcément qu'ils sont invités.

Reprend les bases de la para-virtualisation, sauf que l'exécution de l'hyperviseur se fait sur le ring -1.

##### Définition : hyperviseur
Plate-forme de virtualisation qui permet à plusieurs systèmes d'exploitation de travailler sur une même machine physique en même temps.

- **Type 1 (natif)** : logiciel qui s'éxécute directement sur une plateforme matérielle (considérée comme un outil de contrôle de l'OS) -> On peut utiliser un OS secondaire. L'hyperviseur natif est un noyau hôte allégé et optimisé, il n'a plus à émuler les anneaux de protection et son fonctionnement est donc acceléré.
- **Type 2 (hosted)** : logiciel qui s'exécute à l'intérieur d'un autre OS, il s'exécute en troisième niveau au dessus du matériel. Il n'a pas conscience d'être virtualisé, donc n'ont pas besoin d'être adaptés.  (QEMU, VirtualBox, VMWare...)
![Schéma Hyperviseurs](https://upload.wikimedia.org/wikipedia/commons/e/e1/Hyperviseur.png)

#### Outils :
- KVM (intégré au noyau Linux)
- VMWare, Virtualbox
- éxecution

#### Limitations
- La couche réseau est limitée à du switch virtuel assez basique.
- On utilise quand même des pilotes optimisés, sinon c'est trop lent.
- On ne peut pas virtualiser/émuler tous les composants d'une machine : on sait faire pour le CPU, la RAM, le réseau (+ complexe),du GPU (processeur graphique)(très très compliqué)

### Emulation

On substitue un logiciel à une composante physique.

#### Outils
- QEMU
- IT Guru
- Emulateurs

#### Limitations
- Performances : il faut une machine de guerre pour émuler certains systèmes.
- Connaissance du matériel de base & des micrologiciel implantés
- Aucune garantie sur la précision de l'émulation

----

# Virtualisation du stockage

## Notion de bloc device sous Linux
Le noyeau linux voit les disques comme des fichiers et les manipule avec les interfaces noyeau.

2 types de bloc device :
- blocs physiques
- bloc device virtuels (pas un disque physique, mais un disque virtuel) : ils permettent d'ajouter une couche de traitement, mais ça ne change rien au niveau de l'administration.

## Raid : Redudant Array of Independ Disks

Le RAID est un ensemble de techniques de virtualisation du stockage permettant de répartir des données sur plusieurs disques durs afin d'améliorer soit les performances, soit la sécurité ou la tolérance aux pannes de l'ensemble du ou des systèmes. On compense les défaillances matérielles des disques (mécaniques, flashs, SSD), pour éviter de perdre des données.

Permet la haute disponibilité des données : même si un disque a cassé, et évite de perdre des données. Permet aussi de récupérer les données en cas de crash, sans interruption de service.

Il existe plusieurs niveaux de RAIDs, matériel et logiciel.

Des raid n'enlèvent pas la nécessité de faire des sauvegardes, car la redondance des données n'assure pas qu'elles soient correctes (corruption de données), ne permet pas de revenir en arrière, et ne protège pas des erreurs utilisateurs. **Les raids prémunissent juste des défaillances matérielles**, mais pas humaines ou logicielles.

### Raid 0
Réparti les données entre les différents disques. Pas vraiment un raid : si un disque crash, on perd une partie des données. Cependant, on a une utilisation maximale des disques.

### Raid 1
Le contrôleur écrit les données de manière synchrone sur les deux disques. Attentions, ce n'est pas une duplication, mais une écriture simultanée. On a donc la même performance sur les deux disques.
On a peu de dégradation des performances, mais une très mauvaise utilisation de l'espace disque (problématique de coût). Permet de garder les données si un seul crash, mais pas en cas de crash en cascade.

### Raid 5
Le contrôleur garde en buffer certains blocs, puis fait un calcul de parité (par exemple un XOR), puis écrit ça sur 3 disques. Si on perd un disque, alors le calcul de parité permet de retrouver la donnée perdue.
C'est le plus utilisé aujourd'hui, car il maximise l'espace disque inutile (n-1 disques disponibles). En revanche, on perd en performance en écriture. Un crash unique est récupérable, mais pas les crashs en cascade.

\+ pour reconstruire un disque à partir du XOR, ça prend beaucoup de temps et de ressources (donc peut en faire péter un deuxième oupsi) -> La solution est d'augmenter le nombre de parité.

### Raid 6
On augmente le nombre de parités pour essayer de pallier aux disques en cascade.

### Raid combinés
On essaie de combiner les points forts de chaques raid.

## Logical Volume Management (LVM)

Le LVM est à la fois une méthode et un logiciel de gestion de l'utilisation des espaces de stockage d'un ordinateur. Il permet de  :
- gérer, sécuriser et optimiser de manière souple les espaces de stockage en ligne dans les systèmes d'exploitation de type UNIX
- s'abstraire des disques physiques (d'où la gestion plus souple qu'avec les partitionnements).

### Défintiions
- Physical Volume : partition physique ou un disque complet
- Volume Group : agrégation de plusieurs physical volume
- Logical Volume : partition logique du système, appuyée sur un volume group. On découpe les LV dans les Volume Groups, puis on les formate et on les monte dans des systèmes de fichiers. Ils sont équivalents à des pseudo-partitions.
- Extensions (physiques ou logiques): plus petites unités de stockage LVM. La taille des extensions est la même pour chaque LV d'un même VG. Les extensions non-utilisées permettent d'augmenter la taille d'un VG.

### Processus  (extensions)
1) On découpe les disques en physical extents, des briques (en général 32 mb) sur des physical volumes.
2) On regroupe les physical extents dans un volume groups.
3) Présentation des physical extents pour les logical volumes, formation de logical extents au niveau des logical volumes.

On peut étendre les disques à chaud très facilement : on fait grandir le volume group. Les volumes logiques peuvent dépasser en taille les disques physiques.

### Snapshot (fonction avancée)
Repose sur le principe du copy on write (CoW).
Le snapshot construit une série de liens : indique quel octet consulter pour avoir l'info. Puis, quand une partie du volume d'origine est modifié, on le recopie directement sur le snapshot.

Attention, n'est pas une sauvegarde (si le volume d'origine crash, les "pointeurs" ne permettront pas de retrouver les données), mais peut servir de base. Permet aussi de faire des répliques rapides des données de productions pour faire des tests.

## Systèmes de stockage en réseau
- DAS : Direct Attached Storage : performant et efficace, mais rigide et ne supporte pas la croissance.
Donc on met le stockage en réseau.

### NAS : Network Attached Storage.
On a une machine autonome qui présente des volumees sous forme de partage. Fonctionne en mode fichier, la baie a conscience de ce qu'elle stocke. La ressource de stockage est directement connectée au réseau IP. Le serveur NAS intègre le support de plusieurs systèmes de fichiers réseau.

On a un système d'autorisation descendante : syst. de fichiers exportés -> Options d'exportation -> Options d'importation.

#### Avantages / incovénients.
- Souplesse de mise en oeuvre, on réutilise les infrastructures réseau existantes au maximum
- Grandit relativement mal : si la machine est saturée, il faut en mettre une deuxième et on ne peut pas regrouper les deux.
- Pose aussi des problèmes de cloisonnement de réseau.
- Le fonctionnement par fichier n'est pas forcément optimal (ex : une BDD pour ajouter une ligne devra prendre tout le fichier, écrire sa ligne, puis "renvoyer" le fichier vers le serveur NAS)

### NFS : Network File System
Constuit en UDP.

### SAN : Storage Area Network
Un SAN est un réseau spécialisé permettant de mutualiser des ressources de stockage.
La baie de disque n'a pas conscience de ce qu'elle stocke et on travaille en mode "bloc". Les baies de stockages n'aparaissent pas comme des volumes partagés sur le réseau, mais sont directement accessibles en mode bloc par le système de ficheirs des serveurs : chaque serveur voit l'espace dique d'une baie SAN auquel il a accès comme son propre disque dur.

Protocole de beaucoup plus bas niveau, donc beaucoup plus facile de faire de la redondance (donc forte sécurité).

![stockage réseau](https://upload.wikimedia.org/wikipedia/commons/e/e0/Compingles2.png)

-----

# Identité, Annuaire et Authentification (LDAP)

## Identité
Ce qui caractérise une personne, un service, une machine + toutes les informations techniques nécessaires (par exemple : le mdp) + l'environnement dans l'entreprise

## Annuaire
Regroupement de toutes les identités de l'entreprise : identités, groupes, services... Protocole associé : LDAP

## Authentification
Permet d'identifier formellement. Il existe des protocoles associés (CAS, SAML, OpenID, LDAP, OAuth...)

## LDAP : Lightweight Directory access protocol

LDAP est une norme pour les systèmes d'annuaires, incluant un modèle de données, un modèle de nommage, un modèle fonctionnel basé sur le protocole LDAP, un modèle de sécurité et un modèle de réplication.  Elle est extensible, modulaire, et modifiable, réplicable partiellement ou totalement.

Organisation en arborescence, possibilité de contrôler un à plusieurs niveaux de l'arborescence. Chaque nœud est constitué d'attributs associés à leurs valeurs. L'arbre LDAP doit représenter l'arbre de la société. A chaque noeud ou feuille, on associe des champs obligatoires ou optionnels.

- Plusieurs niveaux d'accès à plusieurs endroits de l'arbre (read/write, authenticate, compare)
- Plusieurs frameworks dans des langages différents.
- Normé de manière internationale
- Notion de schéma : sorte de métadonnée, on peut définir un schéma par exemple le schéma utilisateur (nom prénom id mdp).
- Compris par la grande majorité des OS

### Format LDIF
Formatage d'une entrée.

```C
dn: cn=John Doe,dc=example,dc=org //dn : nom de l'entrée, pas un attribut.
 cn: John Doe //RDN de l'entrée
 givenName: John
 sn: Doe
 telephoneNumber: +1 555 6789
 telephoneNumber: +1 555 1234
 mail: john@example.com
 manager: cn=Barbara Doe,dc=example,dc=com
 objectClass: inetOrgPerson
 objectClass: organizationalPerson
 objectClass: person
 objectClass: top
```

### Comment définit un schéma :
- Chaque entrée a un id unique (Distinguished Name, DN), constitué à partir de son Relative Distinguished Name (RDN) suivi du DN de son parent -> définition récursive.
- Un OID (Object identifier)
- Une dscription
- Des attributs (obligatoires ou optionnels)
