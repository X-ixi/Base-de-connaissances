``` toc

```

# 1. Systèmes de fichiers

## 1.1. Fonctionnement
Vocabulaire :
- *Block* (ou cluster) : bloc de données, permet de subdiviser un fichier en plusieurs bloc
- *Inode* : noeuds d'index, table d'attributs associé à un fichier
- *Super block* : bloc décrivant la structuration du système de fichier (taille des bloc, ...)
- *Sector* : subdivision physique du disque

Méthode d'allocation :
- Contiguë
- Chaînée : un bloc contient des données et un pointeur vers le suivant
- Chaînée et indexée : bloc chaînée et table d'allocation (permet d'accéder directement à une partie éloignée du fichier sans le parcourir intégralement) 

Stocker en contiguë est trop contraignant mais trop fragmenter a aussi ses inconvénients : la tête de lecture aura plus de chemin à parcourir pour lire le fichier (ce n'est plus un problème pour les SSD). 


## 1.2. Historique
### 1.2.1. File Allocation Table (FAT)
Le plus vieux système de fichier : FAT12, FAT16 et FAT32

Au début du disque, une section présente une table d'allocation : tableau dans les entrées sont des index sur 16 bits (pour FAT 16). Chaque entrée désigne si un bloc est libre, utilisé ou défectueux, et quel est le bloc suivant (EOF si c'est le dernier bloc du fichier).
Un catalogue liste les fichiers avec le nom du fichier, la date, la taille, des attributs et l'index du premier bloc.

Ainsi un disque formaté en FAT32 contient dans l'ordre :
- Des constantes relatives au système de fichier (super block)
- La table d'allocation FAT
- Le catalogue de fichier

### 1.2.2. ext, ext2
L'organisation générale ressemble au FAT mais utilise à présent la notion d'inode.
Un inode contient des informations relatives au fichier et des pointeurs sur des blocs de données ou des pointeurs sur des blocs de pointeurs pour gérer les fichiers plus gros.
![[inode.png]]

Un répertoire a un inode particulier qui pointe vers un bloc spécial qui liste les inods des fichiers contenus. 

Les blocs sont regroupés en groupe de bloc pour minimiser le déplacement des têtes de lecture : un fichier sera réparti dans différent blocs du même groupe. Chaque groupe de blocs commence par une copie du super-block pour assurer la redondance du super-block.

Outil pour récupérer les fichiers en cas de corruption du systèmes de fichiers : *fsck*.

### 1.2.3. ext3
Ajout d'un journal pour récupérer les opérations non synchronisées sur le système de fichiers. Le journal peut contenir les métadonnées du fichier (nouvelle taille, date de modifications, ...) et/ou le contenu du fichier (créé et ajouté).

Dans le cas où une panne interrompt le système de fichier, si les modifications sont enregistrées dans le journal, rien ne sera perdu.
Plusieurs modes de journalisation sont possibles avec un compromis lenteur/sûreté selon si on écrit d'abord tout dans le journal ou si des écritures peuvent avoir lieu en parallèle sur le journal et le disque.

Pas utile pour une utilisation personnelle mais peu devenir intéressant pour des applications critiques avec beaucoup d'écritures (ex : supercalculateur).

### 1.2.4. ext4
Remplacement des pointeurs vers des pointeurs dans les inodes de ext3  par des *extends*. Cela permet notamment de stocker une grosse vidéo de manière contiguë en disant simplement sur quelle plage de blocs la vidéo est stockée, plutôt que de s'encombrer de nombreux pointeurs vers des pointeurs.

### 1.2.5. Autres
Quelques autres systèmes de fichier :
- exFAT : mémoire flash, interropérable entre Linux et Windows
- NTFS
- ZFS : nouveau système en plein essor


## 1.3. Cas particulier des mémoires flash
Les SSD, carte SD, clé USB, etc comportent un *block device* qui fait lui-même la gestion hardware des blocs défaillants. C'est transparent et le système qui montent les mémoires flash n'ont souvent pas accès à ces informations
C'est pour cela que la quantité disponible sur les clé USB diminue au fur et à mesure.



# 2. RAID
Permet la redondance de données entre plusieurs disques. Il existe plusieurs modes avec plus ou moins de redondance : RAID 1, RAID 2, ..., RAID 5, RAID 1+0, RAID 0+1, ...

Le RAID peut être effectué au niveau hardware (géré dans le materiel, configuré dans le BIOS) ou au niveau software (géré par l'OS, création de plusieurs disques viruels).

## 2.1. ZFS
Zettabytes File System combine à la fois :
- Un système de fichier
- RAID
- Du chiffrement
-> Un seul software pour gérer toutes ses fonctionnalités

Version open-source de ZFS : OpenZFS


LVM : gestion de périphériques (unifié 10 disques physique en un seul disque logique)
LUKS : chiffrement des diques

