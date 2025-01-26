
# 1 - Double extension
Le serveur vérifie que l'extension est .png ou .jpg.
Il suffit d'upload un fichier `test.php.png` pour que celui_ci soit exécuté.


# 2 - Type MIME
Le serveur vérifie l'extension et le type MIME du fichier. Le type MIME est calculé à partir du magic number en début de fichier.

On ajoute le magic number d'un fichier GIF au début de nombre fichier php
```php
GIF89a;
<?php

echo "Hello World!";

?>
```

Et on le renomme `test.php.gif`.

Ca ne fonctionne pas




