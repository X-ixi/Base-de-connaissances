```toc

```

# Prendre des notes
https://app.osintracker.com

# Technique d'OSINT

## Google Dorks
Utilisation de termes particuliers dans la recherche pour obtenir des résultats cachés normalement.
- **inurl** : recherche de texte particulier dans les URLs
  ex : `inurl:hacking` 
- **filetype** : recherche pour des extensions de fichiers particuliers
  ex : `filetype:pdf "hacking"`
- **site** : recherche tous les URLs du domaine spécifié
  ex : `site:tryhackme.com`
- **cache** : recherche la dernière version mise en cache par google du site
  ex : `cache:tryhackme.com`

Exemple pour rechercher la chaîne de caractère `DB_PASSWORD` dans le sites github :
`site:github.com "DB_PASSWORD"`.

## Wayback machine
Machine à remonter le temps : http://web.archive.org/
Très pratique pour voir l'état d'un site web à une date précise !

## Recherche par image
Les moteurs les plus efficaces sont :
- Google
- Yandex
- Bing
- TinEye

## WHOIS Lookup
Base de données publique sur les noms de domaines : https://who.is/

## Informations sur une entreprise
Plusieurs sites intéressants :
- societe.com
- pappers.fr
- infonet.fr

## Services de l'Etat 
### data.gouv.fr
Le site du gouvernement recense un grand nombre de base de données ouvertes en lien avec le service public : https://www.data.gouv.fr
Chaque ville peut y publier ses jeux de données : commerçants, associations, médecins, emplacements des routes, des écoles, des boites à livres, ...
-> Peut permettre de retrouver des contacts (email, téléphone)
### France Cadastre
Permet de retrouver un permis de construire rapidement.
https://france-cadastre.fr
(Géoportail permet aussi de retrouver rapidement le numéro de la passerelle)

## Base de données de fuite
Les bases de données fuitées contiennent souvent des informations personnelles (email, numéro de téléphone, ...) et ce site permet de savoir si une de ces infos a fuitée : 
https://haveibeenpwned.com/

## Github
Recherche dans les répertoires Github et si un repo a été mal configuré et est public... bingo !

# Geosint
Cf [[Geosint]]

# Réseaux sociaux

## Twitter
Twitter ID : contrairement au nom d'utilisateur qui peut être changé, l'ID d'un compte ne change pas. Pratique pour suivre un compte dans le temps.
Explications pour trouver un ID, ou voir le compte associé à un ID : https://gist.github.com/kentbrew/8942accb5c584f11a775af02d097dd40
Voir le compte twitter associé à un ID : `https://twitter.com/i/user/<id>`

Recherche sur twitter :
- `from:<username> until:2021-10-01` : permet de voir tous les tweet posté sous un username donné avant une date donnée. Cela permet de suspecter si un compte a changé de username.
- `to:<username> until:2021-10-01` : permet de voir tous ceux qui ont commenté un post de username avant la date donnée.
- `... since:2021-10-01` : filtre après une date

Identifier les anciens username d'un compte twitter :
- Identifier les premiers posts du compte connu (`from:<new_username> until:<date>`)
- Rechercher tous les messages à destination de ce compte avant la date des premiers posts de new_username (`to:<new_username> until:<date>`)
- Avec un peu de chance, un de ces massages contient un ancien @ du compte (`ctr-F @`)


## Wayback Machine (again !)
Pour twitter, pour rechercher l'activité sur un username, rechercher l'URL : `twitter.com/<username>` (ou x.com après juillet 2023)
-> Aller dans l'onglet URL !!!