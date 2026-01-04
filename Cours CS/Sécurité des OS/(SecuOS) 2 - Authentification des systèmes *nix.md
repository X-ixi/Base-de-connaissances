``` toc

```

Identification : communication d'une identité
Authentification : vérification de l'identité dont une entité se réclame
Une fois l'authentification faite, l'utilisateur (enfin son processus) obtient un UID et un GID.


# 1. Logins et mot de passe

## 1.1. Identification des utilisateurs
On attribue à chaque utilisateur :
- Un nom d'utilisateur (login)
- Un identifiant d'utilisateur (UID)
- Un identifiant de groupe (GID)

L'utilisateur *root* (UID=0) possède les privilèges absolus sur la machine. A l'inverse, l'utilisateur *nobody* ne possède aucun droit (pas de shell, utilisé parfois pour faire tourner des démons).

Un groupe possède : un nom de groupe, un GID et une liste de membres.


## 1.2. Gestion des mots de passe
Le fichier */etc/group* liste les utilisateurs et leur groupe.
Le fichier */etc/passwd* liste les utilisateurs avec leurs propriétés (homedir, shell, ...).
-> Ce fichier est lisible par tous et ne stocke plus les mots de passe.
Le fichier */etc/shadow* stocke les mots de passe hachés des utilisateurs (fichier accessible uniquement par les utilisateurs root et shadow). 

Le format des mots de passe dans */etc/shadow* est le suivant : `$<prefix>$<options>[$<sel>]$<haché>` où le prefix indique le type de hachage.



# 2. Identification

## 2.1. Name Service Switch (NSS)
C'est le service d'identification centralisé d'un système unix. Il fournit les UID et GID aux applications qui en ont besoin, notamment pour l'authentification mais pas que.
Il contient les informations sur les utilisateurs et groupes, mais aussi sur les noms d'hôtes, de réseaux, de protocoles, de services réseaux, ...

NSS fait le lien avec différents composants pour trouver la réponse : fichiers (/etc/shadow, /etc/hosts, ...) ou autres services (LDAP, DNS, ...).

Le fichier de configuration de NSS est */etc/nsswitch.conf*, et indique où trouver l'information et dans quel ordre la chercher (ex : chercher un hôte dans un fichier avant d'interroger le DNS, ...).

On peut interroger NSS avec la commande `getent` en ligne de commande. On peut aussi l'interroger en C.


## 2.2. Service de noms
Un système peut aussi s'appuyer sur des sources de données externes. Il suffit de configurer dans NSS les sources externes à utiliser.
Les annuaires centralisés les plus commun sont : 
- Winbind (service NSS pour l'intégration avec AD / Kerberos)
- LDAP 



# 3. Pluggable Authentication Modules (PAM)

## 3.1. Principes
Lorsqu'un exécutable veut authentifier un utilisateur local, il doit :  
1. Identifier l’utilisateur (via NSS)
2. Obtenir le mot de passe de l’utilisateur
3. Obtenir l’enregistrement shadow correspondant
4. Hacher le mot de passe de la même manière que dans l’enregistrement shadow
5. Comparer le résultat

Pour éviter de devoir ré-implémenter ça à chaque fois et car il faut des privilèges élevés donc on veut bien maîtriser le code, on fait appel à un module PAM déjà codé.
-> On interagit avec la librairie *libpam*.

![[pam.png]]

PAM offre 4 type de services : authentification, gestion des mots de passe (création, modification, ...), gestion des sessions (initialisation/finalisation - point de montage, exécution de script, ...) et gestion des comptes (validité, ...) 


## 2.2. Configuration
Le comportement de libpam, indiquant les modules qui doivent être chargés (librairie '.so') et de quelle manière ils doivent être interrogés, est déterminé par des fichiers de configuration, répartis dans le dossier : `/etc/pam.d/*`.

Chaque fichier de configuration comporte une suite d'étape et chaque étape peut être suffisante (si elle est satisfaite, on s'arrete), requise ou optionnelle. 
Un fichier de configuration peut en appeler un autre pour éviter de ré-écrire la même chose (ex : fichier *common-auth* utilisé plein de fois ailleurs).


On peut être consommateurs de module PAM, mais on peut aussi implémenter son propre module PAM.




