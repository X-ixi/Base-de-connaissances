```toc

```

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


## WHOIS Lookup
Base de données publique sur les noms de domaines : https://who.is/


## Robot.txt
Fichier qui liste les URLs a indexé ou non sur le site internet.


## Base de données de fuite
Les bases de données fuitées contiennent souvent des informations personnelles (email, numéro de téléphone, ...) et ce site permet de savoir si une de ces infos a fuitée : 
https://haveibeenpwned.com/


## Github
Recherche dans les répertoires Github et si un repo a été mal configuré et est public... bingo !



