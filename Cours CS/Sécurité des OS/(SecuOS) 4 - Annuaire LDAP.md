``` toc

```

# 1. Introduction

Base de données pérennes : les données ne sont pas modifiées très fréquemment, mais sont lues fréquemment (rapport 1000/1). L'annuaire est composé d'entrées qui chacune sont décrit par des attributs.
On ne parle pas de lecture dans un annuaire mais de recherche.

L'annuaire LDAP fournit une abstraction pour l'accès aux données et ne définit pas spécialement d'implémentation.

L'annuaire permet de :
- Gérer de manière centralisée les utilisateurs, les groupes, les machines, les configurations, ...
- Authentification unique SSO
- Gestion distribuée de l'information : un serveur gère une partie de l'annuaire et un autre une autre

*Note* : l'Active Directory Windows est une implémentation de LDAP

LDAP : Lightweight Directory Access Protocol


# 2. Concept LDAP

## 2.1. Modèle de données
Les informations sont stockées sous la forme d'entrées composées d'attributs. Cela se rapproche de la programmation orienté objet.
Un **attribut** est défini par un nom, un identifiant unique (OID), une syntaxe et des règles de comparaison. Il peut hériter d'un autre attribut.
Une **entrée** est une instance d'une ou plusieurs classes. Chaque classes a un ensemble d'attributs optionnels et/ou obligatoire, un identifiant unique (OID) et un type. Une classe peut hériter d'une ou plusieurs classes.

Les classes et leurs attributs son décrits dans des **schémas**. Chaque entrée de l'annuaire appartient à une classe définie dans les schémas de l'annuaire.
Un AD est un LDAP avec les schémas standards de Microsoft.


## 2.2. Modèle de nommage
Les données dans un annuaire LDAP sont représentées sous forme d'arbre : *Directory Information Tree* (DIT). Un annuaire peut être composé de plusieurs arbres.
La racine de l'arbre est appelé la **base**.

Chaque entrée est identifiée de manière unique parmi ses sœurs (même niveau de l'arbre) par un *Relative Distinguish Name* (RDN). Chaque entrée est identifiée de manière unique par son *Distinguish Name* (DN) qui est la concaténation de son RDN et des RDN de tous ces parents jusqu'à la base (comme pour un système de fichiers).

Par convention :
- Les noeuds intermédiaire sont des conteneurs qui reflètent l'organisation de l'entité
- Les feuilles contiennent les données (utilisateurs, machines, ...)

*Note* : un nœud peut être le fils d'un autre sans que la classe de l'entrée fille hérite de celle de l'entrée père. L'agencement des noeuds est juste une question d'organisation 

![[ldap dit.png]]
Le RDN des 2 feuilles de gauches sont composés de 2 attributs pour en assurer l'unicité : cn+uid.

Une bonne hiérarchie de l'arbre a 2 avantages : les performances de la recherche et le contrôle d'accès qui va être basé sur les groupes auxquels appartiennent les utilisateurs.
Evidemment, il ne faut pas sur-complexifier l'arbre et un juste milieu est à trouver.

*LDAP Interchange Format* (LDIF)
Standard de représentation des données qui permet l'ajout, la modification et la suppression de noeuds/d'entrées. Ce format est sous forme textuelle pour simplifier l'utilisation de script.

*Alias* : référence entre entrées au sein d'un même annuaire (comme les hard link des systèmes de fichiers). Si on demande un alias, le serveur le résout automatique et renvoie l'objet original.

*Referals* référence entre entrée de plusieurs annuaires. Ca permet de déléguer la gestion d'un sous-arbre à un autre annuaire.


## 2.3. Protocole et modèle fonctionnel
LDAP est un procole TCP (port standard 389) avec encodage binaire.
Communications possibles :
- Communications client/serveur avec des commandes pour se connecter, rechercher, comparer, créer, modifier ou effacer des données.
- Communications serveur/serveur pour gérer la réplication entre arbres

Sécurisation de LDAP avec l'utilisation de TLS pour chiffrer les données et authentifier les échanges.
-> ldaps (port 636)

Possibilité de faire des requêtes synchrone ou asynchrone, et donc présence d'un identifiant de requête pour identifier la réponse.

Paramètres de requête :
- *baseObject* : le DN à partir duquel effectuer la recherche
- *scope* : 
	- *base* : l’entrée baseObject elle-même (permet de faire une lecture de l'objet baseObject)
	- *one* : entrées immédiatement rattachées au baseObject  
	- *sub* : sous-arbre de l’entrée baseObject  
- *filter* : critères qui déterminent si une entrée fait partie des résultats
  exemples : &, |, !, =, >=, *
- *derefAliases* : indique si la recherche doit suivre les alias  
- *attributes* : liste des attributs à ramener à l’issue de la recherche  
- *sizeLimit* : limitation du nombre d’entrées ramenées à l’issue de la recherche  
- *timeLimit* : limitation du délai de recherche, exprimé en secondes  
- *typesOnly* : ne renvoie que les types d’attribut et non les valeur


## 2.3. Modèle de sécurité
Authentification :
- Par défaut, bind anonyme
- Simple bind : DN et mot de passe transmis en clair

Pour protéger les mots de passe, le plus simple reste de mettre en place TLS pour chiffrer les échanges, mais on peut aussi utiliser MD5 pour haché les mots de passe transmis.

On peut aussi utiliser une source d'autorité externe (Kerberos par exemple), ou des certificats machines pour s'authentifier.

Les mots de passe sont stockés hachés et salés.


## 2.4. Réplication
Afin d'assurer la redondance et de diminuer le temps de réponse, la réplication est intéressante.
Le principe est d'avoir un serveur maître et N serveurs esclaves répliquant chacun une partie de l'annuaire. Le serveur maître reçoit les modifications à faire tandis que les esclaves permettent simplement la recherche.


# 3. Implémentations
Côté serveur :
- Implémentations open-source : OpenLDAP, Apache Directory, Samba LDAP server, ...
  -> OpenLDAP est l'implémentation de référence et est à jour sur le standard LDAP
- Implémentations propriétaire : Microsoft Active Directory, Oracle, Apple, ...

Côté client :
- OpenLDAP tools : ldapadd, ldapsearch, ...
- JXplorer : client graphique
- Active Directory Explorer : client Windows qui peut être utilisé sur n'importe quel LDAP
- phpLDAPadmin : interface web

La configuration du serveur LDAP n'est pas dans un fichier de configuration comme n'importe quel projet (dans /etc/...), mais est dans un **arbre de configuration**. Il ne faut pas se tromper d'arbre lorsqu'on fait des recherches.


# 4. Application : authentification UNIX/Linux
La gestion d'utilisateurs sur des machines UNIX peut être faite grâce à LDAP.
Le choix de la méthode d'authentification est défini dans le fichier */etc/nsswitch.conf* et permet notamment de savoir quel annuaire utilisé (cf [[(SecuOS) 2 - Authentification des systèmes *nix#2.1. Name Service Switch (NSS)]]).






