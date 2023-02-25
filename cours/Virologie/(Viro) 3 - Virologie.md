
```toc

```


# 1. Spécificités techniques des malwares

## 1.1. Primo-infection
Plusieurs points d'entrée très fréquents :
- Social engineering
- Attaque du point d'eau (water holing) : compromission d'un site utilisé par la victime
- Vulnérabilités


## 1.2. Persistance
Assure la survie du malware en cas de redémarrage.

Plusieurs techniques courantes :
- Hijacking : remplacer une librairie légitime par une autre fournissant les mêmes fonctionnalités et assurant en plus la persistance (une DDL par exemple), ou alors modifier un raccourci (word par exemple) pour que l'utilisation de celui ci lance le logiciel légitime ET le malware.
- DLL search order : si Windows appelle une DLL légitime depuis la variable d'environnement PATH, on peut ajouter une DLL malveillante ayant le même nom plus tôt dans le DLL search order.


## 1.3. Furtivité
Passer inaperçu pour l'utilisateur et les outils de détection.

Plusieurs techniques :
- **Mimétisme** : le malware se fait passer pour un logiciel classique (processus à part entière, nom pas soupçonneux, ...)
- **Infection** : le malware s'incruste dans un processus existant qu'il modifie pour exécuter également le code du malware.
- Minimisation des interactions avec l'OS : ne pas faire des appels directs mais plutôt coder une résolution dynamique personnalisée pour être dur à analyser
- **Modification de l'OS** : rootkit
	- Type 1 : modification des constantes du système (modifier des tableaux de pointeurs choisissant le code suivant à exécuter)
	- Type 2 : modification des données (i.e. pas seulement des tables statiques comme le type 1), ex : modification de la liste chaînée des processus pour faire disparaître le malware (tout en gardant le thread du malware dans la liste des processus de l'ordonnanceur sinon il ne sera jamais exécuté)
	- Type 3 : virtualisation de l'OS (le rootkit ne peut plus interagir avec l'OS et doit donc interagir directement avec le materiel)


## 1.4. Protection contre l'analyse
### 1.4.1. Protection du code
Plusieurs techniques statique et dynamique :
- **Chiffrement** : l'idée est de déchiffrer le corps du virus, l'exécuter et le chiffrer avec une nouvelle clé pour contourner la détection par signature. La partie du programme en charge du déchiffrement n'est pas chiffré et peut servir de signature.
- **Polymorphisme** : le binaire généré sera différent pour chaque instance du malware (le but étant que lorsque le malware se réplique sur une autre machine, ce soit un binaire différent)
	- Insertion de code mort (add eax, 0)
	- Substitution de registres (utiliser un registre plutôt qu'un autre)
	- Permutation d'instructions (i.e. modifier l'ordre d'instructions permutables)
	- Substitution d'instructions (xor eax, eax <=> mov eax, 0)
- **Métamorphisme** : création d'un binaire totalement différent pour chaque instance. Le binaire est désassemblé et simplifié pour le représenter en "pseudo-code" (code assembleur d'instructions minimales), puis il est re-étendue avec des règles de réécriture (une instruction devient plusieurs), mélangé avec l'usage de saut inconditionnel et un nouveau binaire est créé.
- **Packers** : à l'origine utilisé pour diminuer la taille d'un programme, en faisant une sorte de compression sur le code et en ajoutant le code d'un "dépacker" capable de décompresser le vrai code au moment de l'exécution. C'est utilisé aujourd'hui de manière générale pour se protéger de la rétro-conception (protéger la propriété intellectuelle, un malware, ...). Des packer personnalisés sont difficiles à détecter par les anti-virus et demande d'analyser dynamiquement l'exécution du malware.


### 1.4.2. Détection des débogueurs
**Breakpoint logiciel** : remplace l'adresse du breakpoint par l'instruction 0xCC qui interrompt le programme et laisse la main au débuggueur.
-> Le malware peut vérifier la présence de 0xCC à la place du code d'origine.

**Breakpoint matériel** : 8 registres de debug indiquant la position de points d'arrêt
-> Le malware peut récupérer la valeur des registres de debug via GetThreadContext (et les modifier avec SetThreadContext).

Autres techniques de détection des débogueurs :
- Des API de détection et des flags existent (IsDebuggerPresent, ...)
- Le programme peut aussi regarder le temps écoulé entre 2 endroits du code (si ce temps est trop long, le debug est surement activé).
- Il ne peut y avoir qu'un seul débogueur sur un processus, le malware peut créer un processus fils et le déboguer : s'il échoue il y a déjà un débogueur.

Une analyse automatique du malware se fera surement avoir par des techniques de détection de debug qui choisira d'exécuter un programme légitime lorsqu'il est débogué.


### 1.4.3. Détection de VM
Par défaut les VM ne cherchent pas à être furtives et peuvent être détectées à travers :
- Les instructions émulées (ex : l'instruction CPUID indique la propriété du processeur - 0 pour une machine physique et 1 pour la VM ; instruction IN spécifique à VMware, ...)
- Les processus et fichiers dédiés (ex: processus Vmwareuser.exe, Vboxservice.exe, ... et idem pour les drivers)
- Leurs configurations (clés de registre spécifiques)
- Leur matériel (préfixe d'adresse MAC spécifique à Vmware ou VirtualBox)

Il y a des logiciels de virtualisation conçue pour la retro qui limite ces indices.



# 2. Détection des malwares

Fiabilité = vrai positif / positifs (tout ce qui doit être détecté l'est)
Pertinence = vrai négatif / négatifs (seul ce qui doit être détecté l'est)

![[detecttion_malware.png]]

## 2.1. Détection statique
Signature : motif permettant d'identifier un binaire
-> il doit être spécifique mais pas trop <=> équilibre faux positif / faux négatif
Ce n'est pas efficace contre les codes mutants (polymorphes ou auto-modifiants).

Quelques indicateurs statiques simples :
- Attributs de section suspects (writable + executable)
- Taille virtuelle incorrecte
- Inconsistance dans les headers (taille du code / données)
- ...

Identification de graphe : 
1. Génération du graphe de flot de contrôle
2. Normalisation des blocs (inverse du polymorphisme)
3. Optimisation inter-bloc
4. Matching avec une base de données de graphes malveillants


## 2.2. Détection dynamique
Nécessite un environnement d'**exécution sûr et transparent** (sûr pour empêcher l'action malveillante et transparent du point de vue du malware).

On peut utiliser une VM mais elle risque d'être détectée malgré tous nos efforts. On peut également émuler le CPU d'une autre architecture, ou même faire de la virtualisation matérielle qui est plus efficace.

L'exécution du malware permet de créer sa **signature comportementale** : appels système utilisés et dans quel ordre.

Il reste cependant des difficultés liées à l'environnement d'exécution : si le malware aux attaquants, ils verront d'ou vient la requête, il est difficile de s'en rendre compte si le malware veut exploiter une vulnérabilité d'un services spécifiques pas forcément présent dans l'environnement, ...



# 3. Analyse des malwares
Threat intelligence : connaissance de la menace
Indicateurs de compromission (IOCs) : hash des binaires, adresse IP, nom de domaine, ...
-> L'objectif est de faire des liens entre les binaires et les groupes d'attaquants
cf [[(Detec) 2 - CTI pour la détection]]

Objectif final : protéger son système

Plusieurs outils possibles :
- Sandbox : exécution du malware dans un environnement maîtrisé (analyse dynamique). Upload du binaire dans une interface, qui crée une VM cloud, exécute le binaire et enregistre son comportement. Ces services vérifient souvent aussi les signatures (analyse statique).
   Ex : Cuckoo sandbox (opensource), JoeSecurity, **Virus Total**
- Corrélation de binaires : basé sur de l'IA ou sur de l'abstraction des blocs de code. Ces services renvoie un pourcentage de correspondance avec des malwares connues.
  Ex : Glimps, Intezer



# 4. Panorama des malwares

## 4.1. Chevaux de Troie
Dissimulation du malware en donnant l’apparence d'un logiciel légitime.

## 4.2. Vers et virus
### 4.2.1. Vers
Ils s'auto-reproduit et essaie d'infecter le maximum de machines.

Prévention : firewall, limiter l'utilisation des services ciblés (SMB, RPC, ...), détecter le virus dans le cas d'un code non-mutant

Le virus est souvent qu'une partie de l'attaque : le virus sert à se propager ET à se connecter à un serveur de commande et contrôle (C&C) pour télécharger un cheval de Troie par exemple.

Exemple du virus Stuxnet crée par la NSA et Israël pour contrer le programme nucléaire de l'Iran. Il ciblait spécifiquement les systèmes industriels SCADA avec pour but d'endommager et de détruire les centrifugeuses d'enrichissement de l'uranium.

### 4.2.2. Virus
Ils s'auto-reproduit et essaie d'infecter le maximum de machine en infectant le programme hôte. Cela lui permet de modifier le flot de contrôle de l'hôte pour s'exécuter.
-> Ne pas éteindre les ordinateurs infectés car le virus peut programmer des actions lorsque l'hôte se termine. 


## 4.3. Rootkit et bootkit
Rootkit : Modification du comportement de l'OS, cf [[#1.3. Furtivité]].
Prévention : patch guard = exécution de test de contrôle d'intégrité sur une partie des images systèmes, registres importants, etc (tests fait par partie et régulièrement)

Bootkit : malware affectant le processus de boot afin de rester actif au sein du système d'exploitation.
Prévention : chaîne de confiance du matériel au système d'exploitation (composant TPM qui vérifie l'intégrité)


## 4.4. Botnets
Réseau de machines compromises permettant de mener des actions distribuées (spam, DDoS, brute force, ...).

Tous les bots du réseau se connectent à un serveur de commande et controle (C&C) pour recevoir leurs instructions.


## 4.5. Ransomware
Blocage de l'ordinateur tant qu'une rançon n'est pas payée.
-> chiffrement de l'ordinateur (sauf les fichiers système pour que l'ordinateur continue de fonctionner)
Ex : Wannacry (2017)

Explosion du nombre de ransomware : plus facile de se faire payer avec les bitcoins, algorithmes de chiffrement robustes, professionnalisation (des spécialistes de chaque morceaux du ransomware - ransomware as a service).

Double-extorsion : les ransomware récents extraient les données et menacent de les publier sur internet si la rançon n'est pas payée.

Conseils : ne pas payer la rançon, ne pas cliquer sur les pièces jointes des mails, maintenir les logiciels à jour et utiliser un antivirus.


