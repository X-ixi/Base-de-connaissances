```toc

```

# 1. Mémoire virtuelle

Un des rôles principaux d’un système d’exploitation est d’**isoler la mémoire des processus**.
Cette isolation est basée sur l’utilisation de la mémoire virtuelle.
Lors de l’exécution d’un programme, le processus ne manipule pas des adresses physiques (sauf pendant la phase de démarrage) mais des **adresses virtuelles**.

Les deux grands mécanismes de traduction d'adresse virtuelles en adresses physiques sont la segmentation (presque plus utilisée aujourd'hui) et la pagination. 

## 1.1. Segmentation
La mémoire virtuelle est organisée sous forme de **segments** : zone de mémoire physique **contiguë** jouant un rôle précis (zone de code, de données, pile, tas, etc). 

Chaque segment est caractérisé par son adresse de début, sa longueur, le privilège minimal pour y accéder (utilisateur, superviseur, ..)  et des droits d'accès (lecture, écriture et/ou exécution).
Chaque segment est décrit par un descripteur de segment qui est conservé dans une table globale. Les segments à utiliser sont programmés dans des registres spécialisés, que l'on peut qualifier de sélecteur de segment.
![[segmentation.png]]

La segmentation a l'avantage d'être **facile à implémenter** et la traduction d'adresse est rapide à réaliser MAIS elle nécessite des zones de mémoires contiguës ! La **fragmentation** de la mémoire (éparpillement de la mémoire disponible en petite zones) est inévitable et rendra impossible la création de nouveau segments même si la quantité de mémoire disponible est suffisante.


## 1.2. Pagination
<u>Objectif</u> : plutôt que de créer des segments à partir de zones contiguës en mémoire physique, on veut assembler de plus **petits fragments éparses** pour les rendre **virtuellement contiguës**. Ces petits fragments sont appelés des **pages**.

On découpe la mémoire physique en **pages de tailles fixe PS** avec `PS = 2^t` représentant quelques kilo-octets. Chaque page est située à une adresse physique pa tel que `pa = 0 (mod PS)`.

Chaque adresse virtuelle peut alors être découpée en 2 parties :
- Les t derniers bits représentent un décalage (**offset**) au sein de la page associée à cette adresse virtuelle
- Les n-t premiers bits représentent un **index** de la **table des pages** qui peut être utilisé pour retrouver l'adresse physique du début de la page
![[pagination.png]]

<u>Remarque</u> : la longueur des adresses virtuelles n'est pas forcément la même que celle des adresse physique (x et n sur le schéma au dessus).



# 2. Pagination dans xv6

## 2.1. Fonctionnement
En RISCV-64 les adresses sont codées sur 64 bits mais la largeur du bus d'adresse (adresses physiques) est de "seulement" 56 bits. Les 8 bits de poids forts sont forcément nuls. La largeur d'une page est de 2¹² octets, soit 4ko. Il y a donc 2^(56-12) = 2⁴⁴ pages en mémoire physique.

### 2.1.1. Registre SATP
Le registre de programmation de la pagination est le **SATP** (Supervisor Address Translation and Protection). C'est un CSR (cf [[(OS) 2 - xv6 et interruptions#3.1. Control and Status Registers (CSR)]]). 
Le registre SATP indique le mode d'adressage à utiliser : adresses physiques au démarrage de l'OS puis d'adresses virtuelles sur 39 bits ensuite.

### 2.1.2. Avec le schéma précédent...
On obtient le schéma précédent avec : t = 12, x = 56 et n = 39 (les adresses virtuelles sont codés sur 64 bits mais seulement 39 sont utilisés).
On a alors 2^(n-t) = 2²⁷ entrées du tableau, chacune d'environ 56 bits, soit environ 8 = 2^3 octets, soit un total de 2³⁰ = 1 Go de mémoire utilisée pour la table des pages d'un seul processus !
-> pas réalisable en pratique

### 2.1.3. Schéma réel
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

### 2.1.4. Les flags
On retrouve notamment les flags suivants :
- V : indique si la page réside en mémoire physique ou non (elle peut avoir été déplacé sur le disque swap par manque de RAM)
- R, W et X : indique si la page pointée est accessible en lecture, écriture ou exécution, ou si c'est un répertoire intermédiaire (RWX = 000 - cas des deux premiers niveau de l'arbre) 
- U : indique si la page est accessible en mode utilisateur
- A et D : indique si la page a été accédé en lecture (A) ou en écriture (D) depuis la dernière fois que le flag a été remis à zéro. Cela permet de choisir une page peu utilisée a déplacée dans le disque swap


## 2.2. Caches
L'utilisation des radix-tree prend certes beaucoup moins de place dans la RAM, mais il nécessite **4 accès mémoire** pour traduire une adresse virtuelle en adresse physique !

Les accès mémoire sont extrêmement lents par rapport au processeur et pour accélérer ces accès et la traduction d'adresses, les processeurs possèdent un **cache** appelé **TLB** qui contient les **dernières traductions d'adresses virtuelles**.

### 2.2.1. Changements de contextes
Chaque changement de contextes modifie SATP ainsi que l'espace d'adressage afin de refléter celui du prochain processus courant. Il y a également une instruction de barrière mémoire pour s'assurer que l'espace d'adressage est changé avant de continuer. Cette barrière mémoire induit un **vidage du TLB**.
-> perte de performances à chaque changement de contexte


## 2.3. Méga et Giga-pages 
Les flags R, W et X permettent d'indiquer si la page pointée par le PPN est un répertoire intermédiaire ou un page physique. On peut alors décider de ne pas faire les 3 étages de traduction mais de n'en faire que 2 (méga-page) ou que 1 (giga-page).

Dans ce cas L0 (et L1) font partis de l'offset de la page physique et la page aura alors une taille de 2^(12+9) pour la méga-page et 2^(12+18) pour la giga-page.

<u>Avantages</u> :
- Avoir une **page contiguë en mémoire de 2 Mo ou 1 Go en seulement 3 ou 2 accès mémoire** (ou lieu de `2⁹*4` ou `2¹⁸*4` accès mémoire).
- Economiser l'espace des tables de pages
- 2 Mo ou 1 Go de mémoire virtuelle n'occupe qu'une entrée du cache TLB

<u>Inconvénient</u> :
- Besoin de grande quantité de mémoire physique contiguë

### 2.3.1. Méga-pages
![[megapage.png]]

### 2.3.2. Giga-pages
![[gigapage.png]]



# 3. Allocateur au sein du noyau

## 3.1. Allocateur de page unique
Cet allocateur alloue une **quantité de mémoire fixe** (1 page) à chaque appel à la fonction **kalloc** et libère une page précédemment allouée avec la fonction **kfree**.
Les fonction kalloc et kfree sont exécutées dans le noyau. 

Implémentation :
- L'ensemble des pages disponibles sont incluses dans une **liste chaînée** (chaque page non-allouée commence par un pointeur vers la prochaine page non-allouée)
- Le noyau conserve un pointeur vers le début de la liste chaînée 
- Il y a un verrou global sur l'allocateur
- kalloc récupère le premier élément de la liste chaînée 
- kfree ajoute un nouvel élément au début de la liste chaînée

<u>Note</u> : kalloc remplit la page à allouer d'un certain caractère et kfree la page à libérer d'un autre, ce qui permet de détecter plus facilement une faute de page.

Cet allocateur est simple, efficace et il stocke toutes les métadonnées nécessaires à son fonctionnement directement dans les pages non-allouées.
MAIS si on veut seulement quelques octets, on est obligé d'allouer une page entière, et si on veut une grande zone mémoire, on est obligé d'allouer seulement une page à la fois.


## 3.2. Buddy allocator
Cet allocateur permet d'obtenir des zones mémoire physiquement contiguë de taille `2^k * GS` octets, avec :
- GS la **granularité**, i.e. la taille minimale allouable par cet allocateur
- k entre 0 et MAX
L'adresse physique pa de la zone retournée est un multiple de `2^k GS`.

L'allocateur maintient un tableau de MAX éléments où le k-ième élément est une liste de zones libres de longueur 2^k GS octets.

### 3.2.1. Initialisation
A l'initialisation, l'ensemble de la mémoire disponible est découpée en zones de taille `2^MAX GS`, comme sur le schéma suivant :
![[buddy allocator 1.png]]

### 3.2.2. Allocation
Lors de la demande d'allocation de n octets, la demande est arrondie à la taille supérieure la plus proche de la forme 2^k0 GS. On positionne k = k0.
Ensuite, on considère la liste à l'index k du tableau : 
- Si la liste est vide, on incrémente k et on recommence
- Si la liste n'est pas vide, la tête de liste permet de servir la demande, mais on a potentiellement k>k0 et donc on alloue trop de mémoire
  -> pour éviter ça, on morcelle la zone 2^k GS en morceaux de taille 2^i GS avec i entre k0 et k-1 que le replace dans le tableau(on obtient 2 morceaux de taille 2^k0 GS et on alloue l'un des 2).
![[buddy allocator 2.png]]

### 3.2.3. Métadonnées supplémentaires
Pour pouvoir libérer un bloc, on a besoin de retrouver, à partir de son adresse pa, la taille de la zone allouée.
-> il faut stocker des **métadonnées supplémentaires** 

Dans xv6, l'implémentation de ces métadonnées utilise une vecteur bitmap split pour chaque taille de bloc. `buddy[k].split[i]` vaut 1 ssi le i-ième bloc de taille 2^k GS est découpé. La taille de la zone allouée peut être déduite de ces métadonnées : c’est le plus petit k tel que le bloc de taille 2 ^(k+1) GS qui contient l’adresse concernée est découpé.

### 3.2.4. Libération
On souhaite libérer le bloc de mémoire à l'adresse pa :
1. On détermine sa taille 2^k GS à l'aide des métadonnées
2. On ajoute pa à la liste d'index k dans le tableau
3. On **défragmente** la liste concernée : on regarde si la zone libérée est à coté d'une autre zone libre de même taille (**buddy zone**) et si c'est le cas on les fusionne et on recommence la défragmentation dans la liste de taille supérieure
![[buddy allocator 3.png]]

### 3.2.5. Avantages et inconvénients
Cet allocateur reste simple à implémenter, alloue des zones de taille variables et permet de ne pas trop fragmenter la mémoire.
MAIS il nécessite de maintenir des **métadonnées** supplémentaires qui ont une **taille inversement proportionnelle à la granularité GS**. Mais plus GS est grand, plus les métadonnées sont petites, mais aussi plus on perd de la mémoire avec les petites allocations (on alloue une grande zone mémoire car GS est grand...).


## 3.3. Slab allocator
Allocateur conçu pour palier au problème du buddy allocator sur l'allocation de petites zones mémoire et en particulier **pour les structures de données**.

<u>Principe</u> : le slab allocator prépare une ou plusieurs pages remplies de structure de données initialisées, puis lorsqu'une structure est demandée, il en renvoie une de libre et lorsqu'une structure doit être libérée, il la garde prête pour la prochaine allocation.

Il est particulièrement adapté aux allocations et désallocations fréquentes sur les structures de données les plus utilisées du noyau.



# 4. Gestion de la mémoire virtuelle des processus

## 4.1. Espace d'adressage d'un processus
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
La fonction main reçoit 2 arguments :
- argc : le nombre d'arguments
- argv : un pointeur vers l'adresse de l'argument 0


## 4.2. Chargement d'un binaire
### 4.2.1. Format d'exécutable ELF
ELF est le format d'exécutable sous Linux et xv6 (format Mach-O sou Mac et PE sous Windows). Il se présente en différentes sections comme suit :
![[format ELF.png]]

Il y a 2 vues des mêmes données :
- Vue "compilation" : découpage en sections, fusionnées par le linker à la compilation (section header table)
- Vue "chargement" : découpage en segments, chargés par l'OS (program header table)

### 4.2.2. Chargement d'un binaire
Le chargement d'un binaire s'effectue avec la fonction **execve** qui provoque une interruption.
![[execve.png]]

Plus en détails, cette interruption provoque les étapes suivantes :
1. Démarrer une transaction avec le système de fichiers
2. Ouvrir le fichier exécutable
3. Prendre le verrou sur fichier
4. Lire l'entête ELF
5. **Allouer un espace d'adressage vide**
6. Pour chaque segment décrit dans la table des segments du fichier ELF (i.e. program header table) :
	1. Lire le segment
	2. Vérifier plusieurs conditions sur le segment (taille, ...)
	3. Allouer de la mémoire physique et modifier la table des pages du processus
	4. **Charger les données du segment** dans la mémoire
7. Relâcher le verrou sur le fichier
8. Finir la transaction avec le système de fichiers
9. **Préparer la pile** :
	1. Allouer 2 pages : une pour la pile et une page de garde
	2. Rendre la page de garde inaccessible en mode utilisateur (permet de détecter les débordements de la pile)
	3. Copier les arguments dans la pile du nouvel espace d'adressage
10. Copier les adresses des arguments dans un tableau qui sera passé à la fonction main (argv)
Si une erreur se produit, on nettoie la table des pages en cours de construction et on renvoie -1.


## 4.3. Optimisation de la mémoire virtuelle des processus
### 4.3.1. Pagination à la demande
La pagination à la demande consiste à n’allouer des pages physiques que lorsque cela est
nécessaire (tentative d’accès à une zone mémoire).
Ainsi, lorsque le processus est chargé en mémoire la première fois, on ne copie pas ses données dans la RAM mais on les laisse sur le disque. Dans la table des pages du processus, on crée le mapping des pages comme s'il elles existaient vraiment mais on les marque comme invalides. Donc, lorsque le processus voudra accéder à la page pour la première fois, un interruption **page fault** sera lancé et l'OS la traitera et copiera la page demandée depuis le disque dans la RAM. 

### 4.3.2. Copy on write
Lors d’un l’appel à fork, l’espace d’adressage du père devrait être copié vers l’espace d’adressage du fils mais c'est coûteux. De plus, bien souvent, le processus fils commence par faire appel a execve, ce qui réinitialise l'espace d'adressage fraîchement copié du père et charger le programme du fils.

Le noyau va plutôt essayer de partager ces pages tant que possible. Seulement lorsqu’une écriture sur une telle page survient, le noyau duplique cette page et affecte une copie à chaque processus.
Pour détecter cette situation, le noyau aura rendu la page en lecture seule.
![[copy on write.png]]



# 5. Allocateur en espace utilisateur
En C, on utilise **malloc** pour allouer de la mémoire. La librairie maintient une réserve de mémoire dans le tas et quand celle-ci devient insuffisante, elle demande au noyau d'**augmenter la taille du tas**. 

Cela se fait avec l'appel système **sbrk** (set break). La fonction sbrk modifie la position du *program break* qui est défini par la fin du **segment data**.