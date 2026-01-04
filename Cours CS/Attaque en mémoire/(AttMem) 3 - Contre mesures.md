``` toc

```

# 1. Contre-mesures 

## 1.1. Bit NX et politique W v X
L'OS enregistre les permissions associées à chaque zone mémoire d'un programme et les applique à l'exécution.

Politique de sécurité W v X (W xor X) : une zone ne doit pas être à la fois inscriptible et exécutable.
Ex : la pile est inscriptible et ne doit donc pas être exécutable

Une exception courante est faite pour les compilateurs just in time qui utilisent des zones RWX (ex de javascript dans les navigateurs).

Le bit NX (No eXecutable) marque une page mémoire comme non exécutable. Il n'est présent que sur les processeurs 64 bits et sur les 32 bits ayant l'extension PAE active (adressage sur 36 bits au lieu de 32). Pour les processeurs antérieur, une zone accessible en lecture est implicitement exécutable.
**-> on ne mélange pas données et code**


## 1.2. ASLR
*Address Space Layout Randomisation* : chargement des segments du programme à des adresses aléatoires.
-> Cela rend très difficile la prédiction de l'adresse de retour par l'attaquant.

Le décalage entre le segment de code et le segment de données est préservé, sinon le programme pourrait ne plus fonctionner.

Le mode de randomisation est défini dans une variable globale du noyau qui peut prendre 3 valeurs :
- 0 : aucune randomisation
- 1 : randomisation des librairies partagées, de la pile, des segments de mémoire projeté avec `mmap` et du VDSO
- 2 : randomisation également du tas

L'ASLR peut être désactiver localement avec la commande `setarch -R ./programme_bin`.


## 1.3. Canaris
Ajout d'une valeur aléatoire dans la pile après l'adresse de retour, appelé canari. A la sortie de la fonction, on dépile le canari et sa valeur est comparée à une copie stockée ailleurs.
Si la comparaison échoue, on appelle la fonction `__stack_chk_fail` et le programme est interrompu.
![[3_canari.png]]

L'ajout de canari alourdit le binaire et n'est pas toujours activée par défaut. Pour l'activer à la compilation avec GCC, il faut utiliser l'option `-f-stack-protector`.


## 1.4. Autres contre mesures
- CFI (Control Flow Integrity) : vérifie l'intégrité des branchements (call, jmp, ret), i.e. que les adresses cibles des branchements correspondent à des zones non modifiées par le programme (la zone commence par une instruction `endbr`)
- DFI (Data Flow Integrity) : rarement implémenté car entraîne un surcoût de calcul important
- Shadow stack : ajout d'une pile supplémentaire dans laquelle on met une copie de chaque  adresse de retour, lorsque l'on utilise l'instruction `ret`, on compare la valeur de retour sur le dessus de la pile avec celle sur le dessus de la shadow stack : si elles ne correspondent pas, le programme échoue.



# 2. Attaques avancées

## 2.1. Return to libc
L'application quasi systématique de la **politique W v X** rend l'exploitation de débordement de buffer dans la pile impossible car la pile ne peut pas contenir de code exécutable.

Mais les **librairies contiennent du code exécutable** et disponible à tout moment (par exemple le code de la librairie C). La fonction `system` de la librairie C permet d'exécuter une commande bash.

| Avant l'attaque | Après l'attaque |
| --------------- | --------------- |
|     ![[3_libc_attack_before.png]]            |![[3_libc_attack_after.png]]                 |

Lorsque la fonction courante termine, la fonction`system` est appelée : dans la pile il y a l'adresse de retour de la fonction `system` (qui n'a pas besoin d'être valide, mais qui permet au programme de se terminer proprement si elle est valide) et le paramètre de la fonction `system`, c'est-à-dire, l'adresse de la  commande à exécuter.

**Toboggan de slash** : répétition du caractère `/` car `////bin/sh` est équivalent à `/bin/sh` et cela donne plus de souplesse (on peut se tromper légèrement dans l'adresse de l'argument passé à `system`).

Cette attaque n'est possible que si l'ASLR est désactivé sur les zones de librairies. Il faut également qu'il n'y ait pas de canaris ou de shadow stack.


## 2.2. Return Oriented Programming
L'ASLR est appliqué à de plus en plus de sections/segments des fichiers exécutables et notamment aux librairies. Cependant, certaines librairies ne sont pas compilées avec les options nécessaires à leur positionnement aléatoire en mémoire (l'ASLR ne pourra pas s'appliquer).
-> On peut détourner le code situé dans ces zones positionnées à des adresses fixes.

En général, on ne peut pas utiliser directement un appel de fonction car il n'y aura pas d'équivalent à `system` à une adresse fixe...


### 2.2.1. Gadget
Un **gadget** est un morceau de code de quelques instructions assembleur se terminant par une instruction de type `ret` et situé à une adresse fixe (i.e. morceau d'une librairie sur laquelle l'ASLR ne s'applique pas).
-> On va combiner plusieurs gadget pour obtenir l'effet souhaité

En pratique, on empile les adresses de ces gadgets dans la pile pour qu'ils s'exécutent successivement => ROP : on enchaîne les `return`.

Il existe des programmes pour chercher automatiquement des gadgets dans le code des librairies et des programmes. Cependant composer automatiquement tous les gadgets est beaucoup plus difficile car il y a trop de possibilités. Des outils composent les gadgets en "motif" de 3-4 gadgets et calcule l'effet du motif. L'attaquant doit alors exprimer sa payload dans un language spécifique en décrivant l'effet recherché de l'attaque.


### 2.2.2. Quand les gadget ne suffisent plus
La charge active peut-être arbitrairement complexe et on n'arrivera pas à la simuler simplement avec des gadgets. On va procéder différemment :
- On a besoin d’une zone mémoire inscriptible ET exécutable de taille suffisante.
- On doit en connaître l’adresse.
- Avec quelques gadgets bien choisis on va écrire la payload complexe dans la zone en question.
- Finalement, on sautera dans le second étage en empilant son adresse de début, et en effectuant un `ret`.

Pour avoir une zone mémoire inscriptible et exécutable, on va faire appel à `nmap` pour créer une zone mémoire satisfaisante. Pour ce faire, on va utiliser des gadget pour mettre les bonnes valeurs dans les registres et faire un appel système à `mmap`.


### 2.2.3. VDSO
VDSO : virtual dynamic shared object
Il ne provient pas du fichier exécutable et est injecté par le noyau au démarrage du processus. Il permet de faire des **appels systèmes sans changement de contexte** : injecte le code de l'appel système dans le programme utilisateur (utilisé seulement pour les appels non critique - récupérer le temps courant par exemple).

Le VDSO était une source de gadget incroyable, cependant, il n'est plus chargé à une adresse fixe maintenant. Dans un vieux programme, on peut exploiter le VDSO, et dans un plus récent, il faut espérer trouver assez de gadget ailleurs.


### 2.2.4. Ne pas utiliser de '0'
Comme on utilise strcpy pour faire déborder notre code dans la pile, il ne faut pas de '0', sinon la copie s'arrête... Il faut ruser pour ne pas utiliser de '0', notamment dans les arguments de `mmap` :
- Le codage des permissions de pile READ | WRITE | EXEC vaut 7, i.e. `0x07`
   -> On écrit `0x01 0x01 ... 0x07` car l'OS testera l'octet de poids faible et n'utilisera pas les autres octets.
- `eax` vaut 192 : on trouve un gadget qui met `eax` à 0 puis un gadget qui lui ajoute un petit nombre et répète le 2ème gadget jusqu'à ce que `eax` valent 192 modulo 256.

