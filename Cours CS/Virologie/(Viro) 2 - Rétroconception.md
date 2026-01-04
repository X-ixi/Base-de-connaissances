``` toc

```

# 1. Introduction à la rétroconception

## 1.1. Définitions
**Rétroconception** : processus inverse de la conception, cherche à comprendre le fonctionnement interne d'un logiciel pour en extraire une abstraction de plus haut niveau (à quoi sert le programme et quelles en sont les étapes).

On travaille sur des fichiers binaires :
- *Désassemblage* pour passer du binaire au code assembleur
- *Décompilation* pour passer de l'assembleur à un code proche du code source
- *Rétroconception* pour passer du code source à l'action réalisée par le programme
![[retroconception.png]]

Il y a 2 contextes d'analyse possibles : en boîte blanche (on a accès au binaire) ou en boîte noire (on a accès seulement aux entrées et sorties du programme).
-> Pour ce cours, on se place en boîte blanche.


## 1.2. Applications et cadre légal
Quelques exemples d'applications :
- Cracking de logiciel
- Vol de propriété intellectuelle (e.g. récupération d'algorithme)
- Analyse de malware
- Interopérabilité avec un logiciel propriétaire (utile pour les projets open source)
- Recherche de vulnérabilités
La rétroconception logicielle est **illégale** exceptée dans le cadre très contrôlé de l’interopérabilité.


## 1.3. Problématiques
La compilation et l'assemblage entraîne une perte forte d'information : noms de symbole, typage, distinction code/données, ...
-> il faut retrouver (en partie) l'information qui a disparu

## 1.4. Approches
![[approches retroconception.png]]


# 2. Langages de programmation

Il existe différents types de langages de programmation et différentes approches de rétroconception associées :
- Langage **compilé** (C, C++, Ocaml, ...) : on doit analyser le code assembleur créé par le compilateur
- Langage **interprété** (python, VBS, php, ...) : on peut lire le code source mais celui ci est souvent obfusqué
- Langage **émulé/virtualisé** (Java, .Net, ...) : des outils existent pour retrouver le code original avec moins de perte d'information que pour les langages compilés

-> Dans ce cours, on se limite au binaires compilés en C.


# 3. Techniques de rétroconception

## 3.1. Analyse statique
Analyse d'un programme sans l'exécuter.

Des outils d'analyse existent mais restent limités car il n'effectue qu'une partie des étapes détaillées après. Ils permettent notamment de générer un **flot de contrôle** mettant en évidence les sauts et les appels de fonctions :
![[flot de controle.png]]

L'analyse statique passe par les étapes suivantes.

### 3.1.1. Désassemblage
Passage du code binaire aux instructions assembleur. Il peut être fait de 2 façon différentes :
	- Linéaire : séquentiel indépendamment du flot de contrôle
	- Récursif : séquentiel jusqu'à une instruction de transfert
Dans les 2 cas, des données peuvent être interprétées comme du code et menées à un désassemblage faux.
  ![[desassemblage de donnees.png]]

### 3.1.2. Propagation de données et de types
Les registres et les variables sont propagées tant qu'ils ne sont pas redéfinis. 
De même, si une variable est définie à partir d'une autre, elle hérite de son type (on connait alors la taille qu'elle prend en mémoire, etc).

### 3.1.3. Détection de bibliothèques par signatures
Cela permet de détecter le code venant d'une librairie lié statiquement du code "propre" à l'application. Cela fait gagne beaucoup de temps car on n'a pas besoin de reverse la fonction *printf* à chaque fois par exemple.

Ces étapes sont faites automatiquement par des outils de retro comme Ghidra mais il peut rester des erreurs (mauvaise interprétation de données en code, ...).


## 3.2. Analyse dynamique
Analyse du programme pendant son exécution, cela permet de déterminer son comportement pour une entrée donnée.

Plusieurs approches :
- Monitoring : surveiller les actions menées par le programmes (lancement de threads, accès à un fichier ou registres, activité réseau, appels systèmes, ...)
  -> *procmon.exe*
- Debug : connaître l'état exact de la machine à un instant donné (registres, mémoire, code en cours d'exécution, ...)
  -> *WinDbg*
- Tracing : enregistrement de l'état du programme à tout instant (registres, mémoire) pour ensuite pouvoir revenir sur chaque instruction
  -> WinDbg Time Travel Debuging

*Note* : attention lorsqu'on debug un malware, celui-ci s'exécute !


## 3.3. Méthodologie
En reverse, il faut réussir à identifier : 
- Des constantes remarquables (chaîne de caractères, code d'erreur, ...)
- Des structures de contrôles (boucles, conditions, ...)
- Des idiomes classiques (strlen, strcmp, memset, memcpy, ...)
- Des structures de données (liste chaînée, ...)
- Des fonctions particulières (fonctions cryptographiques, ...)

Il est recommandé de se concentrer sur une seule partie du programme à la fois et de l'analyser en faisant des hypothèses sur son fonctionnement avant de passer à la suivante.



# 5. Cryptographie

Il est particulièrement utile d'être capable de reconnaître les fonctions cryptographique les plus standards :
- **Fonction de hachage** : prend une entrée de taille quelconque et renvoie une sortie de taille fixe. Le calcul se fait en 3 temps : *HashInit* (initialise le contexte crypto, avec souvent l'utilisation de constantes caractéristiques), *HashUpdate* (met à jour le contexte en fonction de la donnée) et *HashFinal* (récupère le hash final).
- **Cryptographie symétrique** : utilise la même clé secrète pour le chiffrement et le déchiffrement, qui peuvent être fait par bloc ou par flot. Dans les 2 cas, le calcul se fait en 2 temps : *InitKey* (dérive la clé fournie par l'utilisateur pour former la clé secrète) et *Encrypt/Decrypt*.
- **Cryptographie asymétrique** : se base sur l'arithmétique des grands et nécessite donc d'utiliser des nombres bien plus grands que ceux générés nativement par le CPU. On peut alors en identifier l'usage à la présence de bibliothèques de gestion de grands nombres (*bignum* de OpenSSL par exemple).

*Note* : Les fonctions cryptographiques nécessitent des constantes particulières pour fonctionner correctement (Sbox, ...). Ces constantes peuvent permettre d'identifier une fonction, il suffit de chercher ces constantes sur internet !

