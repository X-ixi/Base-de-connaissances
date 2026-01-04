``` toc

```

# 1. Introduction
GPO : Group Policy Object
-> objet dans l'AD qui configure des groupes (machines et utilisateurs en pratique).

GPO :
- Locale : appliqué en local sur une machine
- Centralisé sur l'AD

Configuration possible :
- Mot de passe : complexité, renouvellement, blocage, ...
- Machines : application disponibles, ... jusqu'à la personnalisation du fond d'écran

Ordre d'application des GPO (la plus élevée écrase les précédentes) :
1. Locale 
2. Au niveau d'un site
3. Au niveau du domaine
4. Au niveau d'une unité d'organisation (la plus élevée)

Les GPO sont regroupées dans des conteneurs. Chaque politique est lié à une OU, un domaine ou un site.
On peut aussi utiliser des filtres WMI pour appliquer des GPO à certaines cibles selon leur configuration (espace disques disponibles, version d'OS, ...).

Les GPO sont accessibles et modifiables via un frontend Windows, mais ce sont aussi des objets dans l'AD de type GroupPolicyContainer, donc on peut les retrouver et les modifier via le protocole LDAP.


Côté client :
Récupération automatique des GPOs, puis application des GPOs avec le plus haut niveau de droits. L'application peut-être bloquante ou non-bloquante.


Déploiement de logiciel :
- Déposer le fichier .msi dans le dossier SYSVOL du répertoire partagé (ou un autre répertoire de fichier).
- Un GPO force l'installation chez le client, ou l'autorise.
-> Peu utilisé en pratique, en général utilisation d'un agent sur chaque client et d'un serveur dédié


GPP : Group Policy Preferences
Idem au GPO mais ne sont pas imposées mais suggérées.


Audit
Export puis analyse offline.
Outils utiles : 
- Powerview : extension powershell qui facile le pentest de l'AD (lister des informations, les GPO, ...)
- Bloodhound : à partir d'un export de l'AD (utilisateurs, machines, GPO, ...) puis représentation graphique et calcul de chemin d'attaque entre un groupe utilisateur et une GPO par exemple.

