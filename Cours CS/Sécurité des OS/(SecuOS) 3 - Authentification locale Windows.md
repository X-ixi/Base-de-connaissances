``` toc

```

# 1. Architecture de sécurité Windows

## 1.1. Principaux composants
Le contrôle d'accès Windows utilise les mêmes briques de base que Linux : utilisateurs, groupe, sujets (processus, thread), objets (fichiers, sémaphores, sockets, ...).
Les sujets agissent sur des objets : 
- Le noyau associe un **jeton de sécurité** à chaque sujet, ce jeton contient l'utilisateur pour lequel le processus s'exécute, les groupes auxquels il appartient et les privilèges (équivalent aux capacités sous Linux).
- L'objet est doté d'un **descripteur de sécurité** qui décrit les actions autorisées sur l'objet en fonction de l'identité du sujet.
-> Le noyau effectue le contrôle d'accès via le composant SRM (Security Reference Monitor)

### 1.1.1. Types de contrôle d'accès
Ce contrôle d'accès est dit **discrétionnaire** car c'est le propriétaire de l'objet qui défini qui a accès à son objet.
-> *DAC* : Discretionary Access Control

Ce système est caractérisé d'*IBAC* (Identity Based Access Control) car basé sur l'identité des sujets pour donner l'accès.

Un modèle plus complet est en train d'apparaître dans les AD et sur le cloud : *CBAC* (Claims Based Access Control) qui se base sur un fournisseur d'identité potentiellement différent du donneur d'accès, et permettant plus de granularité en se basant sur des attributs et roles du sujets.

Depuis Windows 7, ce contrôle d'accès discrétionnaire est complété par un contrôle obligatoire *MAC* (Mandatory Access Control), appelé *MIC* (Mandatory Integrity Control).
-> Sous Linux, un MAC est ajouté dans SELinux par exemple

Le MAC/MIC est vérifié en premier, donc il prend le dessus sur le DAC.

### 1.1.2. Architecture globale
![[win_architecture.png]]

**LSASS** : Local Security Authority Subsystem Service
Ce processus exécute le binaire *Lsass.exe* qui gère l'application de la politique de sécurité locale, et est aussi en charge de journaliser tous les évènements liés à l'authentification.

La base de données de LSASS est stockée dans le registre HKLM/SECURITY, qui est protégé par un contrôle d'accès.

Les services fournis par LSASS sont :
- Gestion unifié de toutes les problématiques d'authentification
- Stocke les credentials pour assurer le SSO sans retaper de mot de passe
-> Il est la cible de nombreuses attaques, notamment par l'outil *Mimikatz* (cf [[(Pentest) 2 - AD & test d'intrusion]]).


## 1.2. Contrôle d'accès discrétionnaire

### 1.2.1. Security Identifiers (SID)
Windows utiliser des SID pour identifier les entités qui agissent dans le système (plutôt que des noms).
Le SID a la forme suivante : *S-version-autorithy48-subauthority1-subauthority2-...-rid*
Le RID (Relative Identifier) est en quelque sorte l'équivalent d'un UID sous Linux.

L'administrateur d'un domaine AD a le SID : S-1-5-domaine-500 (ou domaine est une chaîne aléatoire créée lors de la création du domaine).

Dans le jeton de sécurité, l'identité du sujet et son groupe sont en fait désigné par leur SID.

### 1.2.2. Privilèges
Les privilèges sont des droits précis permettant potentiellement d'outrepasser les droits normaux d'un sujet. Cela permet notamment de donner des droits précis sans exécuter le processus en root.

Par exemple, si un sujet a le privilège SeBackupPrivilege il sera autorisé a faire une sauvegarde d'un fichier même s'il n'a initialement pas le droit de lecture sur ce fichier. 

Paradoxe : certains privilèges sont trop puissants et permettent de passer root en effectuant une suite d'actions précises... Ce problème se pose aussi avec les capacités sous Linux.

### 1.2.3. Usurpation d'identité
Chaque processus possède un jeton de sécurité qui s'applique à l'ensemble de ces fils d'exécution (thread).
Mais on aimerait parfois lancer un thread avec une identité différente : par exemple, un serveur agit au nom de son client en utilisant temporairement son identité.
-> On parle alors d'usurpation d'identité (même si c'est totalement légitime !).

Le jeton de sécurité précise un **niveau d'usurpation d'identité** :
- *Anonnymous* : le serveur ne connait pas l'identité du client et ne peut pas usurper son identité
- *Identify* : le serveur a accès au SID du client et peut interroger le SRM pour connaître les droits du client sur un objet
- *Impersonate* : le serveur peut usurper l'identité du client et agir localement en son nom 
- *Delegate* : le serveur peut usurper l'identité du client et agir localement ou à distance en son nom 

Le niveau *Impersonate* est simple et ne pose pas de soucis car SRM peut vérifier la légitimité des accès demandé. 
Cependant, le niveau *Delegate* est dangereux pour l'usurpation à distance car le serveur va envoyer le hash NTLM de l'utilisateur aux services distants pour s'y connecter et le hash NTLM va se retrouver dans les caches des services distants...
-> Une meilleure solution est l'utilisation de *Kerberos*.

### 1.2.4. Descripteurs de sécurité
Chaque objet possède un descripteurs de sécurité décrivant qui peut accéder à l'objet. Ces descripteurs comportnte : des drapeaux, le SID du propriétaire, le SID du groupe propriétaire, une ACL, c'est-à-dire, une liste d'ACE (Access Control Entry) constituant le contrôle d'accès discrétionnaire, et une liste des opérations devant être journalisées.

Les drapeaux indiquent si le contrôle d'accès est activé et si une notion d'héritage de droits est activé (pratique pour les arborescences de fichiers par exemple).

Une ACE possède : un sujet défini par un SID, un masque qui spécifie les types d'accès autorisé ou interdit (lecture, écriture, création, destruction, ...) et un ensemble de drapeaux spécifiant le comportement vis-à-vis de l'héritage.

### 1.2.5. Algorithme de contrôle d'accès
L'algorithme du SRM parcourt la liste des ACE et donne l'accès si aucune ACE n'interdit le droit demandé, et si ce droit est autorisé explicitement par une ACE.
L'algorithme se termine dès qu'une ACE interdit ou autorise explicitement le droit demandé. Par conséquent, l'ordre des ACE dans la liste compte et pour deny-by-default, il faut mettre toutes les ACE d'interdictions au début.


## 1.3. Mandatory Integrity Control
Le MIC est vérifié avant le DAC, et donc avant l'ACL (=liste d'ACE).
Le MIC utilise des niveaux d'intégrité (untrusted, low, medium, high, system) qui sont attribué à chaque sujet et objet. 
On a dans la majorité des cas les règles suivantes :
- *No-write-up* : un sujet d'un certain niveau d'intégrité ne peut pas écrire sur un objet avec un intégrité plus élevé.
- *No-read-up* : empêche un processus de lire la mémoire d'un processus de niveau supérieur

Ex : un navigateur web est exécuté avec un niveau untrusted ou low pour l'empêcher la lecture ou l'écriture sur la très large majorité des fichiers.

Un utilisateur standard a un niveau d'intégrité Medium par défaut, et c'est le niveau de la majorité des fichiers également.

Un processus hérite du niveau d'intégrité du processus parent, sauf si le parent précise explicitement que le niveau de son fils doit être plus faible.
Le niveau d'intégrité des objets est stocké dans le descripteur de fichier.



# 2. Authentification locale

## 2.1. Hash Lan Manager (LM)
Lan Manager OS : OS historique développé entre 1987 et 1994 avec pour but d'interconnecter un parc de machines et de serveurs.
-> Se base sur le protocole **SMB** encore utilisé aujourd'hui (Server Message Block)

Création du hash LM :
1. Prend les 14 premiers caractères du mot de passe (le complète avec des 0 si nécessaire)
2. Passe tous les caractères en majuscules
3. Coupe le mot de passe obtenu en 2 blocs de 7
4. Sur chaque bloc, applique un chiffrement DES (dont la clé est connu aujourd'hui : KGS!@#$%)
5. Concatène le chiffré de chaque bloc et voilà

Ce hash est **facile à casser** car limité à 7 caractères en majuscule, chiffré avec DES sans salage.


## 2.2. Hash NT
Introduit avec Windows NT.

Le hash NT est calculé comme suit :
- Encodage du mot de passe en UTF16 little endian (permet d'encoder toutes les langues, et pas seulement l'ASCII)
- Utilisation de MD4

Ce protocole est nettement meilleur que le hash LM mais elle se base sur MD4 qui est obsolète depuis 1995 et n'utilise toujours pas de salage, donc il est toujours attaquable avec des dictionnaires.


## 2.3. Base de hachés locale
Lorsqu'on s'authentifie localement, LSASS calcule le hash du mot de passe et le compare au hash LM ou NT stocké dans la base **SAM** (Security Account Manager).

La base SAM est stocké dans le registre HKLM/SAM. Elle est stocké sous forme chiffrée mais la clé est stocké en clair dans le registre SYSTEM.
Leur méthode de chiffrement utilise des algorithmes obsolètes (DES, MD5 et RCA) jusqu'à Windows 10 ou ils utilisent enfin de l'AES. Ce qui ne change pas grand chose car la clé de chiffrement est toujours présente dans le registre SYSTEM.
-> On peut dump la base SAM et la clé de chiffrement du registre SYSTEM avec les droits d'administration


## 2.4. Cassage de hachés
2 méthodes :
- On peut remonter au mot de passe en utilisant des dictionnaires, des masques pour la forme du mot de passe, ... (avec *hashcat* ou *JohnTheRipper* par exemple). On peut casser le haché avec des attaques plus avancé avec des compromis temps-mémoire qui nécessite de stocké des hash pré-calculé mais qui casse un mot de passe beaucoup plus vite (attaque Rainbow Tables par exemple).
  -> Le salage permettrait de s'en prémunir
- On peut aussi utiliser directement le hash pour des authentifications réseau



# 3. Authentification réseau
SSO : Single Sign On
Windows propose des briques logiciels pour permettre à l'utilisateur de s'authentifier une seule fois pour plusieurs services (SSO). En particulier, ça permet de s'authentifier une fois en se connectant à la machine et ensuite d'accéder à des services sur le réseau sans se re-authentifier.

SSPI : Security Support Provider Interface
Interface permettant d'unifier plusieurs protocoles d'authentification sous la même API : NTLM, Kerberos, secure channel, ...

Schéma général d'authentification NTLM :
1. L'utilisateur envoie une demande à un serveur
2. Le serveur lui envoie un challenge
3. L'utilisateur lui envoie son hash NTLM
4. Le serveur contacte le contrôleur de domaine avec le protocole *NetLogon* pour savoir si le hash est le bon
5. Une fois la réponse du DC obtenue, le serveur répond à l'utilisateur

Protocoles réseau :
- **LMv1** : le hash LM suffit à répondre au défi, consistant à chiffré en DES le challenge avec une clé dérivée du hash LM.
- **NTLMv1** : idem que LMv1 mais avec le hash NT
- **NTLMv2 Session Security** : le client demande au serveur de s'authentifier par un défi puis utilise la réponse du serveur et le hash NT pour valider son propre défi
- **NTLMv2** : amélioration des protocoles précédents avec utilisation de MD5 et signature du message pour assurer l'intégrité

**Attaque Pass The Hash** : connaître le hash suffit à s'authentifier à la place d'un utilisateur


