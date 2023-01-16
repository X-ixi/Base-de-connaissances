``` toc

```

# 1. Création d'une charge active
## 1.1. Objectif
Mettre en place une charge active de test permettant d'avoir un shell. Cette charge active (payload) doit être légère et faire seulement quelques centaines d'octets.
On va créer cette charge active directement en assembleur et on le chargera en C avec la fonction `asm`.

On souhaite que le code binaire injecté soit équivalent au code C suivant :
```C
# include <stdio.h>
# include <unistd.h>

int main() {
	char* name[] = { ”/bin/sh”, NULL };
	execve(name[0], name, NULL);
	return(0);
}
```


## 1.2. Rappels
Les programmes s'exécutent dans l'espace utilisateur de l'OS et lorsqu'ils ont besoin d'accéder à une ressource (mémoire, périphérique, ...) ils font un appel système qui sera exécuté en espace noyau (espace plus privilégié).

L'appel système `execve` a notamment les effets suivants :
- Utilise l'appel système `fork` pour créer un nouveau processus avec un nouvel espace d'adressage
- Charge le binaire du programme à exécuter dans cet espace d'adressage
- Transfert le flot d'exécution au point d'entrée du binaire
cf [[(OS) 3 - Ordonnancement#1.2. Création d'un processus fils]]

Tous sur les OS ici : [[(OS) 0 - Questions de cours]]


## 1.3. Programme lié statiquement
Lors de la création d'un nouveau processus, le noyau met en place plusieurs zones :
- Code exécutable
- Tas (pour les allocation dynamique, avec malloc par exemple)
- Pile

Les librairies C utilisées par le programme sont présentes directement dans le binaire et sont chargées dans la zone de code exécutable. 
-> c'est simple à produire et le programme fonctionne tout seul (toutes les librairies nécessaires sont dans l'exécutable).
-> MAIS le **binaire est lourd**.


## 1.4. Programme lié dynamiquement
Lors de la création d'un nouveau processus, le noyau met en place plusieurs zones :
- Code exécutable
- Tas
- Librairies
- Chargeur dynamique
- Pile
L'OS crée les zones précédentes, puis, si des librairies partagées sont utilisées par le programme, l'OS passe la main au chargeur dynamique qui fera en sorte de charger les librairies nécessaire dans l'espace d'adressage virtuel du processus.

Le programme suppose que les librairies C utilisées sont installées sur la machine pour s'exécuter.
-> Cela permet de partager les librairies entre les programmes et de réduire la taille mémoire des programmes.
-> MAIS c'est plus complexe à produire par la chaîne de compilation et le fonctionnement nécessite d'**installer toutes les librairies nécessaires au programme**.


## 1.5. Exemple de charge active
On crée un programme qui s'attaque lui même et qui exécute la charge active.
```C
int main(){
	asm("jmp appel_code\n"
		"code:\n"
		"pop esi\n"                         # esi = @("/bin/sh")
		"mov DWORD PTR [esi+0x8],esi\n"
		"mov DWORD PTR [esi+0xc],0x0\n"
		"mov BYTE PTR [esi+0x7],0x0\n"
		"mov eax,0xb\n"
		"mov ebx,esi\n"
		"lea ecx,[esi+0x8]\n"
		"lea edx,[esi+0xc]\n"
		"int 0x80\n"
		"mov eax,0x1\n"
		"int 0x80\n"
		"appel_code:\n"
		"call code\n"
		".string \"/bin/sh\"");
	return(0);
}
```

On commence par sauter à la fin du programme pour ensuite appeler la fonction `code` défini au début : cela permet d'exécuter la fonction `code` avec pour argument la chaîne de caractère `"/bin/sh"`.
Dans la fonction `code`, on remarque notamment les 2 instructions `int 0x80` qui font chacune un appel système dont le numéro est présent dans la variable `eax` (0xb pour l'appel à `execve` et 0x1 pour l'appel à `exit`.

Lorsqu'on essaie d'exécuter le programme précédent, une erreur de segmentation se produit quand on arrive à l'instruction `mov DWORD PTR [esi+0x8],esi`. En effet, on essaie d'écrire `esi` 8 octets après la chaîne de caractères "/bin/sh" mais cette chaîne de caractère se trouve au milieu du code et la page de code n'est pas inscriptible (et dans tous les cas, on ne voudrait pas réécrire sur la suite du code, sinon on écrirait sur le `return(0)`)

=> Pour éviter ce problème, il faut complexifier la payload pour qu'elle fasse une premier appel système à `mmap` pour allouer une page inscriptible qu'on utilisera pour la suite du shellcode.



# 2. Attaque par débordement de buffer
Exécution d'une charge active dans un programme vulnérable

| Avant attaque                        | Après attaque                        |
| ------------------------------------ | ------------------------------------ |
| ![[2_avant_attaque.png]] | ![[2_apres_attaque.png]] |

**Toboggan de NOP** : si l'adresse de retour pointe vers un de ces NOP, l'instruction NOP (No OPeration) sera exécuté avant de passer à la suivante et ainsi de suite jusqu'à l'exécution de la charge active.
-> Cela donne un peu de marge d'erreur : permet de ne pas donner exactement la bonne adresse de retour

Utilisation d'un script python pour créer la charge de payload

2 grosse difficultés :
- deviner l'adresse à laquelle revenir (pas facile avec l'ASLR)
- exécuter du code dans la pile (par défaut, ce n'est pas possible)
