```toc

```

# 1. Qu'est ce qu'un système d'exploitation ?
Le système d'exploitation est l'**interface entre le logiciel et le matériel**. Il expose une API d'appels systèmes permettant aux logiciels d’interagir avec le matériel : gestion des applications, de la mémoire, des E/S, de la communication réseau, ...

Un OS peut être défini par le matériel qu’il gère et sur lequel il peut s’exécuter, et les appels systèmes qu’il offre et comment il les implémente.

Un OS se compose d'au moins 2 espaces :
1. **Espace noyau** : le code du noyau s'exécute dans un espace privilégie avec quasiment tous les droits et accès à toutes les ressources
2. **Espace utilisateur** : les applications et processus utilisateur s'exécute dans un espace de privilège limité et utilisent des appels systèmes qui seront exécutés dans le noyau quand nécessaire.

Le noyau contrôle les périphériques avec des drivers.


# 2. Quelques éléments historiques
1950 : IBM
1957 : Fortran
1959 : Cobol
1964 : IBM annonce que les applications compilées sur une machine seront portables sur toutes autres machines compatibles
1966 : IBM OS
1966 : **Multics**
1969 : **Unix**
1970s : début de la miniaturisation, Intel sort le premier micro-processeur produit industriellement
1971 : création d’ARPANET reliant 23 machines au travers des USA
1972 : nouvelle version d’Unix en C
1972 : Bill Gates fonde Microsoft
1976 : Steve Jobs fonde Apple
1981 : IBM commercialise l’IBM-PC, avec l’OS de Microsoft MS-DOS
1985 : sortie de MS Windows 1.0
1999 : VMWare sort son premier produit de virtualisation pour PC


# 3. Multics
Multiplexed Information and Computing Service : projet commun entre le MIT, les Bell Labs et General Electric. Ils créent un OS extrêmement novateur mais qui ne connaît pas un grand succès.
Parmi les innovations, ils créent les fichiers hiérarchiques, les liens symboliques, les librairies partagées, ...
Les fichiers sont appelés « segments » → ls = list segments


# 4. Unix
Uniplexed Information and Computing Science (jeu de mot avec Multics)
Des chercheurs travaillant sur Multics trouvent l’OS trop complexe et décident d’aller en créer un autre plus simple : Unix.

Dans les années 1950s, un procès anti trust contre l'entreprise américaine AT&T a lieu et celle-ci perd le droit de commercialiser des OS. L'entreprise accepte que les chercheurs travaillent sur Unix à condition qu'ils créent un éditeur de texte graphique.
Par conséquent, Unix est distribué gratuitement (open source).

1972 : réécriture de Unix en C pour être porté sur d’autres processeurs.

1980s : nouveau procès anti-trust, AT&T est divisée en plusieurs compagnies et elles ont le droit de commercialiser Unix.
→ Unix n'est plus open source et est vendu sous licence
=> guerre des Unix : de nombreux OS sont créés à partir de la version open source d’Unix

**Projet GNU** : créer un OS open source et tous les outils de compilation nécessaires
→ de très bon outils de compilation sont créés mais l'OS est peu performant...

**Linus Torvald** : il crée un OS pour bien comprendre le cours d’OS à l’université (en s’inspirant de l’OS pédagogique Minix)
→ il permet ensuite à la communauté d’utiliser son OS gratuitement pour avoir des retours et implémente des améliorations en fonction des retours.

## 4.1. Noyaux monolithique vs micro-noyau
**Noyaux monolithique** : beaucoup de fonctionnalité dans le noyau, cela permet d’aller plus vite mais la moindre erreur de programmation fait planter l’OS.
**Micro-noyau** : mettre le minimum dans le noyau et tout le reste dans l’espace utilisateur, cela nécessite beaucoup de communication ce qui aboutit à de mauvaises performances

Aujourd’hui, tous les OS sont monolithiques, même si la perte de performance des micro noyau serait sûrement négligeable. 

Le code de Linux faisait moins de 200 milles lignes dans sa première version et fait aujourd'hui plus de 30 millions de lignes (les drivers représentent la majorité de ce code), ce qui explique que personne n'a essayé de le migrer vers un micro-noyau.


# 5. Windows
Ordre des versions : 
- MS-DOS, 
- MS-Windows 1 et 2 
- MS-Windows 3 : sur-couche graphique à MS-DOS
- Windows NT : système 32 bits dédié aux serveurs
- Windows XP : basé sur l'API Win32 (comme Windows NT)
- Windows Vista
- Windows 7
Depuis Windows Vista, il n'y a qu'un seul noyau pour toutes les versions, seuls les services, les applications et les paramètres de configuration diffèrent. Les systèmes en version serveur sont optimisés pour la performance des serveurs d'application tandis que les version Home sont optimisés pour avoir une interface graphique réactive.
![[windows_architecture.png]]

## 5.1. Hardware Abstraction Layer (HAL)
La HAL abstrait les variations entre les architectures.
- Gestion des architectures multiprocesseurs
- Types et versions du ou des processeur(s)
- Type de BIOS
- etc

La HAL offre aussi une abstraction des différents bus (PCI, PCI-Express, USB, etc.). Les applications, et les drivers n'ont pas le droit de toucher au hardware directement et doivent passer par la HAL.
Ainsi, les couches supérieures et notamment les drivers n’ont à être développé qu’en une seule version.

## 5.2. DLL de sous-système
Historiquement, Windows a supporté différentes personnalités : Win32, OS/2, POSIX. Ces personnalités constitue une API à laquelle les applications utilisateur font appel.
Les applications ne font pas directement des appels systèmes mais font appel à une librairie d'abstraction : les DLL. Cela permet à Microsoft de modifier les services du noyau sans impacter les appels que les applications utilisateur utilisent.
