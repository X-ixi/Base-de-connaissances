``` toc

```

# 1. Introduction

## 1.1. Rappels sur l'architecture des ordinateurs
Dans ce cours, focus sur les architectures x86-64 tel que AMD64 ou Intel64.
x86-64 expose une architecture CISC ([[(Viro) 1 - x86#2.1. Préambule sur les processeurs]]) pour la compatibilité mais il le transforme ensuite en micro-opérations de type RISC qui sont réellement exécutées.
Ce micro-code n'est pas documenté et est spécifique à un modèle/type de processeur.

Le micro-code d'usine est stocké dans une ROM (Read Only Memory) et lorsque l'ordinateur se lance, il utilise une RAM statique pour mettre à jour le micro-code. 
La mise à jour de cette RAM statique est possible uniquement en ring 0 (donc par le boot firmware, le boot loader, l'hyperviseur ou le kernel) et cette mise à jour est chiffrée et signée.

Hierarchie mémoire :
- SRAM : dans le processeur, c'est les registres et le cache (très petite quantité car très chère)
- DDR-SDRAM : c'est la RAM
- SSD

Les architectures sont de plus en plus complexe et se rapproche de plus en plus des architectures de téléphone ou tous les composants sont sur la puce (pour des raisons de performance).

![[architecture.png]]

A l'intérieur des pointillés, c'est le processeur sur une seul puce, qui vient se brancher sur la carte mère. La carte mère fait la liaison entre le processeur et les autres composants.
Plateform Controller Hub : tout ce qui est dans le processeur sans être intégré au CPU
PCI (ancien) : réseau en bus (une seule liaison centrale à tous)
PCI express : réseau commuté (à l'image d'internet avec les paquets qui passent par des switchs)
eSPI : liaison série extrêmement simple (16 bits) utilisée pour le micro-logiciel de démarrage.
-> Implication en terme de sécurité : il est facile d'injecter des données sur le eSPI

Pour éviter toute latence, la carte vidéo est branchée directement sur le CPU. Aujourd'hui, d'autres port PCI express sont présents sur le CPU et on peut imaginer qu'à terme, ils le seront tous.
Dans le CPU, il y a aussi un GPU interne qui permet de générer l'affichage à l'écran. On peut également ajouter une carte graphique (GPU) externe.

![[architecture_zoom.png]]
Zoom sur le CPU : anneau de communication bi-directionnel
Les caches L1 et L2 sont à l'intérieur des cœurs et les caches L3 sont partagés entre les cœurs (latence plus faible si le cœurs accède à sa slice de cache L3).
Les cœurs et le GPU accède au memory controller pour accéder à la RAM et à la mémoire. 

C'est dans les cœurs que la traduction d'instructions CISC en micro-instructions RISC a lieu, ainsi que la traduction d'adresse virtuelle en adresse réelle.

![[architecture_software.png]]
Il y a des morceaux de software partout ! La carte réseau Ethernet possède un processeur interne et utilise un firmware par exemple.
C'est firmware doivent être chargé de manière synchronisé au démarrage. Ils sont généralement stocké sur des ROM et ne peuvent pas forcément être mis à jour...


## 1.2. Rappels sur les OS
Cf [[(OS) 0 - Questions de cours]].
Les processeurs disposent de différents modes d'exécution avec des privilèges plus ou moins élevés. Ces modes donnent accès à des instructions et à des zones mémoires différentes (notamment grâce à la pagination et aux droits sur les pages).

Le noyau expose une API d'appel système et crée une abstraction du matériel : il est le seul à vraiment gérer la mémoire.
-> Séparation espace noyau et espace utilisateur

Création d'un nouveau processus fait avec `fork`
-> Le processus fils hérite des droits du processus père (en pratique on fait un copy on write)

La sécurité est gérée à l'aide d'identifiants :
- UID : user id
- GID : group id
- EUID, EGID : effective UID, GID
- SUID, SGID : saved UID, GID
- PID : process id (pas vraiment utilisé pour la sécurité)

Ces identifiants sont modifiés lors de l'exécution de `execve` car il charge un nouveau programme, ou sont modifiés par certains appels systèmes privilégiés.


## 1.3. Démarche de sécurisation d'un OS
Principes de base :
- **Principe de minimisation** : réduire la surface d'attaque (supprimer/désactiver les logiciels et composants inutiles, minimiser les fonctionnalités activées)
- **Principe de moindre privilège** : raisonner sur les utilisateurs, les machines et les logiciels pour leur affecter les droits minimum, et filtrer les flux réseaux entrants et sortants
- **Principe de défense en profondeur** : chiffrer les données stockées et les flux réseaux, cloisonner les services et journaliser les évènements

Recommandation de configuration matériel : systèmes 64 bits (plus d'entropie pour l'ASLR), bit NX activé ([[(AttMem) 3 - Contre mesures#1.1. Bit NX et politique W v X]]), activer VT-d pour la virtualisation que si nécessaire, utiliser un TPM, limiter les interfaces (USB, sans-fil, ...), et attention aux fonctionnalités d'administration à distance (AMT, IDRAC, ...).


# 2. Sécurité du boot

## 2.1. UEFI
*Boot firmware* : historiquement appelé *BIOS* (Basic Input Output System).
Code assembleur en charge du démarrage
Intel développe EFI (Extensible Firmware Interface) en 1999 pour les systèmes 64 bits, pour standardiser l'interface du boot firmware. L'EFI se répand et devient *UEFI* (Unified EFI).
-> UEFI regroupe plusieurs applications et notamment : le *boot loader* (chargeur de démarrage, ex : grub) et l'interface graphique de configuration du BIOS
à
UEFI adopte de très nombreuses fonctionnalités (interface graphique, pile TCP/IP pour les mises à jour, ...).
Développement du code en C, ce qui augmente la portabilité et la maintenance du code.

Le boot firmware se trouve sur une mémoire flash, contrairement au boot loader et à l'OS qui se trouve dans un stockage de masse (disque dur) avec un partitionnement adéquat.

UEFI intègre des mécanismes de sécurité par défaut :
- Mise à jour sécurisé du BIOS depuis l'OS (vérification de l'intégrité)
- Secure boot : vérification de l'intégrité des composants (windows signé par Microsoft, ...)
- Trusted boot : vérifie que le système est le même que le système précédent (repose sur l'utilisation d'un TPM)

Phases de démarrage :
1. *Power On* : Exécution d'un bout de code pour initialiser le CPU (une partie du code est en assembleur) et passage en Secure Mode
2. *Platform Initialisation* : chargement des drivers des différents composants (mémoire, carte ethernet, carte USB, ...) de la carte mère pour que chaque composants soient utilisables
3. *Boot Manager* : l'interface UEFI est disponible et les application UEFI sont utilisables
4. *Boot de l'OS*


## 2.2. System Management Mode (SMM)
Mode d’exécution privilégié de certains *runtime services*, qui sont des services qui continue de tourner après le boot de l'OS (la majorité des services utilisé lors du boot sont supprimés).
Exemple : des services qui permettent de mettre à jour le BIOS depuis l'OS, des services pour la gestion de l'énergie, des services pour modifier des variables de configuration UEFI.

Le code et les données des services SMM sont stockées en SMRAM

La pagination est géré par le noyau de l'OS (dans la MMU - Memory Management Unit, bloc de mémoire associé à chaque CPU pour faire la traduction).
-> On ne pas l'utiliser pour séparer l'UEFI et l'OS
=> Le contrôleur mémoire du CPU fait le contrôle d'accès et n'autorise l'accès aux adresses physiques de la SMRAM que lorsqu'on est en mode SMM.

Pour passer en mode SMM, l'OS fait une interruption qui se rapproche d'un appel système.


## 2.3. Implémentations
UEFI est une spécification et il existe plusieurs implémentations.
Tianocore : implémentation open-source développé majoritairement par Intel, et réutilisé dans l'immense majorité des ordinateurs.

Pour autant, on n'est pas obligé d'être conforme à UEFI pour démarrer son pc :
- coreboot : boot firmware open source minimaliste (plus rapide et moins de code). Il permet aussi de lancer Tianocore après la phase de Platform Initialisation.
  -> Utiliser sur les ChromeOS et Purism
- LinuxBoot : utilisation d'un noyau Linux minimal pour charger les drivers et faire la platform initialisation. Ce noyau Linux sert ensuite à charger le véritable OS (Linux, Windows ou autres).
  -> Utiliser en pratique uniquement sur les serveurs
- Slim Bootloader : boot firmware open source léger et dédié à l'embarqué


## 2.4. Converged Security and Management Engine (CSME)
Le CSME est présent sur tous les systèmes Intel.
C'est un sous-système indépendant et isolé du CPU principal (mais sur la même puce que le CPU).

Il participe à l'initialisation de composants matériels et à la sécurité : **vérification d'intégrité** et **chargement des drivers** des composants. Il permet aussi de simuler un TPM virtuel, de supporter des DRM et de vérifier l'intégrité du BIOS UEFI avant son exécution (Boot Guard).
Il permet aussi de faire de l'administration à distance (via Intel AMT).

C'est un **processeur 32 bits** avec toutes les caractéristiques d'un processeur (MMU, SRAM, ROM, ...). Il fait tourner un OS minix. Il a accès à la SRAM du système principal.
Il fonctionne dès que l'ordinateur est sous tension (pas besoin que l'ordinateur soit allumé).


Rappel sur les mode d'exécution d'un pc :
- Full on : totalement allumé, avec interface graphique, etc
- Suspend-to-RAM (hibernation) : contexte sauvegardé sur la RAM (qui est donc alimenté), l'OS ne tourne plus mais peut être rallumé facilement 
- Suspend-to-Disk (hibernation plus profonde - économie d'énergie) : contexte sauvegardé sur le disque, l'OS peut-être relancé rapidement
- Soft off : alimentation minimale, nécessite un reboot
- Mechanical off : plus d’alimentation

Le CSME boot avant le processeur principal et continue de s'exécuter en soft off après la fin du boot.
Il récupère le boot firmware de la flash, puis vérifie son intégrité avec le certificat Intel présent sur la ROM.
Il génère également une clé privée unique et la stocke dans un slot verrouillé qui ne pourra plus être modifié avant le prochain reboot.

Le CSME permet d'implémenter Boot Guard qui vérifie l'intégrité de l'UEFI. Ensuite l'UEFI implémente la suite du Secure Boot pour vérifier la suite des composants et l'OS.

La flash étant sur la carte mère, c'est au vendeur de la carte mère que revient une partie des valeurs que le CSME utilisera pour sa configuration. Intel ne peut donc pas tout verrouiller.
1. Le firmware du CSME est fourni par Intel
2. Le firmware de l'UEFI est fourni par le vendeur de la carte mère
3. Le vendeur de la carte mère met le hash de son UEFI dans une valeur de configuration
4. Il active un fusible pour que la valeur du hash ne puisse plus être modifié (en théorie, certains vendeurs ne le font pas pour donner la possibilité à l'utilisateur de désactiver le CSME - car complètement sous contrôle de Intel, société américaine)


## 2.5. Boot du processeur principal
Election d'un cœur, qui sera nommé BSP (BootStrap Processor) qui sera le premier à booter.
Le BSP démarre dans un mode spécial, similaire au Real Mode (16 bits), ce qui permet la rétro-compatibilité avec des BIOS des années 1980.
Le BSP accède aux données contenu dans la flash de la carte mère contenant le BIOS/UEFI. Cette communication passe par le bus SPI.

La première chose que réalise le BSP est de regarder si des mises à jour du micro-code sont présentes dans la flash, et de les appliquer.
A ce moment-là, la RAM n'est pas initialisé donc le BSP se sert de son cache comme RAM.

Le BSP commence par exécuter un code d'Intel qui se charge de vérifier l'intégrité du début de l'UEFI, grâce au hash disponible via le CSME.

Vérifier l'intégrité = vérifier que le code a été signée par une autorité de confiance (on vérifie uniquement la provenance en réalité)

Le firmware UEFI initialise le CPU et le passe en long mode, ou flat protected mode. Il initialise aussi des registres (indique la zone de SMRAM, ...). Puis il initialise la RAM, copie son code de la flash dans la RAM, et continue son exécution depuis la RAM. Il configure alors le cache pour faire office de cache.

Jusqu'à présent, 1 seul cœur s'exécutait. A présent, on initialise les autres cœurs.


Ensuite viens la phase de découverte des périphériques (carte ethernet, USB, clavier, ...). Historiquement, chaque fabricant de composants met une ROM dans le composant qui permettra de charger les drivers du composant pour l'initialiser. 
Cela ne permet pas d'appliquer des mises à jour (car ROM non modifiable). Aujourd'hui, le firmware pour initialiser les composants peut aussi être présent dans une partition de la flash de l'UEFI.


## 2.6. Runtime services
Certains services lancé pendant le boot UEFI continue de tourner une fois le boot terminé et l'OS lancé.
Parmi ces services, il y a 2 cas :
- Ceux qui peuvent exécuté directement dans le mode de l'OS
- Ceux qui peuvent être exécuté uniquement en mode SMM


## 2.7. Secure boot
![[secure boot.png]]
Chaque morceaux vérifie le suivant.

UEFI vérifie que l'OS a charger est signée par une autorité connue de l'UEFI.
Le fournisseur de la carte mère est le seul à pouvoir mettre à jour l'UEFI, et par conséquent ajouter des clés publiques d'autorité de confiance.
-> Généralement, les seules clés publiques présentes sont celle du fournisseur et celle de Microsoft

Microsoft offre un service de vérification de boot loader (payant 100$) : pour ne pas avoir à vérifier chaque boot loader, Microsoft signe un boot loader intermédiare (shim) qui contient la clé publique de l'éditeur de l'OS à signer.

Shim permet aussi à l'utilisateur de modifier le noyau et d'ajouter des clés personnalisées (application MokManager). Cela permet d'activer le SecureBoot même avec un Linux.


## 2.7. BIOS Guard
Contrairement à Boot Guard qui permet de vérifier l'intégrité du BIOS, BIOS Guard permet de mettre à jour le BIOS.
Il faut être en mode SSM pour modifier la flash et appliquer les mises à jour.
-> Sauf si le fabricant de la carte mère n'a pas activé cette option...



