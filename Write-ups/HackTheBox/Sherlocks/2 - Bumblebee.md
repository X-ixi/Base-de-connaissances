
Un prestataire externe a accédé au forum interne de Forela en utilisant le wifi Guest et il semblerait qu'il ait réussi à volé les identifiants d'un administrateur !

Inputs :
- Logs du forum : requêtes HTTP
- Full database dump au format sqlite3

# Découverte
## Los du forum
Les requêtes proviennent de 2 IP différentes :
- 10.10.0.78
- 10.255.254.2
Et elles sont toutes dirigées sur le forum à l'IP 10.10.0.27.

Les logs s'étendent du 25/04/2023 à 12h jusqu'au 26/04/2023 à 12h, soit sur 24h.

Un script python nous donne les routes utilisées :
```
GET  /index.php
GET  /app.php/feed
GET  /app.php/feed/topics
GET  /app.php/feed/forum/2
GET  /app.php/feed/topic/2
GET  /app.php/feed/topic/1
GET  /viewforum.php
GET  /viewtopic.php

GET  /cron.php
GET  /ucp.php
POST /ucp.php
GET  /posting.php
POST /posting.php
GET  /mcp.php

GET  /adm/index.php
POST /adm/index.php

GET  /store/
GET  /store/backup_1682506471_dcsr71p7fyijoyq8.sql.gz 
```

En particulier la dernier requête récupérant probablement le backup de la base de données a lieu depuis l'IP 10.10.0.78.

## Requêtes admin
On s'intéresse aux requêtes faite sur la route /adm/index.php
On identifie les sid suivants :
- 10.255.254.2 :
	- `ac1490e6c806ac0403c6c116c1d15fa6` : 1 requête GET
	- `0929f9a0759af2b8852c20426857aab2` : 1 requête GET puis 1 POST
	- `041ca559047513ba2267dfc066187582` : beaucoup de requêtes, notamment avec *mode=auth* et *mode=main&action=enable&ext_name=rokx%2Fdborldap&hash=f8bbcf4e*
- 10.10.0.78 :
	- `0bc281afeb61c3b9433da9871518295e` : 1 requête GET puis 1 POST
	- `eca30c1b75dc3eed1720423aa1ff9577` : beaucoup de requêtes, notamment avec *mode=backup&action=download*

## Database
On utilise le logiciel sqlitebrowser.

### Utilisateurs
Dans la table *phpbb_users*, on identifie que peu d'utilisateur ont un mot de passe défini :
- admin
- phpbb-admin
- test
- rsavage001
- apoole
- apoole1
L'email de apoole1 est apoole1@contractor.net, donc c'est le compte du prestataire. Et son IP associé est 10.10.0.78.

### Logs
Dans la table log, on observe 3 lignes intéressantes :

| log_ip     | log_time   | log_operation          | log_data                                          |
| ---------- | ---------- | ---------------------- | ------------------------------------------------- |
| 10.10.0.78 | 1682506392 | LOG_ADMIN_AUTH_SUCCESS |                                                   |
| 10.10.0.78 | 1682506431 | LOG_USERS_ADDED        | a:2:{i:0;s:14:"Administrators";i:1;s:6:"apoole";} |
| 10.10.0.78 | 1682506471 | LOG_DB_BACKUP          |                                                   |

Le prestataire a ajouté son compte au groupe administrateur.
-> time code : 26/04/2023 10:53:51



# Résumé
Actions du prestataire le 26/04/2023 : 
- **10:53:51** : ajout du compte apoole au groupe Administrators
- **11:01:38** : téléchargement de la base de données complète