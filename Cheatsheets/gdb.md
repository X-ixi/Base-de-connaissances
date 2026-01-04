Lancer gdb avec la commande `gdb`.

# Commandes gdb
`layout split` : partage la fenêtre en 3 zones : code source, code assembleur et commande gdb.
`layout asm` : partage la fenêtre en 2 : code assembleur et commande
`layout regs`

`set disassembly-flavor intel` : utilise la convention intel (`mov eax, ebx` <=> copier ebx dans eax)

`quit` : quitter gdb.

## Avancer d'une instruction
`next` ou `n`: avancer d'une instruction en sautant au dessus des appels de fonctions.
`step` ou `s` : avancer d'une instruction en rentrant dans les appels de fonctions.
`nexti` ou `ni`: avancer d'une instruction assembleur en sautant au dessus des appels de fonctions.
`stepi` ou `si` : avancer d'une instruction assembleur en rentrant dans les appels de fonctions.

## Breakpoint & co
On peut utiliser `break` ou `b` pour mettre des breakpoints.
- `b <function>` : met un breakpoint au début de `<function>`. Ex: `b panic`.
- `b <adresse>` : met un breakpoint à l'adresse indiquée. Ex : `b *0x8000228a`.
- `b <filename>:<line_no>` : met un breakpoint à la ligne indiquée. Ex: `b kernel/proc.c:339`.

`continue` ou `c` : continue jusqu'au prochain breakpoint

### Supprimer un breakpoint
`info b` : liste les breakpoints
`del <breakpoint_no>` : supprime le breakpoint numéro `<breakpoint_no>`.


## Debug
`bt` : affiche la "backtrace", i.e. la pile d'appels de fonction qui a mené à ce point du programme

`print` ou `p` permet d'afficher la valeur d'une variable ou d'un registre :
- `p <variable>` : par exemple `p x` ou `p mon_pointeur->attr`.
- `p $<registre>` : par exemple `p $sp`.

`x` permet d'examiner une région de la mémoire
`x/24x $sp` examine 24 morceaux de données (chacune de la taille d'un mot - 4 octets), affichés en hexadécimal, à partir de l'adresse de départ `$sp`.

`x/s $eax` : affiche le registre EAX comme une chaîne de caractères.


`layout regs` (après avoir tapé `layout split`) : la fenêtre du code C est remplacé par l'ensemble des registres.  


`set <varirable> = <value>` : change la valeur d'une variable ou d'un registre
ex : `set $eax = 0`