```toc

```


# 1. Introduction
Lorsqu'on a affaire à un virus, on a seulement le fichier binaire ! On peut alors analyser le virus de deux façons complémentaires :
1. Analyser le comportement du virus : actions réalisés, persistance, E/S, ...
2. Analyser le code du virus : rétro-conception à partir des instructions assembleur.

Pour être capable de faire de la rétro-conception, il faut maîtriser l'assembleur et l'architecture matérielle et logicielle. 


# 2. Architecture x86

## 2.1. Préambule sur les processeurs
Un processeur, ou CPU, comporte les éléments suivant :
- Une unité arithmétique et logique ALU
- Une unité de contrôle : séquenceur qui gère les interruptions (cf cours d'OS [[(OS) 3 - Ordonnancement]])
- Une horloge
- Des **registres** : l'ALU manipulent leur contenu à chaque cycle d'horloge
- Une unité Entrées/Sorties

Il y a 2 catégories de processeur selon le type d'encodage de leurs instructions :
- Les **RISC** (Reduced Intruction Set Computing) pour les encodages à taille fixe (4 octets pour le RISC-V, cf [[(OS) 2 - xv6 et interruptions]])
- Les **CISC** (Complex Instruction Set Computing) pour les encodages à taille variable (de 1 à 15 octets pour le x86)

Le CISC permet d'optimiser la place des instructions en mémoire et accélère l'exécution d'instructions complexes. Cependant, CISC est plus gourmand en énergie et il est plus difficile d'optimiser les instructions générées par les compilateurs.


## 2.2. Les registres
Un registre est un emplacement mémoire interne au processeur : c'est la mémoire la plus rapide mais aussi la plus petite.

L'architecture x86 est sur **32 bits** (tandis que x64 est sur 64 bits), par conséquent la plupart des registres font 32 bits car c'est ce que le processeur est capable d'ingérer.

### 2.3.1. Les registres généraux
Dans x86, les registres généraux font 32 bits et peuvent se décomposer en sous-registres de 16 ou 8 bits, comme le montre l'image suivante.
![[registres_generaux.png]]
Ainsi, l'instruction assembleur `mov al, var_1` met les 8 bits de poids faible de `var_1` dans le registre `al`. De même, si `var_1` est sur 16 bits, `ah`récupère les 8 bits de poids fort.

Il y a 8 registres généraux, qui sont chacun associé à une utilité par convention (rien n'empêche de ne pas la respecter) :
- EAX (AX, AH, AL) : accumulateur
- EBX (BX, BH, BL) : base (adresse de début d'un array par exemple)
- ECX (CX, CH, CL) : compteur
- EDX (DX, DH, DL) : données
- ESI (SI) : source
- EDI (DI) : destination
- ESP (SP) : stack pointer
- EBP (BP) : base pointer

Dans la version 64 bits de x86, nommée x86-64, les registres généraux sont étendus sur 64 bits et sont préfixés par un R à la place du E : RAX, RBX, ... Et 8 nouveaux registres de R8 à R15 sont ajoutés. 

### 2.2.2. Le registre EFLAGS
Ce registre de 32 bits regroupe un ensemble de flag (bit à 1 si le flag est positionné ou à 0 sinon). Les flags les plus importants sont :
- Le **Direction Flag** : oriente les opérations sur une chaîne de caractères (de gauche à droite ou inversement)
- Le **Sign Flag** : correspond au bit de poids fort du résultat
- Le **Zero Flag** : positionné si le résultat est nul
- Le **Carry Flag** : positionné si le résultat génère une retenue (i.e. le résultat est trop grand pour être contenu dans l'espace mémoire qu'il lui est donné)
- L'**Overflow Flag** : positionné si le résultat est de signe différent des deux autres valeurs

### 2.2.3. Types de base
Les principaux types en x86 sont :
- Le **Byte (DB)** : 8 bits, soit 1 octet - c'est le plus petit type utilisé
- Le **Word (DW)** : 16 bits
- Le **Double Word (DD)** : 32 bits
- Le **Quad Word (DQ)** : 64 bits

Le D des acronymes signifie Data : DB = Data Byte.


## 2.3. La mémoire en x86
La mémoire peut être représenté de deux façons :
- **Big Endian** : l'écriture commence par l'octet de poids fort, c'est l'écriture la plus naturelle pour un humain.
- **Little Endian** : l'écriture commence par l'octet de poids faible, c'est l'écriture la plus utilisé par les machines car elle permet certaines optimisations de calculs.
   -> c'est un découpage par octet (ce qui n'est pas vraiment intuitif).

Exemple :  `08 2F 3A C8` en big endian devient `C8 3A 2F 08` en little endian.

Chaque processus manipule des adresses virtuelles qui sont ensuite traduite en adresse physique par le processeur. Cela permet d'isoler chaque processus en lui faisant croire qu'il possède sa propre RAM qu'il gère comme il l'entend. Cela permet aussi de protéger certaines plages d'adresses mémoire. Plus de détails dans le cours d'OS : [[(OS) 5 - Mémoire virtuelle]].

La **pile**, ou **stack**, est une structure de données de typer LIFO permettant de stocker temporairement les valeurs des registres, des arguments ou autres. Le haut de la pile est toujours pointé par **ESP** (RSP en 64 bits). La pile est souvent représenté avec les adresses les plus élevées en haut (à l'inverse de la mémoire donc), cependant, elle se remplit par le bas...
![[stack.png]]
![[memory_in_c_program.png]]

# 3. Assembleur x86
L'assembleur est la représentation humaine du language machine. Voici 3 représentation des mêmes données :
```
1000 1011 0100 0100 0010 0100 0011 0000  # binaire
8B 44 24 30                              # hexadécimal
mov eax, [esp+30h]                       # assembleur
```

<u>Note</u> : En assembleur, 30h désigne la valeur hexadécimal 30, noté dans d'autres domaines 0x30.

## 3.1. Format d'une instruction
Une instruction assembleur à le format suivant :
```
(prefixes) MNEMONIC (Operand 1, Operand 2, Operand 3)
```
Le mnémonique désigne le nom de l'instruction. Le nombre de préfixes n'est pas limité tandis que le nombre d'opérandes est limité à 3.

<u>Remarque</u> : une instruction est de taille variable et est encodée sur au maximum 15 octets, au delà une exception est levée.

Parmi les prefixes, les plus courants sont :
- **lock** : force l'accès exclusif à une mémoire partagée en environnement multiprocesseur
- **repne/repe** : répète l'instruction sur chaque élément d'une chaîne de caractères tant qu'une condition d'égalité n'est pas atteinte (repe) ou de non égalité (repne). Si la condition porte implicitement sur un compteur, ce sera ECX.

Les opérandes peuvent être de 3 types :
- registre
- valeur immédiate
- référence mémoire
```bash
mov eax, ebx            # copie la valeur du registre ebx dans eax
mov eax, 388h           # copie la valeur 0x388 dans le registre eax
mov eax, [ebx]          # copie la valeur à l'adresse contenu dans le registre ebx, dans eax
mov eax, [ebx*2+654h]   # copie la valeur à l'adresse <valeur de ebx>*2+654h dans eax
```


## 3.2. Instructions de base
Voici les instructions les plus utilisées, groupées par catégorie :
- affectation :
	- **MOV** : move - copie l'opérande source vers l'opérande destination
	  `mov eax, ebx` : copie ebx dans eax
	- **LEA** : load effective address - calcule l'adresse effective du second opérande et le stocke dans le premier opérande
	  `lea eax, [ebx+4*esi]` : calcule `valeur_ebx + 4*valeur_esi` et le met dans eax
	  Si on a un vecteur dont l'adresse de début est ebx et esi est la taille de chaque élément, alors eax est l'adresse du 4ème élément du vecteur.

- arithmétique :
	- **INC** : incrémente l'opérande de 1, sans modifier le Carry Flag.
	  `inc eax` : incrémente eax de 1.
	  `inc byte ptr [edx]` : effectue l'opération sur 1 octet. Si edx vaut `a3 ff` alors après l'opération il vaudra `a3 00`.
	- **DEC** : décrémente de 1 sans modifier le Carry Flag.
	- **ADD** : ajoute les 2 opérandes et stocke le résultat dans la première opérande. *ADC* est une variante dans laquelle on ajoute le Carry Flag en plus.
	- **SUB** : soustrait la 2ème opérande à la 1ère et stocke le résultat dans la première. *SBB* est une variante dans laquelle on soustraie le Carry Flag en plus.
	- **MUL** : effectue une multiplication non signée de EAX avec l'opérande spécifiée et stocke le résultat dans EDX:EAX (i.e. dans les 2 registres collés l'un à l'autre car le résultat peut être très grand). *IMUL* effectue une multiplication signée.
	  `mul ebx` <=> `EDX:EAX <- EAX*EBX`
	- **DIV** : effectue une division non signée entre EDX:EAX et l'opérande spécifiée et stocke le quotient dans EAX et le reste dans EDX. *IDIV* effectue la division signée.
	  `div ebx` <=> `EAX <- EDX:EAX // EBX` et `EDX <- EDX:EAX % EBX`
	- **SHR** : Shift Right - décalage à droite de l'opérande destination du nombre de bits spécifié. Le décalage à droite de 1 bit correspond à une division par 2.
	  `shr eax 2` : si `eax = 1011`, alors après l'opération `eax = 0010`
	- **SHL** : Shift Left - décalage à gauche de l'opérande du nombre de bits spécifié. Le décalage à gauche de 1 bit correspond à une multiplication par 2.
	  `shr eax 2` : si `eax = 1011`, alors après l'opération `eax = 101100` (ou `eax = 1100` si eax est stocké sur 4 bits).

- logique :
	- **CMP** : Compare - soustrait la seconde opérande à la première et positionne les flags en fonction du résultat (Carry Flag, Zero Flag, ...). Les opérandes ne sont pas modifiées.
	- **TEST** : calcule le ET logique entre les 2 opérandes et positionne les flags en fonction du résultat. Les opérandes ne sont pas modifiées.
	  `test eax, eax` : le Zero Flag est positionné ssi eax est nul.
	- **XOR** : calcule le OU exclusif entre les 2 opérandes et met le résultat dans la première.
	- **OR** : calcule le OU logique entre les 2 opérandes et met le résultat dans la première.
	- **AND** : calcule le ET logique entre les 2 opérandes et met le résultat dans la première.

- transfert :
	- **JMP** : Jump - transfère le contrôle du programme à l'adresse désignée.
	  `jmp eax` <=> `mov eip, eax`
	- **CALL** : transfère le contrôle du programme à la procédure désignée. C'est équivalent à mettre l'adresse de retour (i.e. où revenir à la fin de la procédure) sur le dessus de la pile et sauter à l'adresse de l'opérande. Si la procédure nécessite des arguments, il doivent être stockés dans des registres ou dans la pile avant l'instruction call.
	  `call eax` <=> `push @Return` puis `jmp eax`
	- **RET** : transfert le contrôle du programme à une adresse de retour stockée sur la pile. Si une opérande est passée à l'instruction ret alors c'est une valeur immediate qui sera ajoutée à ESP (cela permet de corriger la pile, voir plus loin).
	  `ret 08h` <=> `pop <tmp>` puis `add esp, 08h` puis `jmp <tmp>`
	- **J__** : saut conditionnel - vérifie l'état des flags et, selon leurs valeurs, saute à la destination spécifiée ou non.
	  ![[jump_conditions.png]]

- modification de la pile :
	- **PUSH** : décrémente ESP de 4 (4 octets, soit 32 bits) et stocke l'opérande sur le haut de la pile. Ex : `push alpha`
	 ![[push_in_stack.png]]
	- **POP** : charge dans l'opérande la valeur contenue sur le haut de la pile et incrémente ESP de 4. Ex : `pop eax`
![[pop_from_stack.png]]

<u>Note</u> : `add esp, 4` est équivalent à retirer un élément de la pile sans en récupérer la valeur.


## 3.3. Les procédures
Une procédure, ou routine, est un sous programme réalisant une action. Une fonction est un cas particulier de procédure (elle a une valeur de retour).

La signature d'une procédure, aussi appelé son prototype est composé de :
- Un type de retour (void pour les procédures et tout autre type pour les fonctions)
- Une convention d'appel
- Un nom
- D'éventuels arguments

Exemple : `int __fastcall function1(long value1, int value2)`

### 3.3.1. Conventions d'appel
Une convention d'appel définit :
- Comment sont passés les paramètres : registres / pile
- L'ordre de passage des arguments de la pile
- Comment sont récupérées les valeurs de retour
- Les registres sauvegardés
- Comment est effectué la correction de la pile : une fois la procédure terminée, qui s'occupe de retirer les arguments de la pile (i.e. l'appelant ou l'appelé ?)
![[conventions_appel.png]]

En x86, la valeur de retour est toujours mise dans EAX, étendu de EDX si nécessaire. En convention d'appel C, l'appelant corrige la pile (`add esp, 10h`), tandis que dans les autres convention c'est l'appelé (`ret 10h`).

### 3.3.2. Cadre de pile
Le cadre de pile d'une procédure (**stack frame**) correspond à sa vue locale, composée des arguments passés par la pile, de l'adresse de retour et des variables locales. 
![[stack_frame.png]]
Le registre EBP est souvent utilisé comme base de pile, c'est le **frame pointer**. Il facilite notamment l'accès aux variables locales. Pour l'utiliser comme sur l'image précédente, il faut ajouter un prologue au début de la procédure et un épilogue à la fin :
- prologue : 
  1. `push ebp` (sauvegarde l'ancienne valeur)
  2. `mov ebp, esp` (il n'y encore aucune variable locale)
- épilogue : 
  1. `mov esp, ebp` (permet de ramener ESP sur la valeur de retour, qui sera ensuite utilisé avec l'instruction `ret`)
  2. `pop ebp` (rétablit l'ancienne valeur)