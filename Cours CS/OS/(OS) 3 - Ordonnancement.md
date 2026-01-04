```toc

```

# 1. Processus et threads
## 1.1. Processus
Un processus est une abstraction de l'exécution d'une application. Il satisfait les 2 propriétés suivantes :
- **Isolation mémoire** : deux processus ne partagent pas leur données
  -> géré par la *mémoire virtuelle* (cf [[(OS) 5 - Mémoire virtuelle]])

- **Concurrence** : deux processus peuvent s'exécuter en même temps :
	- Concurrence réelle : sur une machine multiprocesseur
	- Concurrence simulée : sur une machine monoprocesseur via l'entrelacement de leur fil d'exécution
  -> géré par l'*ordonnanceur*


## 1.2. Création d'un processus fils
La seule façon de créer un processus est de faire appel à la fonction **fork**. Cette fonction duplique le processus parent (copie de l'espace d'adressage) et retourne :
- 0 dans le processus fils
- le PID du fils dans le processus parent
Le processus fils exécute le même code que le parent, à partir du fork.

Le processus parent a la possibilité d'attendre la fin de l'exécution du fils avec la fonction **wait**. Pendant que le parent attend, il est dans l'état sommeil et sera réveillé quand le fils termine.

Pour que le processus fils exécute un code différent de celui du père, il peut faire appel à la fonction **execve**. Elle réinitialise l'espace d'adressage du processus appelant (le fils dans ce cas), charge un nouveau binaire en mémoire, initialise une nouvelle pile et un nouveau tas et transfère le flot d'exécution au point d'entrée du programme.

Création typique d'un processus fils avec fork, wait et execve :
![[fork execve et wait.png]]

<u>Exemple du shell</u> :
Le shell consiste en une boucle infinie qui parse les commandes envoyées par l’utilisateur sur la ligne de commande et appelle fork afin d’exécuter l’application demandée par l’utilisateur. Le processus fils se change en la commande appelée via l’appel système execve.

## 1.3. Limites des processus
Les 2 critiques suivantes sont faites aux processus :
- La création d'un processus prend du temps car il faut recopier l'intégralité de la mémoire du processus père.
   -> mécanisme **copy on write** pour recopier la mémoire paresseusement (cf [[(OS) 5 - Mémoire virtuelle#4.3.2. Copy on write]]).
- Il est difficile de partager de la mémoire entre le processus père et le processus fils (c'est le principe d'isolation mémoire).
   -> créer des zones de mémoire partagée (shmat en C).

## 1.4. Threads
Les processus légers, ou threads, ou fil d'exécution, séparent le rôle d'isolation mémoire de celui de la gestion concurrente.
A chaque processus on associe au moins un thread, mais il peut en accueillir plus. Tous les threads d'un processus partagent le **même espace d'adressage** : ils peuvent ainsi se partager leurs variables globales mais possèdent chacun une pile séparée afin que leur variable locales soient indépendantes.
-> l'ordonnanceur doit gérer l'exécution des threads


# 2. Généralités sur l'ordonnancement
L'ordonnanceur attribue successivement les processus au(x) processeur(s). Il optimise une ou plusieurs propriétés parmi :
- Equité : chaque processus bénéficie d'un temps d'exécution équitable
- Respect de la priorité : les processus les plus prioritaires bénéficient de plus de temps
processeur
- Équilibre : toutes les parties du système sont occupées au mieux
- Maximisation de l’usage CPU
- Capacité de traitement : optimise le nombre de travaux traités sur une durée donnée.
- Délai de rotation : minimise le délai moyen entre le début et la fin de chaque traitement.
- Réactivité : minimise le temps de réponse aux requêtes de l’utilisateur.

## 2.1. Ordonnancement coopératif
C'est un mode d'ordonnancement historique mais qui n'est plus utilisé aujourd'hui dans les systèmes d'exploitation.

Un algorithme d'ordonnancement coopératif laisse le processus courant s'exécuter jusqu'à ce que celui-ci soit bloqué par :
- Une **demande d'entrée-sortie** (ex :le processus demande de lire/écrire un fichier)
- Une **interruption d'entrée-sortie** (ex : frappe au clavier)
- Une **primitive de synchronisation** (cf [[(OS) 4 - Synchronisation]])
- Un **relachement volontaire** du processeur : le processus appelle la fonction **yield**

La création et la terminaison d’un processus entraı̂nent également un appel à l’ordonnanceur.

L'ordonnancement coopératif ne permet pas d'assurer l'équité et un processus peut potentiellement mobiliser le processeur sans jamais le relâcher !
-> il faut une confiance absolue dans les processus pour l'utiliser.

### 2.1.1. Coroutines
Certains langages de programmation proposent la notion de coroutine qui est une extension de celle de routine (ou fonction) et qui permet de donner l’illusion que plusieurs traitements (les coroutines) ont lieu en parallèle via un ordonnancement coopératif effectué par le support d’exécution (runtime) du langage.

Contrairement aux fonctions classiques d'un même programme qui utilisent toute la même pile, les coroutines ont leur propre environnement d'exécution et donc leur propre pile.

Ex : les générateurs en python sont des coroutines et ils utilisent le mot clé yield.


## 2.2. Ordonnancement préemptif
Un algorithme d'ordonnancement préemptif laisse le processus s'exécuter jusqu'à ce qu'un des évènements évoqués pour l'ordonnancement coopératif se produise, ou que le **quantum de temps** du processus soit écoulé.

Une interruption matérielle (timer) est déclenchée régulièrement et au bout d'un quantum de temps, l'OS redonne la main à l'ordonnanceur, qui change le processus en cours d'exécution.


## 2.3. Changement de contexte
Le passage d’un processus à l’autre par l’ordonnanceur est appelé changement de contexte
(context switch).

L'ordonnanceur :
1. **Sauvegarde** l'état du processus courant (registres, ...) dans une zone prévue à cet effet : la **trapframe**
2. **Sélectionne** le prochain processus (ou thread) à exécuter
3. **Rétablit**, depuis la table des processus, les données du processus à exécuter
4. **Fait reprendre** l'exécution du processus à exécuter

Le changement de contexte est une opération coûteuse, surtout parce que le cache est vidé et reremplit avec la table des pages du nouveau processus (cf [[(OS) 5 - Mémoire virtuelle#2.2. Caches]]).


## 2.4. Algorithmes d'ordonnancement
### 2.4.1. Round-Robin
L'algorithme du tourniquet (Round-Robin) donne à chaque processus un quantum de temps pour s'exécuter et quand ce temps est écoulé, il passe au processus suivant.
Cet algorithme est équitable mais n'est pas réactif et ne permet pas de respecter la priorité des processus.

### 2.4.2. Round-Robin avec priorités fixes
Chaque processus a un niveau de priorité et l’ordonnanceur maintient autant de listes que de niveaux. L’ordonnanceur parcourt les listes par ordre décroissant de priorité et exécute le premier processus prêt qu’il rencontre. À chaque niveau de priorité, le tourniquet est appliqué.
Le tourniquet avec priorités fixes prend en compte la priorité mais n’est pas équitable et peut même entraîner des famines (les processus de priorités élevées monopolisent les processeurs).

### 2.4.3. Round-Robin avec priorités variables
Cette fois-ci les priorités sont variables, ce qui permet d'éviter les famines. L'implémentation exacte dépend des OS mais globalement la priorité d'un processus diminue quand son quantum de temps est atteint et augmente quand il est lancé après avoir attendu une ressource.


# 3. Ordonnanceur de xv6
Il s'agit d'un **Round-Robin préemptif**.

## 3.1. Organisation de la mémoire virtuelle d'un processus
![[espace adressage.png]]
### 3.1.1. Trampoline
La page **trampoline** est projetée à la même adresse virtuelle aussi bien en espace noyau qu'en espace utilisateur. Elle est accessible en lecture/exécution et partagée par tous les processus. Elle **gère le passage d'un mode à l'autre** et contient le code à exécuter pour gérer les interruptions (uservec et userret par exemple).
Si une interruption a lieu en mode utilisateur et doit être gérée en mode superviseur alors le registre stvec contiendra l'adresse de la fonction uservec (cf [[(OS) 2 - xv6 et interruptions]]).

### 3.1.2. Trapframe
La page **trapframe** est projetée sur une page située en dessous de la page trampoline. Elle est accessible en lecture/écriture et différente dans chaque processus. Elle sert à sauvegarder le contexte d’un processus lors de son interruption, notamment les registres.


## 3.2. Traitement des interruptions
### 3.2.1. Interruptions en mode utilisateur
![[interruption mode utilisateur.png]]

### 3.2.2. Interruptions en mode superviseur
Les interruptions en mode superviseur sont traitées dans le noyau.
![[interruption mode superviseur.png]]


## 3.3. En vrac
### 3.3.1 Appel système
On effectue un appel système avec la fonction **ecall**. Cela produit une interruption qui est traitée en mode superviseur. Le registre `scause` vaut 8, ce qui correspond à un appel système, par conséquent la fonction `syscall` sera exécuté.
La fonction `syscall` dispatche les appels systèmes en fonction de la valeur du **registre a7** qui a été sauvegardé dans la trapframe du processus.

### 3.3.2 Représentation CPU et processus
Chaque CPU est doté de son propre ordonnanceur
Chaque CPU est décrit par une structure de données **cpu**, qui contient entre autre :
- Une structure **proc** qui décrit le processus actuellement en train de s'exécuter
- Une structure **context** permettant de sauvegarder le contexte du noyau lorsqu'il passe la main à un nouveau processus

La structure proc contient en particulier :
- state : l'état du processus vis à vis de l'ordonnanceur : **SLEEPING**, **RUNNABLE**, **RUNNING** ou **ZOMBIE** (Le processus est zombie s'il se termine mais son parent ne l'attend avec un wait. Les processus dont le parent termine sont adoptés par le processus initial. Les zombies sont difficiles à éliminer.)
- kstack : l’adresse virtuelle dans l’espace d’adressage du noyau de la page stockant la pile en contexte noyau du processus
- pagetable : table des pages du processus, afin de pouvoir rétabli son espace d'adressage virtuel
- tf : l’adresse virtuelle dans l’espace d’adressage du noyau de la page stockant la trapframe du processus

La fonction `swtch(struct context* old, struct context* new)` fait le changement de contexte, cf [[#2.3. Changement de contexte]].

### 3.3.3. Interruption timer
Les interruptions timer sont en fait traitées par la fonction timervec , en mode machine (M), qui la retransmet au mode superviseur. Le superviseur fait appel à la fonction uservec et analyse la cause de l'interruption : comme c'est une interruption timer, il laisse la main avec la fonction yield. Le processus passe dans l'état RUNNABLE et l'ordonnanceur est appelé pour décider du prochain processus à exécuter.



# 4. Ordonnanceur de Linux
![[ordonnanceur linux.png]]

Ordonnanceur **préemptif à priorités variables**.
Classiquement cet algorithme est linéaire en le nombre de processus (O(n)), cependant cela pose problème quand le nombre de processus devient grand. C'était le cas jusqu'au noyau 2.6 qui utilise une nouvelle version de l'**ordonnanceur à complexité constante**.

Linux implémente trois **classes d’ordonnancement** :
1. SCHED NORMAL : politique d’ordonnancement par défaut
2. SCHED FIFO : politique d’ordonnancement pour des **tâches temps-réel**, utilisant l’algorithme du tourniquet et sans utilisation de quantum de temps.
3. SCHED RR : politique d’ordonnancement pour des **tâches temps-réel**, utilisant l’algorithme du tourniquet **avec un quantum de temps**.

L'ordonnanceur appelle les classes d'ordonnanceur dans l'ordre de la classe la plus prioritaire à la moins prioritaire. Ainsi, les processus temps réel seront traités en priorité.


## 4.1. Priorité statique
Chaque tâche se voit attribuer une **priorité statique**, i.e. un entier entre 0 et 140 (0 étant la priorité maximale). Les tâches temps-réel ont une priorité entre 0 et 99 tandis que les autres tâches ont une priorité statique par défaut de 120. Lors de sa création, le processus a la même priorité que le processus parent.
On peut modifier la priorité d'un processus avec la fonction **nice** (ajout de -20 à +19).

## 4.2. Quantum de temps
Les **processus normaux** (non temps-réel) se voient attribuer un quantum de temps de base, qui découle de leur priorité statique.

Un processus peut s'exécuter sur un processeur tant que :
- Aucun processus plus prioritaire n'est prêt à s'exécuter
- Il n'est pas bloqué par un appel système bloquant
- Il n'a pas épuisé son quantum de temps
Lorsqu'un processus consomme l'intégralité de son quantum de temps, il est préempté et son quantum de temps est rechargé. S'il reste le processus le plus prioritaire, il continue son exécution.

Son P la priorité de la tâche et Q son quantum de temps de base, on a :
`Q = (140 - P)*20` si P est entre 100 et 119
`Q = (140 - P)*5` si P est entre 120 et 139

## 4.3. Tâches interactives VS tâches calculatoires
Une **tâche interactive** est le plus souvent en attente d'une interaction avec l'utilisateur et/ou le materiel et ne consomme presque jamais son quantum de temps d'un seul coup.
Une **tâche calculatoire** fait peut appel au système d'exploitation et consomme la totalité de son quantum de temps sans interruption.

Les tâches peuvent être partiellement interactives ou calculatoires, et peuvent évoluer au cours de leur exécution.

### 4.1.1. Evaluation de la nature d'une tâche
Linux évalue la nature d'une tâche en fonction du temps moyen que le processus passe en attente chaque seconde. Ce temps moyen t est utilisé pour attribuer un **bonus** aux processus. Ce bonus vaut 0 si t est entre 0 et 100ms, 1 si t est entre 100 et 200, ... et 10 si t vaut 1 seconde. Si le bonus vaut 3 alors le processus passe 20-30% de son temps en attente.

On note alors : `priorité dynamique = priorité statique - bonus + 5`.
Par conséquent, plus la tâche est en attente, plus son bonus augmente, plus sa priorité dynamique diminue (les priorités les plus petites sont les plus importantes).

Si `priorité dynamique <= 0.75*priorité statique + 28` alors la tâche est considérée comme interactive, sinon elle est considéré calculatoire.


## 4.4. Algorithme de l'ordonnanceur
Chaque processeur possède deux ensembles de processus :
- Les **processus actifs** : processus n'ayant pas encore consommé la totalité de leur quantum de temps
- Les **processus expirés** : processus ayant consommé la totalité de leur quantum de temps
Les processus expirés ne sont pas ordonnancés. Ils redeviennent actifs lorsqu'ils n'y a plus aucun processus actif.
![[linux ordonnanceur priorites.png]]
Les processus interactifs peuvent être laissés dans les processus actifs lorsque leur quantum de temps est épuisé (et ce quantum réinitialisé) si :
- Le processus expiré le plus ancient est suffisamment jeune
- La priorité du process interactif est plus importante que celle du processus expiré le plus prioritaire

<u>Remarque</u> : lorsqu'il n'y a plus de processus actifs, on intervertit les processus actifs et expirés. Cela se fait par l'inversion des 2 pointeurs (schéma ci dessus) et a une complexité constante !

## 4.5. Critiques et algorithme CFS
### 4.5.1. Critiques de l'ordonnanceur à complexité constante
- Mesurer l'interactivité d'une tâche n'est pas simple.
- La valeur de nice influence de manière non-linéaire la valeur du quantum de temps.
- L'algorithme est mieux adapté aux serveurs qu'aux postes client ou l'interactivité prime sur l'utilisation au plus juste du processeur.
- Si il y a uniquement des processus de priorité faible, ils s'exécuteront tour à tour sur de très petits quantum de temps (alors qu'il y a pas de processus prioritaire et qu'ils pourraient en profiter) et cela créera beaucoup de changements de contexte.

### 4.5.2. Ordonnanceur parfait
Dans l'idéal, si n processus ont la même priorité, ils reçoivent tous une fraction de 1/n du temps des processeurs et cela sur n'importe quel intervalle de temps. Pour réaliser cela il faut couper le temps du processeur en fraction de temps infiniment petite.

En pratique, chaque changement de processus à exécuter entraîne un changement de contexte qui prend du temps (remplacement du cache, etc). Par conséquent, il faut trouver un compromis entre changer souvent pour répartir équitablement les ressources et changer rarement pour limiter le coût des changements de contexte.

### 4.5.3. Ordonnanceur CFS
Pour répondre aux critiques de l'ordonnanceur en à complexité constante, un nouvel ordonnanceur a été introduit dans le noyau 2.6.23 : le **Completely Fair Scheduler** (CFS).

<u>Principe</u> : faire un tourniquet et affecter à chaque processus lors de son exécution un quantum de temps dépendant de sa priorité et de son importance relative aux autres processus en attente (i.e. tient compte du nombre de processus et de leur priorité à tous).

**Latence cible** : intervalle de temps dans lequel tous les processus en attente doivent s'exécuter une fois. Plus elle est faible et plus l'interactivité sera bonne mais plus le coût des changement de contexte sera important.
Pour éviter que le nombre de changement de contexte devienne trop grand quand le nombre de processus augmente, CFS impose une valeur minimale du quantum de temps, qui est appelée **granularité minimale**.

Implémentation :
- Tient à jour le "virtual runtime" de chaque processus, i.e. le temps que le processus a été exécuté divisé par le nombre de processus dans l'état RUNNABLE, le tout pondéré par la priorité du processus et des autres processus en attente
- Lors du choix du nouveau processus a exécuter, choisis le processus avec le plus petit "virtual runtime"


# 5. Ordonnanceur de Windows
Ordonnanceur **préemptif à priorités variables**.

Il existe **32 niveaux de priorité**, de 0 à 15 pour les threads à priorité variable, et de 16 à 31 pour les threads temps-réels.

## 5.1. Calcul de priorité
Les **processus** sont affectés à une **classe de priorité** lors de leur création :
- Temps réel : 24
- Haute : 13
- Au dessus de la normale : 10
- Normal : 8
- En dessous de la normale : 6
- Inactif (idle) : 4

Les **threads** hérite de la priorité du processus et y applique une **priorité relative** :
- Temps critique : +15
- Plus élevé : +2
- Au dessus de la normale : +1
- Normal : 0
- En dessous de la normale : -1
- Plus basse : -2
- Inactif : -15
Les priorités relative temps critique et inactif ne sont pas de réels offset mais correspondent à des seuils de saturation (0, 15, 16 ou 31) : par exemple, un thread temps réel inactif aura une priorité 16 et un thread inactif temps critique aura une priorité 15.

L'ordonnanceur ne se préoccupe que de la priorité finale du thread :
`priorité finale thread = classe priorité processus + priorité relative thread`

![[windows thread states.png]]
Les processeurs sont regroupés en groupes et chaque groupe possède sa liste d'attente de processus dans l'état READY. Cette liste d'attente est organisée en 32 listes : une pour chaque niveau de priorité. Le groupe de processeurs conserve aussi un résumé de ces listes qui consiste en un vecteur de 32 cases valant 1 si la liste de priorité correspondante n'est pas vide et 0 sinon.

## 5.2. Quantum de temps et boost de priorité
Le quantum de temps par défaut de la version server est 6 fois plus élevé que celui de la version personnelle. Ainsi, une requête faite au serveur a plus de chance de se terminer avant de passer à la suivante, mais le système est moins réactif.

Si les quantum de temps variable sont activés (c'est le cas par défaut sur la version personnelle), un thread est considéré comme **au premier plan** s'il fait parti du processus dont la fenêtre est active sur le bureau. Dans ce cas il peut recevoir jusqu'à 2 **boost de priorité** et pour chaque boot de priorité reçu, la priorité du thread augmente de 1 et il hérite d'un quantum de temps supplémentaire !

Des boost de priorité peuvent être appliqués pour d'autres raisons :
- Un thread passe de l'état WAITING à RUNNABLE (une ressource vient de se libérer)
- Un thread est dans l'état RUNNABLE depuis longtemps (éviter les problèmes de famines et d'inversion de priorité, cf [[(OS) 4 - Synchronisation#4.2. Interactions entre l'ordonnanceur et les primitives de synchronisation]])
