```toc

```

# 1. Erreur de segmentation due à un débordement de buffer

Dans le cas où on lance un programme et qu'il crashe avec une erreur "**segmentation fault**", il est possible que le programme soit vulnérable à un débordement de buffer.

Exemple : utilisation de la fonction *strcpy* qui prend en argument un pointeur vers la chaîne de caractère source et un pointeur vers l'adresse de destination. MAIS c'est le programmeur qui a la responsabilité de s'assurer qu'il y a assez de place à l'adresse de destination pour copier l'intégralité de la chaîne de caractère source (i.e. jusqu'au caractère `\0`).  
S'il n'y a pas assez de place, la copie écrit des données dans une zone non-prevue, qui sera surement interprété comme du code ensuite. Si l'instruction correspondant au débordement n'est pas valide, cela produit une "segmentation fault", mais un attaquant peut aussi créer une instruction valide et malveillante...

-> plus safe avec *strlcpy* qui prend un argument supplémentaire précisant le nombre maximal de caractères à copier.

| Avant le débordement              | Après le débordement             |
| --------------------------------- | -------------------------------- |
| ![[1_buffer_overflow_before.png]] | ![[1_buffer_overflow_after.png]] |
|                                   |                                  |


# 2. Détournement du flot d'exécution
S'il n'y a pas de protection sur la pile, une fonction peut modifier toutes les valeurs dans la pile, notamment **modifier la valeur de retour** à laquelle le flot de contrôle va aller à la fin de l'exécution de la fonction courante.
Cela permet de sauter une instruction, exécuter une fonction qui ne devait pas l'être, voire même exécuter du code qui n'est pas présent dans le programme (par exemple exécuter du code injecté avec le débordement de buffer, cf [[(AttMem) 2 - Charge active]]).



# 3. Contre-mesure
Le détournement du flot d'exécution suppose que l'on connait l'adresse à laquelle on souhaite envoyer le flot d'exécution.
L'**ASLR** (Address Space Layout Randomisation) est une contre mesure qui consiste à rendre aléatoire la disposition des espaces d'adressage.

On peut activer / désactiver l'ASLR sur le noyau. Ainsi la commande `setarch -R ./bin_file` exécute le fichier binaire sans randomisation des adresses.

Pour plus de contre-mesures, voir [[(AttMem) 3 - Contre mesures]].



