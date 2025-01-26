
# Alias misconfiguration
Il y a un commentaire sur la page d'accueil `TODO : patch /assets/`.
En se rendant sur /assets/ on accède à un répertoire de fichier vide.

Le nom du challenge laisse penser qu'un alias est défini sur /assets dans le fichier de configuration Nginx.
En allant à l'URL `/assets/../`, on est redirigé vers la page d'acceuil
En allant sur `/assets../`, on a un listing du répertoire `..` avec le flag !

On vient de faire un path traversal !
Le chemin `/../` est détecté et supprimé par défaut mais pas le `/assets..`, qui fonctionne grâce à la mauvaise configuration de l'alias.

Dans le fichier de configuration Nginx on a surement quelque chose comme ça :
```nginx
server {
	...

	location /assets {
		alias /var/www/webapp/static/;
	}
}
```
La bonne configuration serait `/assets/` et non `/assets`.

