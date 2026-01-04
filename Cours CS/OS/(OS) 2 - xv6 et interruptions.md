```toc

```

# 1. xv6 
Système d'exploitation open-source de type Unix à vocation pédagogique.
Il tourne sur l'architecture RISC-V.

Pour le tester, il faut émuler l'architecture RISC-V, par exemple avec `qemu`.
```
git clone https://github.com/mit-pdos/xv6-riscv.git
cd xv6-riscv
make qemu
```


# 2. RISC-V
RISC-V est une architecture récente dont la spécification est open-source.
RISC-V = Reduced Instruction Set Computer 5

32 registres General Purpose : x0 - x31
registres de 32 ou 64 bits selon la machine.

ra : registre de retour   -> l'instruction **ret** signifie `jmp ra`
En RISC-V, le retour d'une fonction est mis dans un registre plutôt que dans la pile comme souvent en x86.
L'utilisation de ces registres définis dans ce tableau sont uniquement des conventions.

Une fonction en appellent une autre... et elles vont toutes les deux utiliser les mêmes registres ! Pour éviter ça, il faut que l'appelant ou l'appelé sauvegarde les registres ailleurs pour pas les modifier.
Par convention :
- Tous les registres sont "caller-save" donc l'appelant doit les sauvegarder
- Sauf pour les registres "s" qui sont "callee-save"

Le processeur RISC-V possède 4 niveaux de privilèges :
1. **Machine** : la machine démarre dans ce mode
2. **Hyperviseur** : un hyperviseur faisant fonctionner plusieurs systèmes d’exploitation en
parallèle fonctionnerait à ce niveau de privilège
3. **Superviseur** : un système d’exploitation fonctionne à ce niveau de privilège. Il peut-être
vu comme une machine virtuelle pour les applications qu’il héberge
4. **Utilisateur** : les applications au sein d’un système d’exploitation fonctionnent à ce
niveau de privilège


# 3. Interruptions sous RISC-V
Il y a 2 types d’événements pouvant interrompre le traitement normal de l'exécution d'un processeur :
- **Exceptions** : évènements **synchronisés** avec l’exécution du code : l’évènement est déclenché par l’exécution d’une instruction.
  Ex : division par zéro, accès à une adresse mémoire invalide, ...
- **Interruptions** : évènements **asynchrones** déclenchés par la survenue d’un évènement extérieurs.
  Ex : pression sur une touche de clavier, interruption timer toutes les X millisecondes, ...
Ces évènements sont qualifiés de **trap**.

## 3.1. Control and Status Registers (CSR)
La gestion des interruptions est régie par un certain nombre de registres de contrôle et d’état (CSR). Les CSR stockent, entre autres,  les informations suivantes :
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

<u>Remarque</u> : on ne peut pas désactiver les exceptions mais on peut désactiver les interruptions (pratique lorsqu'on est en train de traiter une interruption par exemple).


## 3.2. Core Local INTerruptor (CLINT)
Composant attaché à chaque cœur programmable qui peut générer des interruptions à intervalle programmable.
-> **interruption timer**

Etre interrompu à intervalle régulier permet à l'ordonnanceur de répartir le temps d'exécution entre plusieurs processus.

![[routage des interruptions.png]]
<u>Note</u> : HART = Hardware Thread = un cœur CPU (un CPU peut avoir plusieurs cœurs).


## 3.3. Gestionnaire de trappes
Chaque processus possède 2 piles :
- Une pile lorsqu'il s'exécute en mode utilisateur : elle contient les stack frames de chaque fonction appelée (variables locales et adresse de retour), cf [[(Viro) 1 - x86#3.3.2. Cadre de pile]].
- Une pile lorsqu'il s'exécute en mode noyau : lorsqu'une trappe survient, le gestionnaire d'interruption s'exécute et a besoin d'une pile.

Contrôleur d'interruptions :
Le **PLIC** (Platform-Level Interrupt Controller) envoie les interruptions émises par l'UART (interaction clavier-écran) et VIRTIO (disque dur).

Au début de la gestion d'une trappe, tous les registres du processus sont sauvegardés dans la **trapframe** du processus. 



# 4. Appels système
L’instruction assembleur ecall réalise un appel système. Elle génère une trappe qui est traitée par le système d’exploitation (en mode superviseur). L’appel système à réaliser est indiqué via la valeur du registre a7.

Exemple de l'appel système execve :
![[execve.png]]



# 5. Interruptions en x86
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

A chaque interruption est associé un entier entre 0 et 255, qui est codé sur 8 bits et appelé **vecteur d'interruption**.
Ex : 0 pour une division par 0, 3 pour un breakpoint, 14 pour une faute de page, ...


## 5.1. Programmable Interruption Controller (PIC)
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


## 5.2. Niveaux d'exécution
### 5.2.1. Anneaux (rings)
Les processeurs x86 possèdent 4 niveaux de privilège appelés anneaux ou rings. Ils sont numérotés de 0 à 3 avec 0 le niveau le plus privilégié.
La plupart des processeurs RISC ne possèdent que 2 niveaux de privilège (utilisateur et superviseur), donc pour rester portable entre x86 et RISC, Linux et Windows n'utilisent que 2 niveaux de privilège (0 et 3) pour leur fonctionnement interne. Les anneaux intermédiaire sont utilisé par des solutions de virtualisation comme Xen.
-> seule une interruption peut faire passer dans un mode plus privilégié

### 5.2.2. Segments
Le niveau de privilège courant du processeur est géré par l'**unité de segmentation**. Cette unité :
- Gère la mémoire via des segments contigus (cf [[(OS) 5 - Mémoire virtuelle#1.1. Segmentation]]) 
- Met en application une politique d'accès au différent segments : respect des niveaux d'exécution et transition entre niveau d'exécution

Les processeurs RISC ne possède pas d'unité de segmentation mais possède une unité de gestion de la mémoire par pagination. Les processeurs Intel (x86) possèdent les 2 unités : celle de pagination et celle de segmentation.

Linux et Windows limite l'utilisation de l'unité de segmentation à son seul rôle de mise en
application des droits d’accès à la mémoire et transition entre niveaux d’exécution.


## 5.3. Descripteur d'interruption 
À une interruption donnée, on peut associer un des trois types de descripteurs suivants :
- *Task gate* : permet de remplacer le processus en cours d’exécution par un processus donné lorsque l’interruption est levée.
- *Interrupt gate* : permet d’exécuter un **gestionnaire d’interruption** simplement caractérisé par une adresse à laquelle poursuivre l’exécution. L’unité de contrôle masque automatiquement les interruptions masquables.
- *Trap gate* : similaire au type précédent sans masquage des interruptions.

Un descripteur d'interruption indique notamment un sélecteur de segment qui pointe vers le code qui devra être exécuté et le niveau d'exécution minimale auquel l'interruption peut être levée.

Les descripteurs d'interruption sont stockés dans une **table de descripteur d'interruption**.


## 5.4. Gestion des interruptions par l'OS
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