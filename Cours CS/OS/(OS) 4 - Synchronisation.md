```toc

```

# 1. Exemple en espace utilisateur
On prend comme exemple en programme en C qui crée 2 threads modifiant une structure de données partagée : le premier incrémente un compteur de cette structure dans une boucle for et le second le décrémente autant de fois que le premier l'incrémente.
Si les boucles for se font sur un nombre assez grand d'itération, il y aura un problème de synchronisation et le résultat du compteur ne sera pas la valeur initiale !

Ce comportement est dû à un **fossé sémantique** entre ce que pense le programmeur (ctr++ est une instruction atomique) et ce que produit le compilateur (ctr++ devient 4 instructions assembleur).

Les instructions assembleur des 2 thread peuvent se mélanger dans l'ordre d'exécution et créer un problème de synchronisation : le premier thread met la valeur du compteur dans une variable locale, le second en fait de même, le premier incrémente la variable locale et la copie dans le compteur et finalement le second décrémente sa valeur locale (qui n'est plus égale au compteur !) et la copie dans le compteur.

Pour éviter ça, il existe des primitives de synchronisation (comme les **mutex** en C) qui garantissent qu'une ressource (ici la structure de données) n'est accédée que par un seul processus à la fois. Le thread verrouille le mutex, utilise la structure de données puis déverrouille le mutex.



# 2. Synchronisation au sein du noyau
## 2.1. Pseudo-concurrence et concurrence réelle
**Pseudo-concurrence** : un seul flot d’exécution peut avoir lieu à un instant, mais plusieurs fils d’exécution peuvent être entrelacés de manière à donner l’illusion d’une exécution parallèle.
-> architecture **mono-processeur** : l'ordonnanceur entrelace les fils d'exécution de plusieurs processus (cf [[(OS) 3 - Ordonnancement]]).
Il suffit de désactiver les interruptions asynchrones le temps de faire une opération délicate. Mais en pratique on est rarement dans le cas d'une architecture mono-processeur.

**Concurrence réelle** : plusieurs fils d'exécution peuvent avoir lieu simultanément.
-> architecture **multi-processeurs** : plusieurs flots d'exécution peuvent accéder simultanément aux mêmes données

## 2.2. Rôle de l'OS
L'OS doit offrir des **primitives de synchronisation** afin de permettre aux différents fils d’exécution de coopérer harmonieusement.
MAIS, il faut éviter le plus possible d'avoir recours à la synchronisation et essayer de l'utiliser uniquement sur des **structures dédiées** et dans de **petites portions de code**. Sinon, les performances de l'OS seront limité et on ne tirera pas vraiment avantages des multi-processeurs.

Les OS doivent fonctionner pour une architecture mono-processeur aussi bien que pour un calculateur massivement parallèle. C'est pourquoi les OS modernes offrent une panoplie de primitives de synchronisation.



# 3. Primitives de synchronisation
## 3.1. Instructions atomiques
La plupart des processeurs garantissent l'**atomicité de certaines actions simples**. Par exemple, la lecture et l'écriture d'une zone mémoire est souvent atomique, même sur une machine multiprocesseurs.

Sur x86 :
- Les accès mémoire (lecture ou écriture) sont atomiques
- Les instructions du type lecture/modification/écriture sont atomiques si le bus mémoire n’est pas verrouillé par un autre processus au milieu de l’exécution de l’instruction. Elle est donc atomique sur une machine monoprocesseur (l’instruction ne peut être interrompue par une interruption).
- Les instructions précédentes peuvent être rendues atomiques en les préfixant par le modifieur LOCK. Le processeur obtient l’accès exclusif au bus mémoire.

Sur Windows : les applications et drivers doivent passer par la **HAL** (cf [[(OS) 1 - Introduction#5.1. Hardware Abstraction Layer (HAL)]]) pour interagir avec le matériel et cette abstraction propose notamment des fonctions atomiques d'incrémenter, de décrémenter, de comparer une valeur, etc.

Sur Linux : un concept similaire à celui du HAL Windows fournit une abstraction avec un ensemble de fonctions atomiques permettant de lire ou écrire un entier, d'ajouter ou de soustraire une valeur à un entier et de tester un entier. Ces instructions sont implémentées en assembleur pour chacune des architectures supportées par Linux.

Plutôt que d'utiliser des instructions assembleur directement dans le code (ce qui n'est pas portable sur d'autres architecture), on peut utiliser des extensions de compilateur comme les fonctions **builtins** fournies par GCC (ex : `__sync_val_compare_and_swap` est atomique).


## 3.2. Barrières mémoire
### 3.2.1. Ordre d'exécution
L’ordre d’exécution des instructions écrites dans un langage de haut niveau **n’est pas garanti**. Il peut notamment être **modifié par le compilateur** pour optimiser la vitesse d'exécution du code.

MAIS il est **parfois important de garantir l'ordre d'exécution** de deux instructions consécutives ! Par exemple, si l'on veut insérer un élément `new` à la place d'un élément `cur` dans une liste chaînée (`cur` est précédé par l'élément `prev`, on doit modifier 2 pointeurs : `new->next = cur->next` et `prev->next = new`. Si on modifie les pointeurs dans cette ordre, on peut lire la liste pendant qu'on la modifie mais si l'ordre est inversé, cela pose problème !

Pour s'assurer que le compilateur ne modifiera pas l'ordre des 2 instructions, il faut mettre une barrière mémoire entre les deux.

### 3.2.2. Dépendances entre instructions
Prenons comme exemple les 2 instructions suivantes :
```
mov dst1, src1
mov dst2, src2 
```
Peut-on paralléliser ou inverser les 2 instructions ?
-> Cela dépend des opérandes en source et destination !

- Si `src1 = src2`, il n'y a pas de dépendance et les instructions peuvent être interverties
- Si `dst1 = dst2`, la première instruction peut ne pas être exécuté
- Si `dst1 = src2`, il y a **dépendance** car le contenu de src2 dépend de la première instruction
- Si `src1 = dst2`, il y a **dépendance** car le contenu de src1 ne doit pas être modifié par la seconde instruction
- Si `dst2 = [dst1]`, il y a **dépendance** car l'adresse calculée dans la deuxième instruction dépend du résultat de la première

Le processeur analyse ses 3 types de dépendances et les supprime en utilisant plus de registres quand c'est possible. Par exemple :
```
mov eax, ebx
add edx, eax
mov eax, ecx
```
Si on utilise un autre registre que `eax` sur la troisième instructions, on supprime une dépendance !
=> le processeur **supprime des dépendances** pour **exécuter des morceaux de code en parallèle** quand c'est possible, ce qui peut changer l'ordre d'exécution.

### 3.2.3. Barrières mémoire
Une **barrière mémoire** assure que les instructions placées avant elle sont terminées lorsque commence l’exécution des instructions placées après elle.
Une barrière mémoire peut agir uniquement sur les lectures, uniquement sur les écritures ou sur les deux à la fois.

L'implémentation des barrières mémoire dépend de l'architecture. Linux fournit une série de fonction codées en assembleur pour s'abstraire de l'architecture sous-jacente. Windows fait de même avec les instructions `mfence`, `lfence` et `sfence`.


## 3.3. Sémaphores
Un sémaphore permet la **synchronisation entre plusieurs processus** souhaitant accéder de manière exclusive à un objet partagé.

Le sémaphore est créé avec une valeur **val** correspondant au nombre d'**accès simultanés** à la ressource autorisés.
Un processus souhaitant **accéder à une ressource** réalise une opération **P** sur le sémaphore:
- Si val est supérieur à 0, le processus continue et val est décrémenté
- Si val est égale à 0, le processus est bloqué et sera réveillé quand val sera supérieur à 0
Un processus qui **libère une ressource** réalise l'opération **V** sur le sémaphore, qui incrémente val de 1.

### 3.3.1. Mutex VS sémaphore binaire
Le sémaphore binaire ne peut être acquis que par un processus à la fois, comme un mutex, cependant, le **mutex** doit être relâché par le même thread que celui qui l'a acquis, ce qui n'est pas le cas du sémaphore. 
Un **sémaphore binaire** (val = 0 ou 1) peut être utilisé comme primitive de synchronisation ou de signalisation entre 2 processus : le premier thread se bloque en appelant P et attend qu'un second thread le débloque en appelant V. Ainsi, le premier processus attend que le second ait terminé une action et le but n'est pas uniquement d'empêcher un accès concurrent à une ressource mais aussi de coordonner des actions entre 2 threads.

### 3.3.2. Sémaphores et interruptions
Les sémaphores sont conçus pour les **accès concurrents longs** et un processus qui obtient l'accès à une ressource peut être lui-même bloqué alors qu'il détient le verrou.
Les processus en attente du verrou sont mis dans une liste d'attente et l'ordonnanceur ne les exécutera pas tant que le verrou qu'ils attendent n'est pas libéré.

Lorsque l'on appelle le sémaphore dans notre programme (espace utilisateur), il se produit une interruption et la fonction P ou V du sémaphore sera exécuté en mode superviseur. Selon la fonction appelé et l'état de la liste d'attente, l'ordonnanceur sera appelé ensuite.

Le code des fonctions P et V modifie une structure de données partagée (compteur et liste d'attente) et doit être **sûr du point de vue de la concurrence** ! Pour cela, on désactive les interruptions (ce qui est suffisant en environnement mono-processeur), on utilise des instructions atomiques pour modifier le compteur et un **verrou tournant** pour modifier la liste d'attente (on ne peut évidemment pas utiliser un sémaphore pour ça - problème de l’œuf et de la poule).


## 3.4. Verrou tournant (spinlock)
Lorsqu'un processus utilise un verrou tournant, il est en **attente active** (boucle while infinie). 
On utilise un verrou tournant plutôt qu'un sémaphore lorsque l'on souhaite obtenir un verrou pour une **très petite durée** et que la phase d'attente sera courte. Dans ce cas, un sémaphore n'est pas adapté car il met les processus en attente en sommeil, ce qui cause des changements de contexte, ce qui est coûteux (cf [[(OS) 3 - Ordonnancement#2.3. Changement de contexte]]).

Il ne faut pas endormir le processus qui détient le spinlock, sinon il ne libérera pas le verrous et les processus en attente continueront de consommer des ressources CPU (= temps processeur) dans leur attente active.


## 3.5. Sleep lock
C'est une particularité de xv6.
Parfois, il arrive des situations où l'on tient un verrou tournant mais le calcul ne peut plus progresser : il faut que le processus s'endorme en attendant une ressource non disponible. Par conséquent, il faut que le processus relâche le verrou tournant sous peine de bloquer d'autres processus.
Au moment de s'endormir, le processus fait appel à la fonction `sleep(channel, spinlock)` qui relâche le spin lock, met le processus dans l'état SLEEPING et l'abonne au channel (i.e. lorsque la ressource sera libéré, il sera réveillé par l'ordonnanceur). Lorsque le processus se réveille, il ré-acquiert le verrou tournant automatiquement. 


## 3.6. Futexes
Futexe : fast user-space mutex

Les **sémaphores** en espace utilisateur nécessitent un **appel système** pour chacune des opération P et V. Dans le cas où un processus prend le sémaphore sans contention (i.e. le sémaphore est disponible), on pourrait réaliser cette opération en espace utilisateur, sans appel système. C'est exactement le principe des futexes.

Un **futex** est un compteur partagé en espace utilisateur qui vaut :
- 1 si le futex est libre
- 0 s'il est verrouillé sans contention
- -1 (ou négatif) s'il est verrouillé avec contention

Pour verrouiller le futex, le processus décrémente atomiquement le futex et vérifie que sa nouvelle valeur soit 0. Si c'est le cas il a obtenu le futex et peut continuer **sans appel système**. Sinon, un autre processus détient déjà le futex (i.e. il y a contention) et le processus courant s'endort.
Pour déverrouiller le futex, le processus incrémente atomiquement le futex et vérifie que sa valeur précédente soit 0. Si c'est le cas il a libéré le futex et aucun processus n'est en attente du futex donc il peut continuer **sans appel système**. Si la valeur précédente du futex était négative, des processus sont en attente et un appel système a lieu.

Les futexes n'ont **pas vocation à être utilisé directement** par les programmeurs mais ils sont utilisés pour implémenter d'autres librairies de synchronisation avec un niveau d'abstraction plus élevé.
Le problème de cela est qu'un utilisateur bienveillant peut créer des bug en voulant utiliser un futex, et, encore pire, un utilisateur malveillant peut y trouver une vulnérabilité (cf CVE-2014-3153).



# 4. Problèmes liés au verrouillage
## 4.1. Interblocage
Un ensemble de processus est en **interblocage** si chaque processus attend un évènement que seul un autre processus de l’ensemble peut provoquer.

L'**utilisation de plusieurs sémaphores simultanément** peut créer un interblocage. Par exemple, un processus veut prendre le sémaphore A puis le sémaphore B mais il s'exécute en parallèle d'un autre processus qui veut prendre le B puis le A... Chaque processus prend un sémaphore et ils sont tous les deux bloqués.

Dans l'OS, il y a de nombreux processus et de nombreux verrous et il faut s'assurer que peut importe l'ordre d'exécution des processus et lesquels sont en parallèle, il n'y aura pas d'interblocage.
-> c'est complexe


## 4.2. Interactions entre l'ordonnanceur et les primitives de synchronisation
L'ordonnanceur peut s'en mêler et endormir un processus qui a un verrou, bloquant ainsi d'autres processus.

C'est l'exemple de la mission spatiale américaine Mars Pathfinder en 1996. Un processus de priorité basse *P3* prend un verrou, est interrompu par l'ordonnanceur qui donne la main à un processus de priorité élevée *P1*  et s'endort. P1 a besoin du même verrou, par conséquent il relâche le processeur. Ensuite un processus de priorité intermédiaire *P2* est choisi par l'ordonnanceur (car P1 attend une ressource et P3 est de priorité plus faible). 
Seulement P2 s'exécute a une tâche très longue à réaliser MAIS P1 devait réinitialiser un compteur pour éviter un redémarrage automatique...
-> redémarrage automatique puis le problème survient de nouveau, etc

C'est un exemple d'**inversion de priorité** dans lequel un processus de priorité élevé est bloqué par un processus de priorité plus faible.

Ce type de problème a été corrigé nativement dans les OS modernes, notamment en utilisant des futexes.
