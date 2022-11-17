```toc
max_depth: 2
```


# 1. Introduction
## 1. Rôle d'un OS, micro-noyaux/noyaux monolithiques, notion de processus, Hardware Abstraction Layer
cf [[(OS) 1 - Introduction]]

### Rôle d'un OS
Le système d'exploitation est l'**interface entre le logiciel et le matériel**. Il expose une API d'appels systèmes permettant aux logiciels d’interagir avec le matériel : gestion des applications, de la mémoire, des E/S, de la communication réseau, ...

Un OS se compose d'au moins 2 espaces :
1. **Espace noyau** : le code du noyau s'exécute dans un espace privilégie avec quasiment tous les droits et accès à toutes les ressources
2. **Espace utilisateur** : les applications et processus utilisateur s'exécute dans un espace de privilège limité et utilisent des appels systèmes qui seront exécutés dans le noyau quand nécessaire.

Les rôles de l'OS :
- Gérer quel *processus/thread* s'exécute, quand et sur quel processeur : c'est le rôle de l'ordonnanceur
- Gérer les *interruptions* et les *exceptions*
- Gérer la *synchronisation*
- Gérer la *mémoire* : création d'une plage d'adressage virtuelle pour chaque processus
- Gérer les entrées/sorties : les périphériques sont gérés avec des drivers
- Gérer la communication réseau

### Micro-noyau / noyaux monolithiques
**Noyaux monolithique** : beaucoup de fonctionnalité dans le noyau, cela permet d’aller plus vite mais la moindre erreur de programmation fait planter l’OS.
**Micro-noyau** : mettre le minimum dans le noyau et tout le reste dans l’espace utilisateur, cela nécessite beaucoup de communication ce qui aboutit à de mauvaises performances

Aujourd’hui, tous les OS sont monolithiques, même si la perte de performance des micro noyau serait sûrement négligeable. 

Le code de Linux faisait moins de 200 milles lignes dans sa première version et fait aujourd'hui plus de 30 millions de lignes (les drivers représentent la majorité de ce code), ce qui explique que personne n'a essayé de le migrer vers un micro-noyau.

### Notion de processus
cf [[(OS) 3 - Ordonnancement#1. Processus et threads]]
Un processus est un programme en cours d'exécution. Il est défini par :
- Un *ensemble d'instructions* à exécuter, qui sont chargées dans la RAM
- Un *espace d'adressage* dans la RAM pour stocker la pile, le tas, etc.
- Des resources permettant des entrées/sorties de données, comme des ports réseaux

Un processus peut être démarré par un utilisateur (ex : lancement d'une application) ou par un autre processus, appelé processus parent. La création d'un nouveau processus se fait toujours via un appel à la fonction 'fork' qui crée un processus fils avec comme parent le processus courant.

Les processus sont identifiés par des PID.

L'OS est en charge de choisir quel processus exécuter à quel moment, c'est le rôle de l'ordonnanceur. 

Les processus assure l'**isolation mémoire** : deux processus ne partagent pas leur données (espace d'adressage différent).

Les processus légers, ou **threads** (fils d'exécution en français), sont créés par un processus et partage la mémoire du processus. Ils possèdent chacun une pile séparée afin que leur variable locales soient indépendantes. Dans un environnement multi-processeurs, ils permettent notamment à un processus d'exécuter plusieurs threads en parallèle.

### Hardware Abstraction Layer
Concept sous **Windows** (il y a un concept similaire sous Linux avec un nom différent).
![[windows_architecture.png]]
La HAL abstrait les variations entre les architectures.
- Gestion des architectures multiprocesseurs
- Types et versions du ou des processeur(s)
- Type de BIOS
- etc

La HAL offre aussi une abstraction des différents bus (PCI, PCI-Express, USB, etc.)
Ainsi, les couches supérieures et notamment le **noyau** et les **drivers** n’ont à être développé qu’en une seule version.



# 2. xv6 et interruptions
## 2. Interruptions / exceptions sous RISC-V : types, gestion des interruptions, activation / désactivation, ...
cf [[(OS) 2 - xv6 et interruptions#3. Interruptions sous RISC-V]]

### Types
Il y a 2 types d’événements pouvant interrompre le traitement normal de l'exécution d'un processeur :
- **Exceptions** : évènements **synchronisés** avec l’exécution du code : l’évènement est déclenché par l’exécution d’une instruction.
  Ex : division par zéro, accès à une adresse mémoire invalide, ...
- **Interruptions** : évènements **asynchrones** déclenchés par la survenue d’un évènement extérieurs.
  Ex : pression sur une touche de clavier, interruption timer toutes les X millisecondes, ...

### Gestion des interruptions
La gestion des interruptions est régie par un certain nombre de **registres de contrôle et d’état** (**CSR**). Les CSR stockent, entre autres,  les informations suivantes :
- L'adresse du gestionnaire de traps (mtvec, stvec, utvec)
- La cause du trap (mcause, scause, ucause)
- Le niveau de privilège avec lequel traiter le trap (medeleg/mideleg, sedeleg/sideleg)
- Le niveau de privilège auquel revenir une fois le trap traité (mpp, spp)
- L'adresse à laquelle continuer l'exécution une fois le trap traité (mepc, sepc uepc)
- Si les interruptions sont activées / désactivées (mie, sie, uie)

Autres CSR utiles : satp - supervisor address translation and protection. Il indique si la pagination est activée ou non, cf [[(OS) 5 - Mémoire virtuelle]].

Exemple d'une interruption ayant lieu en mode utilisateur et traité en mode machine.
Les registres mepc, mstatus, mcause et mtvec sont des CSR en mode noyau. L'instruction mret remet les registres à leur état d'avant l'interruption.
![[interruptions.png]]
![[interruption mode utilisateur.png]]

Architecture de gestion des interruptions :
![[routage des interruptions.png]]
<u>Note</u> : HART = Hardware Thread = un cœur CPU

**CLINT** (Core Local INTerruptor) : composant attaché à chaque cœur programmable qui peut générer des interruptions à intervalle programmable.
-> **interruption timer**
Etre interrompu à intervalle régulier permet à l'ordonnanceur de réparti le temps d'exécution entre plusieurs processus.

**PLIC** (Platform-Level Interrupt Controller) envoie les interruptions émises par l'UART (interaction clavier-écran) et VIRTIO (disque dur).

### Activation / désactivation
On ne peut pas désactiver les exceptions mais on peut désactiver les interruptions (pratique lorsqu'on est en train de traiter une interruption par exemple).
On modifie un CSR pour activer / désactiver les interruptions.


## 3. Appels système : lien avec les interruptions, passage de paramètres, exemples typiques  
cf [[(OS) 2 - xv6 et interruptions#4. Appels système]] 
et [[(OS) 3 - Ordonnancement#3.3.1 Appel système]]

L'OS est l'interface entre le logiciel et le matériel et il met à disposition une API d'appels systèmes permettant aux logiciels d'interagir avec le matériel : la mémoire, les E/S, ...
-> permet de séparer espace utilisateur / espace noyau
-> le programmeur ne contrôle pas l'interaction avec le matériel (évite les erreurs / vulnérabilités)
Ex : read, write, open, close, waitpid, **fork**, **execve**, chdir, chmod, **nice**, **kill**...

### Lien avec les interruptions
Les appels systèmes son traduit en assembleur en l’instruction **ecall**. Cette instruction génère une **interruption** qui est traitée par le système d’exploitation en **mode superviseur**. 

Lors de la gestion de l'interruption, le gestionnaire regarde le registre CSR `scause` qui vaut 8, ce qui correspond à un appel système, par conséquent la fonction `syscall` sera exécuté.

La fonction `syscall` répartit les appels systèmes en fonction de la valeur du **registre a7** qui a été sauvegardé dans la trapframe du processus.

### Passage de paramètre
Les arguments de l'appel système sont passés dans les registres a0-a6.
Attention :
- Les registres sont transférés dans la trapframe du processus lors du passage en mode superviseur pour traiter l'appel système
- Si des pointeurs sont passés en argument, il faut traduire leur adresse (adresse virtuelle espace utilisateur) en adresse de l'espace noyau 

### Exemples types
- Système de fichiers : read, write, open, close, chdir, chmod, ...
- Contrôle des processus : getpid, getppid, kill, **fork**, **wait**, **execve**, ...
- Communication inter-process : pipe, shmat, ...
- Autres :  **nice**, **sbrk** (malloc, free), ...

fork : crée un processus fils à partir du processus courant (duplique la mémoire) et renvoie 0 dans le processus fils et pid_fils dans le processus parent.

execve : charge un binaire et remplace le processus courant.
Détails sur execve ici : [[(OS) 5 - Mémoire virtuelle#4.2.2. Chargement d'un binaire]]

![[fork execve et wait.png]]



## 4. Interruptions sous x86 : descripteurs d'interruption, segments, gestionnaire d'interruption, vecteur d'interruption, masquage des interruptions, PIC
cf [[(OS) 2 - xv6 et interruptions#5. Interruptions en x86]]

### Masquage des interruptions
Les interruptions peuvent être classées en 2 catégories :
- **Interruptions masquables** : une telle interruption peut être masquée ou non, si elle l'est, elle sera ignorée par le processeur. 
   Ex : interruptions des périphériques d'entrée-sortie.
- **Interruptions non-masquables** : quelques évènements critiques peuvent donner lieu à ces interruptions et le processeur ne peut pas les ignorer.
   Ex : pannes matérielles, surchauffe du processeur, ...

Les exceptions peuvent être classées en 3 catégories :
- **Faute** : elle peut être corrigée et une fois corrigée, l'instruction qui l'a provoquée doit être exécutée de nouveau.
   Ex : faute de page (tentative d'accès à la mémoire à une adresse non disponible)
- **Trappe** : elle est reportée immédiatement après l'exécution de l'instruction qui l'a causée
   Ex : point d'arrêt du debugger
- **Abandon** : erreur sérieuse de l'unité de contrôle, le processeur n'est pas capable de continuer l'exécution

Des exceptions peuvent aussi être programmées avec l'instruction `int` (int 0x80 déclenche un appel système), l'instruction `int3` (ajoute un point d'arrêt), ...
![[x86 interruption.png]]

### Vecteur d'interruption
A chaque interruption est associé un entier entre 0 et 255, qui est codé sur 8 bits et appelé **vecteur d'interruption**.
Ex : 0 pour une division par 0, 3 pour un breakpoint, 14 pour une faute de page, ...

### Programmable Interruption Controller (PIC)
Les interruptions sont gérés au **niveau matériel** par le PIC.
Son travail consiste à :
1. Surveiller les lignes d’interruptions en provenance des périphériques.
2. Lorsqu’une ou plusieurs lignes sont levées, en sélectionner une de manière déterministe puis :
	1. Convertir la ligne d’interruption en un vecteur d’interruption.
	2. Stocker ce vecteur dans un registre d’entrée-sortie auquel le processeur pourra accéder.
	3. Lever le signal d’interruption du processeur.
	4. Attendre l’acquittement du processeur.
	5. Abaisser le signal d’interruption.
3. Reprendre à l’étape 1

Chaque ligne d'interruption peut être masquée, dans ce cas elle n'est pas signalée au processeur. Les interruptions masquées ne sont pas perdues, le contrôleur les mémorise et les enverra au processeur lorsque celui-ci les réactivera.
Une ligne d'interruption peut être partagée par de multiples périphériques.

### Segments
Le niveau de privilège courant du processeur est géré par l'**unité de segmentation**. Cette unité :
- Gère la mémoire via des segments contigus (cf [[(OS) 5 - Mémoire virtuelle#1.1. Segmentation]]) 
- Met en application une politique d'accès au différent segments : respect des niveaux d'exécution et transition entre niveau d'exécution

Les processeurs RISC ne possède pas d'unité de segmentation mais possède une unité de gestion de la mémoire par pagination. Les processeurs Intel (x86) possèdent les 2 unités : celle de pagination et celle de segmentation.

Linux et Windows limite l'utilisation de l'unité de segmentation à son seul rôle de mise en
application des droits d’accès à la mémoire et transition entre niveaux d’exécution.
![[segmentation.png]]

### Descripteur d'interruption
À une interruption donnée, on peut associer un des trois types de descripteurs suivants :
- *Task gate* : permet de remplacer le processus en cours d’exécution par un processus donné lorsque l’interruption est levée.
- *Interrupt gate* : permet d’exécuter un **gestionnaire d’interruption** simplement caractérisé par une adresse à laquelle poursuivre l’exécution. L’unité de contrôle masque automatiquement les interruptions masquables.
- *Trap gate* : similaire au type précédent sans masquage des interruptions.

Un descripteur d'interruption indique notamment un sélecteur de segment qui pointe vers le code qui devra être exécuté et le niveau d'exécution minimale auquel l'interruption peut être levée.

Les descripteurs d'interruption sont stockés dans une **table de descripteur d'interruption**.

### Gestionnaire d'interruption
D’une manière générale, tous les systèmes d’exploitation divisent le traitement des interruptions en deux parties :
- Un **traitement de premier niveau**. Pour x86, c’est celui qui est déclenché via la table de descripteurs d’interruption. Son but est de s’exécuter le plus rapidement possible. Il doit au minimum acquitter l’interruption auprès du périphérique qui l’a levée ; puis transférer les éventuelles données associées à cette interruption depuis ou vers le périphérique.
- Un **traitement de second niveau**. Celui-ci peut être retardé et comporter des calculs plus complexes.

Sur Linux :
- Le gestionnaire d'interruption associé aux interruptions masquables est un unique gestionnaire généraliste
- Lors de l'initialisation d'un périphérique, son pilote demande à utiliser une ligne d'interruption et enregistre un gestionnaire d'interruption spécialisé auprès du gestionnaire généraliste
- En cas d'interruption, le gestionnaire généraliste parcourt la liste des gestionnaires spécialisés enregistrés pour la ligne d'interruption et appelle le gestionnaire correspondant

Sur Windows :
- Les interruptions ont un niveau de priorité associé et sont traités par ordre de priorité décroissant
- Après le traitement de premier niveau d'un interruption, si celle ci doit passer par un traitement de second niveau plus lent, alors ce traitement est noté comme une interruption de priorité faible et sera traité lorsque aucune autre interruption plus prioritaire n'est en attente.



# 3. Ordonnancement
## 5. Processus : états, ordonnancement, changement de contexte, priorité  
cf [[(OS) 3 - Ordonnancement]]

### Etats
*RUNNABLE* : processus prêt à être exécuter
*RUNNING* : processus en train d'être exécuter
*SLEEPING* : processus endormi en attente d'une ressource
*ZOMBIES* : processus mort dont le parent ne l'a pas attendu (si le parent meurt avant le fils, le processus est adopté par le processus initial)
![[ordonnanceur linux.png]]

### Ordonnancement
Choisir quel processus exécuter quand et combien de temps (sur chaque processeur !).
L'ordonnanceur optimise des propriétés telles que l'équité entre les processus, le respect de la priorité, l'utilisation du CPU, la réactivité, nombre de tâches réalisées, ...

Un ordonnanceur peut être de 2 types :
- **coopératif** : le processus courant s'exécute jusqu'à ce qu'il fasse un appel bloquant (primitive de synchronisation ou demande d'entrée-sortie) ou qu'il relâche volontairement le processeur
- **préemptif** : en plus des condition du mode coopératif, l'ordonnanceur interrompt le processus s'il a épuisé son **quantum de temps** (il y a des interruptions timer régulières)
  -> c'est le cas de tous les OS

### Changement de contexte
![[interruption mode utilisateur.png]]
Le passage d’un processus à l’autre par l’ordonnanceur est appelé changement de contexte. L'interruption est traité en mode superviseur et appelle la fonction d'ordonnancement.

L'ordonnanceur :
1. **Sauvegarde** l'état du processus courant (registres, ...) dans une zone prévue à cet effet : la **trapframe**
2. **Sélectionne** le prochain processus (ou thread) à exécuter
3. **Rétablit**, depuis la table des processus, les données du processus à exécuter
4. **Fait reprendre** l'exécution du processus à exécuter

Le changement de contexte est une opération coûteuse, surtout parce que le cache est vidé et reremplit avec la table des pages du nouveau processus (cf [[(OS) 5 - Mémoire virtuelle#2.2. Caches]]).
![[espace adressage.png]]

### Priorité
**Round-Robin**
L'algorithme du tourniquet (Round-Robin) donne à chaque processus un quantum de temps pour s'exécuter et quand ce temps est écoulé, il passe au processus suivant.
Cet algorithme est équitable mais n'est pas réactif et ne permet pas de respecter la priorité des processus.

**Round-Robin avec priorités fixes**
Chaque processus a un niveau de priorité et l’ordonnanceur maintient autant de listes que de niveaux. L’ordonnanceur parcourt les listes par ordre décroissant de priorité et exécute le premier processus prêt qu’il rencontre. À chaque niveau de priorité, le tourniquet est appliqué.
Le tourniquet avec priorités fixes prend en compte la priorité mais n’est pas équitable et peut même entraîner des famines (les processus de priorités élevées monopolisent les processeurs).
-> appel système **nice** pour modifier la priorité 

**Round-Robin avec priorités variables**
Cette fois-ci les priorités sont variables, ce qui permet d'éviter les famines. L'implémentation exacte dépend des OS mais globalement la priorité d'un processus diminue quand son quantum de temps est atteint et augmente quand il est lancé après avoir attendu une ressource.




## 6. Processus vs threads : quelles différences ? proposition d'implémentation des threads dans xv6
cf [[(OS) 3 - Ordonnancement#1. Processus et threads]]

### Processus
Un processus est une abstraction de l'exécution d'une application. Il satisfait les 2 propriétés suivantes :
- **Isolation mémoire** : deux processus ne partagent pas leur données
  -> géré par la *mémoire virtuelle* (cf [[(OS) 5 - Mémoire virtuelle]])

- **Concurrence** : deux processus peuvent s'exécuter en même temps :
	- Concurrence réelle : sur une machine multiprocesseur
	- Concurrence simulée : sur une machine monoprocesseur via l'entrelacement de leur fil d'exécution
  -> géré par l'*ordonnanceur*

**Création d'un processus fils**
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

### Thread
Cependant, c'est difficile de partager de la mémoire entre les processus (c'est l'objectif d'ailleurs !).
-> utilisation de thread qui séparent le rôle d'isolation mémoire de celui de la concurrence

A chaque processus on associe au moins un thread, mais il peut en accueillir plus. Tous les threads d'un processus partagent le **même espace d'adressage** : ils peuvent ainsi se partager leurs variables globales mais possèdent chacun une pile séparée afin que leur variable locales soient indépendantes.
-> l'ordonnanceur doit gérer l'exécution des threads

### Implémentation des threads dans xv6
????


## 7. Ordonnancement sous Linux (classes d'ordonnanceurs, priorité statique et dynamique, CFS)  
cf [[(OS) 3 - Ordonnancement#4. Ordonnanceur de Linux]]

### Classes d'ordonnanceurs
Ordonnanceur **préemptif à priorités variables**.
Classiquement cet algorithme est linéaire en le nombre de processus (O(n)), cependant cela pose problème quand le nombre de processus devient grand. C'était le cas jusqu'au noyau 2.6 qui utilise une nouvelle version de l'**ordonnanceur à complexité constante**.

Linux implémente trois **classes d’ordonnancement** :
1. SCHED NORMAL : politique d’ordonnancement par défaut
2. SCHED FIFO : politique d’ordonnancement pour des **tâches temps-réel**, utilisant l’algorithme du tourniquet et sans utilisation de quantum de temps.
3. SCHED RR : politique d’ordonnancement pour des **tâches temps-réel**, utilisant l’algorithme du tourniquet **avec un quantum de temps**.

L'ordonnanceur appelle les classes d'ordonnanceur dans l'ordre de la classe la plus prioritaire à la moins prioritaire. Ainsi, les processus temps réel seront traités en priorité.
-> on se concentre sur les processus normaux par la suite

### Priorité statique
Chaque tâche se voit attribuer une **priorité statique**, i.e. un entier entre 0 et 140 (0 étant la priorité maximale). Les tâches temps-réel ont une priorité entre 0 et 99 tandis que les autres tâches ont une priorité statique par défaut de 120. Lors de sa création, le processus a la même priorité que le processus parent.
On peut modifier la priorité d'un processus avec la fonction **nice** (ajout de -20 à +19).

-> le **quantum de temps** du processus dépend de sa priorité statique

### Priorité dynamique
`priorité dynamique = priorité statique - bonus + 5`

Linux évalue la nature d'une tâche en fonction du temps moyen que le processus passe en attente chaque seconde. Ce temps moyen t est utilisé pour attribuer un **bonus** aux processus. Ce bonus vaut 0 si t est entre 0 et 100ms, 1 si t est entre 100 et 200, ... et 10 si t vaut 1 seconde. Si le bonus vaut 3 alors le processus passe 20-30% de son temps en attente.

La priorité dynamique permet de catégoriser la tâche entre :
- **Tâche interactive** : le plus souvent en attente d'une interaction avec l'utilisateur et/ou le materiel et ne consomme presque jamais son quantum de temps d'un seul coup.
- **Tâche calculatoire** : fait peut appel au système d'exploitation et consomme la totalité de son quantum de temps sans interruption.

Si `priorité dynamique <= 0.75*priorité statique + 28` alors la tâche est considérée comme interactive, sinon elle est considéré calculatoire.
Lorsqu'une tâche épuise son quantum de temps, elle passe des processus actifs au processus expirés et doit attendre qu'il n'y ait plus de processus actifs pour redevenir active SAUF si elle est interactive et peut rester dans les actifs si :
- Le processus expiré le plus ancient est suffisamment jeune
- La priorité du process interactif est plus importante que celle du processus expiré le plus prioritaire

### CFS
Plusieurs **critiques à l'ordonnanceur en temps constant** :
- Mesurer l'interactivité d'une tâche n'est pas simple
- Si il y a uniquement des processus de priorité faible, ils s'exécuteront tour à tour sur de très petits quantum de temps (alors qu'il y a pas de processus prioritaire et qu'ils pourraient en profiter) et cela créera beaucoup de changements de contexte

**Completely Fair Scheduler**
<u>Principe</u> : faire un tourniquet et affecter à chaque processus lors de son exécution un quantum de temps dépendant de sa priorité et de son importance relative aux autres processus en attente (i.e. tient compte du nombre de processus et de leur priorité à tous).

**Latence cible** : intervalle de temps dans lequel tous les processus en attente doivent s'exécuter une fois. Plus elle est faible et plus l'interactivité sera bonne mais plus le coût des changement de contexte sera important.
Pour éviter que le nombre de changement de contexte devienne trop grand quand le nombre de processus augmente, CFS impose une valeur minimale du quantum de temps, qui est appelée **granularité minimale**.

Implémentation :
- Tient à jour le "virtual runtime" de chaque processus, i.e. le temps que le processus a été exécuté divisé par le nombre de processus dans l'état RUNNABLE, le tout pondéré par la priorité du processus et des autres processus en attente
- Lors du choix du nouveau processus a exécuter, choisis le processus avec le plus petit "virtual runtime"



## 8. Ordonnancement sous Windows (classes d'ordonnanceurs, priorité statique et dynamique)
cf [[(OS) 3 - Ordonnancement#5. Ordonnanceur de Windows]]

### Classes d'ordonnanceurs
Ordonnanceur **préemptif à priorités variables**.

Il existe 32 niveaux de priorité, de 0 à 15 pour les **threads à priorité variable**, et de 16 à 31 pour les **threads temps-réels**.

### Priorité statique
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

### Priorité dynamique
Le quantum de temps par défaut de la version server est 6 fois plus élevé que celui de la version personnelle. Ainsi, une requête faite au serveur a plus de chance de se terminer avant de passer à la suivante, mais le système est moins réactif.

Si les quantum de temps variable sont activés (c'est le cas par défaut sur la version personnelle), un thread est considéré comme **au premier plan** s'il fait parti du processus dont la fenêtre est active sur le bureau. Dans ce cas il peut recevoir jusqu'à 2 **boost de priorité** et pour chaque boot de priorité reçu, la priorité du thread augmente de 1 et il hérite d'un quantum de temps supplémentaire !

Des boost de priorité peuvent être appliqués pour d'autres raisons :
- Un thread passe de l'état WAITING à RUNNABLE (une ressource vient de se libérer)
- Un thread est dans l'état RUNNABLE depuis longtemps (éviter les problèmes de famines et d'inversion de priorité, cf [[(OS) 4 - Synchronisation#4.2. Interactions entre l'ordonnanceur et les primitives de synchronisation]])



# 4. Synchronisation
## 9. Pseudo-concurrence, concurrence réelle, Ordre d'exécution des instructions + barrières mémoire
cf [[(OS) 4 - Synchronisation#2.1. Pseudo-concurrence et concurrence réelle]]
et [[(OS) 4 - Synchronisation#3.2. Barrières mémoire]]

### Pseudo-concurrence et concurrence réelle
**Pseudo-concurrence** : un seul flot d’exécution peut avoir lieu à un instant, mais plusieurs fils d’exécution peuvent être entrelacés de manière à donner l’illusion d’une exécution parallèle.
-> architecture **mono-processeur** : l'ordonnanceur entrelace les fils d'exécution de plusieurs processus (cf [[(OS) 3 - Ordonnancement]]).
Il suffit de désactiver les interruptions asynchrones le temps de faire une opération délicate. Mais en pratique on est rarement dans le cas d'une architecture mono-processeur.

**Concurrence réelle** : plusieurs fils d'exécution peuvent avoir lieu simultanément.
-> architecture **multi-processeur** : plusieurs flots d'exécution peuvent accéder simultanément aux mêmes données

### Ordre d'exécution des instructions
L’ordre d’exécution des instructions écrites dans un langage de haut niveau **n’est pas garanti**. Il peut notamment être **modifié par le compilateur** pour optimiser la vitesse d'exécution du code.

MAIS il est **parfois important de garantir l'ordre d'exécution** de deux instructions consécutives ! Par exemple, si l'on veut insérer un élément `new` à la place d'un élément `cur` dans une liste chaînée (`cur` est précédé par l'élément `prev`, on doit modifier 2 pointeurs : `new->next = cur->next` et `prev->next = new`. Si on modifie les pointeurs dans cette ordre, on peut lire la liste pendant qu'on la modifie mais si l'ordre est inversé, cela pose problème !

Pourquoi le compilateur modifie-t-il l'ordre d'exécution ?
-> pour **accélérer la vitesse de traitement**
Comment ?
En réduisant les dépendances entre instructions et en parallélisant

#### Dépendances
Prenons comme exemple les 2 instructions suivantes :
```
mov dst1, src1
mov dst2, src2 
```
Peut-on paralléliser ou inverser les 2 instructions ?
-> Cela dépend des opérandes en source et destination :
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


### Barrière mémoire
Une **barrière mémoire** assure que les instructions placées avant elle sont terminées lorsque commence l’exécution des instructions placées après elle.
Une barrière mémoire peut agir uniquement sur les lectures, uniquement sur les écritures ou sur les deux à la fois.

L'implémentation des barrières mémoire dépend de l'architecture.



## 10. Problèmes de concurrence dans le noyau, primitives de synchronisation (spinlock et sémaphores), interblocage

### Problèmes de concurrence dans le noyau
cf [[(OS) 4 - Synchronisation#2. Synchronisation au sein du noyau]]
*Pseudo-concurrence* : plusieurs fils d'exécution entrelacé par l'ordonnanceur
-> mono-processeur
-> désactiver les interruptions le temps d'une opération délicate suffit
*Concurrence réelle* : plusieurs fils d'exécution ont lieu en même temps et peuvent accéder aux mêmes données
-> multi-pocesseurs

L'OS doit offrir des **primitives de synchronisation** afin de permettre aux différents fils d’exécution de coopérer harmonieusement.
MAIS, il faut éviter le plus possible d'avoir recours à la synchronisation et essayer de l'utiliser uniquement sur des **structures dédiées** et dans de **petites portions de code**. Sinon, les performances de l'OS seront limité et on ne tirera pas vraiment avantages des multi-processeurs.

Les OS doivent fonctionner pour une architecture mono-processeur aussi bien que pour un calculateur massivement parallèle. C'est pourquoi les OS modernes offrent une panoplie de primitives de synchronisation.

### Primitives de synchronisation : spinlock et sémaphore
cf [[(OS) 4 - Synchronisation#3.3. Sémaphores]]
#### Spinlock
Lorsqu'un processus utilise un verrou tournant, il est en **attente active** (boucle while infinie). 
On utilise un verrou tournant plutôt qu'un sémaphore lorsque l'on souhaite obtenir un verrou pour une **très petite durée** et que la phase d'attente sera courte. Dans ce cas, un sémaphore n'est pas adapté car il met les processus en attente en sommeil, ce qui cause des changements de contexte, ce qui est coûteux (cf [[(OS) 3 - Ordonnancement#2.3. Changement de contexte]]).

Il ne faut pas endormir le processus qui détient le spinlock, sinon il ne libérera pas le verrous et les processus en attente continueront de consommer des ressources CPU (= temps processeur) dans leur attente active.

#### Sémaphore
Un sémaphore permet la **synchronisation entre plusieurs processus** souhaitant accéder de manière exclusive à un objet partagé.

Le sémaphore est créé avec une valeur **val** correspondant au nombre d'**accès simultanés** à la ressource autorisés.
Un processus souhaitant **accéder à une ressource** réalise une opération **P** sur le sémaphore:
- Si val est supérieur à 0, le processus continue et val est décrémenté
- Si val est égale à 0, le processus est bloqué et sera réveillé quand val sera supérieur à 0
Un processus qui **libère une ressource** réalise l'opération **V** sur le sémaphore, qui incrémente val de 1.

Les sémaphores sont conçus pour les **accès concurrents longs** et un processus qui obtient l'accès à une ressource peut être lui-même bloqué alors qu'il détient le verrou.
Les processus en attente du verrou sont mis dans une liste d'attente et l'ordonnanceur ne les exécutera pas tant que le verrou qu'ils attendent n'est pas libéré.

Lorsque l'on appelle le sémaphore dans notre programme (espace utilisateur), il se produit une interruption et la fonction P ou V du sémaphore sera exécuté en mode superviseur. Selon la fonction appelé et l'état de la liste d'attente, l'ordonnanceur sera appelé ensuite.

Le code des fonctions P et V modifie une structure de données partagée (compteur et liste d'attente) et doit être **sûr du point de vue de la concurrence** ! Pour cela, on désactive les interruptions (ce qui est suffisant en environnement mono-processeur), on utilise des instructions atomiques pour modifier le compteur et un **verrou tournant** pour modifier la liste d'attente (on ne peut évidemment pas utiliser un sémaphore pour ça - problème de l’œuf et de la poule).

### Interblocage
cf [[(OS) 4 - Synchronisation#4.1. Interblocage]]
Un ensemble de processus est en **interblocage** si chaque processus attend un évènement que seul un autre processus de l’ensemble peut provoquer.

L'**utilisation de plusieurs sémaphores simultanément** peut créer un interblocage. Par exemple, un processus veut prendre le sémaphore A puis le sémaphore B mais il s'exécute en parallèle d'un autre processus qui veut prendre le B puis le A... Chaque processus prend un sémaphore et ils sont tous les deux bloqués.

Dans l'OS, il y a de nombreux processus et de nombreux verrous et il faut s'assurer que peut importe l'ordre d'exécution des processus et lesquels sont en parallèle, il n'y aura pas d'interblocage.
-> c'est complexe



## 11. Futexes : motivation, principe, inconvénients
cf [[(OS) 4 - Synchronisation#3.6. Futexes]]

### Motivation
Futexe : fast user-space mutex

Les **sémaphores** en espace utilisateur nécessitent un **appel système** pour chacune des opération P et V. Dans le cas où un processus prend le sémaphore sans contention (i.e. le sémaphore est disponible), on pourrait réaliser cette opération en espace utilisateur, sans appel système. C'est exactement le principe des futexes.

### Principe 
Un **futex** est un compteur partagé en espace utilisateur qui vaut :
- 1 si le futex est libre
- 0 s'il est verrouillé sans contention
- -1 (ou négatif) s'il est verrouillé avec contention

Pour verrouiller le futex, le processus décrémente atomiquement le futex et vérifie que sa nouvelle valeur soit 0. Si c'est le cas il a obtenu le futex et peut continuer **sans appel système**. Sinon, un autre processus détient déjà le futex (i.e. il y a contention) et le processus courant s'endort.
Pour déverrouiller le futex, le processus incrémente atomiquement le futex et vérifie que sa valeur précédente soit 0. Si c'est le cas il a libéré le futex et aucun processus n'est en attente du futex donc il peut continuer **sans appel système**. Si la valeur précédente du futex était négative, des processus sont en attente et un appel système a lieu.

### Inconvénients
Les futexes n'ont **pas vocation à être utilisé directement** par les programmeurs mais ils sont utilisés pour implémenter d'autres librairies de synchronisation avec un niveau d'abstraction plus élevé.
Le problème de cela est qu'un utilisateur bienveillant peut créer des bug en voulant utiliser un futex, et, encore pire, un utilisateur malveillant peut y trouver une vulnérabilité (cf CVE-2014-3153).


# 5. Mémoire virtuelle
## 12. VMAs, Copy-On-Write, On-Demand Paging : principe, intérêt (pour tout OS). 
cf [[(OS) 5 - Mémoire virtuelle#4. Gestion de la mémoire virtuelle des processus]]

### VMAs
Dans les OS modernes, l'espace d'adressage d'un processus est organisé en zones virtuellement contiguës mais physiquement discontiguës appelées **VMA** (Virtual Memory Area).
Chaque VMA possède les caractéristiques suivantes :
- Une adresse virtuelle de début calée sur une frontière de page
- Une longueur multiple de la taille de page
- Un niveau de privilège minimum pour accéder à la zone (utilisateur, superviseur, ...)
- Des droits d'accès : lecture, écriture et/ou exécution
-> **sorte de segmentation par dessus la pagination**

Les différentes VMA d'un processus proviennent de différentes origines :
- Au chargement du binaire avec l'appel à *execve*, l'OS crée des zones pour la **pile**, le **tas**, le **code** du programme (zone text sur le schéma) et les **données** chargées statiquement
- Des **librairies** chargées dynamiquement (zone de code et de données)
- Projection de fichiers en mémoire
![[espace adressage xv6.png]]

Intérêt : le processus possède son propre espace mémoire et s'exécute comme s'il était seul sur la machine.
-> sécurité : cela empêche aussi un processus d'accéder à d'autres zones mémoire que les siennes.

### Copy-on-Write
Lors d’un l’appel à fork, l’espace d’adressage du père devrait être copié vers l’espace d’adressage du fils mais c'est coûteux. De plus, bien souvent, le processus fils commence par faire appel a execve, ce qui réinitialise l'espace d'adressage fraîchement copié du père et charger le programme du fils.

Le noyau va plutôt essayer de partager ces pages tant que possible. Seulement lorsqu’une écriture sur une telle page survient, le noyau duplique cette page et affecte une copie à chaque processus.
Pour détecter cette situation, le noyau aura rendu la page en lecture seule.
![[copy on write.png]]

### On-Demand Paging
La pagination à la demande consiste à n’allouer des pages physiques que lorsque cela est
nécessaire (tentative d’accès à une zone mémoire).
Ainsi, lorsque le processus est chargé en mémoire la première fois, on ne copie pas ses données dans la RAM mais on les laisse sur le disque. Dans la table des pages du processus, on crée le mapping des pages comme s'il elles existaient vraiment mais on les marque comme invalides. Donc, lorsque le processus voudra accéder à la page pour la première fois, un interruption **page fault** sera lancé et l'OS la traitera et copiera la page demandée depuis le disque dans la RAM. 

Avantages :
- Libérer de la RAM
- Charger le processus plus rapidement


## 13. Pagination dans RISC-V : structure des tables de page, configuration des tables de pages, traduction d'adresse
cf [[(OS) 5 - Mémoire virtuelle#2. Pagination dans xv6]]

### Structure des tables de page
Contexte : les mémoires est découpé en petits fragments appelés des pages de taille 2^t = 2¹² octets.

On a le schéma général suivant avec :
- **SATP** : registre de contrôle et d'état (CSR) indiquant l'adresse de début de la table des pages ET le mode d'adressage à utiliser (pagination ou pas, adresse virtuelle sur 39 bits)
- Les adresses virtuelles sur 64 bits se découpe en 3 parties
	- Une partie inutile (seulement 39 bits sont utilisés dans notre cas)
	- Une partie de 27 bits servant d'**index** dans la table des pages
	- Une partie de 12 bits servant d'**offset** : décalage au sein de la page physique
- Une ligne de la table des pages contient le début d'une adresse physique et les flags de configuration de la page
![[pagination.png]]

### Configuration des tables de pages
On retrouve notamment les flags suivants :
- V : indique si la page réside en mémoire physique ou non (elle peut avoir été déplacé sur le disque swap par manque de RAM)
- R, W et X : indique si la page pointée est accessible en lecture, écriture ou exécution, ou si c'est un répertoire intermédiaire (RWX = 000 - cas des deux premiers niveau de l'arbre) 
- U : indique si la page est accessible en mode utilisateur
- A et D : indique si la page a été accédé en lecture (A) ou en écriture (D) depuis la dernière fois que le flag a été remis à zéro. Cela permet de choisir une page peu utilisée a déplacée dans le disque swap

### Traduction d'adresse
En réalité, un processus a un **espace d'adressage creux** et n'utilise qu'une très petite partie des 2²⁷ entrées possibles de sa table des pages.
-> choix d'une structure de données adaptée aux espaces creux : les **radix-trees**.

On représente les 2²⁷ entrées dans un arbre de profondeur 3 avec chaque nœud représentant 2⁹ = 512 entrées (cf schéma suivant). On obtient au maximum 2¹⁸ feuilles de chacune 2⁹ entrées. L'intérêt de cette structure est de ne créer que le nombre de feuilles (i.e. pages) nécessaires en indiquant dès la première table des pages (pointée par SATP) les entrées qui ne sont pas valides (et donc qui ne pointe pas sur une autre table des pages).

On procède comme suit :
- satp est l'adresse de début de la première table des page
- L2 est utilisé comme index dans la première table des pages
- La valeur obtenue dans la première table des pages est l'adresse de début de la deuxième table des pages
- L1 est utilisé comme index dans la deuxième tables des pages
- ...
![[pagination dans xv6.png]]

Inconvénient : nécessite 4 accès mémoire
-> utilisation d'un cache TLB contenant les dernières traductions d'adresses virtuelles



## 14. Mémoire virtuelle sous x86 : segments, pagination, niveaux de privilège (rings)
cf [[(OS) 5 - Mémoire virtuelle]]
et [[(OS) 2 - xv6 et interruptions#5. Interruptions en x86]]

x86 implémente à la fois une gestion de la mémoire par segmentation et par pagination.

### Segmentation
La mémoire virtuelle est organisée sous forme de **segments** : zone de mémoire physique **contiguë** jouant un rôle précis (zone de code, de données, pile, tas, etc). 

Chaque segment est caractérisé par son adresse de début, sa longueur, le privilège minimal pour y accéder (utilisateur, superviseur, ..)  et des droits d'accès (lecture, écriture et/ou exécution).
Chaque segment est décrit par un descripteur de segment qui est conservé dans une table globale. Les segments à utiliser sont programmés dans des registres spécialisés, que l'on peut qualifier de sélecteur de segment.
![[segmentation.png]]

La segmentation a l'avantage d'être **facile à implémenter** et la traduction d'adresse est rapide à réaliser MAIS elle nécessite des zones de mémoires contiguës ! La **fragmentation** de la mémoire (éparpillement de la mémoire disponible en petite zones) est inévitable et rendra impossible la création de nouveau segments même si la quantité de mémoire disponible est suffisante.

### Pagination
<u>Objectif</u> : plutôt que de créer des segments à partir de zones contiguës en mémoire physique, on veut assembler de plus **petits fragments éparses** pour les rendre **virtuellement contiguës**. Ces petits fragments sont appelés des **pages**.

On découpe la mémoire physique en **pages de tailles fixe PS** avec `PS = 2^t` représentant quelques kilo-octets. Chaque page est située à une adresse physique pa tel que `pa = 0 (mod PS)`.

Chaque adresse virtuelle peut alors être découpée en 2 parties :
- Les t derniers bits représentent un décalage (**offset**) au sein de la page associée à cette adresse virtuelle
- Les n-t premiers bits représentent un **index** de la **table des pages** qui peut être utilisé pour retrouver l'adresse physique du début de la page
![[pagination.png]]

<u>Remarque</u> : la longueur des adresses virtuelles n'est pas forcément la même que celle des adresse physique (x et n sur le schéma au dessus).

### Niveau de privilège (rings)
Les processeurs x86 possèdent 4 niveaux de privilège appelés anneaux ou rings :
1. **Machine** : la machine démarre dans ce mode
2. **Hyperviseur** : un hyperviseur faisant fonctionner plusieurs systèmes d’exploitation en
parallèle fonctionnerait à ce niveau de privilège
3. **Superviseur** : un système d’exploitation fonctionne à ce niveau de privilège. Il peut-être
vu comme une machine virtuelle pour les applications qu’il héberge
4. **Utilisateur** : les applications au sein d’un système d’exploitation fonctionnent à ce
niveau de privilège

La plupart des processeurs RISC ne possèdent que 2 niveaux de privilège (utilisateur et superviseur), donc pour rester portable entre x86 et RISC, Linux et Windows n'utilisent que 2 niveaux de privilège (0 et 3) pour leur fonctionnement interne. Les anneaux intermédiaire sont utilisé par des solutions de virtualisation comme Xen.


## 15. Mécanismes de protection mémoire : utilisateur vs. noyau, contrôle d'accès à la mémoire au sein d'un processus, isolation entre différents processus
cf [[(OS) 1 - Introduction#1. Qu'est ce qu'un système d'exploitation ?]]
et [[(OS) 3 - Ordonnancement#1.1. Processus]]
et [[(OS) 5 - Mémoire virtuelle]]

### Utilisateur vs noyau
Un OS se compose d'au moins 2 espaces :
1. **Espace noyau** : le code du noyau s'exécute dans un espace privilégie avec quasiment tous les droits et accès à toutes les ressources
2. **Espace utilisateur** : les applications et processus utilisateur s'exécute dans un espace de privilège limité et utilisent des appels systèmes qui seront exécutés dans le noyau quand nécessaire.

Quand une application utilisateur veut accéder à des ressources, elle doit faire un appel système qui sera ensuite validé et traité par le noyau.

### Isolation entre différent processus
Un processus est une abstraction de l'exécution d'une application. Il satisfait les 2 propriétés suivantes :
- **Isolation mémoire** : deux processus ne partagent pas leur données
  -> géré par la *mémoire virtuelle* (cf [[(OS) 5 - Mémoire virtuelle]])

- **Concurrence** : deux processus peuvent s'exécuter en même temps :
	- Concurrence réelle : sur une machine multiprocesseur
	- Concurrence simulée : sur une machine monoprocesseur via l'entrelacement de leur fil d'exécution
  -> géré par l'*ordonnanceur*

Création d'un nouveau processus avec fork = création d'un nouvel espace d'adressage et duplication de la mémoire du processus parent.

### Contrôle d'accès à la mémoire au sein d'un processus
Un des rôles principaux d’un système d’exploitation est d’**isoler la mémoire des processus**.
Cette isolation est basée sur l’utilisation de la mémoire virtuelle.
Lors de l’exécution d’un programme, le processus ne manipule pas des adresses physiques (sauf pendant la phase de démarrage) mais des **adresses virtuelles**.
-> espace d'adressage dédié au processus





## 16. Allocation mémoire : buddy allocator et slab allocator
cf [[(OS) 5 - Mémoire virtuelle#3. Allocateur au sein du noyau]]

L'allocateur mémoire le plus simple consiste à allouer une seule page mémoire à la fois : on stocke toutes les pages mémoires libre dans une liste chaînée et on vient retirer ou ajouter une page à cette liste.
MAIS il alloue toujours la même quantité de mémoire : il en alloue trop pour les besoins de petits espace et il faut l'appeler de nombreuse fois pour allouer de gros espace.

### Buddy allocator
Cet allocateur permet d'obtenir des zones mémoire physiquement contiguë de taille `2^k * GS` octets, avec :
- GS la **granularité**, i.e. la taille minimale allouable par cet allocateur
- k entre 0 et MAX
L'adresse physique pa de la zone retournée est un multiple de `2^k GS`.

L'allocateur maintient un tableau de MAX éléments où le k-ième élément est une liste de zones libres de longueur 2^k GS octets.

#### Initialisation
A l'initialisation, l'ensemble de la mémoire disponible est découpée en zones de taille `2^MAX GS`, comme sur le schéma suivant :
![[buddy allocator 1.png]]

#### Allocation
Lors de la demande d'allocation de n octets, la demande est arrondie à la taille supérieure la plus proche de la forme 2^k0 GS. On positionne k = k0.
Ensuite, on considère la liste à l'index k du tableau : 
- Si la liste est vide, on incrémente k et on recommence
- Si la liste n'est pas vide, la tête de liste permet de servir la demande, mais on a potentiellement k>k0 et donc on alloue trop de mémoire
  -> pour éviter ça, on morcelle la zone 2^k GS en morceaux de taille 2^i GS avec i entre k0 et k-1 que le replace dans le tableau(on obtient 2 morceaux de taille 2^k0 GS et on alloue l'un des 2).
![[buddy allocator 2.png]]
Pour pouvoir libérer un bloc, on a besoin de retrouver, à partir de son adresse pa, la taille de la zone allouée.
-> il faut stocker des **métadonnées supplémentaires** 

#### Libération
On souhaite libérer le bloc de mémoire à l'adresse pa :
1. On détermine sa taille 2^k GS à l'aide des métadonnées
2. On ajoute pa à la liste d'index k dans le tableau
3. On **défragmente** la liste concernée : on regarde si la zone libérée est à coté d'une autre zone libre de même taille (**buddy zone**) et si c'est le cas on les fusionne et on recommence la défragmentation dans la liste de taille supérieure
![[buddy allocator 3.png]]

#### Avantages et inconvénients
Cet allocateur reste simple à implémenter, alloue des zones de taille variables et permet de ne pas trop fragmenter la mémoire.
MAIS il nécessite de maintenir des **métadonnées** supplémentaires qui ont une **taille inversement proportionnelle à la granularité GS**. Mais plus GS est grand, plus les métadonnées sont petites, mais aussi plus on perd de la mémoire avec les petites allocations (on alloue une grande zone mémoire car GS est grand...).


### Slab allocator
Allocateur conçu pour palier au problème du buddy allocator sur l'allocation de petites zones mémoire et en particulier **pour les structures de données**.

<u>Principe</u> : le slab allocator prépare une ou plusieurs pages remplies de structure de données initialisées, puis lorsqu'une structure est demandée, il en renvoie une de libre et lorsqu'une structure doit être libérée, il la garde prête pour la prochaine allocation.

Il est particulièrement adapté aux allocations et désallocations fréquentes sur les structures de données les plus utilisées du noyau.




## 17. Pagination sous RISCV : small pages, mega pages, giga pages - avantages et inconvénients
cf [[(OS) 5 - Mémoire virtuelle#2.3. Méga et Giga-pages]]

### Small pages
Pagination classique.
On procède comme suit :
- satp est l'adresse de début de la première table des page
- L2 est utilisé comme index dans la première table des pages
- La valeur obtenue dans la première table des pages est l'adresse de début de la deuxième table des pages
- L1 est utilisé comme index dans la deuxième tables des pages
- ...
![[pagination dans xv6.png]]

### Mega et giga pages
Les flags R, W et X permettent d'indiquer si la page pointée par le PPN est un répertoire intermédiaire ou un page physique. On peut alors décider de ne pas faire les 3 étages de traduction mais de n'en faire que 2 (méga-page) ou que 1 (giga-page).

Dans ce cas L0 (et L1) font partis de l'offset de la page physique et la page aura alors une taille de 2^(12+9) pour la méga-page et 2^(12+18) pour la giga-page.

<u>Avantages</u> :
- Avoir une **page contiguë en mémoire de 2 Mo ou 1 Go en seulement 3 ou 2 accès mémoire** (ou lieu de `2⁹*4` ou `2¹⁸*4` accès mémoire).
- Economiser l'espace des tables de pages
- 2 Mo ou 1 Go de mémoire virtuelle n'occupe qu'une entrée du cache TLB

<u>Inconvénient</u> :
- Besoin de grande quantité de mémoire physique contiguë

#### Méga-pages
![[megapage.png]]

#### Giga-pages
![[gigapage.png]]


