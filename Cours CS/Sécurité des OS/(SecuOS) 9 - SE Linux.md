``` toc

```

# 1. Introduction
Par défaut, les règles de contrôle d'accès de Linux sont discrétionnaires (DAC), c'est-à-dire choisi par le propriétaire de l'objet.
MAIS tous les processus exécutés par le même utilisateur ont la même identité, donc peuvent accéder aux fichiers des autres : firefox accède aux clés SSH par exemple...
-> Besoin de distinction entre application et utilisateur

Note : android ayant un seul utilisateur, utilise les id utilisateurs pour distincter chaque application.
Les versions récentes de Android utilise SE Linux !


# 2. SE Linux
*Mandatory Access Control* : contrôle d'accès obligatoire, que même le propriétaire de l'objet ne peut pas contourner.
SE Linux est présent par défaut dans les distributions récentes mais pas activé.

Caractéristiques de SE Linux :
- Moindre privilège avec confinement des processus
- Contrôle des flux d'informations multi-niveaux de confidentialité (c'est pour ça que la NSA l'a créé)
- Transparent pour la plupart des applications

Fonctionnement :
- Tout est interdit par défaut et il faut spécifier explicitement chaque autorisation
- La politique de contrôle d'accès est centralisé pour l'ensemble du système
- La politique se configure avec un language spécifique
- C'est une surcouche : on vérifie d'abord le DAC avant de tester le MAC
- Contrairement à linux, touts les objets ne sont pas des fichiers et les autorisations que l'ont peut donner dépendent du type d'objet (fichier, socket et processus) et sont fines (read, write, append, ...).


## 2.1. Contexte
**Contexte de sécurité** : chaque sujet et objet possède un contexte de sécurité qui sert au décision de contrôle d'accès. Un contexte est un quadruplet I, R, T, E : identité (différente des UID Linux), rôle, type (ou domaine) et label.

**Domain and Type Enforcement** : identifiant déclaré et défini par l'auteur d'une politique. Il n'a pas de signification intrinsèque mais permet de regrouper les sujets et objets ayant les mêmes autorisations.
Un sujet ou objet ne peut avoir qu'un seul type (on parle de domaine pour les sujets).

Les types permettent de gérer les autorisations.
Exemple : le processus Apache a le type httpd_t et des fichiers HTTP ont le type http_contentd_t, et on a la règle suivante dans la politique :
allow httpd_t httpd_content_t : file read

Assignation des contextes
- Initialisation à partird'un fichier de configuration de la politique
- Création de sujets : par défaut hérite du processus père, mais possibilité de faire des transition vers d'autres types autorisés dans la politique
- Création d'objets : règle par défaut dépend de chaque classe (répertoires, fichiers, socket, ...) et possibilité de faire des transitions


## 2.2. Politique
Language dédié, compilé
Chargement de module (on installe le module de règles apache en même temps que apache, ...)
...

Classes et permissions : classes (répertoire, fichier, socket, ...) avec chacun des permissions possibles différentes. Possibilité d'héritage de classe (fichier et répertoire héritent de la même).

- Type = ensemble de d’objets/sujets partageant les mêmes règles  
- Attribut = groupe de types (peut être utilisé à la place des types)
  ex : les fichiers de configuration Apache et SSH ont des type différent mais sont tous les 2 des fichiers de configurations, donc utilisation d'un attribut pour ne pas avoir à définir les autorisations des fichiers de configurations 2 fois. 
- Alias = différents identifiants pour un même type (compatibilité, etc.)


Possibilité de journaliser de manière fine les actions des sujets :
allow user_t bin_t : file execute;  
auditallow user_t bin_t : file execute;
Par défaut, tous les accès interdits sont journalisés (possibilité de ne pas le faire avec dontaudit).

Par défaut, tout est interdit. Les règles sont cumulatives et ajouter une gèles ajoute forcément des permissions.
SAUF si on crée une règle *nerverallow* : permet d'être sûr qu'une action qu'on n'a pas autorisé nous même ne sera pas autorisé par ailleurs (pas facile à vérifier avec les alias et les attributs).

...

En pratique, on ajoute rarement des règles à la main mais on utilise des *macros* qui elles éditent les règles.


## Transition de domaine
Exemple : l'utilisateur veut changer son mot de passe. Il lance dont l'exécutable *passwd*, cependant *passwd* hérite du type de l'utilisateur *userd_t* qui n'a pas les droits de modification sur */etc/shadow*...
On donne alors le droit à l'utilisateur de passer de *userd_t* vers *passwd_t* lorsqu'il exécute un objet de type *passwd_exec_t*.

On a alors les règles suivantes :
```
allow user_t passwd_exec_t : file execute;
allow passwd_t passwd_exec_t: file entrypoint;

allow user_t passwd_t: process transition;  
type_transition user_t passwd_exec_t: process passwd_t;

allow passwd_t shadow_t: file { read write append };
```

En pratique, seul les applications les plus sensibles (Firefox par exemple) sont confinés avec des transitions de type bien définies. Les autres (ls, ...) s'exécutent dans un domaine *unconfined* avec tous les droits de l'utilisateur.


## Politiques conditionnelles
Applique des politiques selon des conditions : connexion au wifi ou ethernet, ...
On a des variables boolèenne et on peut faire des *if*, *then*, *else* dessus.

## I,R,L,T
**Identité** : utilisé essentiellement à des fins d'audit

**Rôle** : sert à faire du RBAC pour restreindre le DTE et en particulier restreindre les transition de type - seul certains rôles peuvent faire certaines transition (on est pas obligé de s'en servir). Un utilisateur peut assumer plusieurs rôles mais à un instant donné un sujet n'a qu'un seul rôle.

**Label** : permet de gérer plusieurs niveaux de sécurité
- Belle-Lapadula : no-read-up et no-write-down (évite d'un programme lancé en contexte très secret n'écrive dans un fichier secret) -> assure confidentialité mais pas intégrité
- Biba : l'inverse
En pratique, utiliser pour isoler des VM, pour isoler les applications dans android et pour re-créer des utilisateurs physique dans android.


## Architecture SE Linux

Trois composants :
- *Security Server* : détermine si l'accès doit être autorisé
- *Object Manager* : met en oeuvre le contrôle d'accès
- *Access Vector Cache* : cache pour accélérer les décisions du SS

Linux autorise l'utilisation de module LSM (Linux Security Module). SE Linux est un module LSM.
![[SE_Linux.png]]

L'object manager affecte les SID aux objets et les SID permettent de d'identifier le contexte de sécurité. 


## Modes de fonctionnement
2 modes différents :
- Permissive : pas de MAC, les accès sont autorisés mais sont loggées, cela permet l'apprentissage pour fine-tune la politque
  -> Fonctionnement en IDS : on peut analyser les logs
- Enforced : MAC appliqué


# Autres modules de sécurité

Module LSM :
- SMACK : plus simple que SE Linux, souvent utilisé pour les systèmes embarqués.
- TOMOYO Linux : pas de label car décisions basées sur les chemins des objets. Plus simple à comprendre et utiliser mais offre moins de possibilité que SE Linux.
- AppArmor : similaire à TOMOYO Linux, inclus dans Ubuntu par défaut. Moins de granularité que SE Linux : les seules permissions gérées sont read, write, execute.
=> SE Linux plus populaire

Autres modules qui ne sont pas des LSM :
- GrSecurity : patch de securité du noyau qui propose des éléments supplémentaires (MAC, protection mémoire, audit, hook anti-virus)


SE Linux et les LSM protège l'espace utilisateur mais ne durcit pas le noyau et ne protège pas contre les rootkit et les failles du noyau.