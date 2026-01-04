``` toc

```

# 1. Kerberos

## 1.1. Introduction
Projet Athena du MIT datant des années 1980. Nom dérivé de Cerbère (Cerberus en Latin).
L'objectif du projet est de mettre en oeuvre un protocole d'authentification unique (SSO).

Initialement, Kerberos se base sur le protocole de Needham-Schroeder, qui permet à Alice et Bob de s'échanger un secret partagé en passant par un tier de confiance avec qui ils ont chacun un secret partagé.
Cf [[(Crypto) 1 - Protocoles cryptographiques#2.3.3. Protocole de Needham-Scroeder]].

Pour éviter les attaques par rejeu, il faut s'assurer que les éléments échangés entre Alice, Bob et le serveur sont récents.
-> Besoin d'un système d'**horloge partagé**.

Pendant la guerre froide, les US interdissent l'export d'outils cryptographique avec les pays étrangers. Ce qui limite fortement le développement de Kerberos et favorise le développement de versions parallèles (Heimdal, implémentation GNU, implémentation dans l'AD Microsoft, ...).


## 1.2. Vocabulaire Kerberos
- *Realm* (royaume) : périmètre de sécurité, par convention c'est le DNS converti en majuscule (ex : ATHENA.MIT.EDU)
- *Principal* : machine, utilisateur ou service possédant un nom unique, noté `utilisateur@REALM`, `service/machine@REALM` et `MACHINE$@REALM`.
- *Key Distribution Center* (KDC) : serveur de confiance mettant en oeuvre le protocole Kerberos
- *Authentication Service* (AS) : service responsable d'authentifier les clients. Il délivre les tickets initiaux, ou TGT
- *Ticket Granting Service* (TGS) : service responsable de délivrer aux clients déjà authentifiés (par un TGT) des tickets  permettant d’accéder à d’autres services (ticket TS).
- *Ticket* : ensemble de données regroupant nom du client, estampille temporelle, durée de validité et autres informations
	- *TGT* (Ticket Granting Ticket) : ticket fournit par l'AS qui permet de prouver son identité, d'obtenir des tickets de service et d'assurer le SSO
	- *TS* (Ticket Service) : ticket délivré à un client possédant un TGT valide. Il permet de prouver son identité à un serveur tiers


## 1.3. Fonctionnement du protocole
Trois étapes :
1. Obtention d'un TGT
2. Obtention d'un TGS
3. Utilisation du TGS

![[kerberos_protocol.png]]

Remarque : on peut obtenir un TGT pour un client différent mais si on ne connait pas sa clé (la clé verte précédente), on ne pourra pas se servir du TGT obtenu.


## 1.4. Types de tickets
- Ticket initial : TGT obtenu auprès de l'AS
- Ticket renouvelable : utilisation de ticket à durée de vie courte mais renouvelable. Ces tickets ont deux dates de validité :
	1. Un temps bref de validité avant renouvellement (ex : 8h)
	2. Une échéance de fin de possibilité de renouvellement (ex : 24h)
- Ticket post-daté : ticket invalide mais qui pourra être utilisé pour obtenir un ticket valide dans une période future (pratique pour exécuter un script tous les 6 mois par exemple)
- Ticket mandatable : ticket utilisable par un serveur mandataire à la place du client
- Ticket transférable : ticket qu'un service pourra transmettre au KDC pour obtenir un nouveau ticket pour un autre service (permet la délégation, cf [[#2.1. Délégations]])


## 1.5. Commandes Kerberos
Le fichier */etc/krb5.conf* contient la configuration a utilisé (realm, kdc, algorithmes cryptographiques, ...).
Quelques commandes utiles :
- `kinit` : obtenir un TGT
- `klist` : afficher les tickets présents sur la machine.
- `kdestroy` : supprimer tous les tickets présents

Les commande d'administration ne sont pas standardisées et les commandes dépendent de l'implémentation (`kadmin` sous Unix et commande Powershell pour Windows).

Les tickets peuvent être stocké à plusieurs endroits :
- Un fichier ou un dossier
- La mémoire du processus qui a demandé le ticket (disparaît avec le processus)
- La base LSASS (spécifique à Windows)
- Un serveur dédié KCM (Kerberos Cache Manager) qui stocke les tickets de plusieurs services



# 2. Active Directory

L'AD regroupe au même endroit :
- L'implémentation d'un annuaire LDAP avec un schéma propre à Microsoft, cf [[(SecuOS) 4 - Annuaire LDAP]].
- L'utilisation de Kerberos pour l'authentification des utilisateurs et des machines. 
- L'extension de Kerberos pour gérer aussi les autorisations.
- Éventuellement un serveur DNS
- Un serveur de fichiers dédié à l'administration d'un parc de machines via les GPO


## 2.1. Délégations
Cas d'usage : vous déposez un fichier sur un serveur de fichier puis demander à une imprimante d'aller récupérer ce fichier pour l'imprimer. Cette imprimante doit être autorisé par une délégation à accéder à votre fichier.

### 2.1.1. Délégation totale
Pré-requis : dans les paramètres du client et du serveur, la délégation est autorisé.
Ensuite, la délégation se fait comme suit :
1. Le client demande un TGS pour le serveur
2. Le KDC valide que la délégation est activé et crée un TGT
3. Le KDC lui renvoie ce TGS contenant le TGT du client
4. Le client passe ce TGS au serveur
5. Le serveur a récupéré le TGT du client et peut s'authentifier à sa place 
![[unconstrainted_delegation.png]]
Problème : un TGT se retrouve sur un autre serveur et un attaquant pourrait récupérer des TGT réparti partout sur les machines.

### 2.1.2. Délégation contrainte
Utilisation de tickets TGS transférable pour éviter d'éparpiller des TGT.
![[constrained_delegation.png]]
On peut aussi ajouter à ce protocole un champ précisant quel serveur peut être mandataire de l'utilisateur et le KDC vérifiera cette condition en plus.
-> On parle de *Resource Based Constrained Delegation*.



# 3. Vulnérabilités et contre-mesures

Cf [[(Pentest) 2 - AD & test d'intrusion]]

## 3.1. Kerberoast
On peut demander au KDC un TGT pour n'importe qui, simplement on n'est pas censé y avoir accès car le TGT est chiffré par la clé de l'utilisateur à qui il appartient.
Un attaquant récupère un TGT puis essaie de brute force le chiffrement pour récupérer le mot de passe de l'utilisateur, car la clé de chiffrement est issu du mot de passe utilisateur.

Cette attaque n'est pas possible sur les machines et les services car leur clé de chiffrement est aléatoire et pas issue d'un mot de passe. Sauf si un service tourne sous l'identité d'un utilisateur...

Contre-mesure :
Activer la pré-authentification (c'est fait pas défaut) qui consiste à ajouter un authentifiant récent signé par la clé du demandeur dans la demande de TGT.


## 3.2. Énumération des utilisateurs
Utilisation du script *kerbrute.py* pour tenter de s'authentifier à la place d'utilisateur. Si l'utilisateur existe, la réponse ne sera pas la même que s'il n'existe pas.
Cette attaque permet d'identifier des utilisateurs valides, et elle permet aussi de trouver les utilisateur ne nécessitant pas de pré-authentification.


## 3.3. Attaques cryptographiques
*Pass The Key* : si de vieux protocoles crypto sont supportés par le KDC, on peut utiliser le hash NTLM d'un utilisateur pour l'authentifier

*Pass The Ticket* : récupération des tickets d'un utilisateur (avec *mimikatz* par exemple) et utilisation de ses tickets à sa place
-> Pour s'en protéger, on peut activer *Credential Guard* sous Windows pour protéger toute la base LSASS, et donc le cache de tickets

*Silver Ticket* : on connaît le hash NTLM d'un compte de service et on peut s'en servir pour forger un ticket de service (pour le service hacké) pour n'importe quel utilisateur. Cela permet notamment de faire des actions à la place d'un administrateur sur le service hacké.

*Golden Ticket* : on connaît le hash NTLM du compte *krbtgt* qui permet de forger tous les tickets.